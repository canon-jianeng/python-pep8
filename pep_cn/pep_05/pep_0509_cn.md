
PEP: 509
Title: Add a private version to dict
Version: $Revision$
Last-Modified: $Date$
Author: Victor Stinner <victor.stinner@gmail.com>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 4-January-2016
Python-Version: 3.6


Abstract
========

在内置的 ``dict`` 类型中添加一个新的私有版本,
在每个字典创建和每个字典更改时递增,
用来在命名空间上实现快速保护.


Rationale
=========

在 Python 中, 内置的 ``dict`` 类型被许多指令使用. 例如,
``LOAD_GLOBAL`` 指令在全局命名空间或内置命名空间 (两个字典查找) 中查找变量.
Python 使用 ``dict`` 作为内置命名空间, 全局命名空间, 类型命名空间,
实例命名空间等. 本地命名空间 (函数命名空间) 通常针对数组进行优化, 但它也可以是一个字典.

Python 很难优化, 因为几乎所有东西都是可变的: 内置函数, 函数代码, 全局变量,
局部变量, ... 可以在运行时修改. 实现关于 Python 语义的优化需要检测何时 "发生了变化":
我们将这些检查称为 "guards".

优化的加速取决于防护检查的速度. 该 PEP 建议在字典中添加私有版本,
用来在命名空间上实现快速保护.

如果版本没有更改, 则可以跳过字典查找, 这是大多数命名空间的常见情况.
该版本是全局唯一的, 因此, 检查版本也足以验证命名空间字典未被新字典替换.

当字典版本没有改变时, 保护的性能不依赖于观察字典条目的数量: 复杂度为 O(1).

优化示例: 将全局变量的值复制到函数常量.
此优化需要保护全局变量来检查它是否在复制后被修改.
如果未修改全局变量, 则该函数使用缓存副本.
如果修改了全局变量, 则该函数使用常规查找,
并且还可以对该函数进行去优化 (来消除下一次函数调用的保护检查的开销).

See the `PEP 510 -- 具有守护的专门功能
<https://www.python.org/dev/peps/pep-0510/>`_
用来专门用于保护功能的守护以及 Python 静态优化器的更一般的基本原理.


Guard example
=============

使用假设的 ``dict_get_version(dict)`` 函数检查字典条目
是否被修改 (创建, 更新或删除) 的快速保护的伪代码::

    UNSET = object()

    class GuardDictKey:
        def __init__(self, dict, key):
            self.dict = dict
            self.key = key
            self.value = dict.get(key, UNSET)
            self.version = dict_get_version(dict)

        def check(self):
            """Return True if the dictionary entry did not change
            and the dictionary was not replaced."""

            # read the version of the dictionary
            version = dict_get_version(self.dict)
            if version == self.version:
                # Fast-path: dictionary lookup avoided
                return True

            # lookup in the dictionary
            value = self.dict.get(self.key, UNSET)
            if value is self.value:
                # another key was modified:
                # cache the new dictionary version
                self.version = version
                return True

            # the key was modified
            return False


Usage of the dict version
=========================

Speedup method calls
--------------------

Yury Selivanov 写了一个补丁来优化方法调用
<https://bugs.python.org/issue26110>`_.
该补丁依赖于 "ceval 中的每个操作码缓存"
<https://bugs.python.org/issue26219>`_ 补丁,
如果全局字典或内置字典已被修改, 则需要字典版本使缓存无效.

缓存还要求字典版本是全局唯一的. 可以在命名空间中定义一个函数,
并在不同的命名空间中调用它, 例如使用带有 *globals* 参数的 ``exec()``.
在这种情况下, 替换了全局字典, 并且缓存也必须无效.


Specialized functions using guards
----------------------------------

`PEP 510 -- 具有守护的专用功能
<https://www.python.org/dev/peps/pep-0510/>`_
提出了一个 API 来支持带有守护的专用功能.
它允许在不破坏 Python 语义的情况下为 Python 实现静态优化器.

