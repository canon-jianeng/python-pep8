
PEP: 204
Title: Range Literals
Version: $Revision$
Last-Modified: $Date$
Author: thomas@python.org (Thomas Wouters)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 14-Jul-2000
Python-Version: 2.0
Post-History:


Introduction
============

本 PEP 描述了 Python 2.0 的 "范围文字" 提议.
本 PEP 跟踪此功能的状态和所有权, 计划在 Python 2.0 中引入.
它包含功能的说明, 并概述了支持该功能所需的更改.
本 PEP 总结了在邮件列表论坛中进行的讨论, 并在适当情况下提供了 URL 以获取更多信息.
此文件的 CVS 修订历史记录包含确定的历史记录.


List ranges
===========

范围是固定步进的数字序列, 通常用于 for 循环.
Python for 循环旨在直接迭代序列::

   >>> l = ['a', 'b', 'c', 'd']
   >>> for item in l:
   ...     print item
   a
   b
   c
   d

但是, 这种解决方案并不总是谨慎的. 首先, 当改变 for 循环体中的序列时出现问题,
导致 for 循环跳过项. 其次, 不可能迭代, 例如, 序列的所有第二个元素.
第三, 有时需要根据其索引处理元素, 这在上述构造中不易获得.

对于这些实例, 以及需要一系列数字的其他实例, Python 提供了 ``range`` 内置函数,
它创建了一个数字列表. ``range`` 函数有三个参数, *start*, *end* 和 *step*.
*start* 和 *step* 是可选的, 默认分别为 0 和 1.

``range`` 函数创建一个数字列表, 从 *start* 开始, 步长为 *step*,
最多但不包括 *end*, 这样 ``range(10)`` 产生一个列表, 它有10个项, 数字 0 到 9.

使用 ``range`` 函数, 上面的例子看起来像这样::

   >>> for i in range(len(l)):
   ...     print l[i]
   a
   b
   c
   d

或者, 从 ``l`` 的第二个元素开始, 然后只处理所有第二个元素::

   >>> for i in range(1, len(l), 2):
   ...     print l[i]
   b
   d

这种方法有几个缺点:

- 目的明确: 添加另一个函数调用, 可能需要额外的算术来确定所需的长度和列表的步骤,
  但不会提高代码的可读性. 此外, 可以通过提供具有相同名称的本地或全局变量来 "遮蔽"
  内置 ``range`` 函数, 从而有效地替换它. 这可能是也可能不是期望的效果.

- 效率: 因为 ``range`` 函数可以被覆盖, Python 编译器不能对 for 循环做出假设,
  并且必须维护一个单独的循环计数器.

- 一致性: 已有一种语法用于表示范围, 如下所示. 此语法使用完全相同的参数,
  尽管都是可选的, 但方式完全相同. 将此语法扩展到 ranges, 形成 "范围常量" 似乎是合乎逻辑的.


Slice Indices
=============

在 Python 中, 序列可以通过以下两种方式之一进行索引: 检索单个项或检索一系列项.
检索一系列项会产生与原始序列相同类型的新对象, 其中包含原始序列中的零个或多个项.
这是使用 "范围表示法" 完成的::

   >>> l[2:4]
   ['c', 'd']

此范围表示法由零, 一或两个由冒号分隔的索引组成.
第一个索引是 *start* 索引, 第二个是 *end*.
如果省略其中任何一个, 它们分别默认为序列的开始和结束.

还有一个扩展范围表示法, 它也包含 *step*.
虽然目前大多数内置类型都不支持这种表示法, 但如果是这样, 它将如下工作::

   >>> l[1:4:2]
   ['b', 'd']

切片语法的第三个 "参数" 与 ``range()`` 的 *step* 参数完全相同.
标准的基础机制和这些扩展的切片是完全不同的, 并且不一致,
因为数学包之外的许多类和扩展不实现对扩展变体的支持.
虽然这应该得到解决, 但它超出了本 PEP 的范围.

但是, 扩展切片确实显示已经有一个完全有效且适用的语法来表示范围,
来解决使用 ``range()`` 函数的所有先前声明的缺点:

- 它更清晰, 更简洁的语法, 已经证明既直观又易学.

- 它与 Python 中范围的其他用法一致
  (例如, 切片).

- 因为它是内置语法, 而不是内置函数, 所以它不能被覆盖.
  这意味着检查者可以确定代码的作用, 并且优化器不必担心 ``range()`` 被 "遮蔽".


The Proposed Solution
=====================

建议的范围常量实现将列表常量的语法与 (扩展) 切片的语法相结合, 来形成范围常量::

   >>> [1:10]
   [1, 2, 3, 4, 5, 6, 7, 8, 9]
   >>> [:5]
   [0, 1, 2, 3, 4]
   >>> [5:1:-1]
   [5, 4, 3, 2]

