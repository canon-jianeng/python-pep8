
Abstract
========

此 PEP 提供了用于创建自己的 reStructuredText PEP 的样板或样本模板
结合 PEP 1 [1]_ 中的内容指南, 这可以使您轻松地将自己的 PEP 符合下面列出的格式

注意:
如果您是通过网络阅读此 PEP, 则应该首先获取此 PEP 的文本 (reStructuredText) 来源, 以完成以下步骤
** 不要使用HTML文件作为您的模板! **

这个(或任何) PEP 的源代码可以在 PEPs 存储库中找到, 可以在网站上查看: https：//github.com/python/peps/


Rationale
=========

如果您打算提交 PEP, 您必须结合下面的格式指南使用此模板, 以确保您的 PEP 提交不会因为表单而自动被拒绝

ReStructuredText 为 PEP 作者提供了有用的功能和表达能力, 同时在源文本中保持了易读性
经过处理的 HTML 表单使读者可以访问这些功能: 实时超链接, 样式文本, 表格, 图像和自动目录, 以及其他优点


How to Use This Template
========================

要使用此模板, 您必须首先确定您的 PEP 是否将成为 Informational 或 Standards Track PEP
大多数 PEP 都是 Standards Track, 因为它们为 Python 语言或标准库提出了一个新功能
如有疑问, 请阅读 PEP 1 了解详情或联系 PEP 编辑 <peps@python.org>

一旦您确定了您的 PEP 类型, 请按照以下说明操作

- 制作这个文件的副本 (``.txt``或``.rst``文件, **而不是** HTML!) 并执行以下编辑
  首选``.rst``扩展, 因为这将允许 PEP 在 GitHub 上呈现, 由于遗留原因, ``.txt``扩展名仍然存在

- 由于您尚未进行分配 PEP 编号, 因此请将 "PEP: 12" 标题替换为 "PEP: 9999"

- 将标题更改为您的 PEP 标题

- 保留 Version 和 Last-Modified 标题头部;
  当我们将您的 PEP 检查到 Python 的 子版本存储库时, 我们会处理这些问题
  这些标题由关键字 ("修订版" 和 "日期" 包含在 "$" - 符号中) 组成, 这些关键字由存储库自动扩展
  请不要编辑扩展日期或修订文本

- 更改作者标题以包含您的姓名, 以及您的电子邮件地址
  请务必仔细遵循格式: 您的名字必须首先出现, 并且不得包含在括号中
  您的电子邮件地址可能会显示在第二位 (或者可以省略), 如果显示, 则必须显示在尖括号中
  模糊您的电子邮件地址是可以的

- 如果有用于讨论新功能的邮件列表, 请在 Author 标题后面添加一个 Discussions-To 标题
  如果要使用的邮件列表是 python-ideas@python.org 或 python-dev@python.org,
  或者应该直接向您发送讨论, 则不应该添加 Discussions-To 标题头部,
  大多数 Informational PEP 没有 Discussions-To 标题

- 将状态标题更改为 "Draft"

- 对于 Standards Track PEP, 将 Type 标题更改为 "Standards Track"

- 对于 Informational PEP, 将 Type 标题更改为 "Informational"

- 对于 Standards Track PEP, 如果您的功能取决于其他当前正在开发的 PEP 的接受程度,
  请在 Type 标题头部后面添加 Requires 标题头部, 该值应该是您所依赖的 PEP 的 PEP 编号
  如果在最终 PEP 中描述了相关功能, 请不要添加此标题头部

- 将 Created 标题更改为今天的日期
  请务必仔细遵循格式: 它必须采用``dd-mmm-yyyy``格式,
  其中``mmm``是3个英文字母月缩写, 即 Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec 之一

- 对于 Standards Track PEP, 在 Created 标题之后, 添加一个 Python-Version 标头
  并将值设置为下一个计划版本的 Python, 即您的新功能有望首次出现的版本, 不要使用 alpha 或 beta 版本
  因此，如果 Python 的最后一个版本是 2.2 alpha 1, 并且您希望将新功能添加到 Python 2.2 中, 请将标头设置为::

    Python-Version: 2.2

- 暂时留下 Post-History;
  每次将 PEP 发布到 python-ideas@python.org 或 python-dev@python.org 时, 都会在此标题中添加日期
  如果您在2001年8月14日和2001年9月3日将您的 PEP 发布到列表中, 则 Post-History 标题看起来像::

    Post-History: 14-Aug-2001, 03-Sept-2001

  您必须手动添加新日期并签入, 如果您没有签到权限, 请将更改发送给 PEP 编辑

