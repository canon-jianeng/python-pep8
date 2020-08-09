
PEP: 241
Title: Metadata for Python Software Packages
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 12-Mar-2001
Post-History: 19-Mar-2001


Introduction
============

此 PEP 描述了向 Python 包添加元数据的机制.
它包括字段名称的详细信息, 以及它们的语义和用法.


Including Metadata in Packages
==============================

将修改 Distutils 'sdist' 命令来从参数中提取元数据字段,
并将它们写入生成的 zipfile 或 tarball 中的文件.
此文件将命名为 PKG-INFO, 并将放置在源代码发行版的顶级目录中
(README, INSTALL 和其他文件通常位于此目录中).

开发人员可能不会提供自己的 PKG-INFO 文件. 如果 "sdist" 命令检测到现有的 PKG-INFO 文件,
则会以适当的错误消息终止. 这可以防止 PKG-INFO 和 setup.py 文件不同步导致的混淆.

PKG-INFO 文件格式是 rfc822.py 模块可解析的一组 RFC-822 标头.
以下部分中列出的字段名称用作标题名称. 这种简单的格式没有扩展机制;
Catalog 和 Distutils SIG 将致力于为 Python 2.2 提供更灵活的格式.


Fields
======

此部分指定每个受支持的元数据字段的名称和语义.

标记为 "(多次使用)" 的字段可以在单个 PKG-INFO 文件中多次指定.
其他字段只能在 PKG-INFO 文件中出现一次. 标有 "(可选)" 的字段
不需要出现在有效的 PKG-INFO 文件中, 所有其他字段必须存在.

Metadata-Version
----------------

文件格式的版本; 目前 "1.0" 是这里唯一的合法值.

例如 ::

    Metadata-Version: 1.0

Name
----

包的名称.

例如 ::

    Name: BeagleVote

Version
-------

包含程序包版本号的字符串. 该字段应该由 distutils.version 模块中的一个 Version 类
(StrictVersion 或 LooseVersion) 解析.

例如 ::

    Version: 1.0a2

Platform (multiple use)
-----------------------

以逗号分隔的平台规范列表, 总结了程序包支持的操作系统.
下面列出了主要支持的平台, 但此列表必然不完整.

::

    POSIX, MacOS, Windows, BeOS, PalmOS.

二进制发行版将在其元数据中使用 Supported-Platform 字段来指定编译二进制包的操作系统和 CPU.
本 PEP 中未指定 Supported-Platform 的语义.

例如 ::

    Platform: POSIX, Windows

Summary
-------

包的功能的一行摘要.

例如 ::

    Summary: A module for collecting votes from beagles.

Description (optional)
----------------------

可以运行到几个段落的包的更长描述.
(处理元数据的软件不应该假定该字段的任何最大大小,
但是人们希望人们不会将他们的说明手册作为较长描述.)

例如 ::

    Description: This module collects votes from beagles
                 in order to determine their electoral wishes.
                 Do NOT try to use this module with basset hounds;
                 it makes them grumpy.

Keywords (optional)
-------------------

用于帮助在更大的目录中搜索包的其他关键字的列表.

例如 ::

    Keywords: dog puppy voting election

Home-page (optional)
--------------------

包含程序包主页 URL 的字符串.

例如 ::

    Home-page: http://www.example.com/~cschultz/bvote/

Author (optional)
-----------------

一个至少包含作者姓名的字符串. 还可以添加联系信息, 使用换行符分隔每一行.

例如 ::

    Author: C. Schultz
            Universal Features Syndicate
            Los Angeles, CA

Author-email
------------

包含作者电子邮件地址的字符串. 它可以包含 RFC-822 "发件人:" 标题的法律表单中的名称和电子邮件地址.
它不是可选的, 因为编目系统可以使用该字段的电子邮件部分作为代表作者的唯一键.
目录可以使作者能够存储他们的 GPG 密钥, 个人主页和关于作者的其他元数据,
以及可选地将多个电子邮件地址与同一个人相关联的能力. 本 PEP 不涵盖与作者相关的元数据字段.

例如 ::

    Author-email: "C. Schultz" <cschultz@example.com>

License
-------

从一个简短的选项列表中选择的字符串, 指定涵盖包的许可证. 某些许可证导致软件可以自由再分发,
因此包装商和经销商可以自动知道他们可以自由地重新分发软件.
其他许可证需要人员仔细阅读才能确定如何重新打包和转售软件.

选择是 ::

    Artistic, BSD, DFSG, GNU GPL, GNU LGPL, "MIT",
    Mozilla PL, "public domain", Python, Qt PL, Zope PL, unknown,
    nocommercial, nosell, nosource, shareware, other

Definitions of some of the licenses are:

=============  ===================================================
DFSG           The license conforms to the Debian Free Software
               Guidelines, but does not use one of the other
               DFSG conforming licenses listed here.
               More information is available at:
               http://www.debian.org/social_contract#guidelines

Python         Python 1.6 or higher license.  Version 1.5.2 and
               earlier are under the MIT license.

public domain  Software is public domain, not copyrighted.

unknown        Status is not known

nocommercial   Free private use but commercial use not permitted

nosell         Free use but distribution for profit by arrangement

nosource       Freely distributable but no source code

shareware      Payment is requested if software is used

other          General category for other non-DFSG licenses
=============  ===================================================

其中一些许可证可以解释为软件可以自由再分发.
可再发行许可证列表是 ::

    Artistic, BSD, DFSG, GNU GPL, GNU LGPL, "MIT",
    Mozilla PL, "public domain", Python, Qt PL, Zope PL,
    nosource, shareware

请注意, 可再发行并不意味着一个包符合自由软件的条件,
'nosource' 和 'shareware' 就是例子.

例如 ::

    License: MIT


Acknowledgements
================

Distutils SIG 的读者建议对本文档进行许多更改和重写.
特别是, Sean Reifschneider 经常提供实际文本来包含在本 PEP 中.

许可证列表使用 SourceForge 许可证列表和 Graham Williams 编制的 CTAN 许可证列表进行编译;
Carey Evans 还在此列表中提供了一些有用的建议.


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
