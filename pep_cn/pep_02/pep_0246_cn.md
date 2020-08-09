
PEP: 246
Title: Object Adaptation
Version: $Revision$
Last-Modified: $Date$
Author: aleaxit@gmail.com (Alex Martelli),
    cce@clarkevans.com (Clark C. Evans)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 21-Mar-2001
Python-Version: 2.5
Post-History: 29-Mar-2001, 10-Jan-2005


Rejection Notice
================

我拒绝接受这个 PEP. 即将发生的事情要好得多; 现在说出确切的内容还为时过早,
但它不会过于接近这个 PEP 中的提议, 所以最好开始一个新的 PEP.  GvR.


Abstract
========

该提议提出了一种可扩展的协作机制, 用于将传入对象适配到上下文,
该上下文期望支持特定协议的对象 (比如特定类型, 类或接口).

该提议提供了一个内置的 "adapt" 函数, 对于任何对象 X 和任何协议 Y,
可以用来向 Python 环境询问符合 Y 的 X 版本. 在幕后, 该机制会询问对象 X:
你现在, 或者你知道如何包装自己提供协议 Y 的支持者吗?" 并且, 如果此请求失败,
则函数会询问协议 Y: "对象 X 是否支持您, 或者您是否知道如何将其包装来获得此类支持者?"
这种二元性很重要, 因为协议可以在对象之后开发, 反之亦然, 并且该 PEP 允许任何一种情况相对于预先存在的组件非侵入地支持.

最后, 如果对象和协议都不知道彼此, 则该机制可以检查适配器工厂的注册表,
其中可以动态地注册能够使某些对象适应某些协议的可调用对象.
提案的这一部分是可选的: 通过确保某些类型的协议或对象可以接受适配器工厂的动态注册,
例如, 通过合适的自定义元类, 可以获得相同的效果. 但是,
这个可选部分允许以不妨碍协议或其他对象的方式使适配变得更加灵活和强大,
从而获得适应性, 这与 Python 标准库的 "copy_reg" 模块提供的序列化和续编.

该提议没有具体约束协议是什么, "对协议的合规性" 到底是什么意思, 也不是包装器应该做什么.
这些遗漏旨在使该提议与现有的协议类别兼容, 例如现有的类型和类系统,
以及为 Python 提出或实现的 "接口" 的许多概念, 例如, 一个在 PEP 245 [1]_,
一个在 Zope3 [2]_, 或者在2004年底和2005年初在 BDFL 的 Artima 博客中讨论过 [3]_.
然而, 还包括对这些主题的一些反思, 旨在提出建议性而非规范性.


Motivation
==========

目前, Python 中没有用于检查对象是否支持特定协议的标准化机制. 通常, 某些方法的存在,
特别是诸如 "__getitem__" 的特殊方法, 被用作支持特定协议的指示符.
这种技术适用于 BDFL (Benevolent Dictator for Life) 祝福的一些特定协议.
基于检查 'isinstance' 的替代技术也可以这样说 (内置类 "basestring" 专门用于让你
使用 'isinstance' 来检查一个对象是否是一个 \[built-in\] 字符串).
这两种方法都不容易且通常可扩展到由标准 Python 核心之外的应用程序和第三方框架定义的其他协议.

比检查对象是否已经支持给定协议更重要的是为对象获取合适的适配器 (包装器或代理) 的任务,
如果支持尚不存在的话. 例如, 字符串不支持文件协议, 但您可以将其包装到 StringIO 实例中来获取支持该协议的对象,
并从其包装的字符串中获取其数据; 这样, 您可以将字符串 (适当包装) 传递给子系统, 这些子系统需要作为参数的对象,
这些对象可以作为文件读取. 不幸的是, 目前还没有一种通用的, 标准化的方法来自动化这种极其重要的 "通过包装" 操作.

通常, 今天, 当您将对象传递给期望特定协议的上下文时, 对象都知道上下文并提供自己的包装器,
或者上下文知道对象并适当地包装它. 这些方法的困难在于, 这种调整是一次性的,
不是集中在用户代码的单个位置, 并且不是用通用技术等执行的.
这种标准化的缺乏增加了代码重复与相同的适配器发生在不止一个地方或它鼓励类重写而不是改编.
无论哪种情况, 可维护性都会受到影响.

如果有一个标准函数可以被调用来验证对象是否符合特定协议并提供一个包装器 (如果有一个可用的话),
这将是非常好的 -- 对于具体的, 特定的情况, 所有这些都无需通过每个库的文档来搜索适合的咒语.