- 如果您的 PEP 废弃了早期的 PEP, 请添加替换标头
  此标头的值是新 PEP 要替换的 PEP 的编号
  如果较旧的 PEP 处于 "最终" 形式, 即仅 Accepted, Final 或 Rejected, 则仅添加此标题
  如果您提交的是竞争性想法, 那么您不会更换旧的开放式 PEP

- 现在为您的 PEP 写下您的摘要, 理由和其他内容, 用您自己的文本替换所有这些 gobbledygook
  请务必遵守以下格式指南, 特别是禁止制表符和缩进要求

- 更新您的参考和版权部分
  通常, 您将 PEP 置于公共域名, 在这种情况下, 只需单独保留版权部分
  或者, 您可以使用`Open Publication License`__, 但公共域名仍然是首选

  __ http://www.opencontent.org/openpub/

- 将 Emacs 节留在此文件的末尾, 包括分页符 ("^L" 或 ``\f``)

- 通过 peps@python.org 将您的 PEP 提交发送给 PEP 编辑


ReStructuredText PEP Formatting Requirements
============================================

以下是 reStructuredText 语法的 PEP 特定摘要
为简单和简洁起见, 省略了许多细节, 有关更多详细信息, 请参阅下面的 `Resources`_,
`Literal blocks`_ (其中没有进行标记处理) 用于整个示例, 以说明明文标记


General
-------

您必须遵守 Emacs 惯例, 即在每个句子的末尾添加两个空格
您应该将您的段落填写到第70列, 但在任何情况下都不应该延伸到第79列
如果您的代码示例溢出第79列, 您应该重写它们

标签字符绝不能出现在文档中, PEP应该包括本 PEP 底部的示例包含的标准 Emacs 节


Section Headings
----------------

PEP 标题必须从第0列开始, 每个单词的首字母必须大写, 如书名, 首字母缩略词应该是所有大写字母
章节标题必须装饰有下划线, 一个重复的标点符号, 从第0列开始, 必须至少延伸到标题文本的右边缘 (最少4个字符)
第一级部分标题带有 "="(等号), 带有 "-" (连字符) 的第二级部分标题和带有 "'" (单引号或撇号) 的第三级部分标题
例如::

    First-Level Title
    =================

    Second-Level Title
    ------------------

    Third-Level Title
    '''''''''''''''''

如果您的 PEP 中有多于三个级别的部分, 您可以为第一个和第二个级别插入上划线/下划线标题，如下所示 ::

    ============================
    First-Level Title (optional)
    ============================

    -----------------------------
    Second-Level Title (optional)
    -----------------------------

    Third-Level Title
    =================

    Fourth-Level Title
    ------------------

    Fifth-Level Title
    '''''''''''''''''

您的 PEP 中不应该超过五个级别的部分, 如果你这样做, 你应该考虑重写它

您必须在节的主体的最后一行和下一节标题之间使用两个空行
如果一个小节标题紧跟在标题之后, 则中间的一个空白行就足够了

每个部分的主体通常不会缩进, 尽管某些结构确实使用缩进, 如下所述, 空行用于分隔构造


Paragraphs
----------

段落是由空行分隔的左对齐文本块
段落不是缩进的, 除非它们是缩进构造的一部分 (例如, 块引用或列表项)


Inline Markup
-------------

段落和其他文本块中的部分文本可以设置样式, 例如 ::

    This is a paragraph.

        This is a block quote.

        A block quote may contain many paragraphs.

块引号用于引用来自其他来源的扩展段落
块引号可以嵌套在其他 body 元素中, 每个缩进级别使用4个空格


Literal Blocks
--------------

..
    在下面的文本中, 双反引号用于表示内联文字
    编写 ``::``, 以便冒号以等宽字体显示; 反引号 (``) 是标记, 不是文本的一部分, 请参阅上面的 "内联标记"

    顺便说一下, 这是一条注释, 在下面的 "Comments" 中有描述

文字块用于代码样本或预格式化的 ASCII 艺术
要指示文字块, 请在缩进的文本块前加上 "``::``" (两个冒号)
文字块一直持续到缩进结束, 将文本块缩进4个空格, 例如 ::

    This is a typical paragraph.  A literal block follows.

    ::

        for a in [5,4,3,2,1]:   # this is program code, shown as-is
            print a
        print "it's..."
        # a literal block continues until the indentation ends

只包含 "``::``" 的段落将从输出中完全删除; 没有空段落, "``::``" 也在任何段落的末尾被识别
如果紧接着前面有空格, 则两个冒号都将从输出中删除
当文本紧跟在 "``::``" 之前时. *一个* 冒号将从输出中删除, 只留下一个冒号 (即 "``::``" 将替换为 "``：``")
例如, 一个冒号仍然可见
这里：：

    Paragraph::

        Literal block


Lists
-----

