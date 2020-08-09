
PEP: 258
Title: Docutils Design Specification
Version: $Revision$
Last-Modified: $Date$
Author: David Goodger <goodger@python.org>
Discussions-To: <doc-sig@python.org>
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Requires: 256, 257
Created: 31-May-2001
Post-History: 13-Jun-2001


================
Rejection Notice
================

虽然这可以作为现在独立的文档的有趣的设计文档, 但它不再被包含在标准库中.


==========
 Abstract
==========

此 PEP 记录了 Docutils 的设计问题和实现细节, Docutils 是一个 Python 文档字符串处理系统 (DPS). PPS 256,
"文档字符串处理系统框架" [＃PEP-256]_ 中记录了 DPS 的基本原理和高级概念.
另请参阅 PEP 256 来获取 "Road Map to the Docstring PEPs".

Docutils 采用模块化设计, 因此可轻松更换任何组件.
此外, Docutils 不仅限于处理 Python 文档字符串;
它在几种情况下也处理独立文档.

此 PEP 不需要更改核心 Python 语言. 其可交付成果包括标准库及其文档的包.


===============
 Specification
===============

Docutils Project Model
======================

项目组件和数据流::

                     +---------------------------+
                     |        Docutils:          |
                     | docutils.core.Publisher,  |
                     | docutils.core.publish_*() |
                     +---------------------------+
                      /            |            \
                     /             |             \
            1,3,5   /        6     |              \ 7
           +--------+       +-------------+       +--------+
           | READER | ----> | TRANSFORMER | ====> | WRITER |
           +--------+       +-------------+       +--------+
            /     \\                                  |
           /       \\                                 |
     2    /      4  \\                             8  |
    +-------+   +--------+                        +--------+
    | INPUT |   | PARSER |                        | OUTPUT |
    +-------+   +--------+                        +--------+

每个组件上方的数字表示文档数据的路径. Reader＆Parser 之间
以及 Transformer＆Writer 之间的双宽线表示沿这些路径发送的数据
应该是标准的 (纯的和未扩展的) Docutils doc 树.
单宽度线表示内部树扩展或完全不相关的表示是可能的,
但它们必须在两端都受支持.


Publisher
---------

``docutils.core`` 模块包含一个 "Publisher" facade 类和几个 convenience 函数:
"publish_cmdline()" (用于命令行前端), "publish_file()" (用于编程用于类文件 I/O)
和 "publish_string()" (用于字符串 I/O 的程序化使用).
Publisher 类封装了  Docutils 系统的高级逻辑. Publisher 类全面负责处理,
由 ``Publisher.publish()`` 方法控制:

1. 设置内部设置 (可能包括配置文件和命令行选项) 和 I/O 对象.

2. 调用 Reader 对象来从源 Input 对象读取数据, 并使用 Parser 对象解析数据.
    返回文档对象.

3. 通过附加到文档的 Transformer 对象设置和应用变换.

4. 调用 Writer 对象, 该对象将文档转换为最终输出格式, 并将格式化数据写入目标 Output 对象.
    根据 Output 对象, 输出可以从 Writer 返回, 然后从 ``publish()`` 方法返回.

使用组件名称调用 "发布" 功能 (或实例化 "Publisher" 对象) 将导致默认行为. 对于自定义行为
(自定义组件设置), 首先创建自定义组件对象, 并将它们传递给 Publisher 或 ``publish_ *`` 便利函数.


Readers
-------

Readers 理解输入上下文 (数据来自哪里), 将整个输入或离散的 "块" 发送到解析器,
并提供上下文来将块绑定在一起形成一个有凝聚力的整体.

每个 reader 都是一个模块或包, 导出带有 "read" 方法的 "Reader" 类.
基础 "Reader" 类可以在 ``docutils/readers/__init__.py`` 模块中找到.

大多数 Readers 都必须被告知要使用哪种解析器. 到目前为止 (参见下面的示例列表),
只有 Python Source Reader ("PySource"; 仍然不完整) 才能自行确定解析器.

Responsibilities:

* 从源 I/O 获取输入文本.

