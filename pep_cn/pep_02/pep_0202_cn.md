
PEP: 202
Title: List Comprehensions
Version: $Revision$
Last-Modified: $Date$
Author: barry@python.org (Barry Warsaw)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 13-Jul-2000
Python-Version: 2.0
Post-History:


Introduction
============

本 PEP 描述了 Python 的语法扩展, 列表推导.


The Proposed Solution
=====================

建议允许使用 for 和 if 子句条件构造列表常量.
现在它们将以相同的方式嵌套循环和 if 语句嵌套.


Rationale
=========

列表推导提供了一种更简洁的方法来创建列表,
在当前使用 ``map()`` 和 ``filter()`` 或嵌套循环的情况下.


Examples
========

::

    >>> print [i for i in range(10)]
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

    >>> print [i for i in range(20) if i%2 == 0]
    [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

    >>> nums = [1, 2, 3, 4]
    >>> fruit = ["Apples", "Peaches", "Pears", "Bananas"]
    >>> print [(i, f) for i in nums for f in fruit]
    [(1, 'Apples'), (1, 'Peaches'), (1, 'Pears'), (1, 'Bananas'),
     (2, 'Apples'), (2, 'Peaches'), (2, 'Pears'), (2, 'Bananas'),
     (3, 'Apples'), (3, 'Peaches'), (3, 'Pears'), (3, 'Bananas'),
     (4, 'Apples'), (4, 'Peaches'), (4, 'Pears'), (4, 'Bananas')]
    >>> print [(i, f) for i in nums for f in fruit if f[0] == "P"]
    [(1, 'Peaches'), (1, 'Pears'),
     (2, 'Peaches'), (2, 'Pears'),
     (3, 'Peaches'), (3, 'Pears'),
     (4, 'Peaches'), (4, 'Pears')]
    >>> print [(i, f) for i in nums for f in fruit if f[0] == "P" if i%2 == 1]
    [(1, 'Peaches'), (1, 'Pears'), (3, 'Peaches'), (3, 'Pears')]
    >>> print [i for i in zip(nums, fruit) if i[0]%2==0]
    [(2, 'Peaches'), (4, 'Bananas')]


Reference Implementation
========================

列表推导成为 Python 语言的一部分, 发布 2.0, 在 [1]_ 中有记录.


BDFL Pronouncements
===================
* 上面提出的语法是正确的.

* 形式 ``[x, y for ...]`` 是不允许的;
  一个人需要写 ``[(x, y) for ...]``.

* 形式 ``[... for x... for y...]`` 嵌套, 最后一个索引变化最快, 就像嵌套 for 循环一样.


References
==========

.. [1] http://docs.python.org/reference/expressions.html#list-displays



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
