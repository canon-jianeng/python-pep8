
PEP: 245
Title: Python Interface Syntax
Version: $Revision$
Last-Modified: $Date$
Author: Michel Pelletier <michel@users.sourceforge.net>
Discussions-To: http://www.zope.org/Wikis/Interfaces
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 11-Jan-2001
Python-Version: 2.2
Post-History: 21-Mar-2001


Rejection Notice
================

我拒绝接受这个 PEP. 现在已经五年了. 虽然在某些时候我希望 Python 有接口,
但期望它类似于这个 PEP 中的语法是天真的. 此外, PEP 246 被拒绝支持完全不同的东西;
接口不会在适应中发挥作用, 也不会取代它. GVR.


Introduction
============

此 PEP 描述了用于在 Python 中创建接口对象的建议语法.


Overview
========

除了考虑向 Python 添加静态类型系统之外, Types-SIG 还负责设计 Python 的接口系统.
1998年12月, Jim Fulton 根据 SIG 的讨论发布了原型接口系统.
有关此讨论和原型的许多问题和背景信息都可以在 SIG 档案中找到 [1]_.

大约在2000年底, Digital Creations 开始考虑更好的 Zope 组件模型设计 [2]_.
Zope 未来的组件模型在很大程度上依赖于接口对象. 这导致了 Jim 的 "Scarecrow" 接口原型的进一步发展.
从版本2.3开始, Zope 附带一个接口软件包作为标准软件. Zope 的接口包用作此 PEP 的参考实现.

该 PEP 提出的语法依赖于 PEP 232 [3]_ 中描述的语法增强, 并描述了 PEP 233 [4]_ 可以基于的基础框架.
关于接口对象和代理对象, 有一些工作要做, 所以对于这个 PEP 的可选部分, 您可能希望看到 [5]_.


The Problem
===========

接口很重要, 因为它们解决了开发软件时出现的许多问题:

- Python 中有许多隐含的接口, 通常称为 "协议". 目前确定这些协议是基于实现自我检查, 但通常也会失败.
  例如, 定义 ``__getitem__`` 意味着序列和映射 (前者具有顺序整数键). 开发人员无法明确表示对象打算实现哪些协议.

- 从开发人员的角度来看, Python 在类型和类之间的分离是有限的. 当期望类型时,
  用户使用 'type(foo) == type("")' 之类的代码来确定 'foo' 是否是字符串.
  当期望类的实例时, 用户使用 'isinstance(foo, MyString)' 来确定 'foo' 是否是 'MyString' 类的实例.
  没有用于确定对象是否可以以某种有效方式使用的统一模型.

- Python 的动态类型非常灵活且功能强大, 但它没有提供类型检查的静态类型语言的优势.
  静态类型语言为您提供了更多类型安全性, 但通常过于冗长,
  因为对象只能通过公共子类进行泛化并专门用于转换 (例如, 在 Java 中).

接口试图解决的还有许多文档问题.

- 开发人员浪费大量时间查看系统的源代码, 来确定对象的工作方式.

- 对您的系统不熟悉的开发人员可能会误解您的对象如何工作, 导致并可能传播使用错误.

- 由于缺少接口意味着从源码中推断使用, 开发人员最终可能会使用仅供 "内部使用" 的方法和属性.

- 对于试图正确理解大师编写的代码的新手程序员来说, 代码检查可能很难, 而且非常令人沮丧.

- 当许多人非常努力地理解默默无闻时 (比如未记录的软件), 浪费了大量的时间.
  努力花费前端文档界面最终将节省大部分时间.

接口试图通过为您指定对象的约定义务,
如何使用对象的文档以及用于发现约定和文档的内置机制来解决这些问题.

Python 具有非常有用的自我检查功能. 众所周知, 这使得在交互式解释器中探索概念变得更加容易,
因为 Python 使您能够查看有关对象的各种信息: 类型, doc字符串, 实例字典, 基类, 未绑定方法等等.

其中许多功能都面向自我检查, 使用和更改软件的实现, 其中一个 ("doc strings") 面向提供文档.
该提议描述了对该自然自我检查框架的扩展, 该框架描述了对象的接口.


Overview of the Interface Syntax
================================

在大多数情况下, 接口的语法非常类似于类的语法,
但未来的需求或讨论中提出的需求可能为接口语法定义新的可能性.

稍后在 PEP 中给出正式的 BNF 语法描述, 为了说明的目的,
这里是使用所提出的语法创建的两个不同接口的示例 ::

    interface CountFishInterface:
        "Fish counting interface"

        def oneFish():
            "Increments the fish count by one"

        def twoFish():
            "Increments the fish count by two"

        def getFishCount():
            "Returns the fish count"

    interface ColorFishInterface:
        "Fish coloring interface"

        def redFish():
            "Sets the current fish color to red"

        def blueFish():
            "Sets the current fish color to blue"

        def getFishColor():
            "This returns the current fish color"

