
PEP: 231
Title: __findattr__()
Version: $Revision$
Last-Modified: $Date$
Author: barry@python.org (Barry Warsaw)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 30-Nov-2000
Python-Version: 2.1
Post-History:


Introduction
============

此 PEP 描述了实例属性查找和修改机制的扩展, 它允许许多有趣的编程模型的纯 Python 实现.
此 PEP 跟踪此功能的状态和所有权. 它包含功能的说明, 并概述了支持该功能所需的更改.
本 PEP 总结了在邮件列表论坛中进行的讨论, 并在适当情况下提供了 URL 来获取更多信息.
此文件的 CVS 修订历史记录包含确定的历史记录.


Background
==========

Python 实例的语义允许程序员通过特殊方法 ``__getattr__()``
和 ``__setattr__()`` 来自定义属性查找和属性修改的某些方面. [1]_.

但是, 由于这些方法施加了某些限制, 因此有一些有用的编程技术无法单独用 Python 编写,
例如, 严格的 Java Bean-like [2]_ 接口和 Zope 风格的并购 [3]_.
在后一种情况下, Zope 通过包含一个名为 ExtensionClass [5]_ 的 C 扩展来解决这个问题,
该扩展修改了标准类语义, 并在 Python 的类模型中使用了一个元类钩子,
称为 "Don Beaudry Hook" 或 "Don Beaudry Hack" [6]_.

虽然 Zope 的方法有效, 但它有几个缺点. 首先, 它需要 C 扩展. 其次,
它在 Python 机器中使用了一个非常神秘但 truck-sized 的漏洞.
第三, 其他程序员很难使用和理解 (元类具有众所周知的大脑爆炸特性).
第四, 因为 ExtensionClass 实例不是 "真正的" Python 实例,
所以 Python 运行时系统的某些方面不适用于 ExtensionClass 实例.

解决这个问题的建议经常被归入固定 "class/type 二分法" 的标题之下;
也就是说, 消除了内置类型和类之间的差异 [7]_. 虽然这是一个值得称道的目标,
但为了实现上述编程结构的类型, 修复这个裂缝并不是必需的.
该提议提供了80％的解决方案, 只需对 Python 的类和实例对象进行最少的修改.
它没有解决 type/class 二分法.


Proposal
========

该提议添加了一个名为 ``__findattr__()`` 的新特殊方法, 其语义如下:

* 如果在类中定义, 它将在所有实例属性分辨率上调用, 而不是 ``__getattr__()`` 和 ``__setattr__()``.

* ``__findattr__()`` 永远不会递归调用. 也就是说, 当一个特定实例的 ``__findattr__()`` 在调用堆栈上时,
  该实例的进一步属性访问将使用标准 ``__getattr__()`` 和 ``__setattr__()`` 方法.

* ``__findattr__()`` 被调用属性访问 ('获取') 和属性修改 ('设置'). 不要求删除属性.

* 当调用 get 时, 它会传递一个参数 (不包括 'self'): 被访问属性的名称.

* 当调用设置时, 将使用第三个参数调用它, 该参数是将属性的值设置为.

* ``__findattr__()`` 方法与 ``__getattr__()`` 和 ``__setattr__()`` 具有相同的缓存语义;
  即如果它们在类定义时存在于类中, 则使用它们, 但如果它们随后被添加到类中, 则它们不会.


Key Differences with the Existing Protocol
==========================================

``__findattr__()`` 的语义与关键方式的现有协议不同:

首先, 如果在实例的 ``__dict__`` 中找到属性, 则永远不会调用 ``__getattr__()``.
这是出于效率原因而做的, 因为否则, ``__setattr__()`` 将无法获取实例的属性.

其次, ``__setattr__()`` 不能使用 "普通" 语法来设置实例属性, 例如 "self.name = foo"
因为这会导致递归调用 ``__setattr__()``.

``__findattr__()`` 总是被调用, 无论属性是否在 ``__dict__`` 中,
并且实例对象中的一个标志阻止了对 ``__findattr__()`` 的递归调用.
这使类有机会为每个属性访问执行某些操作. 并且因为它为 gets 和 sets 都调用,
所以很容易为所有属性访问编写类似的策略. 此外, 效率不是问题, 因为它仅在使用扩展机制时支付.


