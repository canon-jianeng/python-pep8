
PEP: 213
Title: Attribute Access Handlers
Version: $Revision$
Last-Modified: $Date$
Author: paul@prescod.net (Paul Prescod)
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 21-Jul-2000
Python-Version: 2.1
Post-History:


Introduction
============

当实例的客户端代码尝试设置属性并执行代码时, Python 代码和扩展模块中可能 (甚至相对常见) "陷阱".
换句话说, 即使底层实现正在进行一些计算而不是直接修改绑定, 也可以允许用户使用属性赋值/检索/删除语法.

此 PEP 描述了一种功能, 可以更轻松, 更高效, 更安全地为 Python 实例实现这些处理程序.


Justification
=============

Scenario 1
----------

您有一个部署的类可以处理名为 "stdout" 的属性. 过了一会儿,
你认为最好在分配时检查 stdout 是否真的是一个带有 "write" 方法的对象.
而不是更改为 setstdout 方法 (与部署的代码不兼容), 您宁愿捕获赋值并检查对象的类型.

Scenario 2
----------

您希望与具有属性赋值概念的对象模型尽可能兼容. 它可以是 W3C 文档对象模型或特定的 COM 接口
(例如, PowerPoint 接口). 在这种情况下, 您可能希望模型中的属性在 Python 界面中显示为属性,
即使底层实现可能根本不使用属性.

Scenario 3
----------

用户想要将属性设置为只读.

简而言之, 此功能允许程序员出于任何目的将其模块的接口与底层实现分开.
同样, 这不是一个新功能, 而只是现有约定的新语法.


Current Solution
================

使某些属性为只读 ::

   class foo:
       def __setattr__( self, name, val ):
           if name=="readonlyattr":
               raise TypeError
           elif name=="readonlyattr2":
               raise TypeError
       ...
       else:
           self.__dict__["name"]=val

这具有以下问题:

1. 该方法的创建者必须密切注意类层次结构 ``__setattr__`` 中的其他位置是否也被捕获用于任何特定目的.
    如果是这样, 她必须专门调用该方法而不是分配给字典. 超载 ``__setattr__`` 有很多不同的原因,
	因此有很好的冲突潜力. 例如, 对象数据库实现经常使 setattr 超载以达到完全不相关的目的.

2. 基于字符串的 switch 语句强制在代码中的一个位置指定所有属性处理程序.
   然后, 他们可能会调度到特定于任务的方法 (用于模块化), 但这可能会导致性能问题.

3. 设置, 获取和删除的逻辑必须存在于 ``__getattr__``, ``__setattr__`` 和 ``__delattr__`` 中.
   再次, 这可以通过额外的方法调用来缓解, 但这是低效的.


Proposed Syntax
===============

特殊方法应该声明以下形式的声明 ::

   class x:
       def __attr_XXX__(self, op, val ):
           if op=="get":
               return someComputedValue(self.internal)
           elif op=="set":
               self.internal=someComputedValue(val)
           elif op=="del":
               del self.internal


客户端代码如下所示 ::

   fooval=x.foo
   x.foo=fooval+5
   del x.foo


Semantics
=========

这三种属性引用都应该调用该方法. op 参数可以是 "get"/"set"/"del".
当然这个字符串将被实现, 因此对字符串的实际检查将非常快.

不允许在与名为 __attr_XXX__ 的方法相同的实例中实际具有名为 XXX 的属性.

__attr_XXX__ 的实现优先于 ``__getattr__`` 的实现, 基于这样的原则:
``__getattr__`` 应该在找到适当的属性失败后调用.

__attr_XXX__ 的实现优先于 ``__setattr__`` 的实现, 以保持一致.
然而, 相反的选择似乎也是可行的. __del_y__ 也是如此.


Proposed Implementation
=======================

有一种称为属性访问处理程序的新对象类型.
此类对象具有以下属性 ::

    name (e.g. XXX, not __attr__XXX__)
    method (pointer to a method object)

在 PyClass_New 中, 将检测适当形式的方法并将其转换为对象 (就像未绑定的方法对象一样).
它们存储在名为 XXX 的类 ``__dict__`` 中. 原始方法以其原始名称存储为未绑定方法.

如果实例中有任何属性访问处理程序, 则设置标志. 我们现在称之为 "I_have_computed_attributes".
派生类从基类继承标志. 实例从类继承标志.

像往常一样获得收益直到返回对象之前. 除了当前检查返回的对象是否是方法之外,
它还将检查返回的对象是否是访问处理程序. 如果是这样, 它将调用 getter 方法并返回值.
要删除属性访问处理程序, 您可以直接使用字典.

通过检查 "I_have_computed_attributes" 标志来继续进行设置. 如果没有设置, 一切都会像今天一样进行.
如果设置了那么我们必须在字典上获取所请求的对象名称. 如果它返回属性访问处理程序, 那么我们用值调用 setter 函数.
如果它返回任何其他对象, 那么我们丢弃结果并继续像今天一样. 请注意,
具有属性访问处理程序将温和地影响特定实例上所有集的属性 "设置" 性能,
但使用 "__setattr__" 时不会超过今天. 使用 ``__getattr__`` 获取比今天更高效.

I_have_computed_attributes 标志旨在消除不使用此功能的对象的每个 "set" 额外 "get" 的性能下降.
检查此标志应该对所有对象产生微不足道的性能影响.

delete 的实现类似于 set 的实现.


Caveats
=======

1. 您可能会注意到, 我没有提出任何逻辑来保持 I_have_computed_attributes 标志是最新的,
   因为属性是从实例的字典中添加和删除的. 这与当前的 Python 一致.
   如果在对象使用后向对象添加一个 ``__setattr__`` 方法, 那么该方法的特性与在 "编译" 时可用的方法不同.
   可以说, 动力不值得额外的实施工作. 此代码段演示了当前的特性 ::

       >>> def prn(*args):print args
       >>> class a:

       ...    __setattr__=prn
       >>> a().foo=5
       (<__main__.a instance at 882890>, 'foo', 5)

       >>> class b: pass
       >>> bi=b()
       >>> bi.__setattr__=prn
       >>> b.foo=5


2. 分配给 __dict __ ["XXX"] 可以覆盖 __attr_XXX__ 的属性访问处理程序.
   通常, 访问处理程序将信息存储在私有的 __XXX 变量中


3. 尝试在对象本身上调用 setattr 或 getattr 的属性访问处理程序可能会导致无限循环
    (与 ``__getattr__`` 一样) 再一次, 解决方案是使用特殊的 (通常是私有的) 变量, 如 __XXX.


Note
====

PEP 252 中描述的描述符机制足够强大, 可以更直接地支持它.
可以在语言中添加 'getset' 构造函数, 使其成为可能::

   class C:
       def get_x(self):
           return self.__x
       def set_x(self, v):
           self.__x = v
       x = getset(get_x, set_x)

可以添加额外的语法糖, 或者可以识别命名约定.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
