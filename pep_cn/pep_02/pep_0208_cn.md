
PEP: 208  
Title: Reworking the Coercion Model  
Version: $Revision$  
Last-Modified: $Date$  
Author: nas@arctrix.com (Neil Schemenauer), mal@lemburg.com (Marc-AndrÃ© Lemburg)  
Status: Final  
Type: Standards Track  
Content-Type: text/x-rst  
Created: 04-Dec-2000  
Python-Version: 2.1  
Post-History:  


Abstract
========

许多 Python 类型实现了数字操作. 当数字操作的参数具有不同类型时,
解释器会尝试将参数强制转换为公共类型. 然后使用这种常见类型执行数字运算.
此 PEP 提出了一个新类型标志, 指示不应该强制类型的数值操作的参数.
不支持提供的类型的操作通过返回一个新的单例对象来指示它.
未设置类型标志的类型以向后兼容的方式处理.
允许操作处理不同类型通常比使用解释器进行强制操作更简单, 更灵活, 更快速.


Rationale
=========

当实现数字或其他相关操作时, 通常希望不仅仅提供一种类型的操作数之间的操作,
例如, 整数+整数, 但也将操作背后的想法概括为其他类型组合, 例如, 整数+浮点数.

这种混合类型情况的一种常见方法是提供一种方法, 将操作数 "提升" 到一个公共类型 (强制),
然后使用该类型的操作数方法作为执行机制. 然而, 这种策略有一些缺点:

* "提升" 过程创建至少一个新的 (临时) 操作数对象,

* 由于强制方法没有被告知要遵循的操作, 因此不可能实现类型的操作特定强制,

* 没有优雅的方式解决情况是一种常见的类型不在旁边，而且

* 在操作方法本身之前, 必须始终调用强制方法.

显然需要针对这种情况的修复, 因为这些缺点使得需要这些特征的类型的实现非常麻烦,
如果不是不可能的话. 举个例子, 看看 ``DateTime`` 和 ``DateTimeDelta`` [1]_ 类型,
第一个是绝对的, 第二个是相对的. 您始终可以将相对值添加到绝对值, 从而获得新的绝对值.
然而, 现有的强制机制没有可用于实施该行动的共同类型.

目前, 解释器专门处理 ``PyInstance`` 类型, 因为它们的数值方法是传递不同类型的参数.
删除此特殊情况简化了解释器, 并允许其他类型实现与实例类型相似的数值方法.
这对 ExtensionClass 等扩展类型特别有用.


Specification
=============

处理不同操作数类型的过程只是留给操作而不是使用核心强制方法.
如果操作发现它无法处理给定的操作数类型组合, 它可能会返回一个特殊的单例作为指示符.

请注意, 用 Python 编写的 "数字" (实现数字协议或其中一部分的任何内容)
已经使用了此策略的第一部分 - 它是我们关注的 C 级 API.

为了保持近 100％ 的向后兼容性, 我们必须非常小心地使对新策略 (旧样式数字)
一无所知的数字与那些期望新方案 (新样式数字) 的数字一样.
此外, 二进制兼容性是必须的, 这意味着如果数字指示这些操作的可用性,
则解释器可以仅访问和使用新样式操作.

当且仅当它设置类型标志 ``Py_TPFLAGS_CHECKTYPES`` 时, 解释器才会考虑新的样式编号.
旧样式数字和新样式数字之间的主要区别在于数字槽函数不能再假设为传递相同类型的参数.
新样式槽必须检查所有参数的正确类型, 并自行实现必要的转换.
这似乎可以代表类型实现者引起更多的工作, 但实际上并不比为旧式强制槽编写相同类型的例程更困难.

如果新样式槽发现它无法处理传递的参数类型组合,
它可能会向调用者返回特殊单例 ``Py_NotImplemented`` 的新引用.
这将导致调用者尝试其他操作数操作槽, 直到找到确实实现特定类型组合操作的槽.
如果没有可能成功的槽, 它会引发一个 ``TypeError``.

为了使实现易于理解 (整个主题足够深奥), 引入了处理数值运算的新层.
该层处理在处理旧样式数字和新样式数字的所有可能组合时需要考虑的所有不同情况.
它由两个静态函数 ``binary_op()`` 和 ``ternary_op()`` 实现, 它们都是内部函数,
只有 Objects/abstract.c 中的函数才能访问. 数字 API (``PyNumber_ *``) 很容易适应这个新层.

作为副作用, 所有数字槽都可以进行 NULL 检查
(无论如何都必须这样做, 因此添加的功能无需额外费用).


