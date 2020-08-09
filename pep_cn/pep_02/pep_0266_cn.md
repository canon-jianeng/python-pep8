
PEP: 266
Title: Optimizing Global Variable/Attribute Access
Version: $Revision$
Last-Modified: $Date$
Author: skip@pobox.com (Skip Montanaro)
Status: Withdrawn
Type: Standards Track
Content-Type: text/x-rst
Created: 13-Aug-2001
Python-Version: 2.3
Post-History:


Abstract
========

大多数全局变量的绑定和其他模块的属性在执行 Python 程序时通常永远不会改变,
但由于 Python 的动态特性, 访问这些全局对象的代码必须在每次需要对象时运行完整查找.
此 PEP 提出了一种机制, 允许访问大多数全局对象的代码将它们视为本地对象,
并将更新引用的负担放在更改此类对象的名称绑定的代码上.


Introduction
============

考虑重负荷函数 ``sre_compile._compile``. 它是 ``sre`` 模块的内部编译功能.
它几乎完全包含对正在编译的模式元素的循环, 将操作码与已知常量值进行比较,
并将标记附加到输出列表. 大多数比较都是从 ``sre_constants`` 模块导入的常量.
这意味着在该模块的编译输出中有许多 ``LOAD_GLOBAL`` 字节码. 只是通过阅读代码,
很明显作者想要 ``LITERAL``, ``NOT_LITERAL``, ``OPCODES`` 和许多其他符号作为常量.
尽管如此, 每次他们参与表达时, 都必须重新查找.

大多数全局访问实际上是对 "almost constants" 的对象. 这包括当前模块中的全局变量以及其他导入模块的属性.
由于它们很少更改, 因此将更新对这些对象的引用的负担放在更改名称绑定的代码上似乎是合理的.
如果将 ``sre_constants.LITERAL`` 更改为引用另一个对象, 那么修改 ``sre_constants`` 模块字典的代码可能
值得更正对该对象的任何活动引用. 通过这样做, 在许多情况下, 全局变量和许多对象的属性可以缓存为局部变量.
如果给予对象的名称和对象本身之间的绑定很少变化, 那么跟踪这些对象的成本应该很低并且潜在的收益相当大.

为了衡量这个提议的效果, 我修改了 Python 发行版中包含的 Pystone 基准程序来缓存全局函数.
它的主要功能 ``Proc0`` 在 ``for`` 循环中调用十个不同的函数. 另外, ``Func2`` 在循环内反复调用 ``Func1``.
如果在输入函数循环之前制作了这11个全局标识符的本地副本, 则此特定基准测试的性能提高约2％
(从我的笔记本电脑上的5561个 pystones 到5685). 它表明通过缓存大多数全局变量访问可以提高性能.
另请注意, pystone 基准测试基本上不会访问全局模块属性, 这是此 PEP 的预期改进领域.


Proposed Change
===============

我建议修改 Python 虚拟机来包含 ``TRACK_OBJECT`` 和 ``UNTRACK_OBJECT`` 操作码.
``TRACK_OBJECT`` 将全局名称或全局名称的属性与局部变量数组中的槽相关联,
并执行相关对象的初始查找, 来使用有效值填充槽. 它创建的关联将由负责更改名称到对象绑定的代码记录,
以使相关的局部变量更新. ``UNTRACK_OBJECT`` 操作码将删除名称和局部变量槽之间的任何关联.


Threads
=======

在线程程序中操作此代码与无线程程序没有什么不同. 如果你需要锁定一个对象来访问它,
你必须在执行 ``TRACK_OBJECT`` 之前这样做并保持锁定直到你停止使用它为止.

FIXME: 我怀疑我需要更多.


Rationale
=========

全局变量和属性很少改变. 例如, 一旦函数导入数学模块, 名称 *math* 与其引用的模块之间的
绑定就不太可能发生变化. 类似地, 如果使用 ``math`` 模块的函数引用其 *sin* 属性, 则不太可能改变.
但是, 每次模块想要调用 ``math.sin`` 函数时, 它必须首先执行一对指令::

    LOAD_GLOBAL     math
    LOAD_ATTR       sin

如果客户端模块总是假设 ``math.sin`` 是一个局部常量,
并且函数外部的 "外力" 负责保持引用正确, 我们可能会有这样的代码::

    TRACK_OBJECT       math.sin
    ...
    LOAD_FAST          math.sin
    ...
    UNTRACK_OBJECT     math.sin

