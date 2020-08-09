
PEP: 11
Title: Removing support for little used platforms
Version: $Revision$
Last-Modified: $Date$
Author: Martin von Lรถwis <martin@v.loewis.de>,
        Brett Cannon <brett@python.org>
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 07-Jul-2002
Post-History: 18-Aug-2007
              16-May-2014
              20-Feb-2015


Abstract
--------

此 PEP 记录了如何在 CPython 中支持操作系统(平台)以及文档过去的支持


Rationale
---------

随着时间的推移, CPython 源代码收集了各种特定于平台的代码,
在某些时候, 这些代码被认为是在特定平台上使用 Python 所必需的
如果无法访问此平台, 则无法确定是否仍需要此代码
因此, 此代码可能会在 Python 的演变过程中中断, 也可能随着平台的发展而变得不必要

越来越多的这些片段带来了不可维护的风险:
没有专家支持大量平台, 就无法确定对 CPython 源代码的某些更改是否适用于所有支持的平台

为了降低这种风险, 此 PEP 指定了 Python 支持的平台所需的内容,
并提供了删除 Python 用户很少或没有的平台的代码的过程


Supporting platforms
--------------------

获得官方平台支持需要两件事, 首先, 核心开发人员需要自愿维护特定于平台的代码
这个核心开发人员既可以是 Python 开发团队的成员, 也可以在维护平台支持的基础上获得贡献者权利
(Python 开发团队可以自行决定是否已准备好拥有此类权利, 即使它只是用于支持特定平台)

其次, 必须提供稳定的 buildbot [2]_
这保证了平台支持不会被没有个人访问平台的 Python 核心开发人员意外破坏
为了使 buildbot 被认为是稳定的, 它需要机器可靠地启动并运行
(但是由 Python 核心开发人员来决定是否要将 buildbot 提升为被认为是稳定的)

该策略不会间接取消支持其他平台的资格
将考虑包含非平台特定, 但仍添加平台支持的修补程序
例如, 如果配置脚本中有必要进行与平台无关的更改, 这些更改有助于支持可接受的特定平台
如果平台没有官方支持, 通常不会接受将特定于平台的代码 (例如, 特定平台的名称) 添加到配置脚本的修补程序

CPU 架构和编译器支持以与平台类似的方式查看
例如, 要考虑支持在 ARM 上运行的 buildbot 的 ARM 体系结构, 以及 Python 开发团队的支持
通常, 不需要在每个可能的平台下运行 CPU 架构以便被视为支持


Unsupporting platforms
----------------------

如果某个当前在 CPython 中有特殊代码的平台被认为没有足够的 Python 用户或缺乏 Python 开发团队
或 buildbot 的适当支持, 则必须在此 PEP 中发布一条说明, 该平台不再受到积极支持, 本说明必须包括:

- 系统的名称
- 不再支持此平台的第一个版本号, 以及
- 主动删除历史支持代码的第一个版本

在某些情况下, 不可能识别使用某些代码的系统的特定列表
(例如, 当 autoconf 测试缺少某些特征时, 这些特征被认为存在于所有支持的系统上)
在这种情况下, 名称将给出不受支持的精确条件 (通常是预处理器符号)

同时, 如果有人试图在此平台上安装 Python, 则必须更改 CPython 源代码以产生构建时错误
在使用 autoconf 的平台上, configure 必须失败, 这为平台的潜在用户提供了前进和提供维护的机会


Re-supporting platforms
-----------------------

如果平台的用户想要再次支持该平台, 他可以自愿维护平台支持
此类要约必须记录在 PEP 中, 用户可以提交补丁以消除构建时错误, 并为平台执行任何其他维护工作


Microsoft Windows
-----------------

Microsoft 已经建立了一个名为产品支持生命周期 [1]_ 的策略
每个产品的生命周期都有一个主流支持阶段, 其中产品通常是商业上可用的, 以及扩展的支持阶段
其中仍然提供付费支持, 并且发布了某些错误修复 (特别是安全修复)

CPython 的 Windows 支持现在遵循此生命周期
新功能版本 X.Y.0 将支持其扩展支持阶段尚未过期的所有 Windows 版本
随后的错误修复版本将支持与原始功能版本相同的 Windows 版本 (即使扩展支持阶段已结束)

由于此策略, 此 PEP 中不需要列出其他 Windows 版本