这个代码在评估时会创建两个名为 ``CountFishInterface`` 和 ``ColorFishInterface`` 的接口.
这些接口由 ``interface`` 语句定义.

接口及其方法的散文文档来自 doc 字符串. 方法签名信息来自 ``def`` 语句的签名.
请注意 def 语句没有正文. 该接口不实现任何服务; 它只描述一个.
接口和接口方法上的文档字符串是必需的, 不能提供 "pass" 语句.
pass 语句的等效接口是一个空文档字符串.

您还可以创建 "扩展" 其他接口的接口. 在这里, 您可以看到一种新的接口类型,
它扩展了 CountFishInterface 和 ColorFishInterface ::

    interface FishMarketInterface(CountFishInterface, ColorFishInterface):
        "This is the documentation for the FishMarketInterface"

        def getFishMonger():
            "Returns the fish monger you can interact with"

        def hireNewFishMonger(name):
            "Hire a new fish monger"

        def buySomeFish(quantity=1):
            "Buy some fish at the market"

FishMarketInteface 扩展到 CountFishInterface 和 ColorfishInterface.


Interface Assertion
===================

下一步是通过创建一个声明它实现接口的具体 Python 类来将类和接口放在一起.
以下是可能执行此操作的 FishMarket 组件示例 ::

    class FishError(Error):
        pass

    class FishMarket implements FishMarketInterface:
        number = 0
        color = None
        monger_name = 'Crusty Barnacles'

        def __init__(self, number, color):
            self.number = number
            self.color = color

        def oneFish(self):
            self.number += 1

        def twoFish(self):
            self.number += 2

        def redFish(self):
            self.color = 'red'

        def blueFish(self):
            self.color = 'blue'

        def getFishCount(self):
            return self.number

        def getFishColor(self):
            return self.color

        def getFishMonger(self):
            return self.monger_name

        def hireNewFishMonger(self, name):
            self.monger_name = name

        def buySomeFish(self, quantity=1):
            if quantity > self.count:
                raise FishError("There's not enough fish")
            self.count -= quantity
            return quantity

这个新类 FishMarket 定义了一个实现 FishMarketInterface 的具体类.
``implements`` 语句后面的对象称为 "接口断言".
接口断言可以是接口对象, 也可以是接口断言的元组.

像这样的 ``class`` 语句中提供的接口断言存储在类的 ``__implements__`` 类属性中.
在解释上面的例子之后, 你会得到一个类声明, 可以使用 'implements' 内置函数进行检查::

    >>> FishMarket
    <class FishMarket at 8140f50>
    >>> FishMarket.__implements__
    (<Interface FishMarketInterface at 81006f0>,)
    >>> f = FishMarket(6, 'red')
    >>> implements(f, FishMarketInterface)
    1
    >>>

一个类可以实现多个接口. 例如, 假设您有一个名为 "ItemInterface" 的接口,
它描述了一个对象如何作为容器对象中的项. 如果你想断言 FishMarket 实例
实现了 ItemInterface 接口以及 FishMarketInterface, 你可以提供一个接口断言,
其中包含 FishMarket 类的接口对象元组 ::

    class FishMarket implements FishMarketInterface, ItemInterface:
        # ...

如果要断言一个类实现接口, 并且另一个类实现的所有接口,
也可以使用接口断言 ::

    class MyFishMarket implements FishMarketInterface, ItemInterface:
        # ...

    class YourFishMarket implements FooInterface, MyFishMarket.__implements__:
        # ...

这个新类 YourFishMarket 断言它实现了 FooInterface, 以及 MyFishMarket 类实现的接口.

有关接口断言的更多细节值得深入研究. 接口断言是接口对象或接口断言的元组. 例如 ::

    FooInterface

    FooInterface, (BarInteface, BobInterface)

    FooInterface, (BarInterface, (BobInterface, MyClass.__implements__))

都是有效的接口断言. 当两个接口定义相同的属性时, 断言中首选信息的顺序是从上到下, 从左到右.

还有其他接口提议, 为了简单起见, 它们结合了类和接口的概念, 来提供简单的接口实现.
接口对象有一个 ``deferred`` 方法, 它返回一个实现此行为的延迟类 ::

    >>> FM = FishMarketInterface.deferred()
    >>> class MyFM(FM): pass

    >>> f = MyFM()
    >>> f.getFishMonger()
    Traceback (innermost last):
      File "<stdin>", line 1, in ?
    Interface.Exceptions.BrokenImplementation:
    An object has failed to implement interface FishMarketInterface

            The getFishMonger attribute was not provided.
    >>>

这通过告诉您忘记执行该接口的操作来提供一些被动接口强制执行.


Formal Interface Syntax
=======================

Python 语法是在 Python 参考手册 [8]_ 中描述的修改后的 BNF 语法符号中定义的.
本节介绍使用此语法的建议接口语法 ::

    interfacedef:   "interface" interfacename [extends] ":" suite
    extends:        "(" [expression_list] ")"
    interfacename:  identifier