`FAT Python <http://faster-cpython.readthedocs.org/fat_python.html>`_
项目的 `fatoptimizer <http://fatoptimizer.readthedocs.org/>`_
是静态 Python 优化器的一个例子. 它实现了许多需要在命名空间上进行保护的优化:

* 调用纯内置函数: 用 ``3`` 替换 ``len("abc")``,
  ``builtins.__dict__['len']`` 和 ``globals()['len']`` 是必要的
* 循环展开: 展开循环 ``for i in range(...): ...``,
  守护 ``builtins.__dict__['range']`` 和 ``globals()['range']`` 是必需的
* 等等.


Pyjion
------

根据 Pyjion 的两个主要开发人员之一 Brett Cannon 的说法,
Pyjion 可以从字典版本中受益来实现优化.

`Pyjion <https://github.com/Microsoft/Pyjion>`_ 是基于 CoreCLR
(Microsoft .NET Core 运行时) 的 Python 的 JIT 编译器.


Cython
------

Cython 可以从字典版本中受益来实现优化.

`Cython <http://cython.org/>`_ 是 Python 编程语言
和扩展的 Cython 编程语言的优化静态编译器.


Unladen Swallow
---------------

即使没有明确提到字典版本, 优化全局变量和内置查找也是
Unladen Swallow 计划的一部分: "实现几个提议的方案之一,
来加速全局变量和内置函数的查找."
(来源: `Unladen Swallow ProjectPlan
<https://code.google.com/p/unladen-swallow/wiki/ProjectPlan>`_).

Unladen Swallow 是 CPython 2.6.1 的一个分支,
添加了一个用 LLVM 实现的 JIT 编译器.
该项目于2011年停止: "Unladen Swallow Retrospective"
<http://qinsb.blogspot.com.au/2011/03/unladen-swallow-retrospective.html>`_.


Changes
=======

将 ``ma_version_tag`` 字段添加到 ``PyDictObject`` 结构中,
使用 C 类型 ``PY_UINT64_T``, 64位无符号整数. 添加全局字典版本.

每次创建字典时, 全局版本都会递增,
字典版本将初始化为全局版本.

每次修改字典内容时, 必须递增全局版本并将其复制到字典版本.
可以修改其内容的字典方法:

* ``clear()``
* ``pop(key)``
* ``popitem()``
* ``setdefault(key, value)``
* ``__delitem__(key)``
* ``__setitem__(key, value)``
* ``update(...)``

当字典方法不改变其内容时, 增加或不增加版本的选择由 Python 实现决定.
Python 实现可以决定不增加版本来避免在守护中进行字典查找.
字典方法不修改其内容的情况示例:

* ``clear()`` 如果字典已经空了
* ``pop(key)`` 如果字典已经为空, 则该键不存在
* ``popitem()`` 如果字典是空的
* ``setdefault(key, value)`` 如果键已存在
* ``__delitem__(key)`` 如果键不存在
* ``__setitem__(key, value)`` 如果新值与当前值相同
* ``update()`` 如果不带参数调用或者新值与当前值相同

将键设置为新值等于旧值也被视为修改字典内容的操作.

两个不同的空字典必须具有不同的版本才能仅通过其版本来识别字典.
它允许在保护中验证未替换命名空间而不存储对字典的强引用.
使用借用的引用不起作用: 如果旧字典被销毁, 则可能在同一内存地址分配新字典.
顺便说一下, 词典不支持弱引用.

版本增加必须是原子的. 在 CPython 中,
全局解释器锁 (GIL) 已经保护 ``dict``
方法来使更改成为原子.

使用假设的 ``dict_get_version(dict)`` 函数的例子::

    >>> d = {}
    >>> dict_get_version(d)
    100
    >>> d['key'] = 'value'
    >>> dict_get_version(d)
    101
    >>> d['key'] = 'new value'
    >>> dict_get_version(d)
    102
    >>> del d['key']
    >>> dict_get_version(d)
    103

该字段被称为 ``ma_version_tag``, 而不是 ``ma_version``,
建议使用 ``version_tag == old_version_tag`` 来比较它,
而不是 ``version <= old_version``, 它在整数溢出后变得错误.