Related Work
============

PEP 213 [9]_ 描述了一种钩子到属性访问和修改的不同方法. PEP 213 中提出的语义
可以使用这里描述的 ``__findattr__()`` 钩子来实现, 但有一点需要注意.
``__findattr__()`` 的当前引用实现不支持钩子属性删除. 如果发现需要, 可以添加. 见下面的例子.


Examples
========

此提议允许的一种编程风格是类似 Java Bean 的对象接口,
其中未经修饰的属性访问和修改透明地映射到函数式接口.  例如.

::

    class Bean:
        def __init__(self, x):
            self.__myfoo = x

        def __findattr__(self, name, *args):
            if name.startswith('_'):
                # Private names
                if args: setattr(self, name, args[0])
                else:    return getattr(self, name)
            else:
                # Public names
                if args: name = '_set_' + name
                else:    name = '_get_' + name
                return getattr(self, name)(*args)

        def _set_foo(self, x):
            self.__myfoo = x

        def _get_foo(self):
            return self.__myfoo


    b = Bean(3)
    print b.foo
    b.foo = 9
    print b.foo


第二个更精细的例子是在纯 Python 中实现隐式和显式获取::

    import types

    class MethodWrapper:
        def __init__(self, container, method):
            self.__container = container
            self.__method = method

        def __call__(self, *args, **kws):
            return self.__method.im_func(self.__container, *args, **kws)


    class WrapperImplicit:
        def __init__(self, contained, container):
            self.__contained = contained
            self.__container = container

        def __repr__(self):
            return '<Wrapper: [%s | %s]>' % (self.__container,
                                             self.__contained)

        def __findattr__(self, name, *args):
            # Some things are our own
            if name.startswith('_WrapperImplicit__'):
                if args: return setattr(self, name, *args)
                else:    return getattr(self, name)
            # setattr stores the name on the contained object directly
            if args:
                return setattr(self.__contained, name, args[0])
            # Other special names
            if name == 'aq_parent':
                return self.__container
            elif name == 'aq_self':
                return self.__contained
            elif name == 'aq_base':
                base = self.__contained
                try:
                    while 1:
                        base = base.aq_self
                except AttributeError:
                    return base
            # no acquisition for _ names
            if name.startswith('_'):
                return getattr(self.__contained, name)
            # Everything else gets wrapped
            missing = []
            which = self.__contained
            obj = getattr(which, name, missing)
            if obj is missing:
                which = self.__container
                obj = getattr(which, name, missing)
                if obj is missing:
                    raise AttributeError, name
            of = getattr(obj, '__of__', missing)
            if of is not missing:
                return of(self)
            elif type(obj) == types.MethodType:
                return MethodWrapper(self, obj)
            return obj


    class WrapperExplicit:
        def __init__(self, contained, container):
            self.__contained = contained
            self.__container = container

        def __repr__(self):
            return '<Wrapper: [%s | %s]>' % (self.__container,
                                             self.__contained)

        def __findattr__(self, name, *args):
            # Some things are our own
            if name.startswith('_WrapperExplicit__'):
                if args: return setattr(self, name, *args)
                else:    return getattr(self, name)
            # setattr stores the name on the contained object directly
            if args:
                return setattr(self.__contained, name, args[0])
            # Other special names
            if name == 'aq_parent':
                return self.__container
            elif name == 'aq_self':
                return self.__contained
            elif name == 'aq_base':
                base = self.__contained
                try:
                    while 1:
                        base = base.aq_self
                except AttributeError:
                    return base
            elif name == 'aq_acquire':
                return self.aq_acquire
            # explicit acquisition only
            obj = getattr(self.__contained, name)
            if type(obj) == types.MethodType:
                return MethodWrapper(self, obj)
            return obj

        def aq_acquire(self, name):
            # Everything else gets wrapped
            missing = []
            which = self.__contained
            obj = getattr(which, name, missing)
            if obj is missing:
                which = self.__container
                obj = getattr(which, name, missing)
                if obj is missing:
                    raise AttributeError, name
            of = getattr(obj, '__of__', missing)
            if of is not missing:
                return of(self)
            elif type(obj) == types.MethodType:
                return MethodWrapper(self, obj)
            return obj


    class Implicit:
        def __of__(self, container):
            return WrapperImplicit(self, container)

        def __findattr__(self, name, *args):
            # ignore setattrs
            if args:
                return setattr(self, name, args[0])
            obj = getattr(self, name)
            missing = []
            of = getattr(obj, '__of__', missing)
            if of is not missing:
                return of(self)
            return obj


    class Explicit(Implicit):
        def __of__(self, container):
            return WrapperExplicit(self, container)


    # tests
    class C(Implicit):
        color = 'red'

    class A(Implicit):
        def report(self):
            return self.color

    # simple implicit acquisition
    c = C()
    a = A()
    c.a = a
    assert c.a.report() == 'red'

    d = C()
    d.color = 'green'
    d.a = a
    assert d.a.report() == 'green'

    try:
        a.report()
    except AttributeError:
        pass
    else:
        assert 0, 'AttributeError expected'


    # special names
    assert c.a.aq_parent is c
    assert c.a.aq_self is a

    c.a.d = d
    assert c.a.d.aq_base is d
    assert c.a is not a


    # no acquisition on _ names
    class E(Implicit):
        _color = 'purple'

    class F(Implicit):
        def report(self):
            return self._color

    e = E()
    f = F()
    e.f = f
    try:
        e.f.report()
    except AttributeError:
        pass
    else:
        assert 0, 'AttributeError expected'


    # explicit
    class G(Explicit):
        color = 'pink'

    class H(Explicit):
        def report(self):
            return self.aq_acquire('color')

        def barf(self):
            return self.color

    g = G()
    h = H()
    g.h = h
    assert g.h.report() == 'pink'

    i = G()
    i.color = 'cyan'
    i.h = h
    assert i.h.report() == 'cyan'

    try:
        g.i.barf()
    except AttributeError:
        pass
    else:
        assert 0, 'AttributeError expected'


