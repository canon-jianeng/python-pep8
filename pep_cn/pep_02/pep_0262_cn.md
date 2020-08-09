
PEP: 262
Title: A Database of Installed Python Packages
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 08-Jul-2001
Post-History: 27-Mar-2002

Introduction
============

此 PEP 描述了系统上安装的 Python 软件数据库的格式.

(在本文档中, 术语 "分布" 用于表示一组开发和分布在一起的代码.
"发行版" 与 Red Hat 或 Debian 包相同, 但术语 "包" 在 Python 术语中已经有了含义,
意思是 "一个带有 ``__init__.py`` 文件的目录.")


Requirements
============

我们需要一种方法来确定系统上安装了哪些发行版以及这些发行版的哪些版本.
我们希望提供类似于 CPAN, APT 或 RPM 的功能. 应该支持的必需用例是:

* 分布 X 是否在系统上?
* 安装了什么版本的分发 X?
* 新版本的 X 版本在哪里可以找到?
  (这可以定义为 "用户可以去寻找下载链接的主页",
  或 "程序可以找到最新版本的地方?" 两者都应该被支持.)
* 分发 X 在我的系统上放了什么文件?
* x/y/z.py 文件的来源是什么?
* 有人在本地修改过 x/y/z.py?
* 该软件还需要哪些其他发行版?
* 这个发行版提供了哪些 Python 模块?


Database Location
=================

数据库存在于一堆文件下
<prefix>/lib/python<version>/install-db/.
此位置将通过此 PEP 的其余部分调用 INSTALLDB.

数据库的结构有意保持简单;
此目录中的每个文件或其子目录 (如果有) 描述单个发布.
然后, 通过将相应的文件安装到 INSTALLDB 目录中,
可以更新 Python 的 Python 软件的二进制包装.

扫描子目录的基本原理是, 如果数据库目录包含太多条目,
我们可以转移到基于目录的索引方案.
例如, 这将让我们透明地从 INSTALLDB/Numeric 切换到
INSTALLDB/N/Nu/Numeric 或类似的散列方案.


Database Contents
=================

INSTALLDB 或其子目录中的每个文件都描述了一个发布, 并具有以下内容:

列出此文件中各节的初始行, 以空格分隔. 目前,
这将始终是 'PKG-INFO FILES REQUIRES PROVIDES'.
这是为了面向未来; 如果我们添加一个新的部分, 例如列出文档文件,
那么我们将添加一个 DOCS 部分并将其列在内容中. 截面始终用空行分隔.

使用 Distutils 进行安装的发行版应该自动更新数据库.
推送自己安装的发行版必须使用数据库的 API 手动添加或更新自己的条目.
系统包管理器 (如 RPM 或 pkgadd) 只能在 INSTALLDB 目录中创建新文件.

文件的每个部分用于不同的目的.

PKG-INFO section
----------------

包含文件分发信息的初始 RFC-822 标题头集, 如 PEP 241,
"Python 软件包的元数据" 中所述.

FILES section
-------------

分发安装的每个文件的条目. 生成的文件 (如 .pyc 和 .pyo 文件) 列在此列表中,
以及由分发安装的原始 .py 文件; 但是, 他们的校验和不会被存储或检查.

每个文件的条目都是一个制表符分隔的行, 其中包含以下字段:

* 文件的完整路径, 安装在系统上.

* 文件的大小

* 该文件的权限. 在 Windows 上, 此字段将始终为 'unknown'

* 文件的所有者和组, 由选项卡分隔. 在 Windows 上, 这些字段都是 'unknown'.

* 文件的 SHA1 摘要, 以十六进制编码. 对于生成的文件, 例如 \*.pyc 文件,
  此字段必须包含字符串 " - ", 表示不应该验证文件的校验和.


REQUIRES section
----------------