Backwards Compatibility
=======================

由于 ``PyDictObject`` 结构不是稳定 ABI 的一部分,
而且新的字典版本没有在 Python 范围内公开, 因此更改是向后兼容的.


Implementation and Performance
==============================

`问题 #26058: PEP 509: 将 ma_version_tag 添加到 PyDictObject
<https://bugs.python.org/issue26058>`_ 包含实现此 PEP 的补丁.

在 pybench 和 timeit 微基准测试中, 补丁似乎没有在字典操作上增加任何开销.
例如, 以下时间微基准测试在更改之前和之后需要318纳秒::

    python3.6 -m timeit 'd={1: 0}; d[2]=0; d[3]=0; d[4]=0; del d[1]; del d[2]; d.clear()'

当版本没有改变时, ``PyDict_GetItem()`` 需要 14.8ns 进行字典查找,
而保护检查只需要 3.8ns. 此外, 守护可以查看多个按键. 例如,
对于在函数中使用10个全局变量进行优化, 10个字典查找的成本为 148ns,
而当版本未更改时, 守护仅花费 3.8ns (快39倍).

`fat 模块
<http://fatoptimizer.readthedocs.org/en/latest/fat.html>`_
实现这样的守护: ``fat.GuardDict`` 是基于字典版本的.


Integer overflow
================

该实现使用 C 类型 ``PY_UINT64_T`` 来存储版本: 64位无符号整数.
C 代码使用 ``version++``. 在整数溢出时,
根据 C 标准将版本包装为 "0" (然后继续递增).

在整数溢出之后, 守护可以成功, 而被监视的字典键被修改.
这个错误只发生在一个守护检查, 如果有自上次守护检查以来的
``2 ** 64`` 字典创建或修改.

如果字典每纳秒修改一次, ``2 ** 64`` 的修改时间超过584年.
使用32位版本, 只需4秒.
这就是为什么64位无符号类型也用于32位系统的原因.
在 C 级别进行字典查找需要 14.8ns.

每584年就有一个漏洞的风险是可以接受的.


Alternatives
============

Expose the version at Python level as a read-only __version__ property
----------------------------------------------------------------------

PEP 的第一个版本建议在 Python 级别将字典版本公开为只读 ``__version__`` 属性,
并将属性添加到 ``collections.UserDict`` (因为这种类型必须模仿 ``dict`` API).

有很多问题:

* 为了保持一致并避免出现意外情况, 必须将版本添加到所有映射类型中.
  实现新的映射类型需要额外的工作, 没有任何好处,
  因为版本只在实践中的 ``dict`` 类型上需要.

* 所有 Python 实现都必须实现这个新属性,
  它为其他实现提供了更多的工作, 而它们可能根本不使用字典版本.

* 在 Python 级别公开字典版本会导致对性能的错误假设.
  在 Python 级别检查 ``dict.__version__`` 并不比字典查找快.
  Python 中的字典查找成本为 48.7ns, 检查版本的成本为 47.5ns,
  差异仅为 1.2ns (3%)::


    $ python3.6 -m timeit -s 'd = {str(i):i for i in range(100)}' 'd["33"] == 33'
    10000000 loops, best of 3: 0.0487 usec per loop
    $ python3.6 -m timeit -s 'd = {str(i):i for i in range(100)}' 'd.__version__ == 100'
    10000000 loops, best of 3: 0.0475 usec per loop

* ``__version__`` 可以包含在整数溢出上. 这很容易出错:
  使用 ``dict.__version__ = guard_version`` 是错误的,
  必须使用 ``dict.__version__ == guard_version`` 来降低整数溢出时出错的风险
  (在实践中即使整数溢出的可能性不大).

在属性名上, 强制性的 bikeshedding:

* ``__cache_token__``: 由 Nick Coghlan 提出的名字,
  名字来自 `abc.get_cache_token()
  <https://docs.python.org/3/library/abc.html#abc.get_cache_token>`_.
* ``__version__``
* ``__version_tag__``
* ``__timestamp__``