* 将输入文本与新的 `document tree`_ root 一起传递给解析器.

Examples:

* 独立 (原始或普通): 只需读取文本文件并进行处理即可.
  需要告诉 reader 使用哪个解析器.

  "Standalone Reader" 已在模块 ``docutils.readers.standalone`` 中实现.

* Python Source: 请参阅下面的 "Python Source Reader".
  该 Reader 目前正在 Docutils 沙箱中开发.

* 电子邮件: RFC-822 标题, 引用的摘录, 签名, MIME 部分.

* PEP: RFC-822 标题头, "PEP xxxx" 和 "RFC xxxx" 转换为 URI.
   "PEP Reader" 已在模块 ``docutils.readers.pep`` 中实现;
  见 PEP 287 和 PEP 12.

* Wiki: 整合到变换中的 "wiki links" 的全局参考查找.
  (仅限 CamelCase 或不受限制?) 懒惰的缩进?

* 网页: 作为独立页面, 但将元字段识别为元标记. 支持某种模板?
  (在 ``<body>`` 之后, ``</ body>`` 之后?)

* FAQ: 结构化的 "question & answer(s)" 结构.

* 复合文档: 将章节合并为一本书. 主要清单文件?


Parsers
-------

解析器分析他们的输入并生成 Docutils `document tree`_.
他们不了解或关心数据的来源或目的地.

每个输入解析器都是使用 "parse" 方法导出 "Parser" 类的模块或包.
基础 "Parser" 类可以在 ``docutils/parsers/__init__.py`` 模块中找到.

职责: 给定原始输入文本和 doctree 根节点, 通过解析输入文本填充 doctree.

示例: 到目前为止, 唯一实现的解析器是 reStructuredText 标记.
它在 ``docutils/parsers/rst/`` 包中实现.

可以并鼓励其他解析器的开发和集成.


.. _transforms:

Transformer
-----------

Transformer 类在 ``docutils/transforms/__init__.py`` 中存储转换并将它们应用于文档.
转换器对象附加到每个新文档树. Publisher_ 调用 ``Transformer.apply_transforms()``
将所有存储的转换应用于文档树. 转换将文档树从一种形式更改为另一种形式, 添加到树中或修整它.
转换解析引用和脚注编号, 处理解释的文本以及执行其他上下文相关的处理.

某些转换特定于组件 (读取器, 分析器, 写入器, 输入, 输出).
标准组件特定的转换在组件类的 ``default_transforms`` 属性中指定.
Reader 完成处理后, Publisher_ 用一个组件列表调用
``Transformer.populate_from_components()`` 并存储所有默认转换.

每个转换都是 ``docutils/transforms/`` 包中的一个类, 它是 ``docutils.transforms.Transform`` 的子类.
转换类每个都有一个 ``default_priority`` 属性, Transformer 使用该属性按顺序应用转换 (从低到高).
将转换添加到 Transformer 对象时, 可以覆盖默认优先级.

Transformer responsibilities:

* 按优先级顺序将转换应用于文档树.

* 存储组件类型名称 ('reader', 'writer' 等) 到组件对象的映射.
  某些转换 (例如, "components.Filter") 使用它们来确定适用性.

Transform responsibilities:

* 原地修改 doctree, 或者将一个结构纯粹转换为另一个结构,
  或者根据 doctree 或外部数据添加新结构.

转换的例子 (在 ``docutils/transforms/`` 包中):

* frontmatter.DocInfo: 文档元数据的转换 (书目信息).

* references.AnonymousHyperlinks: 对相应目标的匿名引用的解析.

* parts.Contents: 生成文档的目录.

* document.Merger: 将多个填充的 doctrees 组合成一个. (尚未实施或完全理解.)

* document.Splitter: 将文档拆分为子文档的树结构, 可能按节.
  它必须适当地转换引用. (两者的实施都没有被远程理解.)

* components.Filter: 包含或排除依赖于特定 Docutils 组件的元素.


Writers
-------

Writers 产生最终输出 (HTML, XML, TeX 等).
Writers 将内部 `document tree`_ 结构转换为最终数据格式,
可能首先运行特定于 Writer 的 transforms_.

