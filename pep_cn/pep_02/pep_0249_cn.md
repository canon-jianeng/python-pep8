
PEP: 249
Title: Python Database API Specification v2.0
Version: $Revision$
Last-Modified: $Date$
Author: mal@lemburg.com (Marc-André Lemburg)
Discussions-To: db-sig@python.org
Status: Final
Type: Informational
Content-Type: text/x-rst
Created:
Post-History:
Replaces: 248


`Introduction`_
===============

已定义此 API 来鼓励用于访问数据库的 Python 模块之间的相似性. 通过这样做,
我们希望实现兼容性, 从而实现更容易理解的模块, 通常在数据库中更容易移植的代码,
以及 Python 更广泛的数据库连接范围.

有关此规范的评论和问题可以针对
`SIG for Database Interfacing with Python <db-sig@python.org>`__.

有关数据库与 Python 和可用软件包的接口的更多信息, 请参阅 "数据库主题指南"
<http://www.python.org/topics/database/>`__.

本文档描述了 Python Database API Specification 2.0 和一组常见的可选扩展.
以前的版本 1.0 版本仍可作为参考, 在 :PEP: `248`. 鼓励包编写者使用此版本的规范作为新接口的基础.


`Module Interface`_
===================

`Constructors`_
---------------

通过连接对象可以访问数据库. 该模块必须为这些提供以下构造函数:

.. _connect:

`connect`_\ ( *parameters...* )
    用于创建与数据库的连接的构造函数.

    返回 Connection_ 对象. 它需要许多与数据库相关的参数. [1]_


`Globals`_
----------

必须定义这些模块全局变量:

.. _apilevel:

`apilevel`_
    字符串常量, 表示支持的 DB API 级别.

    目前只允许使用字符串 "``1.0``" 和 "``2.0``".
	如果没有指定, 应该假设一个 DB-API 1.0 级接口.


.. _threadsafety:

`threadsafety`_
    整数常量表示接口支持的线程安全级别. 可能的值是:

    ============ =======================================================
    threadsafety Meaning
    ============ =======================================================
               0 Threads may not share the module.
               1 Threads may share the module, but not connections.
               2 Threads may share the module and connections.
               3 Threads may share the module, connections and cursors.
    ============ =======================================================

    在上面的上下文中共享意味着两个线程可以使用资源而不使用互斥信号量来包装它来实现资源锁.
	请注意, 通过使用互斥锁管理访问权限, 您无法始终使外部资源保持线程安全:
	资源可能依赖于全局变量或您无法控制的其他外部源.


.. _paramstyle:

`paramstyle`_
    字符串常量, 说明接口所需的参数标记格式的类型. 可能的值是 [2]_:

    ============ ==============================================================
    paramstyle   Meaning
    ============ ==============================================================
    ``qmark``    Question mark style, e.g. ``...WHERE name=?``
    ``numeric``  Numeric, positional style, e.g. ``...WHERE name=:1``
    ``named``    Named style, e.g. ``...WHERE name=:name``
    ``format``   ANSI C printf format codes, e.g. ``...WHERE name=%s``
    ``pyformat`` Python extended format codes, e.g.  ``...WHERE name=%(name)s``
    ============ ==============================================================


`Exceptions`_
-------------

该模块应该通过这些例外或子类提供所有错误信息:

.. _Warning:

`Warning`_
    插入时数据截断等重要警告引发异常等.
	它必须是 Python "StandardError" 的子类
	(在模块异常中定义).


.. _Error:

`Error`_
    异常是所有其他错误异常的基类. 你可以使用它来用一个 ``except`` 语句来捕获所有错误.
	警告不被视为错误, 因此不应该使用此类作为基础. 它必须是 Python "StandardError" 的子类
	(在模块异常中定义).


.. _InterfaceError:

`InterfaceError`_
    针对与数据库接口而非数据库本身相关的错误引发异常.
	它必须是 Error_ 的子类.


.. _DatabaseError:

`DatabaseError`_
    针对与数据库相关的错误引发异常.
	它必须是 Error_ 的子类.


.. _DataError:

`DataError`_
    由于处理数据问题导致的错误引发异常, 例如除以零,
	数值超出范围等. 它必须是 DatabaseError_ 的子类.


.. _OperationalError:

`OperationalError`_
    针对与数据库操作相关的错误而引发的异常, 并且不一定在程序员的控制之下,
	例如, 发生意外断开连接, 找不到数据源名称, 无法处理事务, 处理期间发生内存分配错误等.
	它必须是 DatabaseError_ 的子类.


.. _IntegrityError:

`IntegrityError`_
    当数据库的关系完整性受到影响时引发异常, 例如: 外键检查失败.
	它必须是 DatabaseError_ 的子类.


.. _InternalError:

`InternalError`_
    数据库遇到内部错误时引发异常, 例如: 游标不再有效,
	事务不同步等等. 它必须是 DatabaseError_ 的子类.


