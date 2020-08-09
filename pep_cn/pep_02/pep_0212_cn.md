
PEP: 212
Title: Loop Counter Iteration
Version: $Revision$
Last-Modified: $Date$
Author: nowonder@nowonder.de (Peter Schneider-Kamp)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 22-Aug-2000
Python-Version: 2.1
Post-History:


Rejection Notice
================

该 PEP 已被拒绝. 在 PEP 279 [6]_ 中引入的 ``enumerate()`` 涵盖了本 PEP 中提出的用例,
并且 PEP 作者无法访问.


Introduction
============

该 PEP 描述了在 for 循环中暴露循环计数器的提出的特征. 此 PEP 跟踪此功能的状态和所有权.
它包含功能的说明, 并概述了支持该功能所需的更改. 本 PEP 总结了在邮件列表论坛中进行的讨论,
并在适当情况下提供了 URL 以获取更多信息. 此文件的 CVS 修订历史记录包含确定的历史记录.


Motivation
==========

Python 中的标准 for 循环遍历序列 [1]_ 的元素.
通常希望循环索引或者循环索引元素和索引.

用于实现此目的的常用习语是不直观的.
该 PEP 提出了两种不同的方法来公开指数.


Loop counter iteration
======================

循环索引的当前习惯用法是使用内置的 ``range`` 函数::

    for i in range(len(sequence)):
        # work with index i

循环使用元素和索引可以通过旧的习语或使用新的 ``zip`` 内置函数来实现 [2]_::

    for i in range(len(sequence)):
        e = sequence[i]
        # work with index i and element e

或 ::

   for i, e in zip(range(len(sequence)), sequence):
      # work with index i and element e


The Proposed Solutions
======================

已经讨论了三种解决方案. 一个添加非保留关键字, 另一个添加两个内置函数.
第三种解决方案为序列对象添加方法.


Non-reserved keyword ``indexing``
=================================

这个解决方案通过添加一个可选的 ``<variable> indexing`` 子句来扩展 for 循环的语法,
该子句也可以用来代替 ``<variable> in`` 子句.

因此, 循环序列的索引将成为 ::

    for i indexing sequence:
        # work with index i

对指数和元素进行循环同样如此 ::

    for i indexing e in sequence:
        # work with index i and element e


Built-in functions ``indices`` and ``irange``
=============================================

这个解决方案增加了两个内置函数 ``indices`` 和 ``orange``.
这些语义可以描述如下 ::

    def indices(sequence):
        return range(len(sequence))

    def irange(sequence):
        return zip(range(len(sequence)), sequence)

这些函数可以热切地或懒惰地实现, 并且应该易于扩展, 利于接受多个序列参数.

这些函数的使用将简化循环索引以及元素和索引的惯用语 ::

    for i in indices(sequence):
        # work with index i

    for i, e in irange(sequence):
        # work with index i and element e


Methods for sequence objects
============================

这个解决方案建议在序列中添加 ``indices``, ``items`` 和 ``values`` 方法,
它们只能分别对索引, 索引和元素以及元素进行循环.

这将极大地简化用于循环索引和循环遍及元素和索引的习语 ::

    for i in sequence.indices():
        # work with index i

    for i, e in sequence.items():
        # work with index i and element e

另外, 它允许以一致的方式循环遍历序列和字典的元素 ::

    for e in sequence_or_dict.values():
        # do something with element e


Implementations
===============

对于所有三种解决方案, 在 SourceForge 上存在一些或多或少粗糙的补丁作为补丁:

- ``for i indexing a in l``: 暴露 for 循环计数器 [3]_
- 在内置函数中添加 ``indices()`` 和 ``orange()`` [4]_
- 将 ``items()`` 方法添加到列表对象 [5]_

所有这些都被 BDFL 宣布并拒绝.

请注意, ``indexing`` 关键字在语法中只是一个 ``NAME``,
因此不会妨碍 ``indexing`` 的一般用法.


Backward Compatibility Issues
=============================

由于没有添加关键字并且现有代码的语义保持不变,
因此可以在不破坏现有代码的情况下实现所有三种解决方案.


Copyright
=========

This document has been placed in the public domain.


References
==========

.. [1] http://docs.python.org/reference/compound_stmts.html#for

.. [2] Lockstep Iteration, PEP 201

.. [3] http://sourceforge.net/patch/download.php?id=101138

.. [4] http://sourceforge.net/patch/download.php?id=101129

.. [5] http://sourceforge.net/patch/download.php?id=101178

.. [6] PEP 279, The enumerate() built-in function, Hettinger
       https://www.python.org/dev/peps/pep-0279/


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