当文档到达 Writer 时, 它应该是最终形式. Writer 的工作只是 (并且仅)
从 Docutils doctree 结构转换为目标格式. 可能需要一些小的转换,
但它们应该是本地的和格式特定的.

每个 writer 都是一个模块或包, 它使用 "write" 方法导出 "Writer" 类.
基础 "Writer" 类可以在 ``docutils/writers/__init__.py`` 模块中找到.

Responsibilities:

* 将 doctree 转换为特定的输出格式.

  - 将引用转换为本地格式表单.

* 将已转换的输出写入目标 I/O.

Examples:

* XML: 各种形式, 如下:

  - Docutils XML (内部文档树的表达式, 实现为 ``docutils.writers.docutils_xml``).

  - DocBook (正在 Docutils 沙箱中实施).

* HTML (XHTML 实现为 ``docutils.writers.html4css1``).

* PDF (正在 Docutils 沙箱中开发 ReportLabs 界面).

* TeX (正在沙箱中实现 LaTeX Writer).

* Docutils-native pseudo-XML (实现为 ``docutils.writers.pseudoxml``, 用于测试).

* Plain text

* reStructuredText?


Input/Output
------------

I/O 类为低级输入和输出提供统一的 API. 子类将存在于各种输入/输出机制中.
但是, 它们可以被视为实现细节. 使用与 Publisher_ 关联的一个便利功能应该满足大多数应用程序.

I/O 类目前处于初步阶段; 还有很多工作要做. 问题:

* 如何在 API 中表示多文件输入 (文件和目录)?

* 如何表示多文件输出? 也许是 "Writer" 变体, 每个输出分布类型一个? 或输出具有关联转换的对象?

Responsibilities:

* 从输入源读取数据 (输入对象) 或将数据写入输出目标 (输出对象).

Examples of input sources:

* 磁盘或流上的单个文件 (实现为 ``docutils.io.FileInput``).

* 磁盘上有多个文件 (``MultiFileInput``?).

* Python 源文件: 模块和包.

* 从客户端应用程序接收的 Python 字符串
  (实现为 ``docutils.io.StringInput``).

Examples of output destinations:

* 磁盘或流上的单个文件 (实现为 ``docutils.io.FileOutput``).

* 磁盘上的目录和文件树.

* 一个 Python 字符串, 返回给客户端应用程序
  (实现为 ``docutils.io.StringOutput``).

* 没有输出; 对于只使用一部分正常输出的程序化应用程序很有用
  (实现为 ``docutils.io.NullOutput``).

* 内存中的单个树形数据结构.

* 内存中的其他一些数据结构.


Docutils Package Structure
==========================

