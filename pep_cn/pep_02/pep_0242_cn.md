
PEP: 242
Title: Numeric Kinds
Version: $Revision$
Last-Modified: $Date$
Author: paul@pfdubois.com (Paul F. Dubois)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 17-Mar-2001
Python-Version: 2.2
Post-History: 17-Apr-2001


Abstract
========

该提议为用户提供了对数值计算精度和范围的可选控制,
因此计算可以编写一次并在至少具有所需精度和范围的任何地方运行.
它向后兼容现有代码. 阐明十进制字面量的含义.


Rationale
=========

目前, 除了 Fortran 90 之外, 每种语言都不可能以可移植的方式编写程序,
使用浮点数并获得大致相同的答案, 无论平台如何 - 或者如果不可能则拒绝编译.
Python 目前只有一个浮点数类型, 等于 C 实现中的 C double.

没有类型存在对应于单个或四个浮点数. 它会使语言变得复杂, 试图直接引入这些类型,
并且它们的后续使用将不可移植. 此提议类似于 Fortran 90 "kind" 解决方案, 适用于 Python 环境.
通过此功能, 可以通过更改单行将整个计算从一个精度级别切换到另一个级别.
如果特定机器上不存在所需的精度, 程序将失败而不是得到错误的答案.
由于这种风格的编码会涉及对失败的例程的早期调用, 这是不编译的下一个最好的事情.


Supported Kinds of Ints and Floats
==================================

复数在下面单独处理, 因为 Python 可以在没有它们的情况下构建.

每个 Python 编译器可以根据需要定义尽可能多的 "种类" 的整数和浮点数,
除了它必须支持至少两种对应于现有 int 和 long 的整数,
并且必须支持至少一种浮点数, 相当于现在的浮点数.

这些所需类型的范围和精度与处理器有关,
目前除了 "长整数" 类型, 它可以保持任意整数.

内置函数 ``int()``, ``long()`` 和 ``float()`` 将输入转换为这些默认类型, 就像它们目前所做的那样. 
(请注意, Unicode 字符串实际上是字符串的 "种类", 并且知识渊博的人可能能够扩展此 PEP 来涵盖该情况.)

在每种类型 (整数, 浮点数) 中, 编译器支持线性排序的一组类型,
其顺序由保持增加的范围或精度的数量的能力确定.


Kind Objects
============

在名为 "种类" 的模块中定义了两个新的标准函数. 它们返回称为类对象的可调用对象.
每个整型或浮点数类对象 f 都有签名 ``result = f(x)``,
每个复杂类对象都有签名 ``result = f(x, y=0.)``.

``int_kind(n)``

   对于一个整数参数 ``n >= 1``, 返回一个可调用对象, 其结果是一个整数类型,
   它将在开放区间中保存一个整数 (``-10**n``, ``10**n``).
   kind 对象接受包含 longs 的整数参数. 如果 ``n == 0``,
   则返回与 Python 字面量 0 对应的 kind 对象.

``float_kind(nd, n)``

   对于 ``nd >= 0`` 和 ``n>=1``, 返回一个可调用对象, 其结果是一个浮点数,
   它将保存一个至少有 nd 精度数字的浮点数和一个基数为 10 的浮点数.
   闭区间的指数 ``[-n, n]``. kind 对象接受 integer 或 float 参数.

  如果 nd 和 n 都为零, 则返回与 Python 字面量 0.0 对应的 kind 对象.

编译器将返回一个类型对象, 该对象对应于具有所需属性的该类型的最小种类.
如果在给定的实现中不存在具有所需质量的类型, 则抛出 "OverflowError" 异常.
一个 kind 函数将其参数转换为目标类型, 但如果结果不适合目标类型的范围, 则抛出 "OverflowError" 异常.

除了它们的可调用行为之外, 有类型的对象具有给出所讨论类型的特征的属性.

1. ``name`` 是种类的名字. 标准种类称为 int, long, double.