范围常量和切片语法之间存在一个细微差别:
尽管可以在切片中省略所有 *start*, *end* 和 *step*,
但在范围常量中省略 *end* 是没有意义的.
在切片中, *end* 将默认为列表的末尾, 但这在范围常量中没有意义.


Reference Implementation
========================

建议的实现可以在 SourceForge [1]_ 上找到.
它添加了一个新的字节码, ``BUILD_RANGE``, 从堆栈中获取三个参数,
并在这些参数的基础上构建一个列表. 列表被推送回堆栈.

使用新的字节码对于能够基于其他计算来构建范围是必要的, 其结果在编译时是未知的.

该代码向 ``listobject.c`` 引入了两个新函数,
这些函数在当前私有函数和完整的 API 调用之间徘徊.

``PyList_FromRange()`` 从开始, 结束和步骤构建一个列表,
如果发生错误, 则返回 NULL. 它的原型是::

    PyObject * PyList_FromRange(long start, long end, long step)

``PyList_GetLenOfRange()`` 是一个辅助函数, 用于确定范围的长度.
以前, 它是 ``bltinmodule.c`` 中的静态函数, 但现在在 ``listobject.c``
和 ``bltinmodule.c`` (对于 ``xrange``) 都是必需的.
它是非静态的, 仅用于避免代码重复. 它的原型是::

    long PyList_GetLenOfRange(long start, long end, long step)


Open issues
===========

- 在范围常量中要求 *end* 参数的差异的一种可能的解决方案是允许范围语法创建 "生成器" 而不是列表,
  例如 ``xrange`` 内置函数. 但是, 生成器不是一个列表,
  例如, 它不可能分配给生成器中的项, 或者附加到它上面.

  可以想象, 范围语法可以扩展为包括元组 (即不可变列表), 然后可以安全地将其作为生成器实现.
  这可能是一个理想的解决方案, 特别是对于大量阵列: 生成器在存储和初始化方面需要的很少,
  并且在计算和创建适当的数量请求时只有很小的性能影响.
  (TBD: 有没有? 粗略的测试表明, 即使在长度为 1 的范围内, 性能也相同)

  然而, 即使采用了这个想法, "特殊情况" 第二个参数是明智的,
  在一个语法实例中使它成为可选, 在其他情况下是非可选的吗?

- 是否可以将范围语法与普通列表常量混合, 创建单个列表? E.g.::

     >>> [5, 6, 1:6, 7, 9]

  创建 ::

     [5, 6, 1, 2, 3, 4, 5, 7, 9]

- 范围常量应该如何与另一个提议的新特征相互作用, "列表理解" [2]_?
  具体来说, 是否可以在列表推导中创建列表? E.g.::

     >>> [x:y for x in (1, 2) y in (3, 4)]

  此示例是否应该返回具有多个范围的单个列表::

     [1, 2, 1, 2, 3, 2, 2, 3]

  或列表嵌套列表, 像这样::

     [[1, 2], [1, 2, 3], [2]_, [2, 3]]

  然而, 由于列表推导的语法和语义仍然存在热议,
  这些问题可能最好通过 "列表推导" PEP 来解决.

- 范围常量接受除整数之外的对象: 它对传入的对象执行 ``PyInt_AsLong()``,
  因此只要对象可以被强制转换为整数, 它们就会被接受. 但是, 结果列表始终由标准整数组成.

  范围常量应该创建传入类型的列表吗? 在其他内置类型的情况下可能是理想的, 例如长字符和字符串::

     >>> [ 1L : 2L<<64 : 2<<32L ]
     >>> ["a":"z":"b"]
     >>> ["a":"z":2]

  然而, 这可能太过 "magic". 它也可能会给用户定义的类带来问题:
  即使可以找到基类并创建新实例, 实例也可能需要 ``__init__`` 的其他参数,
  导致创建失败.

- 需要对 ``PyList_FromRange()`` 和 ``PyList_GetLenOfRange()`` 函数进行分类:
  它们是 API 的一部分, 还是应该成为私有函数?


Rejection
=========

经过认真考虑和一段时间的冥想, 这个提议被拒绝了.
公开的问题, 以及范围和切片语法之间的一些混淆, 引发了足够的问题,
Guido 不接受它用于 Python 2.0, 后来完全拒绝该提议.
新语法及其意图被认为不够明显.

[ TBD: Guido, 请修改/确认. 两者最好;
这是一个 PEP, 它应该包含 *所有* 拒绝或重新考虑的原因, 以供将来参考.]


Copyright
=========

This document has been placed in the Public Domain.


References
==========

.. [1] http://sourceforge.net/patch/?func=detailpatch&patch_id=100902&group_id=5470
.. [2] PEP 202, List Comprehensions



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  