* 包 "docutils".

  - 模块 "__init__.py" 包含: 类 "Component", Docutils 组件的基类;
    class "SettingsSpec", 一个用于指定运行时设置的基类
	(由 docutils.frontend 使用); 和类 "TransformSpec", 一个用于指定转换的基类.

  - 模块 "docutils.core" 包含 facade 类 "Publisher" 和便利功能. 参见上面的 `Publisher`_.

  - 模块 "docutils.frontend" 提供运行时设置支持, 用于编程使用和前端工具
    (包括配置文件支持, 命令行参数和选项处理).

  - 模块 "docutils.io" 为低级输入和输出提供统一的 API. 参见上面的 "输入/输出".

  - 模块 "docutils.nodes" 包含 Docutils 文档树元素类库以及树遍历访问者模式基类.
    请参阅下面的 "文档树".

  - 模块 "docutils.statemachine" 包含专用于基于正则表达式的文本过滤器和解析器的有限状态机.
    reStructuredText 解析器实现基于此模块.

  - 模块 "docutils.urischemes" 包含已知 URI 方案的映射 ("http", "ftp", "mail" 等).

  - 模块 "docutils.utils" 包含实用程序函数和类, 包括记录器类 ("Reporter"; 请参阅下面的 `Error Handling`_).

  - 包 "docutils.parsers": 标记 parsers_.

    - 函数 "get_parser_class(parser_name)" 按名称返回解析器模块.
	  "Parser" 类是特定解析器的基类. (``docutils/parsers/__init__.py``)

    - 包 "docutils.parsers.rst": reStructuredText 解析器.

    - 可以添加备用标记解析器.

    请见上面的 `Parsers`_.

  - 包 "docutils.readers": 上下文感知输入 readers.

    - 函数 "get_reader_class(reader_name)" 按名称或别名返回读取器模块.
	  "Reader" 类是特定 readers 的基类. (``docutils/readers/__init__.py``)

    - 模块 "docutils.readers.standalone" 读取独立的文档文件.

    - 模块 "docutils.readers.pep" 读取 PEP (Python 增强建议).

    - 要添加的 Readers: Python 源代码 (结构和文档字符串), 电子邮件,
	  常见问题解答, 以及 Wiki 和其他人.

    请见上面的 `Readers`_.

  - 包 "docutils.writers": 输出格式化的 writers.

    - 函数 "get_writer_class(writer_name)" 按名称返回 writer 模块.
	  "Writer" 类是特定 writers 的基类. (``docutils/writers/__init__.py``)

    - 模块 "docutils.writers.html4css1" 是一个简单的超文本标记语言文档树编辑器,
	  适用于 HTML 4.01 和 CSS1.

    - 模块 "docutils.writers.docutils_xml" 以 XML 格式写入内部文档树.

    - 模块 "docutils.writers.pseudoxml" 是一个简单的内部文档树编辑器;
	  它写了缩进的伪 XML.

    - 要添加的 Writers: HTML 3.2 或 4.01-loose, XML (各种表单, 如 DocBook),
	  PDF, TeX, plaintext, reStructuredText, 以及可能其余的.

    请见上面的 `Writers`_.

  - 包 "docutils.transforms": 树转换类.

    - "Transformer" 类存储转换并将它们应用于文档树.
	  (``docutils/transforms/__init__.py``)

    - "Transform" 类是特定转换的基类.
      (``docutils/transforms/__init__.py``)

    - 每个模块都包含相关的转换类.

    请见上面的 `Transforms`_.

  - 包 "docutils.languages": 语言模块包含依赖于语言的字符串和映射. 它们以其语言标识符命名
    (在下面的 "选择文档字符串格式" 中定义), 将破折号转换为下划线.

    - 函数 "get_language(language_code)", 返回匹配的语言模块.
	  (``docutils/languages/__init__.py``)

    - 模块: en.py (英语), de.py (德语), fr.py (法语), it.py (意大利语), sk.py (斯洛伐克语), sv.py (瑞典语).

    - 其他语言要添加.

* 第三方模块: "extras" 目录. 只有在 Python 安装中不存在这些模块时才会安装这些模块.

  - ``extras/optparse.py`` 和 ``extras/textwrap.py`` 提供选项解析和命令行帮助; 来自 Greg Ward's
    http://optik.sf.net/ project, included for convenience.

  - ``extras/roman.py`` 包含罗马数字转换例程.


Front-End Tools
===============

``tools/`` 目录包含几个用于常见 Docutils 处理的前端.
有关详细信息, 请参阅 `Docutils Front-End Tools`_ .

.. _Docutils Front-End Tools:
   http://docutils.sourceforge.net/docs/user/tools.html


Document Tree
=============

Docutils 在内部使用单个中间数据结构, 在组件之间的接口中;
它在 ``docutils.nodes`` 模块中定义.
不需要在任何组件内部使用此数据结构, 只需在组件之间使用,
如上面 `Docutils Project Model`_ 中的图表所示.

允许自定义节点类型, 前提是 (a) 转换在它们到达 Writer 之前将它们转换为标准 Docutils 节点,
或者 (b) 某些 Writer 明确支持自定义节点, 并且包含在已过滤的 "挂起" 中的节点.
条件 (a) 的一个例子是 `Python Source Reader`_ (见下文), 其中 "stylist" 转换自定义节点.
HTML ``<meta>`` 标签是条件 (b) 的一个例子; 它由 HTML Writer 支持, 但不受其他人支持.
reStructuredText "meta" 指令创建一个 "挂起" 节点, 其中包含嵌入式 "元" 节点只能由 HTML 兼容的编辑器处理的知识.
"pending" 节点由 ``docutils.transforms.components.Filter`` 转换解析, 该转换检查调用编辑器是否支持 HTML;
如果没有, 则从文档中删除 "待定" 节点 (以及封闭的 "元" 节点).

