
PEP: 270
Title: uniq method for list objects
Version: $Revision$
Last-Modified: $Date$
Author: jp@demonseed.net (Jason Petrone)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 21-Aug-2001
Python-Version: 2.2
Post-History:


Notice
======

该 PEP 由作者撤回. 他写:

    从列表中删除重复元素是一项常见任务, 但我只能通过两个原因使其成为内置元素.
	第一个是如果能够更快地完成, 情况并非如此. 第二个是它是否使编写代码变得更加容易.
	引入 ``sets.py`` 消除了这种情况, 因为创建一个没有重复的序列只需要选择一个不同的数据结构:
	一个集合而不是一个列表.

如 PEP 218 中所述, 集合正被添加到 Python 2.3 的标准库中.


Abstract
========

该 PEP 建议添加一种方法来删除列表对象中的重复元素.


Rationale
=========

从列表中删除重复项是一项常见任务. 我认为它在列表对象中属于一个方法是有用的和通用的.
当在 C 中实现时, 它还具有更快执行的潜力, 特别是如果不能使用散列或已排序的优化.

在 comp.lang.python 上有很多很多帖子 [1]_ 询问有关执行此任务的最佳方法.
实现最佳实施有点棘手, 为人们节省自己搞清楚的麻烦会很好.


Considerations
==============

Tim Peters 建议尝试使用哈希表, 然后尝试排序, 最后回到普通的模式匹配算法 [2]_.
uniq 应该以牺牲速度为代价来维护列表顺序?

Is it spelled 'uniq' or 'unique'?


Reference Implementation
========================

我写过 BF 算法版本. 在 ``listobject.c`` 中大约有20行代码.
添加对哈希表的支持和排序的重复删除只需要一个小时左右.


References
==========

.. [1] https://groups.google.com/forum/#!searchin/comp.lang.python/duplicates

.. [2] Tim Peters unique() entry in the Python cookbook:
       http://aspn.activestate.com/ASPN/Cookbook/Python/Recipe/52560/index_txt


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  fill-column: 70  
  End:  