层用于执行二进制操作的方案如下:

===  ===  =================================
v    w    Action taken
===  ===  =================================
new  new  v.op(v,w), w.op(v,w)
new  old  v.op(v,w), coerce(v,w), v.op(v,w)
old  new  w.op(v,w), coerce(v,w), v.op(v,w)
old  old  coerce(v,w), v.op(v,w)
===  ===  =================================

指示的动作序列从左到右执行, 直到操作成功并返回有效结果 (!=``Py_NotImplemented``) 或引发异常.
异常按原样返回到调用函数. 如果一个槽返回 ``Py_NotImplemented``, 则执行序列中的下一个项.

注意, coerce(v, w) 将通过调用 ``PyNumber_Coerce()`` 来使用旧样式 ``nb_coerce`` 槽方法。.

三元运算符还有一些示例要处理:

===  ===  ===  ====================================================
v    w    z    Action taken
===  ===  ===  ====================================================
new  new  new  v.op(v,w,z), w.op(v,w,z), z.op(v,w,z)
new  old  new  v.op(v,w,z), z.op(v,w,z), coerce(v,w,z), v.op(v,w,z)
old  new  new  w.op(v,w,z), z.op(v,w,z), coerce(v,w,z), v.op(v,w,z)
old  old  new  z.op(v,w,z), coerce(v,w,z), v.op(v,w,z)
new  new  old  v.op(v,w,z), w.op(v,w,z), coerce(v,w,z), v.op(v,w,z)
new  old  old  v.op(v,w,z), coerce(v,w,z), v.op(v,w,z)
old  new  old  w.op(v,w,z), coerce(v,w,z), v.op(v,w,z)
old  old  old  coerce(v,w,z), v.op(v,w,z)
===  ===  ===  ====================================================

与上面相同的注释, 除了强制 (v, w, z) 实际上 ::

    if z != Py_None:
        coerce(v,w), coerce(v,z), coerce(w,z)
    else:
        # treat z as absent variable
        coerce(v,w)


目前的实现已经使用了这个方案 (只有一个三元槽: ``nb_pow(a, b, c)``).

注意, 数字协议也用于一些其他相关任务, 例如, 序列连接.
通过对类型组合实施右边操作, 否则这些也可以从新机制中受益,
否则将无法工作. 举个例子, 取字符串连接: 目前你只能做 string + string.
使用新机制, 一个新的类似字符串的类型可以实现 new_type + string 和 string + new_type,
即使字符串对 new_type 一无所知.

由于比较也依赖于强制 (每次将整数与浮点数进行比较时, 首先将整数转换为浮点数, 然后进行比较...),
需要一个新的槽来处理数值比较 ::

    PyObject *nb_cmp(PyObject *v, PyObject *w)

此槽应该比较两个对象并返回一个表示结果的整数对象. 目前, 此结果整数可能只有 -1, 0, 1.
如果槽无法处理类型组合, 它可能会返回对 ``Py_NotImplemented`` 的引用.
[ XXX 注意, 这个时段仍处于不稳定状态, 因为它应该考虑到丰富的比较 (即 PEP 207) ]

数字比较由新的数字协议 API 处理::

    PyObject *PyNumber_Compare(PyObject *v, PyObject *w)

此函数将两个对象比较为 "数字", 并返回一个表示结果的整数对象.
目前, 此结果整数可能只有 -1, 0, 1. 如果给定对象无法处理操作, 则会引发 ``TypeError``.

``PyObject_Compare()`` API 需要相应调整才能使用这个新的 API.

其他更改包括调整一些内置函数 (例如, ``cmp()``) 来使用此 API.
另外, ``PyNumber_CoerceEx()`` 需要在调用 ``nb_coerce`` 槽之前检查新的样式数字.
新样式数字不提供强制槽, 因此无法明确强制.


Reference Implementation
========================

可以通过 Source Forge 补丁管理器获得 CVS 版 Python 的初步补丁 [2]_.


Credits
=======

这个 PEP 和补丁很大程度上基于 Marc-AndrÃ© Lemburg 所做的工作 [3]_.


Copyright
=========

This document has been placed in the public domain.


References
==========

.. [1] http://www.lemburg.com/files/python/mxDateTime.html
.. [2] http://sourceforge.net/patch/?func=detailpatch&patch_id=102652&group_id=5470
.. [3] http://www.lemburg.com/files/python/CoercionProposal.html




..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  sentence-end-double-space: t  
  fill-column: 70  
  coding: utf-8  
  End:  