每个功能版本都是由特定版本的 Microsoft Visual Studio 构建的
在发布时, 该版本应该具有主流支持, 扩展模块的开发人员通常需要使用相同的 Visual Studio 版本;
他们既关注他们需要使用的版本的可用性, 又关注 zoo 版本的小版本
CPython 源代码树将为旧的 Visual Studio 版本保留未维护的构建文件, 这些版本将被接受
在对编译器的扩展支持结束3年后, 这些构建文件将从源树中删除 (但在修订控制中仍然可用)


Legacy C Locale
---------------

从 CPython 3.7.0 开始, \*nix 平台应该提供至少一个 ``C.UTF-8`` (完整语言环境),
``C.utf8`` (完整语言环境)或 ``UTF-8 `` (``LC_CTYPE``-only locale) 作为传统 ``C`` 语言环境的替代

任何与 Unicode 相关的集成问题仅在旧的 ``C`` 语言环境中出现,
并且无法在适当配置的非 ASCII 语言环境中重现, 将被关闭为 "won't fix"


No-longer-supported platforms
-----------------------------

* | Name:             MS-DOS, MS-Windows 3.x
  | Unsupported in:   Python 2.0
  | Code removed in:  Python 2.1

* | Name:             SunOS 4
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             DYNIX
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             dgux
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Minix
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Irix 4 and --with-sgi-dl
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Linux 1
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Systems defining __d6_pthread_create (configure.in)
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Systems defining PY_PTHREAD_D4, PY_PTHREAD_D6,
                      or PY_PTHREAD_D7 in thread_pthread.h
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Systems using --with-dl-dld
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             Systems using --without-universal-newlines,
  | Unsupported in:   Python 2.3
  | Code removed in:  Python 2.4

* | Name:             MacOS 9
  | Unsupported in:   Python 2.4
  | Code removed in:  Python 2.4

* | Name:             Systems using --with-wctype-functions
  | Unsupported in:   Python 2.6
  | Code removed in:  Python 2.6

* | Name:             Win9x, WinME, NT4
  | Unsupported in:   Python 2.6 (warning in 2.5 installer)
  | Code removed in:  Python 2.6

* | Name:             AtheOS
  | Unsupported in:   Python 2.6 (with "AtheOS" changed to "Syllable")
  | Build broken in:  Python 2.7 (edit configure to reenable)
  | Code removed in:  Python 3.0
  | Details:          http://www.syllable.org/discussion.php?id=2320

* | Name:             BeOS
  | Unsupported in:   Python 2.6 (warning in configure)
  | Build broken in:  Python 2.7 (edit configure to reenable)
  | Code removed in:  Python 3.0

* | Name:             Systems using Mach C Threads
  | Unsupported in:   Python 3.2
  | Code removed in:  Python 3.3

* | Name:             SunOS lightweight processes (LWP)
  | Unsupported in:   Python 3.2
  | Code removed in:  Python 3.3

* | Name:             Systems using --with-pth (GNU pth threads)
  | Unsupported in:   Python 3.2
  | Code removed in:  Python 3.3

* | Name:             Systems using Irix threads
  | Unsupported in:   Python 3.2
  | Code removed in:  Python 3.3

* | Name:             OSF* systems (issue 8606)
  | Unsupported in:   Python 3.2
  | Code removed in:  Python 3.3

* | Name:             OS/2 (issue 16135)
  | Unsupported in:   Python 3.3
  | Code removed in:  Python 3.4

* | Name:             VMS (issue 16136)
  | Unsupported in:   Python 3.3
  | Code removed in:  Python 3.4

* | Name:             Windows 2000
  | Unsupported in:   Python 3.3
  | Code removed in:  Python 3.4

* | Name:             Windows systems where COMSPEC points to command.com
  | Unsupported in:   Python 3.3
  | Code removed in:  Python 3.4

* | Name:             RISC OS
  | Unsupported in:   Python 3.0 (some code actually removed)
  | Code removed in:  Python 3.4

* | Name:             IRIX
  | Unsupported in:   Python 3.7
  | Code removed in:  Python 3.7

* | Name:             Systems without multithreading support
  | Unsupported in:   Python 3.7
  | Code removed in:  Python 3.7

References
----------

.. [1] http://support.microsoft.com/lifecycle/
.. [2] http://buildbot.python.org/3.x.stable/

Copyright
---------

本文档已置于公共域名


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   coding: utf-8  
   End:  