.. _ProgrammingError:

`ProgrammingError`_
    针对编程错误引发的异常, 例如: 表未找到或已存在,
	SQL 语句中的语法错误, 指定的参数数量错误等等.
	它必须是 DatabaseError_ 的子类.


.. _NotSupportedError:

`NotSupportedError`_
    在使用数据库不支持的方法或数据库 API 的情况下引发异常, 例如,
	在不支持事务或关闭事务的连接上请求 `.rollback()`_.
	它必须是 DatabaseError_ 的子类.

这是继承布局的异常::

    StandardError
    |__Warning
    |__Error
       |__InterfaceError
       |__DatabaseError
          |__DataError
          |__OperationalError
          |__IntegrityError
          |__InternalError
          |__ProgrammingError
          |__NotSupportedError

.. Note::
    未定义这些异常的值. 但是, 他们应该让用户对错误的原因有一个很好的了解.


.. _Connection:

`Connection Objects`_
=====================

连接对象应该响应以下方法.


`Connection methods`_
---------------------

.. .close():
.. _Connection.close:

`.close() <#Connection.close>`_
    现在关闭连接 (而不是每当调用 ``.__del__()`` 时).

    从这一点开始, 连接将无法使用; 如果尝试对连接进行任何操作,
	将引发 Error_ (或子类) 异常. 这同样适用于尝试使用连接的所有游标对象.
	请注意, 在不先提交更改的情况下关闭连接将导致执行隐式回滚.


.. _.commit:
.. _.commit():

`.commit`_\ ()
    将任何挂起的事务提交到数据库.

    请注意, 如果数据库支持自动提交功能, 则必须先关闭此功能.
	可以提供接口方法来将其重新打开.

    不支持事务的数据库模块应该使用 void 功能实现此方法.


.. _.rollback:
.. _.rollback():

`.rollback`_\ ()
    此方法是可选的, 因为并非所有数据库都提供事务支持. [3]_

    如果数据库确实提供了事务, 则此方法会导致数据库回滚到任何挂起事务的开头.
	在不提交更改的情况下关闭连接将导致执行隐式回滚.


.. _.cursor:

`.cursor`_\ ()
    使用连接返回一个新的 Cursor_ 对象.

    如果数据库未提供直接游标概念,
	则模块必须使用此规范所需的其他方式模拟游标.  [4]_



.. _Cursor:

`Cursor Objects`_
=================

这些对象表示数据库游标, 用于管理获取操作的上下文. 从同一连接创建的游标不是孤立的,
*即*, 游标对数据库所做的任何更改都会被其他游标立即看到. 根据事务支持的实现方式,
可以或不可以隔离从不同连接创建的游标 (另请参阅连接的 `.rollback`_\ () 和 `.commit`_\ () 方法).

游标对象应该响应以下方法和属性.


`Cursor attributes`_
--------------------

.. _.description:

`.description`_
    该只读属性是7项序列的一个序列.

    这些序列中的每一个包含描述一个结果列的信息:

    * ``name``
    * ``type_code``
    * ``display_size``
    * ``internal_size``
    * ``precision``
    * ``scale``
    * ``null_ok``

    前两项 (``name`` 和 ``type_code``) 是强制性的, 另外五项是可选的,
	如果不能提供有意义的值, 则设置为 ``None``.

    对于不返回行的操作, 或者如果游标尚未通过 `.execute*()`_ 方法调用操作,
	该属性将为 ``None``.

    可以通过将 ``type_code`` 与下面部分中指定的 `Type Objects`_ 进行比较来解释它.


.. _.rowcount:

`.rowcount`_
    这个只读属性指定最后一个 `.execute*()`_ 生成的行数 (对于像 ``SELECT`` 这样的 DQL 语句)
	或受影响的行 (对于像 ``UPDATE`` 或 ``INSERT`` 这样的 DML 语句). [9]_

    如果没有对游标执行 `.execute*()`_, 或者上一次操作的 rowcount 无法通过接口确定,
	则该属性为 -1. [7]_

    .. note::
        DB API 规范的未来版本可以重新定义后一种情况,
		使对象返回 ``None`` 而不是 -1.


`Cursor methods`_
-----------------

.. _.callproc:
.. _.callproc():

`.callproc`_\ ( *procname* [, *parameters* ] )
    (此方法是可选的, 因为并非所有数据库都提供存储过程. [3]_)

    使用指定名称调用存储的数据库过程. 参数序列必须包含过程所需的每个参数的一个项.
	调用的结果作为输入序列的修改副本返回. 保持输入参数不变,
	output 和 input/output 参数替换为可能的新值.

    该过程还可以提供结果集作为输出.
	然后必须通过标准的 `.fetch*()`_ 方法使其可用.


.. .close:
.. _Cursor.close:
.. _Cursor.close():

