
PEP: 201
Title: Lockstep Iteration
Version: $Revision$
Last-Modified: $Date$
Author: barry@python.org (Barry Warsaw)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 13-Jul-2000
Python-Version: 2.0
Post-History: 27-Jul-2000



Introduction
============

该 PEP 描述了 "锁步迭代" 提议. 此 PEP 跟踪此功能的状态和所有权,
计划在 Python 2.0 中引入. 它包含功能的说明, 并概述了支持该功能所需的更改.
本 PEP 总结了在邮件列表论坛中进行的讨论, 并在适当情况下提供了 URL 以获取更多信息.
此文件的 CVS 修订历史记录包含确定的历史记录.


Motivation
==========

Python 中的标准 for 循环迭代序列中的每个元素, 直到序列耗尽 [1]_.
但是, for 循环仅迭代单个序列, 并且通常希望以锁步方式循环遍历多个序列.
换句话说, 以这样的方式, 通过循环的第i次迭代返回包含来自每个序列的第i个元素的对象.

用于实现此目的的常用语法是不直观的. 这个 PEP 通过引入一个名为 ``zip`` 的新内置函数,
提出了一种执行这种迭代的标准方法.

虽然 zip() 的主要动机来自锁步迭代, 但通过将 zip() 作为内置函数实现,
它在 for-loops 之外的上下文中具有额外的实用性.

Lockstep For-Loops
==================

锁步 for 循环是对两个或更多个序列的非嵌套迭代, 使得在每次通过循环时,
每个序列中的一个元素被用于组成目标.
这种行为已经可以通过使用 map() 内置函数在 Python 中完成::

    >>> a = (1, 2, 3)
    >>> b = (4, 5, 6)
    >>> for i in map(None, a, b): print i
    ...
    (1, 4)
    (2, 5)
    (3, 6)
    >>> map(None, a, b)
    [(1, 4), (2, 5), (3, 6)]

for 循环正常地迭代这个列表.

虽然 map() 习惯用法在 Python 中是常见的, 但它有几个缺点:

* 对于没有函数式编程背景的程序员来说, 这是不明显的.

* 使用神奇的 ``None`` 第一个参数是不明显的.

* 当列表长度不同时, 它具有任意的, 通常是无意的和不灵活的语义:
  较短的序列用 ``None`` 填充::

    >>> c = (4, 5, 6, 7)
    >>> map(None, a, c)
    [(1, 4), (2, 5), (3, 6), (None, 7)]

出于这些原因, 在 Python 2.0 beta 时间框架中提出了几个建议,
用于对锁步 for 循环的语法支持. 这是两个建议::

  for x in seq1, y in seq2:
    # stuff

::

  for x, y in seq1, seq2:
    # stuff

这些形式都不会起作用, 因为它们已经在 Python 中具有某些意义并且改变含义会破坏现有代码.
对新语法的所有其他建议都遇到了同样的问题,
或者与其他另一个被称为 "列表理解" 的提议功能相冲突 (see PEP 202).

The Proposed Solution
=====================

建议的解决方案是引入一个新的内置序列生成器函数, 可在 ``__builtin__`` 模块中找到.
这个函数叫做 ``zip`` 并具有以下签名::

    zip(seqa, [seqb, [...]])

``zip()`` 接受一个或多个序列并将它们的元素编织在一起,
就像 ``map(None, ...)`` 用相同长度的序列一样.
当最短的序列耗尽时, 编织停止.


Return Value
============

``zip()`` 返回一个真正的 Python 列表, 就像 ``map()`` 那样.


Examples
========

以下是一些基于以下参考实现的示例::

    >>> a = (1, 2, 3, 4)
    >>> b = (5, 6, 7, 8)
    >>> c = (9, 10, 11)
    >>> d = (12, 13)

    >>> zip(a, b)
    [(1, 5), (2, 6), (3, 7), (4, 8)]

    >>> zip(a, d)
    [(1, 12), (2, 13)]

    >>> zip(a, b, c, d)
    [(1, 5, 9, 12), (2, 6, 10, 13)]

请注意, 当序列长度相同时, ``zip()`` 是可逆的::

    >>> a = (1, 2, 3)
    >>> b = (4, 5, 6)
    >>> x = zip(a, b)
    >>> y = zip(*x) # alternatively, apply(zip, x)
    >>> z = zip(*y) # alternatively, apply(zip, y)
    >>> x
    [(1, 4), (2, 5), (3, 6)]
    >>> y
    [(1, 2, 3), (4, 5, 6)]
    >>> z
    [(1, 4), (2, 5), (3, 6)]
    >>> x == z
    1

