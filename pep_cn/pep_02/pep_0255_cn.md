
PEP: 255
Title: Simple Generators
Version: $Revision$
Last-Modified: $Date$
Author: nas@arctrix.com (Neil Schemenauer),
        tim.peters@gmail.com (Tim Peters),
        magnus@hetland.org (Magnus Lie Hetland)
Discussions-To: python-iterators@lists.sourceforge.net
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Requires: 234
Created: 18-May-2001
Python-Version: 2.2
Post-History: 14-Jun-2001, 23-Jun-2001


Abstract
========

这个 PEP 向 Python 引入了生成器的概念,
以及与它们一起使用的新语句, ``yield`` 语句.


Motivation
==========

当生产者函数有足够的工作需要在生成的值之间保持状态时,
除了向生产者的参数列表添加回调函数之外,
大多数编程语言都没有提供令人愉快和有效的解决方案,
需要使用生成的每个值调用.

例如, 标准库中的 ``tokenize.py`` 采用这种方法: 调用者必须将 *tokeneater* 函数
传递给 ``tokenize()``, 每当 ``tokenize()`` 找到下一个标记时调用它.
这允许以自然方式对令牌化进行编码, 但是调用令牌化的程序通常由于需要记住最后看到令牌的回调而复杂.
``tabnanny.py`` 中的 *tokeneater* 函数就是一个很好的例子, 在全局变量中维护一个状态机,
记住它已经看到的回调以及它希望接下来看到的内容. 这很难正常工作, 人们仍然很难理解.
不幸的是, 这是这种方法的典型.

另一种方法是使用 tokenize 在大型列表中同时生成 Python 程序的整个解析.
然后, 可以使用局部变量和本地控制流 (例如, 循环和嵌套的 if 语句) 以自然的方式编写令牌化客户端,
来跟踪其状态. 但这不实用: 程序可能非常大, 因此不能在实现整个解析所需的内存上放置先验限制;
并且一些令牌化客户端只想查看某个特定的东西是否出现在程序的早期 (例如, 未来的语句,
或者如在 IDLE 中所做的那样, 只是第一个缩进语句), 然后首先解析整个程序是一个严重的浪费时间.

另一种方法是将令牌化为迭代器 [1], 每当调用它的 ``.next()`` 方法时传递下一个令牌.
这对于调用者来说是愉快的, 就像一大堆结果一样, 但没有存储和 "如果我想早点出去怎么办?" 缺点.
然而, 这会改变 tokenize 的负担, 利于在 ``.next()`` 祈求之间记住它的状态,
而读者只需要看一下 ``tokenize.tokenize_loop()`` 来实现一个可怕的苦差事.
或者描绘用于生成一般树结构的节点的递归算法: 将其转换为迭代器框架, 需要手动删除递归并手动维护遍历的状态.

第四种选择是在单独的线程中运行生产者和消费者. 这允许两者以自然的方式保持其状态, 因此两者都令人愉快.
实际上, Python 源代码发行版中的 Demo/threads/Generator.py 提供了一个可用的同步通信类,
利于以一般方式执行此操作. 但是, 这在没有线程的平台上不起作用, 而且在平台上运行速度非常慢
(与没有线程的情况相比).

最后一个选择是使用 Python 的 Stackless [2] [3] 变体实现, 它支持轻量级协同程序.
这与线程选项具有相同的程序优势, 但效率更高. 但是, Stackless 是一个有争议的重新思考 Python 核心,
Jython 可能无法实现相同的语义. 这个 PEP 不是讨论的地方, 因此在这里说生成器以一种非常适合
当前 CPython 实现的方式提供 Stackless 功能的有用子集就足够了, 并且被认为对其他 Python 实现来说相对简单.

这耗尽了当前的替代品. 其他一些高级语言提供了令人愉快的解决方案, 特别是 Sather [4] 中的迭代器,
它们受到 CLU 迭代器的启发; 和 [5] 中的生成器, 一种新颖的语言, 其中每个表达式都是生成器.
这些之间存在差异, 但基本思路是相同的: 提供一种函数, 可以将中间结果 ("下一个值") 返回给调用者,
但保持函数的本地状态, 利于可以再次恢复该函数就在它停止的地方.
一个非常简单的例子::

    def fib():
        a, b = 0, 1
        while 1:
           yield b
           a, b = b, a+b