文档树数据结构类似于 DOM 树, 但具有特定的节点名称 (类) 而不是 DOM 的通用节点.
该模式记录在 XML DTD (可扩展标记语言文档类型定义) 中, 它分为两部分:

* the Docutils Generic DTD, docutils.dtd_, 和

* the OASIS Exchange Table Model, soextbl.dtd_.

DTD 定义了一组丰富的元素, 适用于许多输入和输出格式.
DTD 保留重建原始输入文本或其合理传输所需的所有信息.

有关详细信息, 请参阅 `The Docutils Document Tree`_ (不完整).


Error Handling
==============

当解析器在标记中遇到错误时, 它会插入一个系统消息 (DTD 元素 "system_message").
有五个级别的系统消息:

* Level-0, "DEBUG": 内部报告问题. 对处理没有影响. 0级系统消息与其他消息分开处理.

* Level-1, "INFO": 一个可以忽略的小问题. 对处理几乎没有影响. 通常不会报告第1级系统消息.

* Level-2, "WARNING": 一个应该解决的问题. 如果忽略, 输出可能会出现轻微问题.
  通常会报告2级系统消息, 但不会停止处理

* Level-3, "ERROR": 一个应该解决的重大问题. 如果忽略, 输出将包含不可预测的错误.
  通常会报告3级系统消息, 但不会停止处理

* Level-4, "SEVERE": 必须解决的严重错误. 通常, 4级系统消息被转换为停止处理的异常.
  如果忽略, 输出将包含严重错误.

虽然最初的消息级别是独立设计的, 但它们与 "VMS 错误条件严重性级别" 有很强的对应关系.
级别 1 到 4 的引号中的名称是从 VMS 借来的. 从那以后, 错误处理受到了 `log4j project`_ 的影响.


Python Source Reader
====================

Python 源代码 Reader ("PySource") 是 Docutils 组件, 它读取 Python 源文件, 在上下文中提取文档字符串,
然后解析, 链接并将文档字符串组装成一个有凝聚力的整体. 它是一个重要且非平凡的组件,
目前正在 Docutils 沙箱中进行实验性开发. 这里介绍了高级设计问题.


Processing Model
----------------

这个模型将随着时间的推移而发展, 包括经验和发现.

1. PySource Reader 使用 Input 类将 Python 包和模块读入字符串树.

2. 解析 Python 模块, 将字符串树转换为带有 docstring 节点的抽象语法树的树.

3. 抽象语法树被转换为包或模块的内部表示. 提取文档字符串以及代码结构详细信息.
    参见下面的 'AST Mining`_. 构造命名空间在步骤6中进行查找.

4. 一次一个, 解析文档字符串, 生成标准的 Docutils 文档.

5. PySource 将所有单独的 docstrings 的 doctrees 组装成一个特定于 Python 的自定义 Docutils 树,
   与 package/module/class 结构并行; 这是一个自定义的特定于 Reader 的内部表示
   (请参阅 `Docutils Python Source DTD`_). 必须合并命名空间: Python 标识符, 超链接目标.

6. 根据 Python 命名空间查找规则解决从文档字符串 (解释文本) 到 Python 标识符的交叉引用.
    请参阅下面的 "标识符交叉引用".

7. "Stylist" 转换应用于自定义 doctree (通过 Transformer_),
    自定义节点使用标准节点作为基元进行渲染,
    并发出标准文档树. 请参阅下面的 'Stylist Transforms`_.

8. 其他转换由 Transformer_ 应用于标准 doctree.

9. 标准 doctree 被发送到 Writer, Writer 将文档转换为具体格式 (HTML, PDF 等).