当序列长度不同时, 不可能以这种方式颠倒 zip.


Reference Implementation
========================

这是一个参考实现, 在 Python 的 zip() 内置函数中.
在最终批准后, 这将被 C 实施取代::

    def zip(*args):
        if not args:
            raise TypeError('zip() expects one or more sequence arguments')
        ret = []
        i = 0
        try:
            while 1:
                item = []
                for s in args:
                    item.append(s[i])
                ret.append(tuple(item))
                i = i + 1
        except IndexError:
            return ret



BDFL Pronouncements
===================

注意: BDFL 指的是 Guido van Rossum, Python 的仁慈独裁者.

* 功能的名称. 这个 PEP 的早期版本包括一个公开的问题,
  列出了20多个提议的 ``zip()`` 的替代名称.
  面对没有绝对更好的选择, BDFL 强烈倾向于 ``zip()``,
  因为它的 Haskell [2]_ 遗产. 有关备选方案列表, 请参见本 PEP 的 1.7版.

* ``zip()`` 应该是一个内置函数.

* 可选填充. 该 PEP 的早期版本提出了一个可选的 ``pad`` 关键字参数,
  当参数序列的长度不同时, 将使用该参数. 这与 ``map(None, ...)`` 语义类似,
  只是用户可以指定 pad 对象. 由于 KISS 原则, BDFL 拒绝了这一点,
  因为它总是截断到最短的序列. 如果确实需要, 以后添加会更容易.
  如果不需要, 将来仍然无法删除它.

* 懒惰的评价. 这个 PEP 的早期版本提出 ``zip()``
  返回一个使用 ``__getitem __()`` 协议执行惰性求值的内置对象.
  这已被 BDFL 强烈拒绝, 赞成返回真正的 Python 列表.
  如果将来需要延迟评估, BDFL 建议添加一个 ``xzip()`` 函数.

* ``zip()`` 没有参数. BDFL 强烈建议这引发 TypeError 异常.

* ``zip()`` 有一个参数. BDFL 强烈希望返回一个1元组的列表.

* 内外容器控制. 这个 PEP 的早期版本包含了对某些人想要的功能的相当冗长的讨论,
  即能够控制内部和外部容器类型 (在这个版本的 PEP 中它们分别是元组和列表).
  鉴于简化的 API 和实现, 这种详细说明被拒绝. 有关更详细的分析, 请参阅本 PEP 的 1.7版.

Subsequent Change to ``zip()``
==============================

在 Python 2.4 中, 修改了没有参数的 zip() 来返回空列表而不是引发 TypeError 异常.
原始行为的基本原理是缺乏参数被认为表明编程错误. 但是, 这种想法没有预料到使用带有 ``*``
运算符的 zip() 来解压缩变量长度参数列表. 例如, zip 的反转可以定义为: ``unzip = lambda s: zip(*s)``.
该转换还为定义为元组列表的表定义矩阵转置或等效行/列交换.
后一种转换通常在读取数据文件时使用, 其中记录为行, 字段为列. 例如, 代码::

    date, rain, high, low = zip(*csv.reader(file("weather.csv")))

重新排列列数据, 以便将每个字段收集到单个元组中, 以便进行简单的循环和汇总::

    print "Total rainfall", sum(rain)

如果 ``zip(*[])`` 被处理为允许的情况而不是异常, 则使用 ``zip(*args)`` 更容易编码.
当数据从没有记录的情况构建或递归到空案例时, 这尤其有用.

看到这种可能性, BDFL 同意 (有一些疑虑) 让 Py2.4 的特性发生了变化.

Other Changes
=============

* 上面讨论的 ``xzip()`` 函数在 ``itertools`` 模块中的 Py2.3 中实现为 ``itertools.izip()``.
  此函数提供惰性行为, 消耗单个元素并在每次传递时生成单个元组.
  "just-in-time" 风格比基于列表的对应物 "zip()" 更节省内存和运行速度.

* ``itertools`` 模块还添加了 ``itertools.repeat()`` 和 ``itertools.chain()``.
  这些工具可以一起使用来填充 ``None`` 的序列 (来匹配 ``map(None, seqn)`` 的特性)::

      zip(firstseq, chain(secondseq, repeat(None)))


References
==========

.. [1] http://docs.python.org/reference/compound_stmts.html#for

.. [2] http://www.haskell.org/onlinereport/standard-prelude.html#$vzip


Greg Wilson 的一些 CS 研究生的语法问卷调查问卷
http://www.python.org/pipermail/python-dev/2000-July/013139.html


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
