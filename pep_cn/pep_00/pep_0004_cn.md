
PEP: 4
Title: Deprecation of Standard Modules
Version: $Revision$
Last-Modified: $Date$
Author: Brett Cannon <brett@python.org>, Martin von Löwis <martin@v.loewis.de>
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 1-Oct-2000
Post-History:


Introduction
============

当过去将新模块添加到 Python 标准库时, 无法预见它们将来是否仍然有用
即使 Python "Comes With Batteries Included", 电池也会随着时间的推移而放电
携带旧模块是维护者的负担, 特别是当对模块不再感兴趣时

同时, 从分发中移除模块是困难的, 因为一般不知道是否有人仍在使用它
PEP 4 定义了从 Python 标准库中删除模块的过程
模块的使用可能被 "deprecated", 这意味着它可能会从未来的 Python 版本中删除
在 PEP 4 中也收集了弃用模块的基本原理, 如果基本原理有问题, 模块可能会变得 "undeprecated"

Procedure for declaring a module deprecated
===========================================

由于模块弃用的状态记录在 PEP 4 中,
因此必须通过对 PEP 4 的文本进行更改来提出弃用模块的建议, PEP 4 应该是发布到 bugs.python.org 的补丁

弃用模块的提议必须包括建议弃用的日期和弃用的理由
此外, 该提案必须包括对模块文档的更改, 表示该模块已经 "obsolete" 或 "deprecated" 表示弃用
该提案应该包括一个模块源代码的补丁, 通过提高 DeprecationWarning 来指示那里的弃用
该提议必须包含补丁, 用来从标准库中删除对已弃用模块的任何使用

预计不推荐使用的模块包含在 Python 版本中, 该版本紧随弃用之后, 以后的版本可能没有已弃用的模块

For modules existing in both Python 2.7 and Python 3.5
------------------------------------------------------

为了便于同时编写兼容 Python 2 和 python 3 的代码, 在 PEP 373 规定的 Python 2.7 不再受支持之前,
不会从标准库中删除 Python 3.5 和 Python 2.7 中存在的任何模块
此规则是 idlelib 包中的任何模块以及 Python 开发团队授予的任何异常

Procedure for declaring a module undeprecated
=============================================

当模块被弃用时, 会给出其弃用的基本原理
在某些情况下, 会提供相同功能的备用接口, 因此不推荐使用旧接口
在其他情况下, 可能不再需要具有模块的功能

如果基本原理有问题, 则必须再次提交对该 PEP 文本的更改
此更改必须包括未去除的日期和未去除的理由
在 PEP 4 中未初级化的模块必须在 PEP 4 中列出至少一个主要的 Python 版本

Obsolete modules
================

许多模块已在标准库文档中列为过时, 这些列表是为了完整性

    cl, sv, timing

所有这些模块都已经在 Python 2.0 中声明为过时, 有些甚至更早

在 Python 2.5 中删除了以下过时的模块：

    addpack, cmp, cmpcache, codehack, dircmp, dump, find, fmt,
    grep, lockfile, newdir, ni, packmail, Para, poly,
    rand, reconvert, regex, regsub, statcache, tb, tzparse,
    util, whatsound, whrandom, zmod

在 Python 2.6 中删除了以下模块：

    gopherlib, rgbimg, macfs

以下模块目前缺少 DeprecationWarning ：

    rfc822, mimetools, multifile

Deprecated modules
==================