`.close <#Cursor.close>`_\ ()
    现在关闭光标 (而不是每次调用 ``__del__``).

    从这一点开始, 光标将无法使用;
	如果使用游标尝试任何操作, 将引发 Error_ (或子类) 异常.


.. _.execute*:
.. _.execute*():

.. _.execute:
.. _.execute():

`.execute`_\ (*operation* [, *parameters*])
    准备并执行数据库操作 (查询或命令).

    参数可以作为序列或映射提供, 并且将绑定到操作中的变量.
	变量以特定于数据库的表示法指定
	(有关详细信息, 请参阅模块的 paramstyle_ 属性). [5]_

    光标将保留对操作的引用. 如果再次传入相同的操作对象, 则游标可以优化其行为.
	这对于使用相同操作的算法最有效, 但是不同的参数绑定到它 (很多次).

    为了在重用操作时获得最大效率, 最好使用 `.setinputsizes()`_ 方法提前指定参数类型和大小.
	参数与预定义信息不匹配是合法的; 实施应该补偿, 可能会导致效率损失.

    参数也可以被指定为元组中的列表, 例如, 在单个操作中插入多行,
	但不推荐使用这种用法: `.executemany()`_ 应该用来代替.

    返回值未定义.


.. _.executemany:
.. _.executemany():

`.executemany`_\ ( *operation*, *seq_of_parameters* )
    准备数据库操作 (查询或命令), 然后针对序列中找到的
	所有参数序列或映射执行它 * seq_of_parameters*.

    模块可以使用多次调用 `.execute()`_ 方法或使用数组操作来实现此方法,
	用来使数据库在一次调用中整体处理序列.

    将此方法用于生成一个或多个结果集的操作构成未定义的行为,
	并且当检测到通过调用操作创建了结果集时, 允许 (但不是必需) 实现引发异常.

    与 `.execute()`_ 相同的注释也适用于此方法.

    返回值未定义.


.. _.fetch*:
.. _.fetch*():

.. _.fetchone:
.. _.fetchone():

`.fetchone`_\ ()
    获取查询结果集的下一行, 返回单个序列,
	或者在没有更多数据可用时返回 ``None``. [6]_

    如果先前对 `.execute*()`_ 的调用没有产生任何结果集或者还没有发出调用,
	则引发 Error_ (或子类) 异常.


.. _.fetchmany:
.. _.fetchmany():

`.fetchmany`_\ ([*size=cursor.arraysize*])
    获取查询结果的下一组行, 返回序列中的一个序列 (例如, 元组中的一个列表).
	当没有更多行可用时, 返回空序列.

    每次调用要获取的行数由参数指定. 如果未给出, 则游标的 arraysize 确定要获取的行数.
	该方法应该尝试获取 size 参数指示的行数. 如果由于指定的行数不可用而无法执行此操作,
	则可能返回的行数较少.

    如果先前对 `.execute*()`_ 的调用没有产生任何结果集或者还没有发出调用,
	则引发 Error_ (或子类) 异常.

    请注意, *size* 参数涉及性能注意事项. 为获得最佳性能, 通常最好使用 `.arraysize`_ 属性.
	如果使用 size 参数, 那么最好是从一个 `.fetchmany()`_ 调用到下一个保持相同的值.


.. _.fetchall:
.. _.fetchall():

`.fetchall`_\ ()
    获取查询结果的所有 (剩余) 行, 将它们作为序列中的一个序列 (例如, 元组中的一个列表) 返回.
	请注意, 游标的 arraysize 属性可能会影响此操作的性能.

    如果先前对 `.execute*()`_ 的调用没有产生任何结果集或者还没有发出调用,
	则引发 Error_ (或子类) 异常.


.. _.nextset:
.. _.nextset():

`.nextset`_\ ()
    (此方法是可选的, 因为并非所有数据库都支持多个结果集. [3]_)

    此方法将使光标跳到下一个可用集, 从当前集中丢弃任何剩余行.

    如果没有更多集合, 则该方法返回 ``None``. 否则, 它返回一个真值,
	随后对 `.fetch*()`_ 方法的调用将返回下一个结果集中的行.

    如果先前对 `.execute*()`_ 的调用没有产生任何结果集或者还没有发出调用,
	则引发 Error_ (或子类) 异常.


.. _.arraysize:

`.arraysize`_
    此读或写属性使用 `.fetchmany()`_ 指定一次获取的行数.
	默认为1表示一次获取一行.

    实现必须遵循关于 `.fetchmany()`_ 方法的这个值,
	但是可以一次一行地与数据库进行交互.
	它也可以用于 `.executemany()`_ 的实现.


.. _.setinputsizes:
.. _.setinputsizes():