如果 ``LOAD_FAST`` 处于循环中, 则减少的全局负载和属性查找的回报可能很大.

理论上, 该技术可以应用于任何全局变量访问或属性查找. 考虑这段代码::

    l = []
    for i in range(10):
        l.append(math.sin(i))
    return l

即使 *l* 是局部变量, 你仍然需要支付在循环中加载 "l.append" 的十倍的成本.
编译器 (或优化器) 可以识别循环中调用 ``math.sin`` 和 ``l.append`` 并决定生成跟踪的本地代码,
避免内置 ``range()`` 函数, 因为它只在循环设置期间调用一次. 与访问局部变量相关的性能问题
使跟踪 "l.append" 不如跟踪诸如 ``math.sin`` 的全局变量更具吸引力.

根据 Marc-Andre Lemburg [1]_ 对 python-dev 的帖子, ``LOAD_GLOBAL`` 操作码占 Python 虚拟机
执行的所有指令的7％以上. 这可能是一个非常昂贵的指令, 至少相对于 ``LOAD_FAST`` 指令,
它是一个简单的数组索引, 并且不需要虚拟机进行额外的函数调用. 我相信许多 ``LOAD_GLOBAL`` 指令
和 ``LOAD_GLOBAL/LOAD_ATTR`` 对可以转换为 ``LOAD_FAST`` 指令.