着重号列表项以 "-", "*" 或 "+" (连字符, 星号或加号) 之一开头, 后跟空格和列表项主体
列表项主体必须左对齐并相对于项目符号缩进; 着重号确定缩进后的文本, 例如 ::

    This paragraph is followed by a list.

    * This is the first bullet list item.  The blank line above the
      first list item is required; blank lines between list items
      (such as below this paragraph) are optional.

    * This is the first paragraph in the second item in the list.

      This is the second paragraph in the second item in the list.
      The blank line above this paragraph is required.  The left edge
      of this paragraph lines up with the paragraph above, both
      indented relative to the bullet.

      - This is a sublist.  The bullet lines up with the left edge of
        the text blocks above.  A sublist is a new list so requires a
        blank line above and below.

    * This is the third item of the main list.

    This paragraph is not part of the list.

枚举 (编号) 列表项类似, 但使用枚举器而不是着重号
枚举数是数字 (1, 2, 3, ...), 字母 (A, B, C, ...; 大写或小写),
或罗马数字 (i, ii, iii, iv, ...; 大写或小写), 格式为句点后缀 ("1.", "2."),
括号 ("(1)", "(2)") 或右括号后缀 ("1)", "2)"), 例如 ::

    1. As with bullet list items, the left edge of paragraphs must
       align.

    2. Each list item may contain multiple paragraphs, sublists, etc.

       This is the second paragraph of the second list item.

       a) Enumerated lists may be nested.
       b) Blank lines may be omitted between list items.

定义列表是这样写的 ::

    what
        Definition lists associate a term with a definition.

    how
        The term is a one-line phrase, and the definition is one
        or more paragraphs or body elements, indented relative to
        the term.


Tables
------

简单的表格简单紧凑 ::

    =====  =====  =======
      A      B    A and B
    =====  =====  =======
    False  False  False
    True   False  False
    False  True   False
    True   True   True
    =====  =====  =======

表中必须至少有两列 (以区分节标题), 列跨度使用连字符的下划线 ("Inputs" 跨越前两列) ::

    =====  =====  ======
       Inputs     Output
    ------------  ------
      A      B    A or B
    =====  =====  ======
    False  False  False
    True   False  True
    False  True   True
    True   True   True
    =====  =====  ======

第一列单元格中的文本开始一个新行, 第一列中没有文字表示延续线; 其余的部分可能由多条线组成, 例如：：

    =====  =========================
    col 1  col 2
    =====  =========================
    1      Second column of row 1.
    2      Second column of row 2.
           Second line of paragraph.
    3      - Second column of row 3.

           - Second item in bullet
             list (row 3, column 2).
    =====  =========================


Hyperlinks
----------

在 PEP 正文中引用外部网页时, 您应该在文本中包含页面标题,
并带有对 URL 的内联超链接引用或脚注引用 (请参阅下面的 "脚注"), 不要在 PEP 的正文中包含 URL

超链接引用使用反引号和尾随下划线来标记引用文本;
如果引用文本是单个单词, 则反引号是可选的, 例如 ::

    In this paragraph, we refer to the `Python web site`_.

将目标放在 PEP 末尾的参考部分中, 或者在参考之后立即显式目标提供 URL
超链接目标以两个句点和一个空格 ("显式标记开始") 开头,
后面跟前导下划线, 引用文本, 冒号和 URL (绝对或相对) ::

    .. _Python web site: http://www.python.org/

引用文本和目标文本必须匹配 (尽管匹配不区分大小写并忽略空格中的差异)
请注意, 下划线跟踪引用文本, 但在目标文本之前
如果你认为下划线是一个向右箭头, 它会从参考指向 *away* 和 *toward* 目标

相同的机制可用于内部引用, 每个唯一的节标题都隐式定义了一个内部超链接目标
我们可以像这样链接到 Abstract 部分 ::

    Here is a hyperlink reference to the `Abstract`_ section.  The
    backquotes are optional since the reference text is a single word;
    we can also just write: Abstract_.

包含来自外部目标的 URL 的脚注将自动生成在 PEP 的参考部分的末尾, 以及将参考文本与脚注相关联的脚注参考

"PEP x" 或 "RFC x" (其中 "x" 是数字) 形式的文本将自动链接到相应的 URL


Footnotes
---------

脚注引用包括左方括号, 数字, 右方括号和尾随下划线 ::

    This sentence ends with a footnote reference [1]_.

空白必须在脚注参考之前, 在脚注引用和前一个单词之间留一个空格

在引用另一个 PEP 时, 请在正文中包含 PEP 编号, 例如 "PEP 1"
标题可以选择出现, 在标题后添加脚注引用, 例如 ::

    Refer to PEP 1 [2]_ for more information.

