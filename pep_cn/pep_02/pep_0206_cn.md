# @Pep     : 0206
# @Date    : 2018-04-15 09:00:00
# @Author  : Canon
# @Link    : https://www.python.org
# @Version : 3.6.1


PEP: 206
Title: Python Advanced Library
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Withdrawn
Type: Informational
Content-Type: text/x-rst
Created:
Post-History:


Introduction
============

本 PEP 描述了 Python 高级库, 这是一组高质量且经常使用的第三方扩展模块.


Batteries Included Philosophy
=============================

Python 源代码分发长期以来一直保持着 "包含组" 的理念 -- 拥有丰富而通用的标准库,
可以立即使用, 无需用户下载单独的软件包. 这使 Python 语言在许多项目中处于领先地位.

但是, 标准库模块并不总是最适合工作的选择. 有些库模块是快速破解 (例如, ``calendar``, ``commands``),
有些是设计得很差, 现在几乎不可能修复 (``cgi``), 有些已被其他人淘汰了,
更完整的模块 ( ``binascii`` 提供与 ``binhex``, ``uu``, ``base64`` 模块相同的功能.
此 PEP 描述了第三方模块列表, 这些模块使 Python 在各种应用程序域中更具竞争力, 形成了 Python 高级库.

交付成果是一组脚本, 用于检索, 构建和安装特定应用程序域的包.
Python Package Index 现在包含足够的信息, 让软件自动查找软件包并下载它们,
因此实现这一目标的时机已经成熟.

目前, 本文档并未建议 *从标准库中删除* 模块, 这些模块已被第三方模块取代.
这很难做到, 因为它需要许多向后兼容性问题, 因此现在不值得烦恼.

请建议其他感兴趣的域名.


Domain: Web tasks
=================

XML parsing: ElementTree + SAX.

URL retrieval: libcurl? other possibilities?

HTML parsing: mxTidy? HTMLParser?

Async network I/O: Twisted

RDF parser: ???

HTTP serving: ???

HTTP cookie processing: ???

Web framework: A WSGI gateway, perhaps?  Paste?

Graphics: PIL, Chaco.


Domain: Scientific Programming
==============================

Numeric: Numeric, SciPy

Graphics: PIL, Chaco.


Domain: Application Development
===============================

GUI toolkit: ???

Graphics: Reportlab for PDF generation.


Domain: Education
=================

Graphics: PyGame


Software covered by the GNU General Public License
==================================================

其中一些第三方模块受 GNU 通用公共许可证和 GNU 宽通用公共许可证的约束.
提供下载和安装此类软件包的脚本, 或者甚至将所有这些软件包组装到单个 tarball 或 CD-ROM 中,
根据许可证的 "仅仅聚合" 条款, 不应该对 GPL 造成任何困难.


Open Issues
===========

其他哪些应用领域很重要?

这应该只是一组 Ubuntu 或 Debian 软件包吗?
编译诸如 PyGame 之类的东西可能非常复杂, 并且可能太难以自动化.


Acknowledgements
================

PEP 基于 Moshe Zadka 早期的 PEP 草案, 标题为
"2.0 Batteries Included."


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
