PEP: 468
Title: Preserving the order of \*\*kwargs in a function.
Version: $Revision$
Last-Modified: $Date$
Author: Eric Snow <ericsnowcurrently@gmail.com>
Discussions-To: python-ideas@python.org
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 5-Apr-2014
Python-Version: 3.6
Post-History: 5-Apr-2014,8-Sep-2016
Resolution: https://mail.python.org/pipermail/python-dev/2016-September/146329.html


Abstract
========

函数定义中的 **kwargs 语法指示解释器应收集与其他命名参数不对应的所有关键字参数.
但是, Python 不保留将这些收集的关键字参数传递给函数的顺序.
在某些情况下, 顺序很重要。 此 PEP 规定收集的关键字参数作为有序映射在函数体中公开.


Motivation
==========

函数定义中 Python 的 **kwargs 语法提供了动态处理关键字参数的强大方法.
在语法的某些应用程序中 (请参阅 _`Use Cases`), 应用于收集的关键字参数的语义要求保留顺序.
不出所料, 这与 OrderedDict 与 dict 的关系类似.

目前要保留顺序, 您必须手动并与实际函数调用分开.
这涉及构建有序映射, 无论是 OrderedDict 还是2元组的迭代, 然后将其作为单个参数传递给函数.
[#arg_unpacking] _

凭借本 PEP 中描述的功能, 将不再需要该样板.

For comparison, currently::

   kwargs = OrderedDict()
   kwargs['eggs'] = ...
   ...
   def spam(a, kwargs):
       ...

并提出这个建议::

   def spam(a, **kwargs):
       ...

Nick Coglan 谈到了一些用例, 总结得很好
[#nick_obvious]_::

   今天这些 *can* 都可以完成, 但 *not* 通过使用关键字参数.
   在我看来, 想要解决的问题是关键字参数 *look* 像它们应该适用于这些情况,
   因为它们在源代码中有明确的顺序. 他们不工作的唯一原因是因为解释器抛出了顺序信息.

   这是一个教科书案例, 语言特征在某些情况下成为一个有吸引力的麻烦:
   对于上述用例的简单而明显的解决方案 *实际上并不起作用* 原因
   如果你没有牢固把握就不明显 Python 公认的复杂的论证处理.

多年来这个提案的出现以及人们被 OrderedDict 构造函数混淆的无数次支持了这一观察结果.
[#past_threads]_ [#loss_of_order]_ [#compact_dict]_


Use Cases
=========

正如尼克指出的那样, 在人们期望秩序重要的情况下, **kwargs 的当前行为是不直观的.
除了下面概述的更具体的情况之外, 一般而言 "您想要控制迭代顺序的任何其他内容 *和*
在单个调用中设置字段名称和值将有可能受益" [#nick_general]_
对于有序类型的工厂 (例如, __init__()) 很重要.

Serialization
-------------

显然, OrderedDict 会从有序的 kwargs 中受益 (__init__() 和 update()).
但是, 好处还扩展到序列化 API [#nick_obvious]_::

   在序列化的上下文中, 我们学到的一个关键教训是, 当您想要最小化伪差异时,
   任意排序是一个问题, 并且排序不是一个简单的解决方案.

   像 doctest 这样的工具根本不能容忍虚假的差异, 但通常都适合基于排序的答案.

   非常希望能够使用关键字参数来控制键值对集合的显示顺序的情况类似于:

   * 打印输出密钥: 输出 CLI 中的值对
   * 将语义名称映射到 CSV 中的列顺序
   * 在 XML 中序列化特定订单中的属性和元素
   * 序列化地图键, 特别是人类可读格式的顺序, 例如 JSON 和 YAML (特别是当它们将被置于源代码控制之下时)

Debugging
---------

用 Raymond Hettinger 的话来说 [#raymond_debug]_::

   如果参数按创建顺序显示, 则可以更轻松地进行调试.
   AFAICT, 扰乱它们没有任何目的.

Other Use Cases
---------------

* 模拟对象. [#mock]_
* 控制对象呈现.
* 替代 namedtuple(), 可以指定默认值.
* 按顺序指定参数优先级.


Concerns
========

Performance
-----------

如前所述, 有序关键字参数的概念已经出现过多次. 每次遇到相同的响应时,
即保留关键字 arg 顺序会对函数调用性能产生足够不利的影响, 这是不值得做的.
但是, Guido 注意到以下 [#guido_open]_ ::

  订购 **kwds 仍然是开放的, 但需要仔细设计和实施, 以避免减慢无益的函数调用.

如下所述, 有一些方法可以解决这个问题, 但会增加复杂性. 最终, 最简单的方法是最有意义的方法:
将收集的关键字参数打包到 OrderedDict 中. 但是, 如果没有 OrderedDict 的 C 实现, 则没有太多可讨论的内容.
这在 Python 3.5 中发生了变化.
[#c_ordereddict]_

注意: 在 Python 3.6 中, dict 是保持顺序的. 这实际上消除了性能问题.

Other Python Implementations
----------------------------

另一个需要考虑的重要问题是新功能必须认识到多个 Python 实现. 在某些时候, 他们每个人都应该实施有序的 kwargs.
在这方面, 这个想法似乎没有问题. [#ironpython]_ 对主要 Python 实现的非正式调查表明, 此功能不会是一个重大负担.


Specification
=============

从版本 3.6 开始, Python 将保留传递给函数的关键字参数的顺序. 为了实现这一点, 收集的 kwargs 现在将是有序映射.
请注意, 这并不一定意味着 OrderedDict. 现在 CPython 3.6 中的 dict 被命令, 类似于 PyPy.

他的意思仅适用于定义使用 \*\*kwargs 语法收集其他未指定的关键字参数的函数, 仅保留这些关键字参数的顺序.

Relationship to \*\*-unpacking syntax
-------------------------------------

函数调用中的 ** 解包语法与此提议没有特殊关联. 解包提供的关键字参数将以与现在完全相同的方式处理:
匹配已定义参数的关键字将聚集在那里, 其余的将被收集到有序的 kwargs 中 (就像任何其他不匹配的关键字参数一样).

请注意, 解压缩具有未定义顺序的映射 (例如, dict) 将像正常一样保留其迭代顺序.
只是顺序仍未定义. 然后将打包解压缩的键值对的有序映射将不能提供任何备用排序. 这应该不足为奇.

简单地讨论过简单地将这些映射传递给函数 kwargs 而不解包和重新打包它们, 但这不在本提案的范围之内,
并且可能是个坏主意. (这些讨论很简短.)

Relationship to inspect.Signature
---------------------------------

签名对象不需要更改. inspect.BoundArguments (由 Signature.bind() 和 Signature.bind_partial() 返回) 的
`kwargs` 参数将从 dict 变为 OrderedDict.

C-API
-----

No changes.

Syntax
------

No syntax is added or changed by this proposal.

Backward-Compatibility
----------------------

The following will change:

* iteration order of kwargs will now be consistent (except of course in
  the case described above)


Reference Implementation
========================

For CPython there's nothing to do.


Alternate Approaches
====================

Opt-out Decorator
-----------------

这与当前提案相同, 但 Python 也会在 functools 中提供装饰器,
这会导致收集的关键字参数被打包到普通的 dict 而不是 OrderedDict.

预测:

只有在某些不常见的情况下确定性能显着不同或者存在其他无法解决的向后兼容性问题时, 才需要这样做.

Opt-in Decorator
----------------

现状将保持不变. 相反, Python 会在 functools 中提供一个装饰器,
它会将装饰函数注册或标记为应该获得有序关键字参数的函数. 在调用时检查函数的性能开销是微不足道的.

预测:

唯一真正的缺点是函数包装器工厂 (例如, functools.partial 和许多装饰器),
它们旨在通过在包装器定义中使用 kwargs 以及在包装函数的调用中解包 kwargs 来完美地保留关键字参数.
每个包装器都必须单独更新, 尽管有 functools.wraps() 这样做会自动提供帮助.

__kworder__
-----------

关键字参数的顺序将在调用时单独存储在列表中. 该列表将绑定到函数本地的 __kworder__.

预测:

这同样使封装复杂化.

Compact dict with faster iteration
----------------------------------

Raymond Hettinger 介绍了 dict 实现的想法, 这将导致在 dicts 上保留插入顺序 (直到第一次删除).
这对于 kwargs 来说非常合适. [#compact_dict]_

预测:

这个想法在可行性和时间框架上仍然不确定.

请注意, Python 3.6 现在具有此 dict 实现.

\*\*\*kwargs
------------

这将为函数的签名添加一个新表单, 作为与 \*\*kwargs 互斥的并行. 新语法 \*\*\*kwargs (注意有三个星号),
表示 kwargs 应该保留关键字参数的顺序.

预测:

新语法仅在最 *dire* 情况下添加到 Python. 使用其他可用解决方案, 新语法是不合理的.
此外, 与所有选择加入解决方案一样, 新语法会使传递案例复杂化.

annotations
-----------

这是装饰器方法的变体. 您可以在 \*\*kwargs 上使用函数注释, 而不是使用装饰器来标记函数.

预测:

除了传递复杂性之外, Python 核心开发中还积极地禁止使用注释.
使用注释来选择保留命令可能会干扰其他应用程序级别的注释使用.

dict.__order__
--------------

dict 对象将有一个新属性, `__ order__` 默认为 None, 而在 kwargs 情况下,
解释器将以与上述 __kworder__ 相同的方式使用.

预测:

这意味着对 kwargs 性能没有任何影响, 但这种变化会非常具有侵入性 (Python 使用 dict 很多).
此外, 对于包装器案例, 解释器必须小心保存 `__order__`.

KWArgsDict.__order__
--------------------

这与 `dict.__order__` 的想法相同, 但是 kwargs 将是一个提供 `__order__` 属性的新的最小 dict 子类的实例.
相反, dict 将保持不变.

预测:

简单地切换到 OrderedDict 是一种不太复杂和更直观的变化.


Acknowledgements
================

感谢 Andrew Barnert 提供的有用反馈以及所有过去电子邮件主题的参与者.


Footnotes
=========

.. [#arg_unpacking]

   或者, 您也可以在函数定义中用 * 替换 **, 然后传入 key/value 2元组.
   这具有不要求密钥是有效标识符串的优点. 请看
   https://mail.python.org/pipermail/python-ideas/2014-April/027491.html.


References
==========

.. [#nick_obvious]
   https://mail.python.org/pipermail/python-ideas/2014-April/027512.html

.. [#past_threads]
   https://mail.python.org/pipermail/python-ideas/2009-April/004163.html

   https://mail.python.org/pipermail/python-ideas/2010-October/008445.html

   https://mail.python.org/pipermail/python-ideas/2011-January/009037.html

   https://mail.python.org/pipermail/python-ideas/2013-February/019690.html

   https://mail.python.org/pipermail/python-ideas/2013-May/020727.html

   https://mail.python.org/pipermail/python-ideas/2014-March/027225.html

   http://bugs.python.org/issue16276

   http://bugs.python.org/issue16553

   http://bugs.python.org/issue19026

   http://bugs.python.org/issue5397#msg82972

.. [#loss_of_order]
   https://mail.python.org/pipermail/python-dev/2007-February/071310.html

.. [#compact_dict]
   https://mail.python.org/pipermail/python-dev/2012-December/123028.html

     https://mail.python.org/pipermail/python-dev/2012-December/123105.html

   https://mail.python.org/pipermail/python-dev/2013-May/126327.html

     https://mail.python.org/pipermail/python-dev/2013-May/126328.html

.. [#nick_general]
   https://mail.python.org/pipermail/python-dev/2012-December/123105.html

.. [#raymond_debug]
   https://mail.python.org/pipermail/python-dev/2013-May/126327.html

.. [#mock]
   https://mail.python.org/pipermail/python-ideas/2009-April/004163.html

     https://mail.python.org/pipermail/python-ideas/2009-April/004165.html

     https://mail.python.org/pipermail/python-ideas/2009-April/004175.html

.. [#guido_open]
   https://mail.python.org/pipermail/python-dev/2013-May/126404.html

.. [#c_ordereddict]
   http://bugs.python.org/issue16991

.. [#ironpython]
   https://mail.python.org/pipermail/python-dev/2012-December/123100.html


Copyright
=========

本文档已置于公共域名.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   coding: utf-8  
   End:  