添加一个包含 PEP 标题和作者的脚注
它可以选择在单独的行中包含显式 URL, 但仅限于 "引用" 部分
脚注以 ".." (显式标记开头) 开头, 后面跟脚注标记 (无下划), 后面跟脚注体, 例如 ::

    References
    ==========

    .. [2] PEP 1, "PEP Purpose and Guidelines", Warsaw, Hylton
       (http://www.python.org/dev/peps/pep-0001)

如果您决定为 PEP 提供显式 URL, 请将其用作 URL 模板 ::

    http://www.python.org/dev/peps/pep-xxxx

URL 中的 PEP 编号必须用左边的零填充, 以便恰好是4个字符宽, 但文本中的 PEP 编号永远不会填充

在开发 PEP 的过程中, 您可能必须添加, 删除和重新排列脚注引用, 可能导致引用不匹配, 脚注过时和混淆,
自动编号的脚注可以提供更多自由. 使用 "#word" 形式的标签代替数字, 其中 "word" 是由字母数字加上内部连字符,
下划线和句点组成的助记符 (不允许使用空格或其他字符). 例如 ::

    Refer to PEP 1 [#PEP-1]_ for more information.

    References
    ==========

    .. [#PEP-1] PEP 1, "PEP Purpose and Guidelines", Warsaw, Hylton

       http://www.python.org/dev/peps/pep-0001

脚注和脚注引用将自动编号, 数字将始终匹配. 完成 PEP 后, 为简单起见, 应该将自动编号的标签替换为数字.


Images
------

如果您的 PEP 包含图表, 您可以使用 "image" 指令将其包含在已处理的输出中 ::

    .. image:: diagram.png

任何可浏览的图形格式都是可能的: .png, .jpeg, .gif, .tiff 等.

由于 PEP 的读者无法以源文本形式看到此图像, 因此您应该考虑使用注释 (下面) 包含描述或 ASCII 艺术替代方案.


Comments
--------

注释块是紧跟在显式标记开始之后的任意文本的缩进块: 两个句点和空格
将 ".." 单独留在一行, 以确保注释不会被误解为另一个显式标记构造.
注释在已处理的文档中不可见. 为了以源代码形式阅读您的 PEP 的人员的利益,
请考虑包括您所包含的任何图像的 ASCII 艺术替代品的描述. 例如 ::

     .. image:: dataflow.png

     ..
        Data flows from the input module, through the "black box"
        module, and finally into (and through) the output module.

本文档底部的 Emacs 节在注释中.


Escaping Mechanism
------------------

reStructuredText 使用反斜杠 ("``\``") 来覆盖标记字符的特殊含义并自己获取文字字符
想要获得文字反斜杠, 请使用转义反斜杠 ("``\\``").
有两个反斜杠没有特殊含义的上下文: `literal blocks`_ 和 inline literals (参见上面的 `Inline Markup`_).
在这些上下文中, 没有进行标记识别, 并且单个反斜杠表示字面反斜杠, 而不必加倍.

如果您发现需要在文本中使用反斜杠, 请考虑使用内联文字或文字块


Habits to Avoid
===============

许多熟悉 TeX 的程序员经常会写下这样的引号 ::

    `single-quoted' or ``double-quoted''

反引号在 reStructuredText 中很重要, 因此应该避免这种做法
对于普通文本, 请使用普通的 "单引号" 或 "双引号".
对于内联文字文本 (请参阅上面的 "内联标记"), 请使用双反引号 ::

    ``literal text: in here, anything goes!``


Resources
=========

许多其他构造和变体是可能的. 有关 reStructuredText 标记的更多详细信息, 请按顺序逐步增加, 请参阅:

* `A ReStructuredText Primer`__, a gentle introduction.

  __ http://docutils.sourceforge.net/docs/rst/quickstart.html

* `Quick reStructuredText`__, a users' quick reference.

  __ http://docutils.sourceforge.net/docs/rst/quickref.html

* `reStructuredText Markup Specification`__, the final authority.

  __ http://docutils.sourceforge.net/spec/rst/reStructuredText.html

使用 Docutils_ 完成 reStructuredText PEP 的处理
如果您对 reStructuredText 或 Docutils 有疑问或需要帮助, 请`发送消息`_ 到 `Docutils-users 邮件列表`_.
`Docutils 项目网站`_ 有更多信息.

.. _Docutils:
.. _Docutils project web site: http://docutils.sourceforge.net/
.. _post a message:
   mailto:docutils-users@lists.sourceforge.net?subject=PEPs
.. _Docutils-users mailing list:
   http://docutils.sf.net/docs/user/mailing-lists.html#docutils-users


References
==========

.. [1] PEP 1, PEP 目的和指南, Warsaw, Hylton
   (http://www.python.org/dev/peps/pep-0001)


Copyright
=========

本文档已置于公共域名.



..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   coding: utf-8  
   End:  
