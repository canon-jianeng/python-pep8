
PEP: 232
Title: Function Attributes
Version: $Revision$
Last-Modified: $Date$
Author: Barry Warsaw <barry@python.org>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 02-Dec-2000
Python-Version: 2.1
Post-History: 20-Feb-2001


Introduction
============

本 PEP 描述了 Python 的扩展, 为函数和方法添加了属性字典.
此 PEP 跟踪此功能的状态和所有权. 它包含功能的说明,
并概述了支持该功能所需的更改. 本 PEP 总结了在邮件列表论坛中进行的讨论,
并在适当情况下提供了 UR 来获取更多信息. 此文件的 CVS 修订历史记录包含确定的历史记录.


Background
==========

函数已经具有许多属性, 其中一些属性是可写的, 例如, ``func_doc``, a.k.a.
``func .__doc__``. ``func_doc`` 有一个有趣的属性, 即在函数 (和方法) 定义中有特殊的语法来隐式设置属性.
这种便利已被反复利用, 使用额外的语义重载文档字符串.

例如, John Aycock 编写了一个系统, 其中 docstrings 用于定义解析规则. [1]_
Zope 的 ZPublisher ORB [2]_ 使用 docstrings 来发出可发布方法的信号,
即可以通过网络调用的方法.

这种方法的问题是重载的语义可能彼此冲突. 例如, 如果我们想要将一个 doctest 单元测试添加到 Zope 方法中,
该方法不应该通过 Web 发布.


Proposal
========

该提议为函数对象添加了一个新的字典, 称为 ``func_dict`` (a.k.a. ``__dict__``).
可以使用普通属性集和 get 语法设置和获取此字典.

方法也获得 getter 语法, 它们当前通过底层函数对象的字典访问该属性.
除非在底层函数对象上显式执行, 否则无法在绑定或未绑定方法上设置属性.
有关后续 Python 版本中的方法, 请参阅下面的 "未来方向" 讨论.

函数对象的 ``__dict__`` 也可以设置, 但只能设置为字典对象. 删除函数的 ``__dict__``,
或将其设置为除具体字典对象之外的任何内容都会导致 ``TypeError``.
如果没有设置任何函数属性, 函数的 ``__dict__`` 将为空.


Examples
========

以下是您可以使用此功能执行的一些示例.

::

    def a():
        pass

    a.publish = 1
    a.unittest = '''...'''

    if a.publish:
        print a()

    if hasattr(a, 'unittest'):
        testframework.execute(a.unittest)

    class C:
        def a(self):
            'just a docstring'
            a.publish = 1

    c = C()
    if c.a.publish:
        publish(c.a())


Other Uses
----------

Paul Prescod 在 `python-dev thread`_ 上列举了许多其他用法.

.. _python-dev thread: https://mail.python.org/pipermail/python-dev/2000-April/003364.html


Future Directions
=================

以下是一些未来的方向. 任何采用这些想法都需要一个新的 PEP,
它引用了这个, 并且必须针对 2.1 版本之后的 Python 版本.

- 此 PEP 的先前版本允许在未绑定方法上使用 setter 和 getter 属性,
  并且仅在绑定方法上使用 getter. 该策略发现了许多问题.

  因为方法属性存储在底层函数中, 所以这会导致几个可能令人惊讶的结果 ::

      class C:
          def a(self): pass

      c1 = C()
      c2 = C()
      c1.a.publish = 1
      # c2.a.publish would now be == 1 also!

  因为对 ``a`` 绑定 ``c1`` 的更改也导致 ``a`` 的变化绑定到 ``c2``,
  所以不允许在绑定方法上设置属性. 但是, 即使允许在未绑定方法上设置属性也有其含糊之处 ::

      class D(C): pass
      class E(C): pass

      D.a.publish = 1
      # E.a.publish would now be == 1 also!

  因此, 当前 PEP 不允许在绑定或未绑定方法上设置属性,
  但允许获取任一属性 - 都返回基础函数对象的属性值.

  未来的 PEP 可能会建议通过使用特殊命名约定在实例或类上设置属性来实现设置
  (绑定或未绑定) 方法属性. 即 ::

      class C:
          def a(self): pass

      C.a.publish = 1
      C.__a_publish__ == 1 # true

      c = C()
      c.a.publish = 2
      c.__a_publish__ == 2 # true

      d = C()
      d.__a_publish__ == 1 # true

  在这里, 对实例的查找首先查看实例的字典, 然后查找类的字典, 最后查找函数对象的字典.

- 目前, Python 仅支持 Python 函数的函数属性 (即那些用 Python 编写的函数, 而不是内置的函数).
  如果值得, 可以制作一个单独的补丁, 它将向内置函数添加函数属性.

- ``__doc__`` 是目前唯一具有语法支持利于设置的函数属性. 
  最终可能有必要增强支持简单功能属性设置的语言.
  以下是 PEP 审稿人建议的一些语法 ::

      def a {
          'publish' : 1,
          'unittest': '''...''',
          }
          (args):
          # ...

      def a(args):
          """The usual docstring."""
          {'publish' : 1,
           'unittest': '''...''',
           # etc.
           }

      def a(args) having (publish = 1):
          # see reference [3]
          pass

  BDFL 目前正在反对设置任意函数属性的任何此类特殊语法支持.
  任何语法提案都必须在新的 PEP 中概述.


Dissenting Opinion
==================

当在2000年4月的 python-dev 邮件列表中讨论这个问题时, 提出了一些不同意见.
为了完整起见, 讨论线程从 `python-dev`_ 开始.

.. _python-dev: https://mail.python.org/pipermail/python-dev/2000-April/003361.html

不同意见似乎属于以下类别:

- 没有明确的目的 (它会给你带来什么?)
- 其他方法 (例如, 映射为类属性)
- 没有用, 直到包含语法支持

反驳其中一些论点是观察到, 使用 vanilla Python 2.0, ``__doc__`` 实际上可以设置为任何类型的对象,
因此一些可写函数属性的外观已经可行. 但这种方法又是 ``__doc__`` 的另一种腐败.

虽然当然可以将类映射添加到类对象 (或者在函数属性的情况下, 添加到函数的模块中),
但是如何提取属性值来进行检查更加困难和不太明显.

最后, 可能需要添加语法支持, 就像存在 ``__doc__`` 语法支持一样.
这可以与实际设置和获取函数属性的能力分开考虑.


Reference Implementation
========================

此 PEP 已被接受, 并且已将实现集成到 Python 2.1 中.


References
==========

.. [1] Aycock, "Compiling Little Languages in Python",
   http://www.foretec.com/python/workshops/1998-11/proceedings/papers/aycock-little/aycock-little.html

.. [2] http://classic.zope.org:8080/Documentation/Reference/ORB

.. [3] Hudson, Michael, SourceForge patch implementing this syntax,
   http://sourceforge.net/tracker/index.php?func=detail&aid=403441&group_id=5470&atid=305470


Copyright
=========

This document has been placed in the public domain.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   coding: utf-8  
   End:  