`.setinputsizes`_\ (*sizes*)
    这可以在调用 `.execute*()`_ 之前使用,
	来预定义操作参数的内存区域.

    *sizes* 被指定为一个序列 -- 每个输入参数一个项.
	该项应该是与将要使用的输入对应的 Type 对象,
	或者应该是指定字符串参数的最大长度的整数.
	如果该项为 ``None``, 则不会为该列保留预定义的内存区域
	(这对于避免大输入的预定义区域很有用).

    在调用 `.execute*()`_ 方法之前将使用此方法.

    实现可以自由地使用此方法, 用户可以不使用它.


.. _.setoutputsize:
.. _.setoutputsize():

`.setoutputsize`_\ (*size* [, *column*])
    为大量列的提取设置列缓冲区大小 (例如, ``LONG``\s, ``BLOB``\s 等).
	该列被指定为结果序列的索引. 不指定列将为游标中的所有大量列设置默认大小.

    在调用 `.execute*()`_ 方法之前将使用此方法.

    实现可以自由地使用此方法, 用户可以不使用它.


.. _Type Objects:

`Type Objects and Constructors`_
================================

许多数据库需要以特定格式输入来绑定到操作的输入参数. 例如, 如果输入是预定的 ``DATE`` 列,
那么它必须以特定的字符串格式绑定到数据库. ``Row ID`` 列或大型二进制项 (例如, blob 或 ``RAW`` 列)
存在类似的问题. 这给 Python 带来了问题, 因为 `.execute*()`_ 方法的参数是无类型的.
当数据库模块看到一个 Python 字符串对象时, 它不知道它是应该绑定为一个简单的 ``CHAR`` 列,
作为一个原始的 ``BINARY`` 项, 还是一个 ``DATE``.

为了克服这个问题, 模块必须提供下面定义的构造函数来创建可以保存特殊值的对象.
传递给游标方法时, 模块可以检测输入参数的正确类型并相应地绑定它.