Requirements
============

在考虑对象是否符合协议时, 有几种情况需要检查:

a) 当协议是一个类型或类, 并且该对象具有该类型或者是该类的实例 (不是子类).
    在这种情况下, 合规性是自动的.

b) 当对象知道协议时, 要么认为自己是合规的, 要么知道如何适当地包装自己.

c) 当协议知道对象时, 对象已经符合或者协议知道如何适当地包装对象.

d) 当协议是类型或类时, 对象是子类的成员. 这与上面的第一种情况 (a) 不同,
    因为继承 (不幸的是) 并不一定意味着可替代性, 因此必须小心处理.

e) 当上下文知道对象和协议并知道如何调整对象来满足所需的协议时.
    这可以使用适配器注册表或类似的方法.

上面的第四个示例是微妙的. 当子类更改方法的签名时, 可以发生可替代性的中断,
或者限制为方法的参数接受的域 (参数类型上的 "协方差"),
或者扩展共域来包含基类可能永远不会返回的返回值产生 (返回类型的 "contra-variance").
虽然基于类继承的合规性应该是自动的, 但此提议允许对象发出信号表明它不符合基类协议.

但是, 如果 Python 获得了某些接口的标准 "官方" 机制, 那么 "快速路径" 情况 (a)
可以而且应该扩展到作为接口的协议, 并且该接口对象是声称符合的类型或类的实例.
例如, 如果 [3]_ 中讨论的 "interface" 关键字被采用到 Python 中, 则可以使用 case (a) 的 "快速路径",
因为实现接口的可实例化类不允许破坏可替代性.


Specification
=============

该提议引入了一个新的内置函数 ``adapt()``, 它是支持这些要求的基础.

``adapt()`` 函数有三个参数:

- ``obj``, 要适应的对象

- ``protocol``, 对象请求的协议

- ``alternate``, 如果无法调整对象, 则返回的可选对象

``adapt()`` 函数的成功结果返回传递 ``obj`` 的对象, 如果对象已经符合协议,
或者返回辅助对象 ``wrapper``, 它提供了一个符合协议的对象. 包装器的定义是故意模糊的,
如果需要, 允许包装器成为具有自己状态的完整对象. 但是, 设计意图是适配包装器应该
保存对它包装的原始对象的引用, 加上 (如果需要) 最小的额外状态, 它不能委托给包装器对象.

自适应包装器的一个很好的例子是 StringIO 的一个实例, 它调整传入的字符串,
就像它是一个文本文件一样: 包装器保存对字符串的引用, 但它自己处理 "当前读取点"
(从其中在包装的字符串中将是下一个字符, 例如 "readline" 调用源于该实例),
因为它不能将它委托给包装对象 (字符串没有 "当前读取点" 的概念, 甚至远程也没有与该概念有关).

无法使对象适应协议会引发一个 ``AdaptationError`` (它是 ``TypeError`` 的子类),
除非使用了备用参数, 在这种情况下会返回替换参数.

为了启用需求中列出的第一种情况, ``adapt()`` 函数首先检查对象的类型或对象的类是否与协议相同.
如果是这样, 那么 ``adapt()`` 函数会直接返回对象, 而不会更加轻松.

要启用第二种情况, 当对象知道协议时, 对象必须具有 ``__conform__()`` 方法.
此可选方法有两个参数:

- ``self``, 被改编的对象

- ``protocol``, 请求的协议

就像今天的 Python 中的任何其他特殊方法一样, ``__conform__`` 意味着从对象的类中获取,
而不是从对象本身获取 (对于所有对象, 除了 "经典类" 的实例, 只要我们仍然必须支持后者).
如果需要, 这可以在将来将可能的 'tp_conform' 槽添加到 Python 的类型对象中.

对象可以作为 "__conform__" 的结果返回, 来表示符合性. 或者, 该对象还可以选择返回符合协议的包装器对象.
如果对象知道它不符合, 虽然它属于一个类型是协议的子类, 那么 ``__conform__`` 应该引发一个 ``LiskovViolation`` 异常
(``AdaptationError`` 的子类). 最后, 如果对象无法确定其符合性, 则应返回 "None" 来启用其余机制.
如果 ``__conform__`` 引发任何其他异常, "adapt" 只会传播它.

要启用第三种情况, 当协议知道对象时, 协议必须具有 ``__adapt__()`` 方法.
此可选方法有两个参数:

- ``self``, 请求的协议

