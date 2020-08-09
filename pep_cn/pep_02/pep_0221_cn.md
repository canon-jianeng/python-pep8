
PEP: 221
Title: Import As
Version: $Revision$
Last-Modified: $Date$
Author: thomas@python.org (Thomas Wouters)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 15-Aug-2000
Python-Version: 2.0
Post-History:


Introduction
============

这个 PEP 描述了 Python 2.0 的``import as``提议. 此 PEP 跟踪此功能的状态和所有权.
它包含功能的说明, 并概述了支持该功能所需的更改. 此文件的 CVS 修订历史记录包含确定的历史记录.


Rationale
=========

这个 PEP 提出了关于 ``import`` 和 ``from <module> import`` 语句的 Python 语法的扩展.
这些语句在模块中加载, 并将该模块绑定到本地名称, 或将该模块中的对象绑定到本地名称.
但是, 有时需要将这些对象绑定到不同的名称, 例如以避免名称冲突. 目前可以使用以下惯用语法来实现::

    import os
    real_os = os
    del os

同样对于 ``from ... import`` 语句::

    from os import fdopen, exit, stat
    os_fdopen = fdopen
    os_stat = stat
    del fdopen, stat

建议的语法更改将为这两个语句添加一个可选的 ``as`` 子句, 如下所示 ::

    import os as real_os
    from os import fdopen as os_fdopen, exit, stat as os_stat

``as`` 这个名字并不是一个关键字, 必须用一些技巧来说服 CPython 解析器它不是一个.
但是, 对于更高级的解析器或标记器, 这应该不是问题.

导入子模块存在一些特殊情况. 该声明 ::

    import os.path

将模块 ``os`` 本地存储为 ``os``, 以便导入的子模块 ``path`` 可以作为 ``os.path`` 访问.
结果是, ::

    import os.path as p

在 ``p`` 中存储 ``os.path``, 而不是 ``os``.
这使得它实际上与之相同 ::

    from os import path as p


Implementation details
======================

此 PEP 已被接受, 并且已检查建议的代码更改. 补丁仍可在 SourceForge 补丁管理器 [1]_ 中找到.
目前, 在语法中使用 ``NAME`` 字段而不是裸字符串, 以避免关键字问题. 它引入了一个新的字节码,
``IMPORT_STAR``, 它执行 ``from module import *`` 行为, 并改变``IMPORT_FROM``字节码的行为,
以便它加载所请求的名称 (总是一个名称) 到堆栈上, 随后由 "STORE" 操作码存储.
因此, 现在显式导入的所有名称都遵循 ``global`` 指令.

``from module import *``的特殊情况仍然是一个特殊情况, 因为它不能容纳 ``as`` 子句,
并且没有生成 ``STORE`` 操作码; 导入的对象直接加载到本地名称空间.
这也意味着以这种方式导入的名称始终是本地的, 并且不遵循 ``global`` 指令.

还建议对此语法进行其他更改, 以概括在 ``as`` 子句后给出的表达式.
可以允许它是任何产生有效左值的表达式, 而不是单个名称; 任何可以分配的东西.
正如补丁[2]_ 所证明的那样, 适应这种情况的变化是最小的,
并且由此产生的泛化允许许多与其他 Python 赋值结构完全并行运行的新构造.
然而, 这个想法被 Guido 拒绝为 "超常化".


Copyright
=========

This document has been placed in the Public Domain.


References
==========

.. [1] https://hg.python.org/cpython/rev/18385172fac0

.. [2] http://sourceforge.net/patch/?func=detailpatch&patch_id=101234&group_id=5470



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
