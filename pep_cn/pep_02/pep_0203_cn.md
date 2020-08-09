
PEP: 203
Title: Augmented Assignments
Version: $Revision$
Last-Modified: $Date$
Author: thomas@python.org (Thomas Wouters)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 13-Jul-2000
Python-Version: 2.0
Post-History: 14-Aug-2000


Introduction
============

此 PEP 描述了 Python 2.0 的 *增强任务* 提议. 此 PEP 跟踪此功能的状态和所有权,
计划在 Python 2.0 中引入. 它包含功能的说明, 并概述了支持该功能所需的更改.
本 PEP 总结了邮件列表论坛 [1]_ 中的讨论, 并在适当的地方提供了 URL 以获取更多信息.
此文件的 CVS 修订历史记录包含确定的历史记录.


Proposed Semantics
==================

为 Python 添加扩充赋值的建议补丁引入了以下新运算符::

+= -= *= /= %= **= <<= >>= &= ^= |=

它们实现与它们的普通二进制形式相同的运算符, 除了当左侧对象支持它时操作 *in-place*, 并且左侧只评估一次.

它们真正表现为增强赋值, 因为它们执行所有正常的加载和存储操作, 以及它们要执行的二进制操作.
所以, 特定表达式::

    x += y

加载对象 ``x``, 然后添加 ``y``, 并将生成的对象存回原始位置.
对两个参数执行的精确操作取决于 ``x`` 的类型, 可能还有 ``y``.

在 Python 中扩充赋值背后的想法是, 它不仅是一种更简单的方法来编写将二进制操作的结果
存储在其左侧操作数中的常见做法, 而且还有一种方法可以解决左侧操作数的问题.
知道它应该 *自己* 运行, 而不是创建自己的修改副本.

为了实现这一点, 在 Python 类和 C 扩展类型中添加了许多新的 *钩子*,
当有问题的对象被用作扩充赋值操作的左侧时, 它们被调用.
如果类或类型未实现 *in-place* 钩子, 则使用特定二进制操作的常规钩子.

因此, 给定一个实例对象 ``x``, 表达式::

    x += y

试图调用 ``x.__iadd__(y)``, 这是 ``__add__`` 的 *in-place* 变体.
如果 ``__iadd__`` 不存在, 则尝试 ``x.__add__(y)``,
如果 ``__add__`` 也丢失了, 最后 ``y.__radd__(x)``.
没有 ``__iadd__`` 的 *right-hand-side* 变体,
因为这需要 ``y`` 知道如何原位修改 ``x``, 这至少可以说是不安全的.
``__iadd__`` 钩子的行为类似于 ``__add__``, 返回操作的结果
(可能是 ``self``), 它将被分配给变量 ``x``.

对于 C 扩展类型, *hooks* 是 ``PyNumberMethods`` 和 ``PySequenceMethods`` 结构的成员.
一些特殊的语义适用于使用这些方法, 以及 Python 实例对象和 C 类型的混合, 尽可能不足为奇.

在 ``x <augop> y`` (或使用 ``PyNumber_InPlace`` API 函数的类似情况) 的一般情况下,
正在操作的主要对象是 ``x``. 这与普通的二进制操作不同, 其中 ``x`` 和 ``y`` 可以被认为是 *co-operated*,
因为与二进制操作不同, 原位操作中的操作数不能被交换。 但是，当不支持原位修改时,
原位操作会回退到正常的二进制操作, 从而导致以下规则:

- 如果左手对象 (``x``) 是一个实例对象, 并且它有一个 ``__coerce__`` 方法,
  则用 ``y`` 作为参数调用该函数. 如果强制成功, 并且得到的左手对象是一个与 ``x`` 不同的对象,
  则停止处理它就位并调用适当的函数进行正常的二进制操作, 强制执行 ``x`` 和 ``y`` 作为参数.
  操作的结果是该函数返回的任何内容.

  如果强制不会为 ``x`` 产生不同的对象, 或者 ``x`` 没有定义 ``__coerce__`` 方法,
  并且 ``x`` 对于此操作有适当的 ``__ihook__``, 用 ``y`` 作为参数调用该方法,
  操作的结果是该方法返回的任何内容.