- ``obj``, 被改编的对象

如果协议发现对象符合要求, 则可以直接返回 obj. 或者, 该方法可以返回符合协议的包装器.
如果协议知道对象不符合, 尽管它属于协议的子类, 那么 ``__adapt__`` 应该引发一个 ``LiskovViolation`` 异常
(``AdaptationError`` 的子类). 最后, 当无法确定合规性时, 此方法应返回 None 来启用其余机制.
如果 ``__adapt__`` 引发任何其他异常, "adapt" 只会传播它.

第四种情况, 当对象的类是协议的子类时, 由内置的 ``adapt()`` 函数处理. 在正常情况下,
如果 "isinstance(object, protocol)" 则 ``adapt()`` 直接返回对象. 但是, 如果对象不是可替换的,
那么 ``__conform__()`` 或 ``__adapt__()`` 方法, 如上所述, 可能会引发 ``LiskovViolation``
(``AdaptationError`` 的子类) 防止这种默认行为.

如果前四种机制都没有工作, 作为最后的尝试, 'adapt' 会回退到检查适配器工厂的注册表,
由协议和 ``obj`` 的类型索引, 来满足第五种情况. 可以从该注册表动态注册和删除适配器工厂,
来对对象或协议不具有侵入性的方式提供彼此不了解的对象和协议的 "第三方适配".


Intended Use
============