Cursor_ 对象的 description 属性返回有关查询的每个结果列的信息.
``type_code`` 必须等于下面定义的 Type 对象之一. 类型对象可能等于多个类型代码
(例如, ``DATETIME`` 可能等于日期, 时间和时间戳列的类型代码; 有关详细信息, 请参阅下面的 `实现提示`_）.

该模块导出以下构造函数和单例:

.. _Date:

`Date`_\ (*year*, *month*, *day*)
    此函数构造一个包含日期值的对象.


.. _Time:

`Time`_\ (*hour*, *minute*, *second*)
    此函数构造一个包含时间值的对象.


.. _Timestamp:

`Timestamp`_\ (*year*, *month*, *day*, *hour*, *minute*, *second*)
    此函数构造一个包含时间戳值的对象.


.. _DateFromTicks:

`DateFromTicks`_\ (*ticks*)
    此函数构造一个对象, 其中包含给定 ticks 值的日期值 (自纪元以来的秒数;
	请参阅标准 Python 时间模块的文档 <http://docs.python.org/library/time.html>` __ 详情).

.. _TimeFromTicks:

`TimeFromTicks`_\ (*ticks*)
    此函数构造一个对象, 其中包含给定 ticks 值的时间值
	(自纪元以来的秒数; 有关详细信息, 请参阅标准 Python 时间模块的文档).


.. _TimeStampFromTicks:

`TimestampFromTicks`_\ (*ticks*)
    此函数构造一个对象, 其中包含给定 ticks 值的时间戳值
	(自纪元以来的秒数; 有关详细信息, 请参阅标准 Python 时间模块的文档).


.. _Binary:

`Binary`_\ (*string*)
    此函数构造一个能够保存二进制 (长) 字符串值的对象.


.. _STRING:

`STRING`_ type
    此类型对象用于描述数据库中基于字符串的列 (例如, ``CHAR``).


.. _Binary type:

`BINARY`_ type
    这个类型对象用于描述数据库中的 (长) 二进制列
	(例如, ``LONG``, ``RAW``, ``BLOB``\s).


.. _NUMBER:

`NUMBER`_ type
    此类型对象用于描述数据库中的数字列.


.. _DATETIME:

`DATETIME`_ type
    此类型对象用于描述数据库中的日期或时间列.

.. _ROWID:

`ROWID`_ type
    此类型对象用于描述数据库中的 "Row ID" 列.


SQL ``NULL`` 值由输入和输出上的 Python ``None`` 单例表示.

.. Note::
    使用 Unix ticks 进行数据库接口可能会因为它们所涵盖的日期范围有限而导致麻烦.



.. _Implementation Hints:

`Implementation Hints for Module Authors`_
==========================================

* 日期或时间对象可以实现为 `Python datetime module <http://docs.python.org/library/datetime.html>`__ 对象
   (自 Python 2.3 以来可用, 自 2.4 以来的 C API) 或使用 `mxDateTime
   <http://www.egenix.com/products/python/mxBase/mxDateTime/>`_ 包 (适用于 1.5.2 以来的所有 Python 版本).
   它们都提供 Python 和 C 级别的所有必要的构造函数和方法.

* 下面是基于 Unix ticks 的构造函数的示例实现,
   用于将工作委托给泛型构造函数的日期或时间::

        import time

        def DateFromTicks(ticks):
            return Date(*time.localtime(ticks)[:3])

        def TimeFromTicks(ticks):
            return Time(*time.localtime(ticks)[3:6])

        def TimestampFromTicks(ticks):
            return Timestamp(*time.localtime(ticks)[:6])

* 二进制对象的首选对象类型是从 1.5.2 版开始的标准 Python 中可用的缓冲区类型.
  有关详细信息, 请参阅 Python 文档. 有关 C 接口的信息, 请查看 Python 源代码分发中的
  ``Include/bufferobject.h`` 和 ``objects/bufferobject.c``.

* 此 Python 类允许实现上述类型对象,
  即使描述类型代码字段为 on 类型对象生成多个值::

        class DBAPITypeObject:
            def __init__(self,*values):
                self.values = values
            def __cmp__(self,other):
                if other in self.values:
                    return 0
                if other < self.values:
                    return 1
                else:
                    return -1

  生成的类型对象比较等于传递给构造函数的所有值.

* 下面是一段 Python 代码, 它实现了上面定义的异常层次结构::

        import exceptions

        class Error(exceptions.StandardError):
            pass

        class Warning(exceptions.StandardError):
            pass

        class InterfaceError(Error):
            pass

        class DatabaseError(Error):
            pass

        class InternalError(DatabaseError):
            pass

        class OperationalError(DatabaseError):
            pass

        class ProgrammingError(DatabaseError):
            pass

        class IntegrityError(DatabaseError):
            pass

        class DataError(DatabaseError):
            pass

        class NotSupportedError(DatabaseError):
            pass

  在 C 中, 您可以使用 ``PyErr_NewException(fullname, base, NULL)`` API 来创建异常对象.


`Optional DB API Extensions`_
=============================

在 DB API 2.0 的生命周期中, 模块作者经常将其实现扩展到此 DB API 规范所要求的范围之外.
为了增强兼容性并为可能的未来版本规范提供干净的升级路径, 本节定义了一组核心 DB API 2.0 规范的通用扩展.

与所有 DB API 可选功能一样, 数据库模块作者可以自由地不实现这些附加属性和方法
(使用它们将导致 ``AttributeError``) 或引发 NotSupportedError_ 来防止只能检查可用性运行.

已经提出通过 Python 警告框架发出 Python 警告, 使程序员可以选择使用这些扩展.
要使此功能有用, 必须对警告消息进行标准化, 利于能够屏蔽它们. 这些标准消息在下面称为 *警告消息*.


.. _.rownumber:

Cursor\ `.rownumber`_
    如果无法确定索引, 则此只读属性应该提供结果集中游标的当前基于 0 的索引或 ``None``.

    索引可以看作序列中光标的索引 (结果集). 下一个提取操作将获取该序列中由 `.rownumber`_ 索引的行.

    *Warning Message:* "DB-API 扩展 cursor.rownumber 使用"


.. _Connection.Error:
.. _Connection.ProgrammingError:

`Connection.Error`_, `Connection.ProgrammingError`_, etc.
    DB API 标准定义的所有异常类都应该作为属性公开在 Connection_ 对象上
	(除了在模块范围内可用).

    这些属性简化了多连接环境中的错误处理.

    *Warning Message:* "DB-API 扩展 connection.<exception> 使用"


.. _.connection:

Cursor\ `.connection`_
    此只读属性返回对创建游标的 Connection_ 对象的引用.

    该属性简化了在多连接环境中编写多态代码的过程.

    *Warning Message:* "DB-API 扩展 cursor.connection 使用"


.. _.scroll:
.. _.scroll():

Cursor\ `.scroll`_\ (*value* [, *mode='relative'* ])
    根据 *模式* 将结果集中的光标滚动到新位置.

    如果 mode 是 ``relative`` (默认值), 则将值作为结果集中当前位置的偏移量,
	如果设置为 "绝对值", 则表示绝对目标位置.

    如果滚动操作将离开结果集, 则应该引发 ``IndexError``.
	在这种情况下, 光标位置未定义 (理想情况下根本不移动光标).

    .. Note::
        此方法应该使用本机可滚动游标 (如果可用), 或者还原为仅向前滚动游标的模拟.
		该方法可能引发 NotSupportedError_ 来指示数据库不支持特定操作
		(例如, 向后滚动).

    *Warning Message:* "DB-API 扩展 cursor.scroll() 使用"


.. _Cursor.messages:

`Cursor.messages`_
    这是一个 Python 列表对象的界面附加的,
	接口从底层数据库接收用于该光标的所有消息元组 (异常类, 异常值).

    列表由所有标准游标方法调用 (在执行调用之前) 清除,
	除了 `.fetch*()`_ 自动调用来避免过多的内存使用,
	也可以通过执行 ``del cursor.messages[: ]``.

    数据库生成的所有错误和警告消息都放在此列表中,
	因此, 检查列表允许用户验证方法调用的正确操作.

    此属性的目的是消除对经常导致问题的警告异常的需要
	(某些警告实际上只有信息字符).

    *Warning Message:* "DB-API 扩展 cursor.messages 使用"


.. _Connection.messages:

`Connection.messages`_
    与 Cursor.messages_ 相同,
	但列表中的消息是面向连接的.

    所有标准连接方法调用 (在执行调用之前) 都会自动清除该列表,
	来避免过多的内存使用, 也可以通过执行 ``del connection.messages[: ]`` 来清除它.

    *Warning Message:* "DB-API 扩展 connection.messages 使用"


.. _.next:
.. _.next():

Cursor\ `.next`_\ ()
    使用与 `.fetchone()`_ 相同的语义从当前执行的 SQL 语句返回下一行.
	当 Python 版本 2.2 及更高版本的结果集用尽时, 会引发 ``StopIteration`` 异常.
	以前的版本没有 ``StopIteration`` 异常, 因此, 该方法应该引发一个 ``IndexError``.

    *Warning Message:* "DB-API 扩展 cursor.next() 使用"


.. _.__iter__:
.. _.__iter__():

Cursor\ `.__iter__`_\ ()
    返回 self 来使游标与迭代协议兼容 [8]_.

    *Warning Message:* "DB-API 扩展 cursor.__iter__() 使用"


.. _.lastrowid:

Cursor\ `.lastrowid`_
    
277/5000
	此只读属性提供最后一个修改行的 rowid (大多数数据库仅在执行单个 ``INSERT`` 操作时返回 rowid).
	如果操作未设置 rowid 或数据库不支持 rowid, 则此属性应该设置为 ``None``.

    如果最后执行的语句修改了多行, 例如 ``.lastrowid`` 的语义是未定义的.
	当 ``INSERT`` 与 ``.executemany()`` 一起使用时.

    *Warning Message:* "DB-API 扩展 cursor.lastrowid 使用"


`Optional Error Handling Extensions`_
=====================================

核心 DB API 规范仅引入了一组异常, 可以引发这些异常来向用户报告错误.
在某些情况下, 异常可能对程序流程造成太大破坏, 甚至无法执行.

对于这些情况, 为了简化处理数据库时的错误处理,
数据库模块作者可能会选择实现用户可定义的错误处理程序.
本节介绍定义这些错误处理程序的标准方法.

.. _Connection.errorhandler:
.. _Cursor.errorhandler:

`Connection.errorhandler`_, `Cursor.errorhandler`_
    读或写属性, 它引用了一个错误处理程序, 利于在满足错误条件时调用.

    处理程序必须是 Python 可调用的, 具有以下参数:

    .. parsed-literal::

        errorhandler(*connection*, *cursor*, *errorclass*, *errorvalue*)

    其中 connection 是对游标操作的连接的引用, cursor 是对游标的引用
	(如果错误不适用于游标, 则为 ``None``), *errorclass* 是一个错误类,
	用于实例化 *errorvalue* 作为构造参数.

    标准错误处理程序应该将错误信息添加到相应的 ``.messages`` 属性
	(`Connection.messages`_ 或 `Cursor.messages`_)
	并引发由给定的 *errorclass* 和 *errorvalue* 参数定义的异常.

    如果没有设置 ``.errorhandler`` (属性为 ``None``),
	则应该应用上面列出的标准错误处理方案.

    *Warning Message:* "DB-API 扩展 .errorhandler 使用"

游标应该在创建游标时从其连接对象继承 ``.errorhandler`` 设置.


`Optional Two-Phase Commit Extensions`_
=======================================

许多数据库都支持两阶段提交 (TPC),
它允许跨多个数据库连接和其他资源管理事务.

如果数据库后台提供对两阶段提交的支持, 并且数据库模块作者希望公开此支持,
则应该实现以下 API. 如果只能在运行时检查数据库后台对两阶段提交的支持,
则应该引发 NotSupportedError_.

`TPC Transaction IDs`_
----------------------

由于许多数据库遵循 XA 规范, 因此事务 ID 由三个组件组成:

* a format ID
* a global transaction ID
* a branch qualifier

对于特定的全局事务, 前两个组件对于所有资源应该是相同的.
应该为全局事务中的每个资源分配不同的分支限定符.

各种组件必须满足以下标准:

* format ID: 非负32位整数.

* global transaction ID and branch qualifier: 字节串不超过64个字符.

使用 `.xid()`_ Connection 方法创建事务 ID:


.. _.xid:
.. _.xid():

`.xid`_\ (*format_id*, *global_transaction_id*, *branch_qualifier*)
    返回适合传递给此连接的 `.tpc_*()`_ 方法的事务 ID 对象.

    如果数据库连接不支持 TPC, 则引发 NotSupportedError_.

    `.xid()`_ 返回的对象类型未定义, 但必须提供序列行为, 允许访问这三个组件.
	符合要求的数据库模块可以选择使用元组, 而不是自定义对象来表示事务 ID.


`TPC Connection Methods`_
-------------------------

.. _.tpc_*:
.. _.tpc_*():

.. _.tpc_begin:
.. _.tpc_begin():

`.tpc_begin`_\ (*xid*)
    使用给定的事务 ID *xid* 开始 TPC 事务.

    应该在事务之外调用此方法 (*即* 自上一个 `.commit()`_
	或 `.rollback()`_ 之后可能没有执行任何操作.

    此外, 在 TPC 事务中调用 `.commit()`_ 或 `.rollback()`_ 是错误的.
	如果应用程序在活动 TPC 事务期间调用 `.commit()`_ 或 `.rollback()`_,
	则引发 ProgrammingError_.

    如果数据库连接不支持 TPC, 则引发 NotSupportedError_.


.. _.tpc_prepare:
.. _.tpc_prepare():

`.tpc_prepare`_\ ()
    执行以 `.tpc_begin()`_ 开头的事务的第一阶段.
	如果此方法在 TPC 事务之外, 则应该引发 ProgrammingError_.

    在调用 `.tpc_prepare()`_ 之后, 在调用 `.tpc_commit()`_
	或 `.tpc_rollback()`_ 之前, 不能执行任何语句.


.. _.tpc_commit:
.. _.tpc_commit():

`.tpc_commit`_\ ([ *xid* ])
    当没有参数调用时, `.tpc_commit()`_ 提交一个先前
	使用 `.tpc_prepare()`_ 编写的 TPC 事务.

    如果在 `.tpc_prepare()`_ 之前调用 `.tpc_commit()`_, 则执行单阶段提交.
	如果只有一个资源参与全局事务, 则事务管理器可以选择执行此操作.

    使用事务 ID *xid* 调用时, 数据库将提交给定的事务.
	如果提供了无效的事务 ID, 则会引发 ProgrammingError_.
	此表单应该在事务外部调用, 旨在用于恢复.

    返回时, TPC 交易结束.


.. _.tpc_rollback:
.. _.tpc_rollback():

`.tpc_rollback`_\ ([ *xid* ])
    当没有参数调用时, `.tpc_rollback()`_ 回滚一个 TPC 事务.
	它可以在 `.tpc_prepare()`_ 之前或之后调用.

    使用事务 ID *xid* 调用时, 它将回滚给定的事务.
	如果提供了无效的事务 ID, 则会引发 ProgrammingError_.
	此表单应该在事务外部调用, 旨在用于恢复.

    返回时, TPC 交易结束.

.. _.tpc_recover:
.. _.tpc_recover():

`.tpc_recover`_\ ()
    返回一个适用于 ``.tpc_commit(xid)``
	或 ``.tpc_rollback(xid)`` 的挂起事务 ID 列表.

    如果数据库不支持事务恢复,
	则可能返回空列表或引发 NotSupportedError_.



`Frequently Asked Questions`_
=============================

数据库 SIG 经常看到有关 DB API 规范的重复出现的问题.
本节介绍人们有时会对规范提出的一些问题.

**Question:**

如何从 `.fetch*()`_ 返回的元组构造一个字典?:

**Answer:**

有几种现有的工具可以为这项任务提供帮助.
他们中的大多数使用的方法是使用游标属性 `.description`_ 中
定义的列名作为行字典中键的基础.

请注意, 不扩展 DB API 规范来支持 `.fetch*()`_ 方法的字典
返回值的原因是这种方法有几个缺点:

* 某些数据库不支持区分大小写的列名称
   或将它们自动转换为全部小写或全部大写字符.

* 结果集中由查询生成的列 (例如, 使用 SQL 函数) 不映射到表列名称,
  数据库通常以特定于数据库的方式生成这些列的名称.

因此, 通过字典键访问列在数据库之间是不同的,
并且使得编写可移植代码变得不可能.



`Major Changes from Version 1.0 to Version 2.0`_
================================================

与 1.0 版本相比, Python Database API 2.0 引入了一些重大更改.
由于其中一些更改会导致现有的基于 DB API 1.0 的脚本中断,
因此调整了主要版本号来反映此更改.

这些是从 1.0 到 2.0 的最重要的变化:

* 删除了对单独 dbi 模块的需求, 并将功能合并到模块接口本身.

* 新的构造函数和 `Type Objects`_ 被添加了日期或时间值,
   ``RAW`` 类型对象被重命名为 ``BINARY``.
   结果集应涵盖现代 SQL 数据库中常见的所有基本数据类型.

* 添加了新常量 (apilevel_, threadsafety_, paramstyle_)
   和方法 (`.executemany()`_, `.nextset()`_) 来提供更好的数据库绑定.

* 现在可以清楚地定义调用存储过程所需的 `.callproc()`_ 的语义.

* `.execute()`_ 返回值的定义已更改. 以前, 返回值基于 SQL 语句类型 (很难正确实现) -- 现在未定义;
   使用更灵活的 `.rowcount`_ 属性. 模块可以自由返回旧样式返回值,
   但这些不再是规范要求的, 应该被视为依赖于数据库接口.

* 基于类的异常被纳入规范. 模块实现者可以通过继承定义的异常类来自由扩展本规范中定义的异常布局.


发布后添加到 DB API 2.0 规范:

* 指定了对核心功能集的附加可选 DB API 扩展.


`Open Issues`_
==============

尽管 2.0 版规范澄清了 1.0 版本中尚未公开的许多问题,
但仍有一些问题需要在未来的版本中解决:

* 为新的结果集可用的情况定义 `.nextset()`_ 的有用返回值.

* 整合 `十进制模块 <http://docs.python.org/library/decimal.html>`__
   ``Decimal`` 对象用作减少金钱损失和十进制交换格式.



`Footnotes`_
============

.. [1] 作为指导, 连接构造函数参数应该实现为关键字参数,
       利于更直观地使用, 并遵循此参数顺序:

    ============= ====================================
    Parameter     Meaning
    ============= ====================================
    ``dsn``       Data source name as string
    ``user``      User name as string (optional)
    ``password``  Password as string (optional)
    ``host``      Hostname (optional)
    ``database``  Database name (optional)
    ============= ====================================

    例如. 连接可能看起来像这样::

        connect(dsn='myhost:MYDB', user='guido', password='234$')

.. [2] 模块实现者应该比其他格式更喜欢 ``numeric``, ``named`` 或 ``pyformat``,
	   因为这些提供了更多的清晰度和灵活性.


.. [3] 如果数据库不支持该方法所需的功能, 则在使用该方法时, 接口应该抛出异常.

    首选方法是不实现该方法, 因此如果请求方法, Python 会生成 ``AttributeError``.
	这允许程序员使用标准的 ``hasattr()`` 函数检查数据库功能.

    对于某些动态配置的接口, 可能不适合要求动态地使该方法可用.
	然后, 这些接口应该引发一个 ``NotSupportedError``, 表示在调用该方法时无法执行回滚.

.. [4] 数据库接口可以通过允许方法的字符串参数来选择支持命名游标.
       此功能不是规范的一部分, 因为它使 `.fetch*()`_ 方法的语义复杂化.

.. [5] 该模块将使用参数对象的 ``__getitem__`` 方法将位置 (整数) 或名称 (字符串) 映射到参数值.
       这允许将序列和映射用作输入.

    术语 *bound* 是指将输入值绑定到数据库执行缓冲区的过程. 实际上,
	这意味着输入值直接用作操作中的值. 不应该要求客户端 "转义" 该值,
	利于可以使用它 -- 该值应该等于实际的数据库值.

.. [6] 请注意, 接口可以使用数组和其他优化实现行提取.
       不能保证对此方法的调用只会将关联的光标向前移动一行.

.. [7] 可以以动态更新其值的方式对 ``rowcount`` 属性进行编码.
       这对于仅在第一次调用 `.fetch*()`_ 方法后返回可用的 ``rowcount`` 值的数据库非常有用.

.. [8] 实现注意: Python C 扩展必须在游标对象上实现 ``tp_iter`` 槽
       而不是 ``.__iter__()`` 方法.

.. [9] 术语 *number of affected rows* 通常是指由数据库游标上运行的最后一个语句删除, 更新或插入的行数.
       大多数数据库将返回由语句的相应 ``WHERE`` 子句找到的总行数.
	   有些数据库对 ``UPDATE``\s 使用不同的解释, 只返回由 ``UPDATE`` 更改的行数,
	   即使语句的 ``WHERE`` 子句可能找到了更多匹配的行. 数据库模块作者应该尝试实现更常见的解释,
	   即返回 ``WHERE`` 子句找到的行总数, 或者清楚地记录 ``.rowcount`` 属性的不同解释.


`Acknowledgements`_
===================

非常感谢 Andrew Kuchling, 他将 Python 数据库 API 规范 2.0
从原始 HTML 格式转换为 PEP 格式.

非常感谢 James Henstridge 带领讨论,
这导致了两阶段提交 API 扩展的标准化.

非常感谢 Daniele Varrazzo 将规范从文本 PEP 格式转换为 ReST PEP 格式,
允许链接到各个部分.

`Copyright`_
============

This document has been placed in the Public Domain.
