
PEP: 265
Title: Sorting Dictionaries by Value
Version: $Revision$
Last-Modified: $Date$
Author: g2@iowegian.com (Grant Griffin)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 8-Aug-2001
Python-Version: 2.2
Post-History:


Abstract
========

该 PEP 建议对字典进行 "按值排序" 操作.
主要的好处是在 "内置电池" 方面支持一种常见的 Python 惯用语法,
以其当前的形式, 对于初学者来说难以理解并且对于所有人来说都很麻烦.


BDFL Pronouncement
==================

这个 PEP 被拒绝是因为 Py2.4 的 ``sorted()`` 内置函数很大程度上满足了它的需要::

    >>> sorted(d.iteritems(), key=itemgetter(1), reverse=True)
    [('b', 23), ('d', 17), ('c', 5), ('a', 2), ('e', 1)]

或仅仅是 keys::

    sorted(d, key=d.__getitem__, reverse=True)
    ['b', 'd', 'c', 'a', 'e']

此外, Python 2.5 的 ``heapq.nlargest()`` 函数解决了仅查找一些最高价值项的常见用例::

    >>> nlargest(2, d.iteritems(), itemgetter(1))
    [('b', 23), ('d', 17)]


Motivation
==========

字典的一个常见用法是通过在第一次出现时将 ``d[key]`` 的值设置为1来计算出现次数,
然后在每次后续出现时递增该值. 这可以通过几种不同的方式完成, 但 ``get()`` 方法最简洁::

        d[key] = d.get(key, 0) + 1

一旦计算了所有事件, 生成的字典的常见用法是通常首先使用最大值, 打印发生排序顺序的命令.

这导致需要按值对字典的项进行排序. 在 Python 中执行此操作的规范方法是
首先使用 ``d.items()`` 来获取字典项的列表, 然后将每个项的元组的顺序
从 (key, value) 反转为 (value, key), 然后对列表进行排序;
由于 Python 基于元组的第一项对列表进行排序, 因此 (反转) 项的列表按值排序.
如果需要, 然后可以反转列表, 并且可以将元组重新反转回 (key, value).
(但是, 根据我的经验, 反向元组排序适用于大多数用途, 例如, 打印出列表.)

例如, 给定的出现次数为::

    >>> d = {'a':2, 'b':23, 'c':5, 'd':17, 'e':1}

我们可能会做::

    >>> items = [(v, k) for k, v in d.items()]
    >>> items.sort()
    >>> items.reverse()             # so largest is first
    >>> items = [(k, v) for v, k in items]

导致::

    >>> items
    [('b', 23), ('d', 17), ('c', 5), ('a', 2), ('e', 1)]

它按值从大到小顺序显示列表. (在这种情况下, 发现 ``'b'`` 的次数最多.)

这很好, 但在两个方面 "难以使用". 首先, 虽然老练的 Pythoneers 知道这个习惯用法,
但对于新手来说这一点并不明显 -- 无论是在算法上 (反转项为元组的顺序)
还是实现 (使用列表推导 -- 这是一个高级的 Python 特性).
其次, 它需要反复输入大量的 "垃圾", 导致乏味和错误.

因此, 我们宁愿 Python 提供一种按值排序字典的方法, 这对新手来说很容易理解
(或者, 更好的是, 不要必须理解) 并且更容易让所有人使用.


Rationale
=========

正如 Tim Peters 所指出的那样, 这种事情带来了试图成为所有人的所有事情的问题.
因此, 我们将限制其范围, 试图打出 "the sweet spot". 当然,
可以使用现有方法 "手动" 处理不寻常的情况 (例如, 通过自定义比较功能进行分类).

这里有一些简单的可能性:

可以使用具有默认值的新参数来扩展 ``items()`` 词典方法,
该默认值提供完全向后兼容性::

    (1) items(sort_by_values=0, reversed=0)

或者只是::

    (2) items(sort_by_values=0)

因为反转列表很容易.

或者, ``items()`` 可以让我们控制 (key, value) 顺序::

    (3) items(values_first=0)

同样, 这完全向后兼容. 它的工作量比其他工作少,
但它至少可以简化按值排序问题中最复杂或最棘手的部分:
反转项为元组的顺序. 使用它非常简单::

    items = d.items(1)
    items.sort()
    items.reverse()         # (if desired)

由于必须处理默认参数, 前面三种方法的主要缺点是无参数 ``items()`` 情况的额外开销.
(但是, 如果假设 ``items()`` 主要用于创建按值排序列表, 那么这在实践中并不是真正的缺点.)

或者, 我们可能会添加一个新的字典方法, 它以某种方式体现 "排序". 这种方法有两个优点.
首先, 它避免了增加 ``items()`` 方法的开销. 其次, 对于新手来说, 它可能更容易获得:
当他们去寻找一种排序字典的方法时, 他们希望遇到这个,
他们不必理解元组反转和列表排序的细节点来实现按值排序.

为了允许按键或值和前进或后退顺序的四种基本可能性, 我们可以添加此方法::

    (4) sorted_items(by_value=0, reversed=0)

我相信最常见的情况实际上是 ``by_value=1, revers=1``,
但这里给出的默认值可能会减少用户的惊喜:
``sorted_items()`` 与 ``items()`` 相同, 后跟 ``sort()``.

最后 (作为最后的手段), 我们可以使用::

    (5) items_sorted_by_value(reversed=0)


Implementation
==============

拟议的字典方法必然会用 C 实现. 据推测, 实现起来相当简单,
因为它只需要添加一些调用 Python 的现有机制.


Concerns
========

除了在可能性1到3中已经解决的运行时开销之外, 对该提议的关注可能将
属于 "feature bloat" 或 "code bloat" 的类别. 但是,
我相信这里提出的一些建议会导致极少的臃肿, 导致臃肿和 "增值" 之间的良好权衡.

Tim Peters 指出, 在 C 语言中实现它可能并不比在 Python 中实现它快得多. 然而,
这里的主要好处是 "可访问性" 和 "易用性", 而不是 "速度". 因此, 只要它没有明显变慢
(在简单的 ``items()`` 的情况下, 速度不必考虑.)


References
==========

2001年8月 comp.lang.python 上出现了一个名为 "计数事件" 的相关线程.
这包括通过将其作为可重用的 Python 函数和类实现系统化按值排序问题的方法示例.


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  