此部分是一个字符串列表, 提供此模块分发正常运行所需的服务.
此列表包括分发名称 ("python-stdlib") 和模块名称
("rfc822", "htmllib", "email", "email.Charset").
它将由 ``distutils.core.setup()`` 函数的额外 'requires' 参数指定.
例如::

    setup(..., requires=['xml.utils.iso8601',

最终可能会有自动化工具查看所有代码并生成需求列表,
但这些工具不太可能处理所有可能的情况;
始终需要手动指定要求的方式.


PROVIDES section
----------------

此部分是一个字符串列表, 提供已安装发行版提供的服务.
此列表包括分发名称 ("python-stdlib") 和模块名称
("rfc822", "htmllib", "email", "email.Charset").

XXX 应该列出文件吗? 例如 $PREFIX/lib/color-table.txt,
用于获取数据文件, 所需脚本等.

最终可能有一个选项让模块开发人员将自己的字符串添加到此部分. 例如,
您可以在此部分添加 "XML 解析器", 然后其他模块发行版可以将 "XML 解析器"
列为其依赖项之一, 来指示可以使用多个不同的 XML 解析器.
目前这种能力不受支持, 因为它引发了太多问题:
我们是否需要一个合法字符串的中心的注册表,
或者只是让人们放任何他们喜欢的东西?  等等., 等等...


API Description
===============

有一个基础类, InstallationDatabase. 它的代码位于 distutils/install_db.py 中.
(XXX 对标准库中的备用位置或备用模块名称的任何建议?)

InstallationDatabase 返回 Distribution 的实例, 其中包含有关已安装分发的所有信息.

XXX Distribution 中的几个字段与 distutils.dist.Distribution 中的字段重复.
可能它们应该被分解到这里提出的 Distribution 类中, 但是这可以以向后兼容的方式完成?

InstallationDatabase 具有以下界面::

    class InstallationDatabase:
        def __init__ (self, path=None):
            """InstallationDatabase(path:string)
            Read the installation database rooted at the specified path.
            If path is None, INSTALLDB is used as the default.
            """

        def get_distribution (self, distribution_name):
            """get_distribution(distribution_name:string) : Distribution
            Get the object corresponding to a single distribution.
            """

        def list_distributions (self):
            """list_distributions() : [Distribution]
            Return a list of all distributions installed on the system,
            enumerated in no particular order.
            """

        def find_distribution (self, path):
            """find_file(path:string) : Distribution
            Search and return the distribution containing the file 'path'.
            Returns None if the file doesn't belong to any distribution
            that the InstallationDatabase knows about.
            XXX should this work for directories?
            """

    class Distribution:
        """Instance attributes:
        name : string
          Distribution name
        files : {string : (size:int, perms:int, owner:string, group:string,
                           digest:string)}
           Dictionary mapping the path of a file installed by this distribution
           to information about the file.

        The following fields all come from PEP 241.

        version : distutils.version.Version
          Version of this distribution
        platform : [string]
        summary : string
        description : string
        keywords : string
        home_page : string
        author : string
        author_email : string
        license : string
        """

        def add_file (self, path):
            """add_file(path:string):None
            Record the size, ownership, &c., information for an installed file.
            XXX as written, this would stat() the file.  Should the size/perms/
            checksum all be provided as parameters to this method instead?
            """

        def has_file (self, path):
            """has_file(path:string) : Boolean
            Returns true if the specified path belongs to a file in this
            distribution.
            """

         def check_file (self, path):
            """check_file(path:string) : Boolean
            Checks whether the file's size, checksum, and ownership match,
            returning true if they do.
            """

Deliverables
============

要添加到此 PEP 的数据库 API 的说明.

对 Distutils 的修补程序
1) 实现 InstallationDatabase 类,
2) 安装新分发时更新数据库.
3) 添加一个简单的包管理工具, 将要添加到此 PEP 的功能.
   (或者应该是一个单独的 PEP?) 参见当前补丁的 [2]_.


Open Issues
===========

PJE 建议安装数据库 "可能存在于 sys.path 中的每个目录中, 内容以 sys.path 顺序合并.
这将允许主目录或其他备用位置安装工作, 并简化 distutils 安装的过程命令写入文件."
不错的功能: 它确实意味着包管理器工具可以考虑用户私有安装的 Python 包.

AMK 想知道: 如果告诉安装包到不在 sys.path 上的目录, setup.py 会怎么做?
它是否将 install-db 目录编写到它要写入的目录中, 或者它什么都不做?

包数据库文件本身是否应该包含在文件列表中?
(PJE 会认为是, 但当然它不能包含自己的校验和.
AMK 无法想到包含 DB 文件的用例.)

PJE 想知道在安装任何其他文件之前首先编写包 DB 文件,
利于失败的部分安装都可以被撤消, 并被识别为已损坏.
该 PEP 可能必须指定一些用于识别这种情况的算法.

我们是否应该保证安装数据库的格式在 Python 版本中保持兼容,
还是可以随意更改? 可能我们需要保证兼容性.


Rejected Suggestions
====================

可以使用一个大文本文件或 anydbm 文件, 而不是每个分发使用一个文本文件.
由于一些原因, 这被拒绝了. 首先, 性能可能不是一个非常紧迫的问题,
因为数据库仅在安装或删除软件时使用, 这是一项相对不常见的任务.
可扩展性也可能不是问题, 因为人们可能安装了数百个 Python 软件包,
但似乎不可能有数千或数万个. 最后, 单个文本文件与 RPM 或 DPKG 等安装程序兼容,
因为二进制打包程序可以将新数据库文件放入数据库目录中.
如果使用了一个大型文本文件或二进制文件, 则必须通过运行 postinstall 脚本来更新 Python 数据库.

在 Windows 上, 不存储文件的权限和所有者或组. Windows 确实支持所有权和访问权限,
但是读取和设置它们需要 win32all 扩展, 并且它们不存在于 Windows 的基本 Python 安装程序中.


References
==========

.. [1] Michael Muller 的补丁 (1999年12月28日左右发布到 Distutils-SIG) 生成了已安装文件的列表.

.. [2] 实现此 PEP 的补丁将在 SourceForge 上作为补丁 #562100 进行跟踪.
       http://www.python.org/sf/562100.
	   实现安装数据库的代码目前位于 nondist/sandbox/pep262 目录中的 Python CVS 中.


Acknowledgements
================

这个 PEP 的想法最初来自 Greg Ward, Fred L. Drake Jr., Thomas Heller,
Mats Wichmann, Phillip J. Eby 和其他人的帖子.

Distutils SIG 的读者建议对本文档进行许多更改和重写.


Copyright
=========

This document has been placed in the public domain.

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