10. Writer 使用 Output 类将结果数据写入其目标 (磁盘文件, 目录和文件等)。


AST Mining
----------

抽象语法树挖掘代码将被编写 (或调整), 扫描解析的 Python 模块,
并返回包含名称, 文档字符串 (包括属性和其他文档字符串; 见下文)的有序树,
以及所有的附加信息 (在下面的括号中) 以下对象:

* packages
* modules
* module attributes (+ initial values)
* classes (+ inheritance)
* class attributes (+ initial values)
* instance attributes (+ initial values)
* methods (+ parameters & defaults)
* functions (+ parameters & defaults)

(提取评论呢? 例如, 模块开头的注释将是书目字段列表的好地方.)

为了评估解释的文本交叉引用, 还需要上述每个的命名空间.

请参阅 python-dev/docstring-develop 线程 "AST mining",
于2001-08-14开始.


Docstring Extraction Rules
--------------------------

1. 要检查什么:

   a) 如果正在记录的模块中存在 "``__all__``" 变量, 则只检查 "``__all__``中列出的标识符是否存在文档字符串.

   b) 在没有 "``__all__``" 的情况下, 将检查所有标识符, 但名称为私有的名称除外
       (名称以 "_" 开头但不以 "__" 开头和结尾).

   c) 运行时设置可以覆盖 1a 和 1b.

2. 在哪里:

   Docstrings 是字符串文字表达式, 可在 Python 模块的以下位置识别:

   a) 在任何注释之后的模块的开头, 函数定义, 类定义或方法定义.
       这是 Python ``__doc__`` 属性的标准.

   b) 在任何注释之后, 立即在模块的顶层, 类定义或 ``__init__`` 方法定义进行简单赋值.
       请参阅下面的 "属性文档字符串".

   c) 在 (a) 和 (b) 中的文档字符串之后立即找到的附加字符串文字将被识别, 提取和连接.
      请参阅下面的 "其他文档字符串".

   d)  docstrings 属性的 @@@ 2.2-style "properties"? 等待语法?

3. 怎么样:

   只要有可能, Python 模块应该由 Docutils 解析, 而不是导入.
   有几个原因:

   - 导入不受信任的代码本质上是不安全的.

   - 使用自我检查导入的模块 (例如, 注释和定义的顺序) 时, 源中的信息将丢失.

   - 在字节码编译器忽略字符串文字表达式 (上面的 2b 和 2c) 的地方要识别文档字符串,
     这意味着导入模块将丢失这些文档字符串.

   当然, 应该使用标准的 Python 解析工具, 例如 "parser" 库模块.

   当模块的 Python 源代码不可用时 (即只存在 ``.pyc`` 文件) 或 C 扩展模块时,
   要访问文档字符串, 只能导入模块, 并且必须满足任何限制.

由于 Python 字节码编译器忽略了属性 docstrings 和其他文档字符串,
因此使用它们不会产生命名空间污染或运行时膨胀.
它们未分配给 ``__doc__`` 或任何其他属性. 模块的初始解析可能会略微影响性能.


Attribute Docstrings
''''''''''''''''''''

(这是 PEP 224 的简化版本 [#PEP-224]_.)

紧跟在赋值语句之后的字符串文字由 docstring 提取机制解释为赋值语句的目标的 docstring,
在以下条件下:

1. 赋值必须在以下某个上下文中:

   a) 在模块的顶层 (即, 未嵌套在诸如循环或条件的复合语句内): 模块属性.

   b) 在类定义的顶层: 类属性.

   c) 在类的 "``__init__``" 方法定义的顶层: 实例属性.
       假定在其他方法中分配的实例属性是实现细节.
	   (@@@ ``__ new__`` 方法?)

   d) 在模块或类定义的顶级进行的函数属性赋值.

   由于上述每个上下文都位于顶层 (即, 在定义的最外层),
   因此可能需要为有条件地或循环地分配的属性放置虚拟赋值。.

2. 赋值必须是单个目标, 而不是列表或目标元组.