首次调用 ``fib()`` 时, 它将 *a* 设置为0, 将 *b* 设置为1, 然后将 *b* 返回给它的调用者.
调用者看到1. 当 ``fib`` 被恢复时, 从它的角度来看, ``yield`` 语句实际上与 ``print`` 语句相同:
在 yield 与所有本地状态完好无损之后, ``fib`` 继续. *a* 和 *b* 然后变为1和1,
并且 ``fib`` 循环回到 ``yield``, 对其调用者产生1, 等等. 从 ``fib`` 的角度来看,
它只是提供一系列结果, 就像通过回调一样. 但是从调用者的角度来看,
``fib`` 调用是一个可以随意恢复的可迭代对象. 与线程方法一样, 这允许双方以最自然的方式进行编码;
但与线程方法不同, 这可以在所有平台上高效完成. 实际上, 恢复生成器应该不比函数调用昂贵.

同样的方法适用于许多生产者或消费者功能. 例如,
``tokenize.py``可以产生下一个令牌而不是用它作为参数调用回调函数,
而 tokenize 客户端可以以自然的方式迭代令牌:
Python 生成器是一种 Python 迭代器 [1]_, 但是特别有力的一种.


Specification:  Yield
=====================

引入了一个新的声明::

    yield_stmt:    "yield" expression_list

``yield`` 是一个新的关键字, 因此需要一个 ``future`` 语句 [8]_ 来进行调整:
在初始版本中, 希望使用生成器的模块必须包含该行::

    from __future__ import generators

靠近顶部 (详见 PEP 236 [8]_). 使用标识符 ``yield`` 而没有 ``future`` 语句的模块将触发警告.
在下面的版本中, ``yield`` 将是一个语言关键字, 不再需要 ``future`` 语句.

``yield`` 语句只能在函数内部使用. 包含 ``yield`` 语句的函数称为生成器函数.
生成器函数在所有方面都是普通的函数对象,
但在代码对象的 co_flags 成员中设置了新的 ``CO_GENERATOR`` 标志.

当调用生成器函数时, 实际参数以通常的方式绑定到函数本地形式参数名称,
但不执行函数体中的代码. 而是返回一个 generator-iterator 对象;
这符合迭代器协议 [6]_, 因此特别可以以自然的方式用于 for 循环.
请注意, 当从上下文中清除 intent 时, 非限定名称 "generator" 可用于表示生成器函数或生成器迭代器.

每次调用 generator-iterator 的 ``.next()`` 方法时, 执行生成器函数体中的代码,
直到 ``yield`` 或 ``return`` 语句 (见下文) 遇到, 或直到 body 的末端到达.

如果遇到 ``yield`` 语句, 则冻结函数的状态, 并将 *expression_list* 的值返回给 `.next()` 的调用者.
"frozen" 是指保留所有本地状态, 包括局部变量, 指令指针和内部评估堆栈的当前绑定: 保存足够的信息,
利于下次调用 ``.next()``, 该函数可以完全像 ``yield`` 语句只是另一个外部调用.

限制: ``try/finally`` 结构的 ``try`` 子句中不允许使用 ``yield`` 语句. 困难在于无法保证生成器将被恢复,
因此无法保证 finally 块将被执行; 这太违反了最终的目的.

限制: 生成器在主动运行时无法恢复::

    >>> def g():
    ...     i = me.next()
    ...     yield i
    >>> me = g()
    >>> me.next()
    Traceback (most recent call last):
     ...
     File "<string>", line 2, in g
    ValueError: generator already executing


Specification:  Return
======================

生成器函数还可以包含构成的 return 语句::

    return

请注意, 生成器主体中的 return 语句不允许使用 *expression_list*
(当然, 它们可能出现在嵌套在生成器中的非生成器函数的主体中).

当遇到 return 语句时, 控制继续执行任何函数返回, 执行适当的 ``finally`` 子句 (如果存在).
然后引发一个 ``StopIteration`` 异常, 表示迭代器已经耗尽.
如果控制流出生成器的末尾而没有显式返回, 则也会引发 ``StopIteration`` 异常.

请注意, 对于生成器函数和非生成器函数, 返回意味着 "我已完成, 并且没有任何有趣的返回".

请注意, 返回并不总是等同于引发 ``StopIteration``:
区别在于如何处理封闭 ``try/except`` 结构. 例如,::

    >>> def f1():
    ...     try:
    ...         return
    ...     except:
    ...        yield 1
    >>> print list(f1())
    []

因为, 就像在任何函数中一样, ``return`` 只是退出, 但是::

    >>> def f2():
    ...     try:
    ...         raise StopIteration
    ...     except:
    ...         yield 42
    >>> print list(f2())
    [42]

