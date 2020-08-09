
PEP: 248
Title: Python Database API Specification v1.0
Version: $Revision$
Last-Modified: $Date$
Author: mal@lemburg.com (Marc-André Lemburg)
Discussions-To: db-sig@python.org
Status: Final
Type: Informational
Content-Type: text/x-rst
Created:
Post-History:
Superseded-By: 249


Introduction
============

已定义此 API 来鼓励用于访问数据库的 Python 模块之间的相似性.
通过这样做, 我们希望实现兼容性, 从而实现更容易理解的模块,
通常在数据库中更容易移植的代码, 以及 Python 更广泛的数据库连接范围.

该接口规范由几个项目组成:

* 模块接口
* 连接对象
* 游标对象
* DBI 帮助对象

有关此规范的注释和问题可能会针对 Python 中的表格数据库上的 SIG
(http://www.python.org/sigs/db-sig).

该规范文档最后更新于: 1996年4月9日. 它将被称为本规范的 1.0 版本.


Module Interface
================

数据库接口模块通常应该使用 ``db`` 终止的名称命名.
现有的例子是: ``oracledb``, ``informixdb`` 和 ``pg95db``.
这些模块应该导出多个名称:

``modulename(connection_string)``
   用于创建与数据库的连接的构造函数.
   返回一个连接对象.

``error``
   针对数据库模块中的错误引发异常.


Connection Objects
==================

连接对象应该响应以下方法:

``close()``
   现在关闭连接 (而不是每次调用 ``__del__``).
   从这一点开始, 连接将无法使用; 如果尝试连接任何操作, 将引发异常.

``commit()``
   将任何挂起的事务提交到数据库.

``rollback()``
   将数据库回滚到任何挂起事务的开头.

``cursor()``
   返回一个新的游标对象. 如果数据库不支持游标概念, 则可能抛出异常.

``callproc([params])``
   (注意: 此方法尚未明确定义.) 使用给定 (可选) 参数调用存储的数据库过程.
   返回存储过程的结果.

(all Cursor Object attributes and methods)
   对于没有游标的数据库和不需要游标复杂性的简单应用程序,
   连接对象应该响应游标对象的每个属性和方法.
   具有游标的数据库可以通过使用隐式内部游标来实现此目的.


Cursor Objects
==============

这些对象表示数据库游标, 用于管理获取操作的上下文.

游标对象应该响应以下方法和属性:

``arraysize``
   这个读或写属性用 ``fetchmany()`` 指定一次获取的行数. 当一次插入多行时,
   也会使用此值 (将 tuples/lists 的 tuple/list 作为参数值传递给 ``execute()``).
   此属性默认为单行.

   请注意, 数组大小是可选的, 仅用于更高性能的数据库交互.
   实现应该遵循 ``fetchmany()`` 方法来观察它,
   但是可以一次一行地与数据库进行交互.

``description``
   此只读属性是7元组的元组. 每个7元组包含描述每个结果列的信息:
   (name, type_code, display_size, internal_size, precision, scale, null_ok).
   对于不返回行的操作, 或者如果游标尚未通过 ``execute()`` 方法调用操作,
   该属性将为 ``None``.

   'type_code' 是以下部分中指定的 'dbi' 值之一.

   注意: 这有点不稳定. 通常, 7元组的前两项将始终存在;
   其他可能是数据库特定的.

``close()``
   现在关闭光标 (而不是每次调用 ``__del__``). 从这一点开始,
   光标将无法使用; 如果使用光标尝试任何操作, 将引发异常.

``execute(operation [,params])``
   执行 (准备) 数据库操作 (查询或命令). 可以提供参数 (作为序列 (例如, 元组或列表))
   并且将参数绑定到操作中的变量. 变量来特定于数据库的表示法指定,
   该表示法基于参数元组中的索引 (基于位置而不是基于名称).

   参数还可以被指定为多个序列的单个序列 (例如, 元组的列表)
   用来在单个操作中插入多行.

   光标将保留对操作的引用. 如果再次传入相同的操作对象, 则游标可以优化其行为.
   这对于使用相同操作的算法最有效, 但是不同的参数绑定到它 (很多次).

   为了在重用操作时获得最大效率, 最好使用 ``setinputsizes()`` 方法提前指定参数类型和大小.
   参数与预定义信息不匹配是合法的; 实施应该补偿, 可能会导致效率损失.

   使用 SQL 术语, 这些是 ``execute()`` 方法可能的结果值:

   - 如果语句是 DDL (例如, ``CREATE TABLE``), 则返回 1.

   - 如果语句是 DML (例如, ``UPDATE`` 或 ``INSERT``),
     则返回受影响的行数 (0 或正整数).

   - 如果语句是 DQL (例如, ``SELECT``), 则返回 ``None``,
     表示在使用 'fetch' 方法之一, 之前语句不完全完成.

``fetchone()``
   获取查询结果的下一行, 返回单个元组.

``fetchmany([size])``
   获取查询结果的下一组行, 返回为元组列表. 当没有更多行可用时, 返回空列表.
   要获取的行数由参数指定. 如果是 ``None``, 则光标的 arraysize 确定要获取的行数.

   请注意, size 参数涉及性能方面的考虑因素. 为获得最佳性能, 通常最好使用 arraysize 属性.
   如果使用 size 参数, 那么最好是从一个 ``fetchmany()`` 调用到下一个调用保持相同的值.

``fetchall()``
   获取查询结果的所有行, 作为元组列表返回.
   请注意, 游标的 arraysize 属性可能会影响此操作的性能.

``setinputsizes(sizes)``
   (注意: 这个方法还没有很好地定义.) 这可以在调用 ``execute()`` 之前使用, 来预定义操作参数的内存区域.
   sizes 被指定为元组 -- 每个输入参数一个项. 该项应该是与将要使用的输入对应的 Type 对象,
   或者应该是指定字符串参数的最大长度的整数. 如果该项为 ``None``,
   则不会为该列保留预定义的内存区域 (这对于避免大输入的预定义区域很有用).

   在调用 ``execute()`` 方法之前, 将使用此方法.

   请注意, 此方法是可选的, 仅用于更高性能的数据库交互.
   实现可以随意执行任何操作, 用户可以不使用它.

``setoutputsize(size [,col])``
   (注意: 此方法尚未明确定义.)

   为大量列的提取设置列缓冲区大小 (例如 LONG). 该列被指定为结果元组的索引.
   使用 ``None`` 列将为光标中的所有大量列设置默认大小.

   在调用 ``execute()`` 方法之前, 将使用此方法.

   请注意, 此方法是可选的, 仅用于更高性能的数据库交互.
   实现可以随意执行任何操作, 用户可以不使用它.


DBI Helper Objects
==================

许多数据库需要以特定格式输入来绑定到操作的输入参数. 例如,
如果输入的目的地是 ``DATE`` 列, 那么它必须以特定的字符串格式绑定到数据库.
"Row ID" 列或大型二进制项 (例如 blob 或 ``RAW`` 列) 存在类似的问题.
这给 Python 带来了问题, 因为 ``execute()`` 方法的参数是无类型的.
当数据库模块看到 Python 字符串对象时, 它不知道它是应该绑定为简单的 CHAR 列,
作为原始二进制项, 还是作为 ``DATE``.

为了克服这个问题, 创建了 'dbi' 模块. 此模块指定了一些用于处理数据库的基本数据库接口类型.
有两个类: 'dbiDate' 和 'dbiRaw'. 这些是包装值的简单容器类. 当传递给数据库模块时,
模块可以检测到输入参数是 ``DATE`` 或 ``RAW``. 对于对称性,
数据库模块将返回 ``DATE`` 和 ``RAW`` 列作为这些类的实例.

游标对象的 'description' 属性返回有关查询的每个结果列的信息.
'type_code' 被定义为该模块导出的五种类型之一: ``STRING``, ``RAW``,
``NUMBER``, ``DATE`` 或 ``ROWID``.

该模块导出以下名称:

``dbiDate(value)``
   此函数构造一个包含日期值的 'dbiDate' 实例. 该值应该指定为自 "epoch" 以来的整数秒数
   (例如 ``time.time()``).

``dbiRaw(value)``
   此函数构造一个保存原始 (二进制) 值的 'dbiRaw' 实例.
   该值应该指定为 Python 字符串.

``STRING``
   此对象用于描述数据库中基于字符串的列
   (例如 CHAR).

``RAW``
   此对象用于描述数据库中的 (大) 二进制列
   (例如 LONG RAW, blobs).

``NUMBER``
   此对象用于描述数据库中的数字列.

``DATE``
   此对象用于描述数据库中的日期列.

``ROWID``
   此对象用于描述数据库中的 "Row ID" 列.


Acknowledgements
================

非常感谢 Andrew Kuchling, 他将 Python 数据库 API
规范 1.0 从原始 HTML 格式转换为 PEP 格式.


Copyright
=========

This document has been placed in the Public Domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