3. 目标的形式:

   a) 对于上面的上下文 1a 和 1b, 目标必须是简单标识符
       (不是点标识符, 下标表达式或切片表达式).

   b) 对于上面的上下文 1c, 目标必须是 "``self.attrib``" 形式,
       其中 "``self``" 匹配 "``__init__``" 方法的第一个参数 (实例参数)
	   和 "attrib" 是 3a 中的简单标识符.

   c) 对于上面的上下文 1d, 目标必须是 "``name.attrib``" 形式,
      其中 "``name``" 匹配已经定义的函数或方法名称,
	  "attrib" 是一个简单的标识符, 如 3a.

在属性 docstrings 之后可以使用空行来强调赋值和 docstring 之间的连接.

例如::

    g = 'module attribute (module-global variable)'
    """This is g's docstring."""

    class AClass:

        c = 'class attribute'
        """This is AClass.c's docstring."""

        def __init__(self):
            """Method __init__'s docstring."""

            self.i = 'instance attribute'
            """This is self.i's docstring."""

    def f(x):
        """Function f's docstring."""
        return x**2

    f.a = 1
    """Function attribute f.a's docstring."""


Additional Docstrings
'''''''''''''''''''''

(这个想法改编自 PEP 216 [#PEP-216]_.)

许多程序员希望广泛使用 API 文档的文档字符串. 但是, docstrings 确实占用了运行程序中的空间,
因此一些程序员不愿意 "bloat up" 他们的代码. 此外, 并非所有 API 文档都适用于交互式环境,
其中将显示 ``__doc__``.

Docutils 的文档字符串提取工具将连接出现在定义开头或简单赋值后的所有字符串文字表达式.
只有定义中的第一个字符串可用作 ``__doc__``, 并且可用于适用于交互式会话的简短用法文本;
Python 字节码编译器会忽略后续的字符串文字和所有属性文档字符串, 并且可能包含更多的 API 信息.

例如::

    def function(arg):
        """This is __doc__, function's docstring."""
        """
        This is an additional docstring, ignored by the byte-code
        compiler, but extracted by Docutils.
        """
        pass

.. 主题:: 问题: ``from __future__ import``

   这将破坏 Python 2.1 中为多个模块文档字符串 (主文档字符串加上附加文档字符串)
   引入的 "``_ from __future__ import``" 语句. Python 参考手册指定:

       未来的声明必须出现在模块顶部附近.
	   在未来声明之前可以出现的唯一行:

       * 模块 docstring (如果有的话),
       * 注释,
       * 空行, 和
       * 其他未来的陈述.

   Resolution?

   1. 我们应该在 ``__future__`` 声明之后搜索文档字符串吗? 十分难看.

   2. 重新定义 ``__future__`` 语句来允许多个前面的字符串文字?

   3. 或者我们甚至不应该担心这个? 毕竟, 生产代码中可能不应该有 ``__future__`` 语句.
      也许带有 ``__future__`` 语句的模块只需要忍受单文档字符串限制.


Choice of Docstring Format
--------------------------

处理系统不允许强制每个人使用单个文档字符串格式, 而是允许多种输入格式.
在任何函数或类定义之前, 特殊变量 ``__docformat__`` 可能出现在模块的顶层.
随着时间的推移或通过法令, 应该出现标准格式或一组格式.

模块的 ``__docformat__`` 变量仅适用于模块文件中定义的对象.
特别是, 包的 ``__init __.py`` 文件中的 ``__docformat__`` 变量不适用于子包和子模块中定义的对象.

``__docformat__`` 变量是一个字符串, 包含正在使用的格式的名称, 一个不区分大小写的字符串,
与输入解析器的模块或包名称相匹配 (即, 与 "导入" 模块或包所需的名称相同), 或者注册的别名.
如果没有指定 ``__docformat__``, 则默认格式为 "plaintext"; 如果有人建立, 可以将其更改为标准格式.

``__docformat__`` 字符串可以包含一个可选的第二个字段, 它与格式名称 (第一个字段) 之间用一个空格分隔:
RFC 1766 中定义的不区分大小写的语言标识符. 典型的语言标识符由2个字母组成来自 `ISO 639`_ 的语言代码
(仅当不存在双字母代码时, 才使用3字母代码; RFC 1766 目前正在修订来允许3字母代码). 如果未指定语言标识符,
则英语的默认值为 "en". 语言标识符将传递给解析器, 并可用于与语言相关的标记功能.


Identifier Cross-References
---------------------------

在 Python 文档字符串中, 解释文本用于对程序标识符进行分类和标记, 例如变量, 函数, 类和模块的名称.
如果仅给出标识符, 则根据 Python 命名空间查找规则隐式推断其角色.
对于函数和方法 (即使动态分配), 也可以包括括号 ('()') ::

   这个函数使用 `another()` 来完成它的工作.

对于类, 实例和模块属性, 必要时使用点标识符.
例如 (使用 reStructuredText 标记) ::

    class Keeper(Storer):

        """
        Extend `Storer`.  Class attribute `instances` keeps track
        of the number of `Keeper` objects instantiated.
        """

        instances = 0
        """How many `Keeper` objects are there?"""

        def __init__(self):
            """
            Extend `Storer.__init__()` to keep track of instances.

            Keep count in `Keeper.instances`, data in `self.data`.
            """
            Storer.__init__(self)
            Keeper.instances += 1

            self.data = []
            """Store data in a list, most recent last."""

        def store_data(self, data):
            """
            Extend `Storer.store_data()`; append new `data` to a
            list (in `self.data`).
            """
            self.data = data

用反引号 ("`") 引用的每个标识符将成为对标识符本身的定义的引用.


Stylist Transforms
------------------

Stylist 转换是特定于 PySource Reader 的专用变换. PySource Reader 不必对样式做出任何决定;
它只生成一个逻辑构造的文档树, 解析和链接, 包括自定义节点类型.
Stylist 转换了解 Reader 创建的自定义节点并将其转换为标准 Docutils 节点.

可以实现多个 Stylist 转换, 并且可以在运行时选择一个
(通过 "--style" 或 "--stylist" 命令行选项).
每个 Stylist 转换实现不同的布局或样式; 因此这个名字.
它们将 Reader 的上下文理解部分与处理的布局生成部分分离,
从而形成更灵活, 更健壮的系统. 这也是 "将样式与内容分开",
SGML/XML 的理想选择.

通过保持造型小而模块化的代码片段, 人们可以更轻松地滚动自己的样式.
使用现有工具, "barrier to entry" 过高; 提取 stylist 代码将大大降低障碍.


==========================
 References and Footnotes
==========================

.. [#PEP-256] PEP 256, 文档字符串处理系统框架, Goodger
   (http://www.python.org/dev/peps/pep-0256/)

.. [#PEP-224] PEP 224, 属性文档字符串, Lemburg
   (http://www.python.org/dev/peps/pep-0224/)

.. [#PEP-216] PEP 216, 文档字符串格式, Zadka
   (http://www.python.org/dev/peps/pep-0216/)

.. _docutils.dtd:
   http://docutils.sourceforge.net/docs/ref/docutils.dtd

.. _soextbl.dtd:
   http://docutils.sourceforge.net/docs/ref/soextblx.dtd

.. _The Docutils 文档树:
   http://docutils.sourceforge.net/docs/ref/doctree.html

.. _VMS 错误条件严重性级别:
   http://www.openvms.compaq.com:8000/73final/5841/841pro_027.html
   #error_cond_severity

.. _log4j project: http://logging.apache.org/log4j/docs/index.html

.. _Docutils Python Source DTD:
   http://docutils.sourceforge.net/docs/dev/pysource.dtd

.. _ISO 639: http://lcweb.loc.gov/standards/iso639-2/englangn.html

.. _Python Doc-SIG: http://www.python.org/sigs/doc-sig/



==================
 Project Web Site
==================

已经为此项工作设立了 SourceForge 项目
http://docutils.sourceforge.net/.


===========
 Copyright
===========

This document has been placed in the public domain.


==================
 Acknowledgements
==================

This document borrows ideas from the archives of the `Python
Doc-SIG`_.  Thanks to all members past & present.



..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   End:  