因为 ``StopIteration`` 被一个简单的 ``except`` 捕获, 就像任何异常一样.


Specification:  Generators and Exception Propagation
====================================================

如果一个未处理的异常 -- 包括但不限于 ``StopIteration`` -- 由生成器函数引发或通过它,
那么异常将以通常的方式传递给调用者, 并随后尝试恢复生成器函数引发 ``StopIteration``.
换句话说, 未处理的异常终止了生成器的使用寿命.

示例 (不是常用的, 但为了说明这一点) ::

    >>> def f():
    ...     return 1/0
    >>> def g():
    ...     yield f()  # the zero division exception propagates
    ...     yield 42   # and we'll never get here
    >>> k = g()
    >>> k.next()
    Traceback (most recent call last):
      File "<stdin>", line 1, in ?
      File "<stdin>", line 2, in g
      File "<stdin>", line 2, in f
    ZeroDivisionError: integer division or modulo by zero
    >>> k.next()  # and the generator cannot be resumed
    Traceback (most recent call last):
      File "<stdin>", line 1, in ?
    StopIteration
    >>>


Specification:  Try/Except/Finally
==================================

如前所述, 在 ``try/finally`` 结构的 ``try`` 子句中不允许 ``yield``.
结果是生成器应该非常谨慎地分配关键资源. ``yield`` 没有限制,
否则出现在 ``finally`` 子句, ``except`` 子句或 ``try/except`` 结构的 ``try`` 子句中::

    >>> def f():
    ...     try:
    ...         yield 1
    ...         try:
    ...             yield 2
    ...             1/0
    ...             yield 3  # never get here
    ...         except ZeroDivisionError:
    ...             yield 4
    ...             yield 5
    ...             raise
    ...         except:
    ...             yield 6
    ...         yield 7     # the "raise" above stops this
    ...     except:
    ...         yield 8
    ...     yield 9
    ...     try:
    ...         x = 12
    ...     finally:
    ...        yield 10
    ...     yield 11
    >>> print list(f())
    [1, 2, 4, 5, 8, 9, 10, 11]
    >>>


Example
=======

::

    # A binary tree class.
    class Tree:

        def __init__(self, label, left=None, right=None):
            self.label = label
            self.left = left
            self.right = right

        def __repr__(self, level=0, indent="    "):
            s = level*indent + `self.label`
            if self.left:
                s = s + "\n" + self.left.__repr__(level+1, indent)
            if self.right:
                s = s + "\n" + self.right.__repr__(level+1, indent)
            return s

        def __iter__(self):
            return inorder(self)

    # Create a Tree from a list.
    def tree(list):
        n = len(list)
        if n == 0:
            return []
        i = n / 2
        return Tree(list[i], tree(list[:i]), tree(list[i+1:]))

    # A recursive generator that generates Tree labels in in-order.
    def inorder(t):
        if t:
            for x in inorder(t.left):
                yield x
            yield t.label
            for x in inorder(t.right):
                yield x

    # Show it off: create a tree.
    t = tree("ABCDEFGHIJKLMNOPQRSTUVWXYZ")
    # Print the nodes of the tree in in-order.
    for x in t:
        print x,
    print

    # A non-recursive generator.
    def inorder(node):
        stack = []
        while node:
            while node.left:
                stack.append(node)
                node = node.left
            yield node.label
            while not node.right:
                try:
                    node = stack.pop()
                except IndexError:
                    return
                yield node.label
            node = node.right

    # 练习非递归生成器.
    for x in t:
        print x,
    print

两个输出块都显示::

    A B C D E F G H I J K L M N O P Q R S T U V W X Y Z


Q & A
=====

为什么不使用新关键字而不是重复使用 ``def``?
-------------------------------------------------

请参阅下面的 BDFL 声明部分.

为什么 ``yield`` 新关键字?  为什么不是内置函数呢?
---------------------------------------------------------------------

