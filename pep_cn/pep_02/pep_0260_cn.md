
PEP: 260
Title: Simplify xrange()
Version: $Revision$
Last-Modified: $Date$
Author: guido@python.org (Guido van Rossum)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 26-Jun-2001
Python-Version: 2.2
Post-History: 26-Jun-2001


Abstract
========

这个 PEP 建议从 ``x[i:j]`` 和 ``x*n`` 等一些很少使用的行为中去掉 ``xrange()`` 对象.


Problem
=======

``xrange()`` 函数有一个惯用的用法::

    for i in xrange(...): ...

然而, ``xrange()`` 对象有一堆很少使用的行为, 试图让它更像序列. 这些很少使用,
以至于历史上它们都有严重的错误 (例如, 一个错误), 这些错误在几个版本中未被发现.

我声称最好丢弃这些未使用的功能.
这将简化实施, 测试和文档, 并减少维护和代码大小.


Proposed Solution
=================

我建议将 ``xrange()`` 对象剥离到最低限度. 唯一保留的序列行为是 ``x[i]``,
``len(x)`` 和 ``repr(x)``. 特别是, 这些行为将被取消:

* ``x[i:j]`` (切片)
* ``x*n``, ``n*x`` (序列重复)
* ``cmp(x1, x2)`` (对比)
* ``i in x`` (容量测试)
* ``x.tolist()`` 方法
* ``x.start``, ``x.stop``, ``x.step`` 属性

我还建议更改 ``PyRange_New()`` C API 的签名来删除第4个参数 (重复计数).

通过实现自定义迭代器类型, 我们可以加快常用, 但这是可选的
(默认序列迭代器就可以了).


Scope
=====

这个 PEP 影响 ``xrange()`` 内置函数和 ``PyRange_New()`` C API.


Risks
=====

有人的代码可能依赖于扩展代码, 而且这段代码会破坏. 但是,
鉴于历史上扩展代码中的错误已经长时间未被发现, 因此很多代码不太可能受到影响.


Transition
==========

为了向后兼容, 现有功能仍将存在于 Python 2.2 中, 但会触发警告.
在 Python 2.2 final 发布一年后 (可能在 2.4 中), 该功能将被删除.


Copyright
=========

This document has been placed in the public domain.


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