适应的典型用途是在代码中, 它从外部接收了一些对象 X, 作为参数或调用某个函数的结果,
并且需要根据某个协议 Y 使用该对象. "协议" 例如 Y 表示一个接口, 通常富含一些语义约束
(例如, 通常用于 "按合同设计" 方式), 并且通常还有一些实用的期望 (例如 "某个操作的运行时间")
应该不比 O(N) "更差"; 该提案没有具体说明协议是如何设计的, 也没有说明如何或是否检查协议的合规性,
以及声称合规但未实际交付的后果可能是什么 (缺乏 "语法" 合规性 -- 名称和签名方法 -- 通常会导致异常被提出;
缺乏 "语义" 合规性可能导致微妙的, 也许是偶然的错误 [想象一种方法声称是线程安全的,
但事实上它实际上受到一些微妙的竞争条件的影响]; 缺乏 "务实" 合规性通常会导致代码运行 "正确",
但实际使用速度太慢, 或者有时耗尽内存或磁盘空间等资源.

当协议 Y 是具体类型或类时, 对它的遵从性意味着对象允许可以在 Y 的实例上执行的所有操作, 具有 "可比较的" 语义和语用.
例如, 作为单链表的假设对象 X 不应该声称符合协议 'list', 即使它实现了 list 的所有方法: 索引 ``X[n]`` 需要时间 O(n),
同样的操作将是列表中的 O(1), 有所作为. 另一方面, ``StringIO.StringIO`` 的实例确实符合协议 '文件',
即使某些操作 (例如, 模块 'marshal' 的操作) 可能不允许将一个操作替换为另一个操作, 因为它们执行显式类型 -checks:
从协议合规性的角度来看, 这种类型检查 "超越了范围".

虽然这种约定使得使用具体类型或类作为本提案的目的的协议是可行的, 但是这种使用通常不是最佳的.
调用 'adapt' 的代码很少需要某种具体类型的所有特性, 特别是对于像文件, 列表, 字典这样的丰富类型;
很少有所有这些功能都可以通过具有良好语用的包装器提供, 以及语法和语义实际上与具体类型相同.

相反, 一旦这个提议被接受, 设计工作就需要开始确定当前在 Python 中使用的那些协议的基本特征,
特别是在标准库中, 并使用某种 "接口" 构造来形式化它们 (不一定需要任何新的语法: 一个简单的自定义元类将让我们开始,
并且工作的结果可以稍后迁移到任何 "接口" 构造最终被接受到 Python 语言中). 通过这种更正式设计的协议,
使用 "适应" 的代码将能够要求, 比如适应 "可读和可搜索的文件类对象", 或者其他任何其他特定需要的代码 "粒度",
而不是太一般地要求遵守 '文件' 协议.

适配不是 "造型". 当对象 X 本身不符合协议 Y 时, 将 X 调整为 Y 意味着使用某种包装对象Z, 它包含对 X 的引用,
并实现 Y 所需的任何操作, 主要是通过以适当的方式委托给 X. 例如, 如果 X 是一个字符串, 而 Y 是 'file',
那么使 X 适应 Y 的正确方法是使 ``StringIO(X)``, **NOT** 来调用 ``file(X)`` [会尝试打开由 X 命名的文件].

然而, 数字类型和协议可能需要是 "适配不是造型" 的咒语的异常.


Guido's "Optional Static Typing: Stop the Flames" Blog Entry
============================================================


一个典型的简单使用示例是 ::

    def f(X):
        X = adapt(X, Y)
        # continue by using X according to protocol Y

在 [4]_ 中, BDFL 提出了引入语法 ::

    def f(X: Y):
        # continue by using X according to protocol Y

作为一个方便的快捷方式, 恰好这种典型的使用适配, 并作为实验的基础,
直到解析器被修改为接受这种新语法, 语义等效的装饰器 ::

    @arguments(Y)
    def f(X):
        # continue by using X according to protocol Y

这些 BDFL 想法与此提案完全兼容,
Guido 在同一博客中的其他建议也是如此.



Reference Implementation and Test Cases
=======================================

以下参考实现不涉及经典类: 它只考虑新式类. 如果需要支持经典类, 那么添加内容应该非常清楚,
虽然有点乱 (``x.__class__`` vs ``type(x)``, 直接从对象而不是类型中获取 bound 方法, 等等).

::

    -----------------------------------------------------------------
    adapt.py
    -----------------------------------------------------------------
    class AdaptationError(TypeError):
        pass
    class LiskovViolation(AdaptationError):
        pass

    _adapter_factory_registry = {}

    def registerAdapterFactory(objtype, protocol, factory):
        _adapter_factory_registry[objtype, protocol] = factory

    def unregisterAdapterFactory(objtype, protocol):
        del _adapter_factory_registry[objtype, protocol]

    def _adapt_by_registry(obj, protocol, alternate):
        factory = _adapter_factory_registry.get((type(obj), protocol))
        if factory is None:
            adapter = alternate
        else:
            adapter = factory(obj, protocol, alternate)
        if adapter is AdaptationError:
            raise AdaptationError
        else:
            return adapter


    def adapt(obj, protocol, alternate=AdaptationError):

        t = type(obj)

        # (a) first check to see if object has the exact protocol
        if t is protocol:
           return obj

        try:
            # (b) next check if t.__conform__ exists & likes protocol
            conform = getattr(t, '__conform__', None)
            if conform is not None:
                result = conform(obj, protocol)
                if result is not None:
                    return result

            # (c) then check if protocol.__adapt__ exists & likes obj
            adapt = getattr(type(protocol), '__adapt__', None)
            if adapt is not None:
                result = adapt(protocol, obj)
                if result is not None:
                    return result
        except LiskovViolation:
            pass
        else:
            # (d) check if object is instance of protocol
            if isinstance(obj, protocol):
                return obj

        # (e) last chance: try the registry
        return _adapt_by_registry(obj, protocol, alternate)

    -----------------------------------------------------------------
    test.py
    -----------------------------------------------------------------
    from adapt import AdaptationError, LiskovViolation, adapt
    from adapt import registerAdapterFactory, unregisterAdapterFactory
    import doctest

    class A(object):
        '''
        >>> a = A()
        >>> a is adapt(a, A)   # case (a)
        True
        '''

    class B(A):
        '''
        >>> b = B()
        >>> b is adapt(b, A)   # case (d)
        True
        '''

    class C(object):
        '''
        >>> c = C()
        >>> c is adapt(c, B)   # case (b)
        True
        >>> c is adapt(c, A)   # a failure case
        Traceback (most recent call last):
            ...
        AdaptationError
        '''
        def __conform__(self, protocol):
            if protocol is B:
                return self

    class D(C):
        '''
        >>> d = D()
        >>> d is adapt(d, D)   # case (a)
        True
        >>> d is adapt(d, C)   # case (d) explicitly blocked
        Traceback (most recent call last):
            ...
        AdaptationError
        '''
        def __conform__(self, protocol):
            if protocol is C:
                raise LiskovViolation

    class MetaAdaptingProtocol(type):
        def __adapt__(cls, obj):
            return cls.adapt(obj)

    class AdaptingProtocol:
        __metaclass__ = MetaAdaptingProtocol
        @classmethod
        def adapt(cls, obj):
            pass

    class E(AdaptingProtocol):
        '''
        >>> a = A()
        >>> a is adapt(a, E)   # case (c)
        True
        >>> b = A()
        >>> b is adapt(b, E)   # case (c)
        True
        >>> c = C()
        >>> c is adapt(c, E)   # a failure case
        Traceback (most recent call last):
            ...
        AdaptationError
        '''
        @classmethod
        def adapt(cls, obj):
            if isinstance(obj, A):
                return obj

    class F(object):
        pass

    def adapt_F_to_A(obj, protocol, alternate):
        if isinstance(obj, F) and issubclass(protocol, A):
            return obj
        else:
            return alternate

    def test_registry():
        '''
        >>> f = F()
        >>> f is adapt(f, A)   # a failure case
        Traceback (most recent call last):
            ...
        AdaptationError
        >>> registerAdapterFactory(F, A, adapt_F_to_A)
        >>> f is adapt(f, A)   # case (e)
        True
        >>> unregisterAdapterFactory(F, A)
        >>> f is adapt(f, A)   # a failure case again
        Traceback (most recent call last):
            ...
        AdaptationError
        >>> registerAdapterFactory(F, A, adapt_F_to_A)
        '''

    doctest.testmod()


Relationship To Microsoft's QueryInterface
==========================================

尽管此提议与 Microsoft（COM）QueryInterface 有一些相似之处, 但它有许多不同之处.

首先, 该提议中的自适应是双向的, 允许查询接口 (协议), 这提供了更多的动态能力 (更多 Pythonic).
其次, 没有特殊的 "IUnknown" 接口可用于检查或获得原始的展开对象标识,
尽管这可以作为那些 "特殊" 祝福接口协议标识符之一提出. 第三, 使用 QueryInterface,
一旦对象支持特定接口, 它必须在支持此接口后始终存在; 这个提议没有做出这样的保证,
因为特别是适配器工厂可以动态地添加到注册中并在以后再次删除.

第四, Microsoft 的 QueryInterface 的实现必须支持一种等价关系 -- 它们必须具有反射性,
对称性和传递性, 在特定的意义上. 根据该提议的协议适应的等效条件也将代表期望的特性 ::

    # given, to start with, a successful adaptation:
    X_as_Y = adapt(X, Y)

    # reflexive:
    assert adapt(X_as_Y, Y) is X_as_Y

    # transitive:
    X_as_Z = adapt(X, Z, None)
    X_as_Y_as_Z = adapt(X_as_Y, Z, None)
    assert (X_as_Y_as_Z is None) == (X_as_Z is None)

    # symmetrical:
    X_as_Z_as_Y = adapt(X_as_Z, Y, None)
    assert (X_as_Y_as_Z is None) == (X_as_Z_as_Y is None)

然而, 虽然这些特性是可取的, 但在所有情况下都可能无法保证它们.
QueryInterface 可以强加它们的等价物, 因为它在某种程度上决定了对象,
接口和适配器的编码方式; 这个提议意味着不一定是侵入性的, 可用的,
并且在相互未知的两个框架之间 "改进”适应, 而不必修改任何一个框架.

事实上, 适应的传递性在某种程度上存在争议, 适应和继承之间的关系 (如果有的话) 也是如此.

如果我们知道继承总是意味着 Liskov 的可替代性, 那么后者就不会引起争议, 遗憾的是我们没有.
如果某些特殊形式, 例如 [4]_ 中提出的接口, 确实可以确保 Liskov 可替代性, 那么对于那种继承,
我们可能断言如果 X 符合 Y, 而 Y 继承自 Z, 则 X 符合 Z ... 但只有在非常强烈的意义上采用可替代性来包含语义和语用,
这似乎是值得怀疑的. (对于它的价值: 在 QueryInterface 中, 继承不需要也不暗示一致性).
除了上面具体详细说明的小规模之外, 该提案不包括继承的任何 "强烈" 影响.

同样, 传递性可能意味着多个 "内部" 适应过程通过一些中间 Y 得到 "adapt(X, Z)" 的结果,
本质上就像 ``adapt(adapt(X, Y), Z)``, 一些合适且自动选择的 Y.
同样, 这可能在适当的强约束下可行, 但这种方案的实际意义仍然不清楚这个提议的作者.
因此, 在任何情况下, 该提议都不包括任何自动或隐含的适应传递性.

对于该提案的原始版本的实现, 其在传递性和继承的效果方面执行更高级的处理,
参见 Phillip J.Eby 的 ``PyProtocols`` [5]_. 伴随着 ``PyProtocols`` 的文档非常值得研究,
因为它考虑了如何编码和使用适配器, 以及如何适应可以消除应用程序代码中对类型检查的任何需求.


Questions and Answers
=====================

* Q: 该提案提供了哪些好处?

  A: 典型的 Python 程序员是一个集成商, 他是连接各个供应商组件的人. 通常,
  要在这些组件之间进行接口, 需要中间适配器. 通常, 程序员负担研究由一个组件公开并由另一个组件所需的接口,
  确定它们是否直接兼容, 或开发适配器. 有时供应商甚至可能包含适当的适配器, 但即使这样,
  搜索适配器并确定如何部署适配器也需要时间.

  这种技术使供应商能够直接相互合作, 必要时实施 "__conform__" 或 "__adapt__". 这使集成商无需制作自己的适配器.
  实质上, 这允许组件之间具有简单的对话. 集成器只是将一个组件连接到另一个组件, 如果类型不自动匹配, 则内置适配机制.

  此外, 由于适配器注册表, "第四方" 可能提供适配器来允许完全不知道的框架互相操作彼此, 非入侵性,
  并且不需要集成商做任何事情, 而不是在注册表启动时, 安装适当的适配器工厂.

  只要库和框架与这里提出的适应基础设施合作 (主要是通过适当地定义和使用协议,
  并根据收到的参数和回调工厂函数的结果调用 '适应'), 集成商的工作就变得更加简单了.

  例如, 考虑 SAX1 和 SAX2 接口: 需要在它们之间切换所需的适配器. 通常, 程序员必须意识到这一点;
  然而, 有了这个适应方案, 现在已经不再是这样了 -- 实际上, 由于适配器注册表,
  即使提供 SAX1 的框架和需要 SAX2 的框架彼此不知道, 也可以删除这种需求.


* Q: 为什么这必须是内置的, 不能是独立的?

  A: 是的, 它确实独立运作. 但是, 如果它是内置的, 则它有更大的使用机会.
      该提案的价值主要在于标准化: 拥有来自不同供应商的库和框架, 包括 Python 标准库,
      使用单一的方法进行调整. 此外:

  0.  该机制本质上是一个单独的.

  1.  如果频繁使用, 内置速度会快得多.

  2.  它是可扩展和不张扬的.

  3.  一旦 'adapt' 内置, 它就可以支持语法扩展, 甚至对类型推理系统有所帮助.


* Q: 为什么动词 ``__conform__`` 和 ``__adapt__``?

  A: 符合, 动词不及物

  1. 在形式或性质上对应; 相似.
  2. 采取行动或达成一致或协议; 执行.
  3. 按照现行约定或模式做事.

  适应, 动词传递

  1. 为了使适合或适应特定使用或情况.

  Source:  The American Heritage Dictionary of the English
  Language, Third Edition


Backwards Compatibility
=======================

向后兼容性应该没有问题, 除非有人在其他方面使用了特殊名称 ``__conform__`` 或 ``__adapt__``,
但这似乎不太可能, 并且在任何情况下, 用户代码都不应该使用非特殊名称 -- 标准目的.

可以在不更改解释器的情况下实施和测试该提议.


Credits
=======

这个提议很大程度上是由 Python 主要邮件列表和类型信息列表上的人才反馈创建的. 提名具体的贡献者 (如果我们错过任何人就道歉!),
除了提案的作者之外: 该提案的第一个版本的主要建议来自 Paul Prescod, 得到了 Robin Thomas 的重要反馈,
我们也借用了 Marcin'Qrczak'Kowalczyk 和 Carlos Ribeiro 的想法.

其他贡献者 (通过评论) 包括 Michel Pelletier, Jeremy Hylton, Aahz Maruch, Fredrik Lundh, Rainer Deyke, Timothy Delaney,
和 Huaiyu Zhu. 目前的版本很大程度上要与 Phillip J. Eby, Guido van Rossum, Bruce Eckel, Jim Fulton 和 Ka-Ping Yee
(以及其他人) 进行讨论, 并研究和反思他们的建议, 实施和有关使用和适用于 Python 中的接口和协议.


References and Footnotes
========================

.. [1] PEP 245, Python 接口语法, Pelletier
       http://www.python.org/dev/peps/pep-0245/

.. [2] http://www.zope.org/Wikis/Interfaces/FrontPage

.. [3] http://www.artima.com/weblogs/index.jsp?blogger=guido

.. [4] http://www.artima.com/weblogs/viewpost.jsp?thread=87182

.. [5] http://peak.telecommunity.com/PyProtocols.html


Copyright
=========

This document has been placed in the public domain.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   End:  