::

    Module name:   posixfile
    Rationale:     fcntl.lockf（）可以更好地完成锁定
    Date:          2000-10-01 之前
    Documentation: 已经记录为过时的, 在 Python 2.6 中添加了弃用警告

    Module name:   gopherlib
    Rationale:     gopher 协议不再有效使用
    Date:          2000-10-01
    Documentation: 自 Python 2.5 起被记录为已弃用, 在 Python 2.6 中删除

    Module name:   rgbimgmodule
    Rationale:     在 2001-04-24 c.l.py 帖子中, Jason Petrone 提到他偶尔使用它, 截至2003-11-19, 没有其他参考资料可供参考
    Date:          2000-10-01
    Documentation: 自 Python 2.5 起被记录为已弃用, 在 Python 2.6 中删除

    Module name:   pre
    Rationale:     底层 PCRE 引擎不支持 Unicode, 并且自 Python 1.5.2 以来一直没有维护
    Date:          2002-04-10
    Documentation: 它仅作为实现细节被提及, 并且从未有过自己的部分, 这一提及现已被删除

    Module name:   whrandom
    Rationale:     该模块的默认种子计算本质上是不安全的, 应该使用随机模块
    Date:          2002-04-11
    Documentation: 自 Python 2.1 以来, 该模块已被记录为过时, 但 PEP 4 中的列表被忽略了
                   弃用警告将在 Python 2.3 发布一年后添加到模块中, 并且模块将在一年后删除

    Module name:   rfc822
    Rationale:     由 Python 2.2 的 email 模块包替代
    Date:          2002-03-18
    Documentation: 自 Python 2.2.2 起被记录为 "自 python 2.3 版以来已弃用"

    Module name:   mimetools
    Rationale:     由 Python 2.2 的 email 模块包替代
    Date:          2002-03-18
    Documentation: 自 Python 2.2.2 起被记录为 "自 python 2.3 版以来已弃用"

    Module name:   MimeWriter
    Rationale:     由 Python 2.2 的 email 模块包替代
    Date:          2002-03-18
    Documentation: 自 Python 2.2.2 起被记录为 "自 python 2.3 版以来已弃用", 从 Python 2.6 开始提高 DeprecationWarning

    Module name:   mimify
    Rationale:     由 Python 2.2 的 email 模块包替代
    Date:          2002-03-18
    Documentation: 自 Python 2.2.2 起被记录为 "自 python 2.3 版以来已弃用", 从 Python 2.6 开始提高 DeprecationWarning

    Module name:   rotor
    Rationale:     使用不安全的算法
    Date:          2003-04-24
    Documentation: 该文档已从 Python 2.4 中的标准库引用中删除

    Module name:   TERMIOS.py
    Rationale:     此文件中的常量现在位于 "termios" 模块中
    Date:          2004-08-10
    Documentation: 自 Python 2.1 以来, 该模块已被记录为过时, 但 PEP 4 中的列表被忽略了
                   从 Python 2.4 中的标准库引用中删除

    Module name:   statcache
    Rationale:     使用缓存可能很脆弱且容易出错, 应用程序应该直接使用 os.stat()
    Date:          2004-08-10
    Documentation: 自 Python 2.2 以来, 该模块已被记录为过时, 但 PEP 4 中的列表被忽略了
                   从 Python 2.5 中的标准库引用中删除

    Module name:   mpz
    Rationale:     第三方软件包提供类似的功能, 并包含更多 GMP 的 API
    Date:          2004-08-10
    Documentation: 自 Python 2.2 以来, 该模块已被记录为过时, 但 PEP 4 中的列表被忽略了
                   从 Python 2.4 中的标准库引用中删除

    Module name:   xreadlines
    Rationale:     在 python 2.3 中引入的 'for line in file' 是最好的
    Date:          2004-08-10
    Documentation: 自 Python 2.3 以来, 该模块已被记录为过时, 但 PEP 4 中的列表被忽略了
                   从 Python 2.4 中的标准库引用中删除

    Module name:   multifile
    Rationale:     由 email 模块包替代
    Date:          2006-02-21
    Documentation: 从 Python 2.5 开始被记录为已弃用

    Module name:   sets
    Rationale:     Python 2.4 中引入的内置 set/frozenset 类型取代了模块
    Date:          2007-01-12
    Documentation: 从 Python 2.6 开始被记录为已弃用

    Module name:   buildtools
    Rationale:     未知
    Date:          2007-05-15
    Documentation: 在 Python 2.3 中被记录为已弃用, 但 PEP 4 中的列表被忽略了
                   从 Python 2.6 开始提出了 DeprecationWarning

    Module name:   cfmfile
    Rationale:     未知
    Date:          2007-05-15
    Documentation: 在 Python 2.4 中被记录为已弃用, 但 PEP 4 中的列表被忽略了
                   Python 2.6 中添加了 DeprecationWarning

    Module name:   macfs
    Rationale:     未知
    Date:          2007-05-15
    Documentation: 在 Python 2.3 中被记录为已弃用, 但 PEP 4 中的列表被忽略了
                   在 Python 2.6 中删除

    Module name:   md5
    Rationale:     由 "hashlib" 模块替换
    Date:          2007-05-15
    Documentation: 在 Python 2.5 中被记录为已弃用，但 PEP 4 中的列表被忽略了
                   从 Python 2.6 开始提出弃用警告

    Module name:   sha
    Rationale:     由 "hashlib" 模块替换
    Date:          2007-05-15
    Documentation: 在 Python 2.5 中被记录为已弃用，但 PEP 4 中的列表被忽略了
                   从 Python 2.6 开始提出弃用警告

    Module name:   plat-freebsd2/IN and plat-freebsd3/IN
    Rationale:     平台已经过时 (最近一次发布于2000年), 从 Python 2.6 中删除
    Date:          2007-05-15
    Documentation: None

    Module name:   plat-freebsd4/IN and possibly plat-freebsd5/IN
    Rationale:     平台已过时或不受支持, 从 Python 2.7 中删除
    Date:          2007-05-15
    Documentation: None

    Module name:   imp
    Rationale:     由 importlib 模块替换
    Date:          2013-02-10
    Documentation: 从 Python 3.4 开始不推荐使用

    Module name:   formatter
    Rationale:     在社区中缺乏使用, 没有测试来保持代码正常工作
    Date:          2013-08-12
    Documentation: 从 Python 3.4 开始不推荐使用

    Module name:   macpath
    Rationale:     过时的 macpath 模块危险，应该删除
    Date:          2017-05-15
    Documentation: 平台已过时或不受支持

Deprecation of modules removed in Python 3.0
============================================

PEP 3108 列出了从 Python 3.0 中删除的所有模块
它们都在 Python 2.6 中被记录为已弃用, 如果 -3 标志被激活, 则引发 DeprecationWarning(弃用警告)

Undeprecated modules
====================

未知

Copyright
=========

本文档已置于公共域名

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   coding: utf-8  
   End:  