类似于 C++ 的访问控制也可以完成, 虽然不太干净,
因为很难确定从运行时调用堆栈, 调用什么方法 ::

    import sys
    import types

    PUBLIC = 0
    PROTECTED = 1
    PRIVATE = 2

    try:
        getframe = sys._getframe
    except ImportError:
        def getframe(n):
            try: raise Exception
            except Exception:
                frame = sys.exc_info()[2].tb_frame
            while n > 0:
                frame = frame.f_back
                if frame is None:
                    raise ValueError, 'call stack is not deep enough'
            return frame


    class AccessViolation(Exception):
        pass


    class Access:
        def __findattr__(self, name, *args):
            methcache = self.__dict__.setdefault('__cache__', {})
            missing = []
            obj = getattr(self, name, missing)
            # if obj is missing we better be doing a setattr for
            # the first time
            if obj is not missing and type(obj) == types.MethodType:
                # Digusting hack because there's no way to
                # dynamically figure out what the method being
                # called is from the stack frame.
                methcache[obj.im_func.func_code] = obj.im_class
            #
            # What's the access permissions for this name?
            access, klass = getattr(self, '__access__', {}).get(
                name, (PUBLIC, 0))
            if access is not PUBLIC:
                # Now try to see which method is calling us
                frame = getframe(0).f_back
                if frame is None:
                    raise AccessViolation
                # Get the class of the method that's accessing
                # this attribute, by using the code object cache
                if frame.f_code.co_name == '__init__':
                    # There aren't entries in the cache for ctors,
                    # because the calling mechanism doesn't go
                    # through __findattr__().  Are there other
                    # methods that might have the same behavior?
                    # Since we can't know who's __init__ we're in,
                    # for now we'll assume that only protected and
                    # public attrs can be accessed.
                    if access is PRIVATE:
                        raise AccessViolation
                else:
                    methclass = self.__cache__.get(frame.f_code)
                    if not methclass:
                        raise AccessViolation
                    if access is PRIVATE and methclass is not klass:
                        raise AccessViolation
                    if access is PROTECTED and not issubclass(methclass,
                                                              klass):
                        raise AccessViolation
            # If we got here, it must be okay to access the attribute
            if args:
                return setattr(self, name, *args)
            return obj

    # tests
    class A(Access):
        def __init__(self, foo=0, name='A'):
            self._foo = foo
            # can't set private names in __init__
            self.__initprivate(name)

        def __initprivate(self, name):
            self._name = name

        def getfoo(self):
            return self._foo

        def setfoo(self, newfoo):
            self._foo = newfoo

        def getname(self):
            return self._name

    A.__access__ = {'_foo'      : (PROTECTED, A),
                    '_name'     : (PRIVATE, A),
                    '__dict__'  : (PRIVATE, A),
                    '__access__': (PRIVATE, A),
                    }

    class B(A):
        def setfoo(self, newfoo):
            self._foo = newfoo + 3

        def setname(self, name):
            self._name = name

    b = B(1)
    b.getfoo()

    a = A(1)
    assert a.getfoo() == 1
    a.setfoo(2)
    assert a.getfoo() == 2

    try:
        a._foo
    except AccessViolation:
        pass
    else:
        assert 0, 'AccessViolation expected'

    try:
        a._foo = 3
    except AccessViolation:
        pass
    else:
        assert 0, 'AccessViolation expected'

    try:
        a.__dict__['_foo']
    except AccessViolation:
        pass
    else:
        assert 0, 'AccessViolation expected'


    b = B()
    assert b.getfoo() == 0
    b.setfoo(2)
    assert b.getfoo() == 5
    try:
        b.setname('B')
    except AccessViolation:
        pass
    else:
        assert 0, 'AccessViolation expected'

    assert b.getname() == 'A'