Add a version to each dict entry
--------------------------------

每个字典的单个版本需要保持对值的强引用,
这可以使值保持活动的时间长于预期.
如果我们还为每个字典条目添加一个版本,
则 guard 只能存储条目版本 (一个简单的整数)
来避免对该值的强引用: 只需要对字典和键的强引用.

更改: 在 ``PyDictKeyEntry`` 结构中添加一个 ``me_version_tag`` 字段,
该字段具有 C 类型 ``PY_UINT64_T``. 创建或修改键时, 条目版本设置为字典版本,
在任何更改时递增 (创建, 修改, 删除).

使用假设的 ``dict_get_version(dict)`` 和 ``dict_get_entry_version(dict)``
函数检查字典键是否被修改的快速保护的伪代码::

    UNSET = object()

    class GuardDictKey:
        def __init__(self, dict, key):
            self.dict = dict
            self.key = key
            self.dict_version = dict_get_version(dict)
            self.entry_version = dict_get_entry_version(dict, key)

        def check(self):
            """Return True if the dictionary entry did not change
            and the dictionary was not replaced."""

            # read the version of the dictionary
            dict_version = dict_get_version(self.dict)
            if dict_version == self.version:
                # Fast-path: dictionary lookup avoided
                return True

            # lookup in the dictionary to read the entry version
            entry_version = get_dict_key_version(dict, key)
            if entry_version == self.entry_version:
                # another key was modified:
                # cache the new dictionary version
                self.dict_version = dict_version
                self.entry_version = entry_version
                return True

            # the key was modified
            return False

此选项的主要缺点是对内存占用的影响. 它增加了每个字典条目的大小,
因此开销取决于 buckets 的数量 (字典条目, 使用或未使用).
例如, 它在64位系统上将每个字典条目的大小增加了8个字节.

在 Python 中, 内存占用很重要, 趋势是减少它.
例子:

* `PEP 393 -- 灵活的字符串表示
  <https://www.python.org/dev/peps/pep-0393/>`_
* `PEP 412 -- 键共享字典
  <https://www.python.org/dev/peps/pep-0412/>`_


Add a new dict subtype
----------------------

添加一个新的 ``verdict`` 类型, ``dict`` 的子类型.
当需要守护时, 使用名称空间 (模块命名空间, 类型命名空间,
实例命名空间等) 的 ``verdict`` 而不是 ``dict``.

保持 ``dict`` 类型不变, 利于在不使用守护时不添加任何开销 (CPU, 内存占用).

技术问题: 野外的很多 C 代码, 包括 CPython 核心,
都期望精确的``dict``类型。问题:

* ``exec()`` 需要全局和本地的 ``dict``. 很多代码都使用 ``globals={}``.
  不可能将 ``dict`` 强制转换为 ``dict`` 子类型,
  因为调用者希望修改 ``globals`` 参数 (``dict`` 是可变的).

* C 函数直接调用 ``PyDict_xxx()`` 函数, 而不是调用 ``PyObject_xxx()``,
  如果对象是 ``dict`` 子类型

* ``PyDict_CheckExact()`` 检查 ``dict`` 子类型失败,
  而有些函数需要精确的 ``dict`` 类型.

* ``Python/ceval.c`` 并不完全支持命名空间的字典子类型


``exec()`` 问题是一个阻塞问题.

其他问题:

* 垃圾收集器有一个特殊的代码来 "跟踪" ``dict`` 实例.
  如果 ``dict`` 子类型用于名称空间, 则垃圾收集器可能无法破坏某些引用循环.

* 有些函数有一个 ``dict`` 的快速路径, 它不会被用于 ``dict`` 子类型,
  所以它会让 Python 变慢一点.


Prior Art
=========

Method cache and type version tag
---------------------------------

2007年, Armin Rigo 编写了一个补丁来实现方法缓存. 它被合并到 Python 2.6 中.
补丁为类型添加 "类型属性缓存版本标记" (``tp_version_tag``)
和 "有效版本标记" 标记 (``PyTypeObject`` 结构).

在 Python 级别中类型版本标记未公开.