通过 Python 中的关键字更好地表达控制流, yield 是一个控制结构.
它还认为 Jython 中的高效实现要求编译器能够在编译时确定潜在的挂起点,
并且新的关键字使这一点变得容易. CPython 参考实现也大量使用它来检测
哪些函数是 generator-functions (虽然用一个新的关键字代替 ``def`` 可以
解决 CPython 的问题 -- 但人们会问 "为什么一个新的关键字?" 问题不想要任何新的关键字）.

那么为什么没有一个新的关键字的其他特殊语法?
-------------------------------------------------------------

例如, 其中一个, 而不是 ``yield 3``::
 
    return 3 and continue
    return and continue 3
    return generating 3
    continue return 3
    return >> , 3
    from generator return 3
    return >> 3
    return << 3
    >> 3
    << 3
    * 3

我错过了一个 <wink> 吗? 在数百条消息中, 我计算了三个建议这样的替代方案, 并从中提取了上述内容.
不需要一个新的关键字会更好, 但更好地使 ``yield`` 非常清楚 -- 我不希望从推断出一个先前毫无意义的关键词
序列产生关键字或运算符. 尽管如此, 如果这引起足够的兴趣, 支持者应该基于一个共识建议, 而 Guido 将在其上发表意见.

为什么要允许 ``return``?  为什么不强制终止拼写 ``raise StopIteration``?
----------------------------------------------------------------------------------------------

"StopIteration" 的机制是低级细节, 就像 Python 2.1 中 "IndexError" 的机制一样:
实现需要做一些定义明确的内容, 而 Python 为高级用户公开了这些机制.
但这并不是强迫每个人都在这个级别工作的论据. ``return`` 在任何一种功能中都意味着 "我已经完成",
这很容易解释和使用. 请注意, ``return`` 并不总是等同于 try/except 结构中的 ``raise StopIteration``
(参见 "规范: 返回" 部分).

那么为什么不允许在 ``return`` 上使用表达式?
---------------------------------------------------

也许有一天我们会. 在 Icon 中, ``return expr`` 意味着 "我已经完成"
和 "但我还有一个最终有用的值也可以返回, 这就是它". 在开始时,
并且在没有令人信服的用于 ``return expr`` 的情况下, 使用 ``yield`` 专门用于传递值更简洁.


BDFL Pronouncements
===================

Issue
-----

引入另一个新关键字 (比如, ``gen`` 或 ``generator``) 来代替 ``def``,
或以其他方式改变语法, 来区分生成器函数和非生成器函数.

Con
---

在实践中 (你如何看待它们), 生成器是函数, 但它们具有可恢复性.
它们如何建立的机制是一个相对较小的技术问题,
引入一个新的关键字无助于过分强调生成器如何启动的机制
(生成器生命中至关重要的一小部分).

Pro
---

实际上 (你如何看待它们), 生成器函数实际上是工厂函数, 它们通过魔术产生生成器迭代器.
在这方面, 它们与非生成器函数完全不同, 更像是构造函数而不是函数,
因此重用 ``def`` 最多是令人困惑的. 埋藏在正文中的 ``yield`` 语句不足以警告, 语义是如此不同.

BDFL
----

``def`` 它留下来. 任何一方都没有任何争论是完全令人信服的, 所以我咨询了我的语言设计师的直觉.
它告诉我 PEP 中提出的语法是完全正确的 -- 刚刚好. 但是, 就像希腊神话中的 Delphi 中的甲骨文一样,
它并没有告诉我原因, 所以我没有对反对 PEP 语法的论点进行反驳. 我能想出的最好的
(除了同意已经做出的反驳) 是 "FUD". 如果这从第一天开始就是语言的一部分,
我非常怀疑它会不会做出 Andrew Kuchling 的 "Python Warts" 页面.


Reference Implementation
========================

当前的实现, 处于初步状态 (没有文档, 但经过良好测试和可靠),
是 Python 的 CVS 开发树 [9]_ 的一部分. 使用它需要您从源代码构建 Python.

这是来自 Neil Schemenauer 的早期补丁 [7]_.


Footnotes and References
========================

.. [1] PEP 234, 迭代器, Yee, Van Rossum
       http://www.python.org/dev/peps/pep-0234/

.. [2] http://www.stackless.com/

.. [3] PEP 219, 无堆栈的 Python, McMillan
       http://www.python.org/dev/peps/pep-0219/

.. [4] "Sather 的迭代抽象"
       Murer, Omohundro, Stoutamire and Szyperski
       http://www.icsi.berkeley.edu/~sather/Publications/toplas.html

.. [5] http://www.cs.arizona.edu/icon/

.. [6] 迭代器的概念在 PEP 234 中描述.  见上文 [1].

.. [7] http://python.ca/nas/python/generator.diff

.. [8] PEP 236, 返回 __future__, Peters
       http://www.python.org/dev/peps/pep-0236/

.. [9] 要试验这个实现, 请根据 http://sf.net/cvs/?group_id=5470 上的说明
       从 CVS 中查看 Python. 请注意, std 测试 ``Lib/test/test_generators.py`` 包含很多示例,
	   包括本 PEP 中的所有人员.


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
