
PEP: 256
Title: Docstring Processing System Framework
Version: $Revision$
Last-Modified: $Date$
Author: David Goodger <goodger@python.org>
Discussions-To: <doc-sig@python.org>
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 01-Jun-2001
Post-History: 13-Jun-2001


Rejection Notice
================

这个提议似乎已经失去了动力.


Abstract
========

Python 适用于内联文档. 凭借其内置的 docstring 语法, Python 中很容易
使用有限形式的 "Literate Programming`_". 但是, 没有令人满意的标准工具
来提取和处理 Python 文档字符串. 缺乏标准工具集是 Python 基础架构中的一个重大差距;
该 PEP 旨在填补这一空白.

围绕文档字符串处理的问题一直存在争议并且难以解决.
该 PEP 提出了一个通用的文档字符串处理系统 (DPS) 框架,
该框架将组件 (程序和概念) 分离出来, 通过共识 (一个解决方案)
或通过分歧 (许多) 来解决个别问题. 它促进了标准接口,
允许使用各种插件组件 (输入上下文读取器, 标记解析器和输出格式编写器).

DPS 框架的概念与实现细节无关.


Road Map to the Docstring PEPs
==============================

docstring 处理有很多方面. "Docstring PEP" 已经解决了这些问题,
利于单独地或尽可能接近地处理每个问题. 各个方面和相关的 PEP 如下:

* Docstring 语法. PEP 287, "reStructuredText Docstring Format" [#PEP-287]_,
  提出了 Python 文档字符串, PEP 和其他用法的语法.

* Docstring 语义至少包含两个方面:

  - 约定: docstrings 的高级结构. 在 PEP 257 中处理 "Docstring Conventions" [＃PEP-257]_.

  - 方法论: docstrings 的信息内容规则. 没有解决.

* 处理机制. 此 PEP (PEP 256) 概述了抽象文档字符串处理系统 (DPS) 的高级问题和规范. PEP 258,
  "Docutils 设计规范" [＃PEP-258]_, 概述了正在开发的一个 DPS 的设计和实施.

* 输出样式: 开发人员希望从源代码生成的文档看起来很好, 并且对于这意味着什么有很多不同的想法.
  PEP 258 涉及 "设计师改造". 文档字符串处理的这一方面尚未得到充分探索.

通过分离问题, 我们可以更容易地形成共识 (更小的斗争;-), 并更容易接受分歧.


Rationale
=========

其他一些语言有标准的内联文档系统. 例如, Perl 有 POD_ ("普通旧文档"), Java 有 Javadoc_,
但这些都没有 Pythonic 方式. POD 语法非常明确, 但在可读性方面取代了 Perl.
Javadoc 是以 HTML 为中心的; 除了 "``@field``" 标签外, 原始 HTML 用于标记.
还有一些通用工具, 如 Autoduck_ 和 Web_ (Tangle＆Weave), 可用于多种语言.

已经有很多尝试为 Python 编写自动文档系统 (不是详尽的列表):

- Marc-Andre Lemburg 的 doc.py_

- Daniel Larsson 的 pythondoc_ & gendoc_

- Doug Hellmann 的 HappyDoc_

- Laurence Tratt 的 Crystal (网络上不再提供)

- Ka-Ping Yee 的 pydoc_ (pydoc.py 现在是 Python 标准库的一部分; 见下文)

- Tony Ibbs 的 docutils_ (Tony 已将此名称捐赠给 `Docutils project'_)

- Edward Loper 的 STminus_ 形式化和相关的成就

这些系统各有不同的目标, 取得了不同程度的成功. 许多上述系统的一个问题是过度的野心和不灵活性.
他们提供了一组独立的组件: 文档字符串提取系统, 标记解析器, 内部处理系统和一个或多个具有固定样式的
输出格式编写器. 不可避免地, 每个系统的一个或多个方面都存在严重的缺点, 并且不容易扩展或修改它们,
从而阻止它们被用作标准工具.

至少对于这位作者来说, 已经很清楚 "全有或全无" 的方法不会成功,
因为所有有关方面都不可能达成单一的独立系统. 设计用于扩展的模块化组件方法
(其中组件可以多次实现) 可能是成功的唯一机会. 标准的组件间 API 将使 DPS 组件易于理解,
而无需详细了解整体, 降低贡献的障碍, 并最终导致丰富多样的系统.

文档字符串处理系统的每个组件都应该独立开发. 应该选择 "最佳" 系统,
或者从现有系统合并, 或重新开发. 该系统应该包含在 Python 的标准库中.


PyDoc & Other Existing Systems
------------------------------

从版本 2.1 开始, PyDoc 成为 Python 标准库的一部分.
它从 Python 交互式解释器中提取并显示文档字符串, 从 shell 命令行,
从 GUI 窗口到 Web 浏览器 (HTML). 虽然 PyDoc 是一个非常有用的工具,
但它有几个不足之处, 包括:

- 在 GUI/HTML 的情况下, 除了标识符名称的一些启发式超链接之外,
  不进行文档字符串的格式化. 它们在 ``<p> <small> <tt>`` 标签中显示,
  来避免不必要的换行. 不幸的是, 结果并不吸引人.

- PyDoc 从导入的模块对象中提取文档字符串和结构信息 (类标识符, 方法签名等).
  导入不受信任的代码涉及安全问题. 此外, 导入时来自源的信息会丢失, 例如注释,
  "附加文档字符串" (非文档字符串上下文中的字符串文字; 请参阅 PEP 258 [#PEP-258]_),
  以及定义的顺序.

在提供 HTML 页面时, 可以将此 PEP 中提出的功能添加到 PyDoc 或由 PyDoc 使用.
建议的文档字符串处理系统的功能远不止 PyDoc 目前的形式. 可以开发一个独立的工具
(PyDoc 可能使用也可能不使用), 或者可以扩展 PyDoc 来包含此功能, 并成为文档字符串处理系统
(或一个这样的系统). 该决定超出了本 PEP 的范围.

类似地, 对于其他现有文档字符串处理系统, 其作者可能会也可能不会选择与此框架的兼容性.
但是, 如果该框架被接受并作为 Python 标准采用, 则兼容性将成为这些系统未来的重要考虑因素.


Specification
=============

文档字符串处理系统框架分解如下:

1. 文档字符串约定. 文件问题如下:

   - 应该记录在哪里.

   - 第一行是单行概要.

   PEP 257 [#PEP-257]_ 记录其中的一些问题.

2. 文档字符串处理系统设计规范. 文件问题如下:

   - 高级规范: DPS 的作用.

   - 可执行脚本的命令行界面.

   - 系统 Python API.

   - 文档字符串提取规则.

   - Readers 封装了输入上下文.

   - 解析器.

   - readersDocument 树: 中间内部数据结构.
     解析器和读取器的输出以及 Writer 的输入都共享相同的数据结构.

   - 转换, 修改文档树.

   - 输出格式的 Writers .

   - 处理输出管理的分配者 (一个文件, 许多文件或内存中的对象).

   这些问题适用于任何文档字符串处理系统实现.
   PEP 258 [＃PEP-258]_ 记录了这些问题.

3. Docstring 处理系统实现.

4. 输入标记规范: docstring 语法.
    PEP 287 [＃PEP-287]_ 提出了标准语法.

5. 输入解析器实现.

6. 输入上下文阅读器 ("模式": Python 源代码, PEP, 独立文本文件, 电子邮件等) 和实现.

7. Stylists: 某些输入上下文阅读器可能具有关联的 Stylists, 允许各种输出文档样式.

8. 输出格式 (HTML, XML, TeX, DocBook, info 等) 和编辑器实现.

组件 1, 2/3/5 和 4 是个别伴随 PEP 的主题. 如果存在框架或 语法/解析器 的另一个实现,
则可能需要额外的 PEP. 将需要组件 6 和 7 中的每一个的多个实现; 对于这些组件, PEP 机制可能过度.


Project Web Site
================

已经为此项工作设立了 SourceForge 项目
http://docutils.sourceforge.net/.


References and Footnotes
========================

.. [#PEP-287] PEP 287, reStructuredText 文档字符串格式, Goodger
   (http://www.python.org/dev/peps/pep-0287/)

.. [#PEP-257] PEP 257, Docstring 约定, Goodger, Van Rossum
   (http://www.python.org/dev/peps/pep-0257/)

.. [#PEP-258] PEP 258, Docutils 设计规范, Goodger
   (http://www.python.org/dev/peps/pep-0258/)

.. _Literate Programming: http://www.literateprogramming.com/

.. _POD: http://www.perldoc.com/perl5.6/pod/perlpod.html

.. _Javadoc: http://java.sun.com/j2se/javadoc/

.. _Autoduck:
   http://www.helpmaster.com/hlp-developmentaids-autoduck.htm

.. _Web: http://www-cs-faculty.stanford.edu/~knuth/cweb.html

.. _doc.py:
   http://www.egenix.com/files/python/SoftwareDescriptions.html#doc.py

.. _pythondoc:
.. _gendoc: http://starship.python.net/crew/danilo/pythondoc/

.. _HappyDoc: http://happydoc.sourceforge.net/

.. _pydoc: http://docs.python.org/library/pydoc.html

.. _docutils: http://www.tibsnjoan.co.uk/docutils.html

.. _Docutils project: http://docutils.sourceforge.net/

.. _STMinus: http://www.cis.upenn.edu/~edloper/pydoc/

.. _Python Doc-SIG: http://www.python.org/sigs/doc-sig/


Copyright
=========

This document has been placed in the public domain.


Acknowledgements
================

This document borrows ideas from the archives of the `Python
Doc-SIG`_.  Thanks to all members past & present.



..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   End:  