版本标签的 C 类型为 ``unsigned int``. 缓存是4096个条目的全局哈希表,
由所有类型共享. 缓存是全局的, "使其快速, 具有确定性和低内存占用, 并且容易失效".
每个缓存条目都有一个版本标记. 全局版本标记用于创建下一个版本标记,
它还具有 C 类型 ``unsigned int``.

默认情况下, 类型的 "有效版本标记" 标志已清除, 表示版本标记无效.
缓存该类型的第一个方法时, 将设置版本标记和 "有效版本标记" 标记.
修改类型时, 将清除该类型及其子类的 "有效版本标记" 标志.
稍后, 当使用这些类型的缓存条目时, 将删除该条目, 因为其版本标记已过时.

在整数溢出时, 清除整个缓存并将全局版本标记重置为 "0".

请参阅 `方法缓存 (问题 #1685986)
<https://bugs.python.org/issue1685986>`_
和 `针对 Python 2.6 更新了 Armin 的方法缓存优化 (问题 #1700288)
<https://bugs.python.org/issue1700288>`_.


Globals / builtins cache
------------------------

2010年, Antoine Pitrou 提出了一个 `Globals/builtins 缓存 (问题 #10401)
<http://bugs.python.org/issue10401>`_,
它为 ``PyDictObject`` 结构添加了一个私有的 ``ma_version`` 字段 (``dict`` 类型),
该字段具有 C 类型 ``Py_ssize_t``.

补丁为函数和框架添加了 "全局和内置缓存",
并更改了 ``LOAD_GLOBAL`` 和 ``STORE_GLOBAL`` 指令来使用缓存.

``PyDictObject`` 结构的变化与此 PEP 非常相似.


Cached globals+builtins lookup
------------------------------

2006年, Andrea Griffini 提出了一个实现
`Cached globals + builtins 查找优化的补丁
<https://bugs.python.org/issue1616125>`_.
补丁为 ``PyDictObject`` 结构 (``dict`` 类型)
添加了一个私有的 ``timestamp`` 字段, 该字段具有 C 类型 ``size_t``

python-dev 上的线程: `关于字典查找缓存
<https://mail.python.org/pipermail/python-dev/2006-December/070348.html>`_
(December 2006).


Guard against changing dict during iteration
--------------------------------------------

2013年, Serhiy Storchaka 提出 "防止在迭代期间改变字典 (问题 #19332)
<https://bugs.python.org/issue19332>`_,
它将 ``ma_count`` 字段添加到 ``PyDictObject`` 结构中 (``dict`` 类型),
该字段具有 C 类型 ``size_t``. 修改字典时, 此字段会递增.


PySizer
-------

`PySizer <http://pysizer.8325.org/>`_: 一个用于 Python 的内存分析器,
由 Nick Smallbone 开发的代码2005项目的 Google Summer.

这个项目有一个 CPython 2.4 的补丁, 它将 ``key_time`` 和 ``value_time``
字段添加到字典条目中. 它使用全局进程范围的字典计数器,
每次修改字典时都会增加. 时间用于决定子对象何时首次出现在其父对象中.


Discussion
==========

邮件列表上的线程:

* python-dev: `更新 PEP 509
  <https://mail.python.org/pipermail/python-dev/2016-April/144250.html>`_
* python-dev: `RFC: PEP 509: 添加私有版本到字典
  <https://mail.python.org/pipermail/python-dev/2016-April/144137.html>`_
* python-dev: `PEP 509: 添加私有版本到字典
  <https://mail.python.org/pipermail/python-dev/2016-January/142685.html>`_
  (january 2016)
* python-ideas: `RFC: PEP: 添加 dict.__version__
  <https://mail.python.org/pipermail/python-ideas/2016-January/037702.html>`_
  (january 2016)


Acceptance
==========

PEP 在 2016-09-07 被 Guido van Rossum 接受
<https://mail.python.org/pipermail/python-dev/2016-September/146298.html>`_.
此后, PEP 实现已提交到存储库.


Copyright
=========

This document has been placed in the public domain.