使用全局变量的代码经常使用各种技巧来避免全局变量和属性查找. 前面提到的 ``sre_compile._compile`` 函数
缓存了不断增长的输出列表的 ``append`` 方法. 许多人通常滥用函数的默认参数功能来缓存全局变量查找.
这两种方案都是 hackish, 很少能解决所有可用的优化机会. (例如, ``sre_compile._compile`` 不会缓存它最常使用的
两个全局变量: 内置 ``len`` 函数和从 ``sre_constants.py`` 导入的全局 ``OPCODES`` 数组.


Questions
=========

线程怎么样? 如果 ``math.sin`` 在缓存中发生变化怎么办?
-----------------------------------------------------------------

我相信全局解释器锁将保护值不被破坏. 无论如何, 情况不会比今天更糟.
如果一个线程在另一个线程之后修改了 ``math.sin`` 已经执行了 ``LOAD_GLOBAL math``,
但在执行 ``LOAD_ATTR sin`` 之前, 客户端线程会看到 ``math.sin`` 的旧值.

这个想法是这样的. 我使用下面的多属性加载作为示例, 不是因为它会经常发生,
而是因为通过额外调用来演示递归性质, 希望它会变得更加清晰.
假设模块 ``foo`` 中定义的函数想要访问 ``spam.eggs.ham``
并且 ``spam`` 是在 ``foo`` 模块级别导入的模块::

    import spam
    ...
    def somefunc():
    ...
    x = spam.eggs.ham

进入 ``somefunc`` 后, 将执行 ``TRACK_GLOBAL`` 指令::

    TRACK_GLOBAL spam.eggs.ham n

*spam.eggs.ham* 是存储在函数常量数组中的字符串字面量. *n* 是一个 fastlocals 索引.
``＆fastlocals[n]`` 是对执行帧的 ``fastlocals`` 数组中 slot *n* 的引用,
该数组是存储 *spam.eggs.ham* 引用的位置. 这就是我想象中发生的事情:

1. ``TRACK_GLOBAL`` 指令定位名称 *spam* 引用的对象,
   并在其模块范围内找到它. 然后它执行类似的 C 函数::

       _PyObject_TrackName(m, "spam.eggs.ham", &fastlocals[n])

   其中 ``m`` 是具有属性 ``spam`` 的模块对象.

2. 模块对象剥离前导垃圾邮件. 并存储必要的信息 (*eggs.ham* 和 ``＆fastlocals[n]``),
   来防止它的名称 *eggs* 的绑定发生变化. 然后它在其字典中定位键 *eggs* 引用的对象并递归调用::

       _PyObject_TrackName(eggs, "eggs.ham", &fastlocals[n])

3. ``eggs`` 对象剥离前导 *eggs.*, 存储 (*ham*, ＆fastlocals[n]) 信息,
   将对象定位在名为 ``ham`` 的命名空间中, 再次调用 ``_PyObject_TrackName``::

       _PyObject_TrackName(ham, "ham", &fastlocals[n])

4. ``ham`` 对象剥离了前导字符串 (这次没有 ".", 但这是一个小点), 看到结果为空,
   然后使用自己的值 (可能是 ``self``) 来更新它的位置::

       Py_XDECREF(&fastlocals[n]);
       &fastlocals[n] = self;
       Py_INCREF(&fastlocals[n]);

   此时, 解析 ``spam.eggs.ham`` 所涉及的每个对象都知道需要跟踪其命名空间中的
   哪个条目以及该名称更改时要更新的位置. 此外, 如果它在本地存储中跟踪的一个名称发生更改,
   则一旦进行了更改, 它就可以使用新对象调用 "_PyObject_TrackName". 在食物链的最底端,
   最后一个对象将始终删除一个名称, 查看空字符串并知道它的值应该填入它传递的位置.

   当虚线表达式 ``spam.eggs.ham`` 引用的对象超出范围时,
   执行 ``UNTRACK_GLOBAL spam.eggs.ham n`` 指令.
   它具有删除所有 ``TRACK_GLOBAL`` 建立的跟踪信息的效果.

   跟踪操作可能看起来很昂贵, 但回想一下, 被跟踪的对象被假定为 "almost constant",
   因此设置成本将与希望多个本地而非全局负载进行折衷. 对于具有属性的全局变量,
   跟踪设置成本会增加, 但会通过避免额外的 ``LOAD_ATTR`` 成本来抵消.
   ``TRACK_GLOBAL`` 指令需要为链中的第一个名称执行 ``PyDict_GetItemString`` 来确定顶级对象所在的位置.
   链中的每个对象都必须在某处存储字符串和地址, 可能在使用存储位置作为键的字典中
   (例如, ``＆fastlocals[n]``) 和字符串作为值. (这个 dict 可能是 dicts 的中心字典, 其键是对象地址而不是每个对象的字典.)
   它不应该是另一种方式, 因为多个活动帧可能想要跟踪 ``spam.eggs.ham` `,
   但只有一个框架想要将该名称与其快速本地槽之一相关联.


Unresolved Issues
=================

Threading
---------

这个 (伪) 代码怎么样?::

    l = []
    lock = threading.Lock()
    ...
    def fill_l()::
       for i in range(1000)::
          lock.acquire()
          l.append(math.sin(i))
          lock.release()
    ...
    def consume_l()::
       while 1::
          lock.acquire()
          if l::
             elt = l.pop()
          lock.release()
          fiddle(elt)

从代码的静态分析中不清楚锁保护什么. (在编译时你无法告诉线程甚至你可以涉及到吗?)
它会或者应该影响在 ``fill_l`` 函数中跟踪 ``l.append`` 或 ``math.sin`` 的尝试?

如果我们使用神秘的 ``track_object`` 和 ``untrack_object`` 内置来注释代码
(我不建议这样做, 只是说明内容会去哪里!), 我们得到::

    l = []
    lock = threading.Lock()
    ...
    def fill_l()::
       track_object("l.append", append)
       track_object("math.sin", sin)
       for i in range(1000)::
          lock.acquire()
          append(sin(i))
          lock.release()
       untrack_object("math.sin", sin)
       untrack_object("l.append", append)
    ...
    def consume_l()::
       while 1::
          lock.acquire()
          if l::
             elt = l.pop()
          lock.release()
          fiddle(elt)

无论有没有线程都是正确的 (或者至少同样不正确的线程和没有线程)?

Nested Scopes
-------------

嵌套作用域的存在将影响 ``TRACK_GLOBAL`` 找到全局变量的位置,
但在此之后不应该影响任何内容. (我认为.）

Missing Attributes
------------------

假设我正在跟踪 ``spam.eggs.ham`` 引用的对象, 并且 ``spam.eggs`` 被反弹到一个没有 ``ham`` 属性的对象.
很明显, 如果程序员试图在当前的 Python 虚拟机中解析 ``spam.eggs.ham``, 那将是一个 ``AttributeError``,
但是假设程序员已经预料到了这种情况::

    if hasattr(spam.eggs, "ham"):
        print spam.eggs.ham
    elif hasattr(spam.eggs, "bacon"):
        print spam.eggs.bacon
    else:
        print "what? no meat?"

重新计算跟踪信息时, 不能引发 ``AttributeError``.
如果它没有引发 ``AttributeError`` 而是让跟踪, 它可能会为程序员设置一个非常微妙的错误.

该问题的一个解决方案是跟踪函数直接引用的每个虚线表达式的最短可能 root.
在上面的例子中, 将跟踪 ``spam.eggs``, 但不会跟踪 ``spam.eggs.ham`` 和 ``spam.eggs.bacon``.

Who does the dirty work?
------------------------

在问题部分, 我假设存在一个 ``_PyObject_TrackName`` 函数. 虽然 API 很容易指定,
但幕后实现并不那么明显. 可以使用中心字典来跟踪名称或位置映射,
但似乎可能需要修改所有 ``setattr`` 函数来适应这种新功能.

如果所有类型都使用 ``PyObject_GenericSetAttr`` 函数来设置稍微本地化更新代码的属性.
但它们并没有 (这并不太令人惊讶), 所以似乎所有 ``getattrfunc`` 和 ``getattrofunc`` 函数都必须更新.
另外, 这会对 C 扩展模块作者提出一个绝对要求, 即当属性改变值时调用某个函数 (``PyObject_TrackUpdate``?).

最后, 很可能一些属性将由副作用设置, 而不是通过任何直接调用某种类型的 ``setattr`` 方法.
考虑一个具有中断例程的设备接口模块, 该例程将设备寄存器的内容复制到对象的 ``struct`` 中的槽中.
在这些情况下, 模块作者必须进行更广泛的修改. 在编译时识别这种情况是不可能的.
我认为可以在 ``PyTypeObjects`` 中添加一个额外的槽来指示对象的代码是否对全局跟踪是安全的.
它的默认值为 0 (``Py_TRACKING_NOT_SAFE``). 如果扩展模块作者已实现必要的跟踪支持,
则该字段可以初始化为1 (``Py_TRACKING_SAFE``). ``_PyObject_TrackName`` 可以检查该字段并发出警告,
如果要求它跟踪作者没有明确表示可以安全跟踪的对象.


Discussion
==========

Jeremy Hylton 在桌子上有另一个提议 [2]_. 他的提议旨在创建一个混合字典或列表对象, 用于全局名称查找,
使全局变量访问看起来更像本地变量访问. 虽然没有可用的 C 代码可供检查,
但他的提案中给出的 Python 实现似乎仍然需要字典键查找. 他的提议似乎没有加速局部变量属性查找,
如果可以解决潜在的性能负担, 这在某些情况下可能是值得的.


Backwards Compatibility
=======================

我不相信会有任何严重的向后兼容问题. 显然,
早期版本的解释器无法执行包含 ``TRACK_OBJECT`` 操作码的 Python 字节码,
但在版本之间通常会假设字节码级别出现断裂.


Implementation
==============

TBD. 这是我需要帮助的地方. 我相信应该有一个中心名称或位置注册表或修改对象属性的代码应该被修改,
但我不确定最好的方法来解决这个问题. 如果你看一下实现 ``STORE_GLOBAL`` 和 ``STORE_ATTR`` 操作码的代码,
似乎需要对 ``PyDict_SetItem`` 和 ``PyObject_SetAttr`` 或它们的 String 变体进行一些修改. 理想情况下,
有一个相当重要的位置来本地化这些变化. 如果您开始考虑跟踪局部变量的属性, 那么您也会遇到修改 ``STORE_FAST`` 的问题,
这可能是一个问题, 因为局部变量的名称绑定会更频繁地更改.
(我认为优化器可以避免为变量名称绑定发生变化的任何局部变量的属性插入跟踪代码.)


Performance
===========

我相信 (虽然我现在没有代码来证明它), 实现 ``TRACK_OBJECT`` 通常不会比单个 ``LOAD_GLOBAL`` 指令
或 ``LOAD_GLOBAL`` /``LOAD_ATTR`` 对贵得多. 优化器应该能够避免将 ``LOAD_GLOBAL``
和 ``LOAD_GLOBAL`` /``LOAD_ATTR`` 转换为新方案, 除非对象访问发生在循环中. 更进一步,
面向寄存器的替代当前的 Python 虚拟机 [3]_ 可以想象地消除了大多数 ``LOAD_FAST`` 指令.

跟踪对象的数量应该相对较小. 可以想象, 所有活动线程的所有活动帧都可以跟踪对象,
但与给定应用程序中定义的函数数量相比, 这似乎很小.


References
==========

.. [1] https://mail.python.org/pipermail/python-dev/2000-July/007609.html

.. [2] http://www.zope.org/Members/jeremy/CurrentAndFutureProjects/FastGlobalsPEP

.. [3] http://www.musi-cal.com/~skip/python/rattlesnake20010813.tar.gz


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  fill-column: 70  
  End:  