2. ``typecode`` 是一个单字母字符串, 适合与 ``Numeric`` 或模块 ``array`` 一起使用来形成这种数组.
    标准类型的类型代码分别是 'i', 'O', 'd'.

3. 整数类具有以下附加属性: ``MAX``, 等于此类允许的最大整数, 或者 long 类的 "None".
    ``MIN``, 等于负的允许整数类, 或者 long 类的 ``None``.

4. Float 种类具有这些附加属性, 其属性等于标准头文件 "float.h" 中相应 C 类型的对应值.
    ``MAX``, ``MIN``, ``DIG``, ``MANT_DIG``, ``EPSILON``, ``MAX_EXP``, ``MAX_10_EXP``,
	``MIN_EXP``, `` MIN_10_EXP``, ``RADIX``, ``ROUNDS`` (==``FLT_RADIX``, 在 float.h 中的 ``FLT_ROUNDS``).
	这些值的类型是整数, 除了 ``MAX``, ``MIN`` 和 ``EPSILON``, 它们是类型对应的 Python 浮点类型.


Attributes of Module kinds
==========================

``int_kinds`` 是可用整数种类的列表, 从最低到最高种类排序.
根据定义, ``int_kinds[-1]`` 是 long 类型.

``float_kinds`` 是可用浮点类型的列表, 从最低到最高类型排序.

``default_int_kind`` 是对应于 Python 字面量 0 的种类对象

``default_long_kind`` 是对应于 Python 字面量 0L 的种类对象

``default_float_kind`` 是对应于 Python 字面量 0.0 的种类对象


Complex Numbers
===============

如果支持复数具有实部和虚部, 这些部分是具有相同类型的浮点数.
如果它支持复数, Python 编译器必须支持它支持的每个浮点类型的复合模拟.

如果支持复数, 则模块类型中提供以下内容:

``complex_kind(nd, n)``

   返回一个可调用对象, 其结果是一个复合类型, 它将包含一个复数, 其中每个组件 (.real, .imag)
   都是 ``float_kind(nd, n)``. kind 对象将接受一个任何整数, 实数或复数类型的参数, 或两个参数, 每个整数或实数.

``complex_kinds`` 是可用 complex 种类的列表, 从最低到最高种类排序.

``default_complex_kind`` 是对应于 Python 字面量 0.0j 的种类对象.
这种名称是 doublecomplex, 其类型代码为 'D'.

Complex  类型对象具有这些添加属性:

``floatkind`` 是相应浮点类型的类型对象.


Examples
========

在模块 myprecision.py 中 ::

    import kinds
    tinyint = kinds.int_kind(1)
    single = kinds.float_kind(6, 90)
    double = kinds.float_kind(15, 300)
    csingle = kinds.complex_kind(6, 90)

在我的其余代码中 ::

    from myprecision import tinyint, single, double, csingle
    n = tinyint(3)
    x = double(1.e20)
    z = 1.2
    # builtin float gets you the default float kind, properties unknown
    w = x * float(x)
    # but in the following case we know w has kind "double".
    w = x * double(z)

    u = csingle(x + z * 1.0j)
    u2 = csingle(x+z, 1.0)

请注意, 通过更改 myprecision.py 中的参数, 可以将整个代码更改为更高的精度.

注释: 请注意, 你不约定 single != double; 但你应该约定 ``double(1.e20)`` 将保存一个精度为15位十进制数的数字
和一个高达 ``10**300`` 的范围或者 ``float_kind`` 调用将失败.


Open Issues
===========

目前尚未提出任何未解决的问题.


Rejection
=========

该 PEP 已被作者关闭. kinds 模块不会添加到标准库中.

没有人反对该提案, 只是对使用它的兴趣不大, 不足以证明将模块添加到标准库中是合理的.
相反, 它将作为 Numerical Python 站点上的单独分发项提供.
在 Numerical Python 的下一个版本中, 它将不再是 Numeric 发行版的一部分.


Copyright
=========

This document has been placed in the public domain.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