- 否则, 如果左侧对象不是实例对象, 但其类型确定了此操作的原位操作函数,
  则使用 ``x`` 和 ``y`` 作为参数调用该函数, 结果操作是该函数返回的任何内容.

  请注意, 在这种情况下, 不会对 ``x`` 或 ``y`` 进行强制, 并且对于 C 类型来说,
  接收实例对象作为第二个参数是完全有效的. 这是普通二进制操作不可能发生的事情.

- 否则, 将其处理为正常的二进制操作 (非原位), 包括参数强制.
  简而言之, 如果任一参数是实例对象, 则通过 ``__coerce__``, ``__hook__``
  和 ``__rhook__`` 解析操作. 否则, 两个对象都是 C 类型, 它们被强制转换并传递给适当的函数.

- 如果找不到处理操作的方法, 则引发一个带有特定于操作的错误消息的 ``TypeError``.

- 存在一些特殊的外壳来解释 ``+`` 和 ``*`` 的情况, 这对于序列具有特殊的意义:
  对于 ``+``, 序列连接, 没有强制, 如果 C 做了什么呢? type 定义 ``sq_concat``
  或 ``sq_inplace_concat``. 对于 ``*``, 序列重复, ``y`` 在调用 ``sq_inplace_repeat``
  和 ``sq_repeat`` 之前转换为 C 整数. 即使 ``y`` 是一个实例, 也可以这样做,
  但如果 ``x`` 是一个实例则不行.

原位函数应该始终返回一个新引用, 如果操作确实原位执行,
则返回旧的 ``x`` 对象, 或者返回给新对象.


Rationale
=========

将此功能添加到 Python 有两个主要原因: 表达式的简单性和对原位操作的支持.
最终结果是在语法的简单性和表达的简单性之间进行权衡; 与大多数新功能一样,
增强分配不会添加以前不可能的任何内容. 它只是让这些事情变得更容易.

添加扩充赋值将使 Python 的语法更复杂. 现在有12个赋值操作, 而不是单个赋值操作,
其中11个也执行二进制操作. 然而, 这11种新的赋值形式很容易理解为赋值和二元运算之间的耦合,
并且它们不需要大的概念上的飞跃来理解. 此外, 具有增强赋值的语言已经表明它们是一种流行的, 常用的功能.
表格的表达式::

    <x> = <x> <operator> <y>

在这些语言中很常见, 以使额外的语法值得, 而 Python 没有那么多的表达式.
事实上, 恰恰相反, 因为在 Python 中你也可以用二元运算符连接列表,
这是经常做的事情. 将上面的表达式写为::

    <x> <operator>= <y>

它更易读, 更容易出错, 因为对于读者来说, 它是 ``<x>`` 正在被改变,
而不是 ``<x>`` 几乎被所取代的东西, 但是不完全, 完全不像 ``<x>``.

新的原位操作对于矩阵计算和需要大对象的其他应用程序特别有用.
为了有效地处理可用的程序存储器, 这样的包不能盲目地使用当前的二进制操作.
因为这些操作总是创建一个新对象, 所以向现有 (大) 对象添加单个项将导致复制整个对象
(这可能导致应用程序内存不足), 添加单个项, 然后可能删除原始对象, 取决于引用计数.

要解决此问题, 当前包必须使用方法或函数来原位修改对象, 这绝对不如扩充赋值表达式可读.
增强赋值不会解决这些包的所有问题, 因为有些操作无法在有限的二元运算符集中表示,
但它是一个开始. 一个不同的 PEP [3]_ 正在考虑添加新的运算符.


New methods
===========

建议的实现添加了以下11个可能的 *钩子*, Python 类可以实现这些钩子来重载扩充的赋值操作::

    __iadd__
    __isub__
    __imul__
    __idiv__
    __imod__
    __ipow__
    __ilshift__
    __irshift__
    __iand__
    __ixor__
    __ior__

``__iadd__`` 中的 *i* 代表 *原位*.

对于 C 扩展类型, 添加以下结构成员.

对于 ``PyNumberMethods``::

    binaryfunc nb_inplace_add;
    binaryfunc nb_inplace_subtract;
    binaryfunc nb_inplace_multiply;
    binaryfunc nb_inplace_divide;
    binaryfunc nb_inplace_remainder;
    binaryfunc nb_inplace_power;
    binaryfunc nb_inplace_lshift;
    binaryfunc nb_inplace_rshift;
    binaryfunc nb_inplace_and;
    binaryfunc nb_inplace_xor;
    binaryfunc nb_inplace_or;