这是 PEP 213 中描述的属性钩子的实现 (除了当前参考实现不支持钩子属性删除).

::

    class Pep213:
        def __findattr__(self, name, *args):
            hookname = '__attr_%s__' % name
            if args:
                op = 'set'
            else:
                op = 'get'
            # XXX: op = 'del' currently not supported
            missing = []
            meth = getattr(self, hookname, missing)
            if meth is missing:
                if op == 'set':
                    return setattr(self, name, *args)
                else:
                    return getattr(self, name)
            else:
                return meth(op, *args)


    def computation(i):
        print 'doing computation:', i
        return i + 3


    def rev_computation(i):
        print 'doing rev_computation:', i
        return i - 3


    class X(Pep213):
        def __init__(self, foo=0):
            self.__foo = foo

        def __attr_foo__(self, op, val=None):
            if op == 'get':
                return computation(self.__foo)
            elif op == 'set':
                self.__foo = rev_computation(val)
            # XXX: 'del' not yet supported

    x = X()
    fooval = x.foo
    print fooval
    x.foo = fooval + 5
    print x.foo
    # del x.foo


Reference Implementation
========================

作为 Python 核心的补丁, 可以在此 URL 中找到参考实现:

http://sourceforge.net/patch/?func=detailpatch&patch_id=102613&group_id=5470


References
==========

.. [1] http://docs.python.org/reference/datamodel.html#customizing-attribute-access
.. [2] http://www.javasoft.com/products/javabeans/
.. [3] http://www.digicool.com/releases/ExtensionClass/Acquisition.html
.. [5] http://www.digicool.com/releases/ExtensionClass
.. [6] http://www.python.org/doc/essays/metaclasses/
.. [7] http://www.foretec.com/python/workshops/1998-11/dd-ascher-sum.html
.. [8] http://docs.python.org/howto/regex.html
.. [9] PEP 213, Attribute Access Handlers, Prescod
    http://www.python.org/dev/peps/pep-0213/


Rejection
=========

递归保护特征存在严重问题. 如此处所述, 它不是线程安全的,
并且线程安全的解决方案还有其他问题. 一般来说,
目前尚不清楚递归保护特征有多大帮助;
这使得编写需要在 ``__findattr__`` 内部以及在其外部调用的代码变得很困难.
但是没有递归保护, 很难实现 ``__findattr__``
(因为 ``__findattr__`` 会为它试图访问的每个属性递归调用自己).
这里似乎没有好的解决方案.

获得和设置属性都支持 ``__findattr__`` 是多么有用 - 在所有情况下都会调用 ``__setattr__``.

如果注意不要存储在自己的名字的实例变量, 这些示例都可以使用 ``__getattr__`` 实现.


Copyright
=========

This document has been placed in the Public Domain.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