接口定义是可执行语句. 它首先评估扩展列表 (如果存在). 扩展列表中的每个项都应该评估为接口对象.

然后使用新创建的本地命名空间和原始全局命名空间, 在新的执行框架中执行接口的套件
(参见 Python 参考手册, 第4.1节). 当接口的套件完成执行时, 其执行帧将被丢弃,
但其本地名称空间将保存为接口元素. 然后使用基本接口的扩展列表和保存的接口元素创建接口对象.
接口名称绑定到原始本地名称空间中的此接口对象.

此 PEP 还建议扩展 Python 的 "类" 语句 ::

    classdef:    "class" classname [inheritance] [implements] ":" suite
    implements:  "implements" implist
    implist:     expression-list

    classname,
    inheritance,
    suite,
    expression-list:  see the Python Reference Manual

在执行类的套件之前, 将评估 "继承" 和 "实现" 语句 (如果存在). 
"继承" 特性未在 "语言参考”的第 7.6 节中定义.

'implements' (如果存在) 在继承后进行评估. 这必须评估接口规范,
接口规范是接口或接口规范的元组. 如果存在有效的接口规范,
则将断言分配给类对象的 '__implements__' 属性, 作为元组.

本 PEP 不建议对函数定义或赋值的语法进行任何更改.


Classes and Interfaces
======================

上面的示例接口没有描述其方法的任何特性, 它们只描述了典型的 FishMarket 对象将实现的接口.

您可能会注意到从其他接口扩展的接口和从其他类进行子类化的接口之间的相似性.
这是一个类似的概念. 但是, 重要的是要注意接口扩展接口和类子类.
您不能扩展接口的类或子类. 类和接口是分开的.

类的目的是共享对象如何工作的实现. 接口的目的是记录如何使用对象, 而不是如何实现对象.
可以使用具有非常不同实现的若干不同类实现相同的接口.

也可以实现一个具有许多类的接口, 这些类混合了接口的功能, 或者相反, 可以让一个类实现许多接口.
因此, 不应该混淆或混合接口和类.


Interface-aware built-ins
=========================

根据接口对象对 Python 的内置函数列表进行有用的扩展将是 ``implements()``.
这个内置函数需要两个参数, 一个对象和一个接口, 如果对象实现了接口, 则返回 true 值,
否则返回 false. 例如 ::

    >>> interface FooInterface: pass
    >>> class Foo implements FooInterface: pass
    >>> f = Foo()
    >>> implements(f, FooInterface)
    1

目前, 该功能在参数实现中作为 ``Interface`` 包中的函数存在, 需要 "导入接口" 才能使用它.
它作为内置的存在纯粹是为了方便, 而不是使用接口所必需的, 类似于 ``isinstance()`` 用于类.


Backward Compatibility
======================

建议的接口模型不会在 Python 中引入任何向后兼容性问题.
但是, 提出的语法确实如此.

任何使用 ``interface`` 作为标识符的现有代码都会中断. 可能存在其他类型的向后不兼容性,
将 ``interface`` 定义为新关键字将引入. Python 语法的这种扩展不会以任何向后不兼容的方式更改任何现有语法.

新的 ``from __future__`` Python 语法 [6]_ 和新的警告框架 [7]_ 是解决这种向后不兼容性的理想选择.
要立即使用接口语法, 开发人员可以使用该语句 ::

    from __future__ import interfaces

此外, 任何使用关键字 ``interface`` 作为标识符的代码都将从 Python 发出警告.
在适当的时间段之后, 接口语法将成为标准, 上面的 import 语句将不执行任何操作,
任何名为 ``interface`` 的标识符都会引发异常. 这段时间建议为24个月.


Summary of Proposed Changes to Python
=====================================

添加新的 ``interface`` 键字并使用 ``implements`` 扩展类语法.

扩展类接口来包含 ``__implements__``.

添加内置的 'implements(obj, interface)' .


Risks
=====

这个 PEP 建议在 Python 语言中添加一个新的关键字 ``interface``.
这将打破代码.


Open Issues
===========

Goals
-----

Syntax
------

Architecture
------------


Dissenting Opinion
==================

这个 PEP 还没有在 python-dev 上讨论过.


References
==========

.. [1] https://mail.python.org/pipermail/types-sig/1998-December/date.html

.. [2] http://www.zope.org

.. [3] PEP 232, 函数属性, Warsaw
       http://www.python.org/dev/peps/pep-0232/

.. [4] PEP 233, Python 在线帮助, Prescod
       http://www.python.org/dev/peps/pep-0233/

.. [5] http://www.lemburg.com/files/python/mxProxy.html

.. [6] PEP 236, 返回到 __future__, Peters
       http://www.python.org/dev/peps/pep-0236/

.. [7] PEP 230, 警告框架, van Rossum
       http://www.python.org/dev/peps/pep-0236/

.. [8] Python 参考手册
       http://docs.python.org/reference/


Copyright
=========

This document has been placed in the public domain.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