对于 ``PySequenceMethods``::

    binaryfunc sq_inplace_concat;
    intargfunc sq_inplace_repeat;

为了保持二进制兼容性, ``tp_flags`` TypeObject 成员用于
确定所讨论的 TypeObject 是否为这些槽分配了空间.
直到二进制兼容性的彻底中断 （在 2.0 之前可能发生或可能不发生),
想要使用其中一个新结构成员的代码必须首先检查它们是否可用于 ``PyType_HasFeature()`` 宏::

    if (PyType_HasFeature(x->ob_type, Py_TPFLAGS_HAVE_INPLACE_OPS) &&
        x->ob_type->tp_as_number && x->ob_type->tp_as_number->nb_inplace_add) {
            /* ... */
 
甚至在测试 ``NULL`` 值的方法槽之前, 必须进行此检查!
宏只测试槽是否可用, 而不是它们是否是填充方法.


Implementation
==============

增强赋值的当前实现 [2]_ 除了已经涵盖的方法和槽外,
还增加了13个新的字节码和13个新的 API 函数.

API 函数只是当前二进制操作 API 函数的原位版本::

    PyNumber_InPlaceAdd(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceSubtract(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceMultiply(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceDivide(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceRemainder(PyObject *o1, PyObject *o2);
    PyNumber_InPlacePower(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceLshift(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceRshift(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceAnd(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceXor(PyObject *o1, PyObject *o2);
    PyNumber_InPlaceOr(PyObject *o1, PyObject *o2);
    PySequence_InPlaceConcat(PyObject *o1, PyObject *o2);
    PySequence_InPlaceRepeat(PyObject *o, int count);

它们调用 Python 类钩子 (如果其中一个对象是 Python 类实例) 或 C 类型的数字或序列方法.

新的字节码是::

    INPLACE_ADD
    INPLACE_SUBTRACT
    INPLACE_MULTIPLY
    INPLACE_DIVIDE
    INPLACE_REMAINDER
    INPLACE_POWER
    INPLACE_LEFTSHIFT
    INPLACE_RIGHTSHIFT
    INPLACE_AND
    INPLACE_XOR
    INPLACE_OR
    ROT_FOUR
    DUP_TOPX

``INPLACE_ *`` 字节码反映了 ``BINARY_ *`` 字节码, 除了它们被实现为对 ``InPlace`` API 函数的调用.
另外两个字节码是 *utility* 字节码: ``ROT_FOUR`` 的行为类似于 ``ROT_THREE``,
只是四个最顶层的堆栈项被旋转.

``DUP_TOPX`` 是一个字节码, 它接受一个参数, 该参数应该是1到5之间的整数 (包括 1 和 5),
这是一个块中要复制的项目数. 给定这样的堆栈 (列表的右侧是堆栈的 *顶部*) ::

    [1, 2, 3, 4, 5]

``DUP_TOPX 3`` 将复制前3项, 从而产生这个堆栈::

    [1, 2, 3, 4, 5, 3, 4, 5]

参数为 1 的 ``DUP_TOPX`` 与 ``DUP_TOP`` 相同. 限制 5 纯粹是一个实现限制.
增强赋值的实现只需要带有参数 2 和 3 的 ``DUP_TOPX``, 并且可以不使用这个新的操作码,
代价是 ``DUP_TOP`` 和 ``ROT_ *``.


Open Issues
===========

``PyNumber_InPlace`` API 只是普通 ``PyNumber`` API 的一个子集:
只包括支持扩充赋值语法所需的那些函数. 如果需要其他原位 API 函数, 可以稍后添加它们.

``DUP_TOPX`` 字节码是一个方便的字节码, 实际上并不是必需的.
应该考虑这个字节码是否值得拥有. 此时似乎没有其他可能的用于此字节码.


Copyright
=========

This document has been placed in the public domain.


References
==========

.. [1] http://www.python.org/pipermail/python-list/2000-June/059556.html

.. [2] http://sourceforge.net/patch?func=detailpatch&patch_id=100699&group_id=5470

.. [3] PEP 211, Adding A New Outer Product Operator, Wilson
       http://www.python.org/dev/peps/pep-0211/


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
