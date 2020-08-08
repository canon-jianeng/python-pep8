##### @Pep     : 0001
##### @Date    : 2018-04-15 09:00:00
##### @Author  : Canon
##### @Link    : https://www.python.org
##### @Version : 3.6.1


PEP: 1
Title: PEP Purpose and Guidelines
Author: Barry Warsaw, Jeremy Hylton, David Goodger, Nick Coghlan
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 13-Jun-2000
Post-History: 21-Mar-2001, 29-Jul-2002, 03-May-2003, 05-May-2012, 07-Apr-2013

What is a PEP?
==============

PEP 全称: Python Enhancement Proposal
PEP 是一个为 Python 社区提供信息或描述 Python 或其进程或环境的新功能的设计文档
PEP 应该提供该功能的简明技术规范和该功能的基本原理

我们希望 PEP 成为提出主要新功能，收集社区对问题的意见以及记录 Python 中的设计决策的主要机制
PEP 作者负责在社区内建立共识并记录不同意见
由于 PEP 在版本化存储库中作为文本文件维护，因此其修订历史记录是功能提议的历史记录 [1]

PEP Types
=========

1.  A ** Standards Track ** PEP 描述了Python的新功能或实现
    它还可能描述在后续 PEP 在未来版本中添加标准库支持之前，将支持当前 Python 版本的标准库外部的互操作性标准

2.  An **Informational** PEP 描述了 Python 设计问题，或者向 Python 社区提供了一般指导或信息，但没有提出新功能
    Informational PEPs 不一定代表 Python 社区的共识或建议，因此用户和实施者可以自由地忽略 Informational PEPs 或遵循他们的建议

3.  A ** Process ** PEP 描述了围绕 Python 的过程，或者建议对过程进行更改(或事件)
    Process PEP 与 Standards Track PEP 类似，但适用于 Python 语言本身以外的区域
    他们可能会提出一个实现，但不是 Python 的代码库; 他们经常需要社区共识; 与 Informational PEP 不同，它们不仅仅是建议，用户通常也不能自由忽略它们
    示例包括过程，指南，决策过程的更改以及 Python 开发中使用的工具或环境的更改
    任何 meta-PEP 也被视为 Process PEP

Python's BDFL
-------------
PEP 中有几个引用 "BDFL"
这个缩写代表“仁慈的生活独裁者”，指的是 Guido van Rossum
他是Python编程语言的最初创作者和最终设计权威

PEP Editors
-------------
PEP 编辑者负责管理 PEP 工作流程的管理和编辑方面（例如，分配PEP编号和更改其状态）
有关详细信息，请参阅 "PEP 作者职责和工作流程"
目前的编辑者是
* Chris Angelico
* Anthony Baxter
* Georg Brandl
* Brett Cannon
* David Goodger
* R. David Murray
* Jesse Noller
* Berker Peksag
* Guido van Rossum
* Barry Warsaw

PEP 编辑者是受当前编辑者的邀请，可以通过地址 peps@python.org 联系，但您可能只需要使用它来私下联系编辑者
所有 PEP 工作流程都可以通过 GitHub `PEP repository`_ issues 和拉取请求

Start with an idea for Python
-----------------------------
PEP 流程从 Python 的新想法开始
强烈建议单个 PEP 包含单个关键提议或新想法
小的增强或补丁通常不需要 PEP，可以通过向 Python `issue tracker`_ 提交补丁来注入 Python 开发工作流程
PEP 越集中，它就越成功, PEP 编辑者保留拒绝 PEP 提案的权利
如果它们看起来太过分散或过于宽泛，如果有疑问，请将您的 PEP 分成几个注重焦点的 PEP

每个 PEP 必须有一个拥护者 - 使用下面描述的风格和格式编写 PEP 的人，在适当的论坛中指导讨论，并尝试围绕这个想法建立社区共识
PEP 拥护者（a.k.a.作者）应首先尝试确定该想法是否具有 PEP 能力
发布到 comp.lang.python 新闻组（a.k.a. python-list@python.org 邮件列表）或 python-ideas@python.org 邮件列表是最好的方法

在编写 PEP 之前公开征求一个想法意味着节省潜在编写时间
由于各种原因，已经提出了许多改变 Python 的想法
首先询问 Python 社区，如果一个想法是原创的，有助于防止花费太多时间花在基于事先讨论的保证被拒绝的事情上
（搜索整个互联网并不总是这样做）
它还有助于确保该想法适用于整个社区而不仅仅是作者
仅仅因为一个想法对作者来说听起来不错并不意味着它适用于大多数使用 Python 的人

一旦拥护者向 Python 社区询问一个想法是否有任何接受的机会，就应该将一个 PEP 草案提交给 python-ideas
这使作者有机会充实 PEP 草案，使格式正确，质量高，并解决对提案的初步担忧

Submitting a PEP
----------------
在讨论了 python-ideas 之后，该提议应该通过一个 'GitHub pull request`_ 作为 PEP 草案提交
草案必须以 PEP 方式编写，如下所述，否则将无法立即审核（尽管作者可能会纠正小错误）

标准的 PEP 工作流程是:
* 你是 PEP 的作者，分出 `PEP repository`_ 分支, 然后创建一个名为 "pep-9999.rst" 的文件，其中包含你的新 PEP
  使用 "9999" 作为您的草案 PEP 编号
* 将其推送到您的 GitHub 分支并提交拉取请求
* PEP 作者会检查您的 PR 的结构，格式和其他错误
* 一旦获得批准，他们将为您的 PEP 分配一个编号，并将其标记为 Standards Track, Informational, Process 并将其标记为 "草稿"

审核过程完成后，PEP 作者会批准 (请注意，这与接受您的 PEP 不同), 他们会将您的拉取请求提交到 master 上

PEP 作者不会无理拒绝 PEP
拒绝 PEP 状态的原因包括重复工作，技术上不健全，没有提供适当的动机或解决向后兼容性，或者不符合Python理念
在批准阶段可以咨询 BDFL，并且是草案 PEP 能力的最终仲裁者

具有针对 `PEP repository`_ 的 git 推送权限的开发人员可以通过创建和提交新的 PEP 直接声明 PEP 编号
这样做的话，开发人员必须处理 PEP 作者通常要处理的任务（参见 "PEP 作者职责和工作流程"）
这包括确保初始版本符合提交 PEP 的预期标准，或者，甚至开发人员也可以选择通过拉取请求提交 PEP
这样做的话，让 PEP 作者知道你有 git 推送权限，他们可以指导你直接更新 PEP 存储库

由于需要更新，PEP 作者可以检查新版本（如果他们 (或合作开发人员) 具有 git 推送权限）

在分配 PEP 编号之后，可以进一步讨论关于 python-ideas 的 PEP 草案（获得早期分配的 PEP 编号对于易于参考是有用的，尤其是在同时考虑多个草案 PEP 时）
最终，必须将所有 Standards Track PEP 发送到 `python-dev list <mailto：python-dev@python.org>` 进行审核，如下一节所述

Standards Track PEP 由两部分组成，即设计文档和参考实现
通常建议至少将一个原型实施与 PEP 共同开发，因为原则上听起来不错的想法在进行实施测试时有时会变得不切实际

PEP 作者负责在提交 PEP 之前收集社区对 PEP 的反馈意见
但是，应尽可能避免在公共邮件列表上进行长时间的开放式讨论
保持讨论有效的策略包括：为主题设置单独的 SIG 邮件列表，
让 PEP 作者在早期设计阶段接受私人评论，设置 wiki 页面等
PEP 作者应在此处使用他们的自由裁量权

PEP Review & Resolution
-----------------------
一旦作者完成了 PEP，他们可以要求对 PEP 编辑的风格和一致性进行审查
但是，必须要求 BDFL 提供 PEP 的内容和最终接受，通常是通过电子邮件发送到 python-dev 邮件列表
PEP 由 BDFL 及其选定的顾问进行审核，他们可以接受或拒绝 PEP 或将其发回给作者进行修订
对于预先确定为可接受的 PEP（例如，它是一个明显的胜利, 现状它的实施已经在被检查），
BDFL 也可以启动 PEP 审查，首先通知 PEP 作者并给予 他们有机会进行修改

PEP 批准的最终权力是 BDFL
然而，无论何时提出新的 PEP，任何认为他们具有适当经验以对该 PEP 作出最终决定的核心开发者可以提供作为该 PEP 的 BDFL 代表（或 "PEP 沙皇"）
如果他们的自我提名被其他核心开发者和 BDFL 接受，那么他们将有权批准（或拒绝）该 PEP
这个过程最常发生在 PEP 上，其中 BDFL 原则上批准了要做的事情，但是在 PEP 被接受之前需要制定细节

如果 PEP 的最终决定是由代表而不是直接由 BDFL 做出，则将通过在 PEP 中包含 "BDFL-Delegate" 标题来记录

PEP 审查和解决也可能发生在 python-dev 以外的列表中（例如，distutils-sig 用于打包不会立即影响标准库的 PEP）
在这种情况下，PEP 中的 "Discussions-To" 标题将确定适当的备选列表，其中将对 PEP 进行讨论，审查和声明

要接受 PEP，必须满足某些最低标准, 它必须是对提议的增强的清晰和完整的描述, 增强必须代表一个根本性的提高
如果适用，拟议的实施必须是可靠的，不得过度使翻译复杂化
最后，建议的增强必须是 "pythonic" 才能被 BDFL 接受
（但是, "pythonic" 是一个不精确的术语;它可以被定义为BDFL可接受的任何东西。这个逻辑是故意循环的）
参见 PEP 2 [2] 用于标准库模块验收标准

一旦接受 PEP，就必须完成参考实施
当参考实现完成并合并到主源代码库中时，状态将更改为 "Final"

PEP 也可以被指定为 "Deferred" 状态
当 PEP 没有取得进展时，PEP 作者或编辑可以将 PEP 指定为此状态
一旦 PEP 被推迟，PEP 编辑者就可以将其重新分配为 "draft" 状态

PEP 也可以 "Rejected", 也许毕竟说完了并不是一个好主意, 记录这一事实仍然很重要
"Withdrawn" 状态类似 - 这意味着 PEP 作者, 自己已经确定 PEP 实际上是一个坏主意，或者已经接受了一个与之竞争的提案是一个更好的选择

当 PEP 被接受，拒绝或撤销时，应该相应地更新 PEP
除了更新状态字段之外，至少应该在 Resolution-header 邮件列表档案中添加一个指向相关帖子的链接的 Resolution 标题

PEP 也可以被不同的 PEP 取代，使得原始 PEP 过时
这适用于 Informational PEP，其中 API 的版本2可以替换版本1

PEP 状态的可能路径如下: image:: pep-0001-1.png

如果从未打算完成，某些信息和流程 PEP 也可能具有 "Active" 状态, 例如: PEP 1（本PEP）

PEP Maintenance
---------------
一般而言，Standards track PEPs 在达到最终状态后不再被修改
完成 PEP 后，语言和标准库参考将成为预期行为的正式文档

Informational and Process PEPs 可能会随着时间的推移而更新，以反映开发实践和其他细节的变化
在这些情况下遵循的精确过程将取决于正在更新的 PEP 的性质和目的

What belongs in a successful PEP?
=================================

每个PEP应包含以下部分:

1.  Preamble -- 
    包含有关 PEP 的元数据的 RFC 822 样式标题，包括 PEP 编号，简短描述性标题（限制为最多44个字符），名称以及每个作者的联系信息等
2.  Abstract -- 
    对正在解决的技术问题的简短（约200字）描述
3.  Copyright/public domain -- 
    每个 PEP 必须明确标记为放置在公共域名中（请参阅此 PEP 作为示例）或根据 `Open Publication License` 授权
4.  Specification -- 
    技术规范应描述任何新语言特性的语法和语义, 规范应该足够详细，以允许至少当前主要 Python 平台（CPython，Jython，IronPython，PyPy）的竞争性，可互操作的实现
5.  Motivation -- 
    对于想要改变 Python 语言的 PEP 来说, 动机至关重要, 它应该清楚地解释为什么现有的语言规范不足以解决 PEP 解决的问题, 没有足够动力的 PEP 提交可能会被彻底拒绝
6.  Rationale -- 
    基本原理描述什么是设计的动机, 为什么作了特别的设计决定了规范, 它应该描述被考虑的替代设计和相关工作，例如， 如何在其他语言中支持该功能
    解释应该提供社区内共识的证据，并讨论在讨论中提出的重要异议或关注
7.  Backwards Compatibility -- 
    引入向后不兼容性的所有 PEP 必须包含描述这些不兼容性及其严重性的部分
    PEP 必须解释作者如何处理这些不兼容性, 没有足够的向后兼容性论述的 PEP 提交可能会被彻底拒绝
8.  Reference Implementation -- 
    必须在任何 PEP 被赋予 "Final" 状态之前完成参考实现，但不能在 PEP 被接受之前完成
    虽然在编写代码之前就规范和基本原理达成共识的方法是有价值的，但是在解决 API 细节的许多讨论时，"rough consensus and running code" 的原则仍然很有用
    最终实现必须包含适用于 Python 语言参考或标准库参考的测试代码和文档

PEP Formats and Templates
=========================

PEP 是使用 reStructuredText_ 格式的 UTF-8 编码的文本文件
ReStructuredText_ 允许仍然非常容易阅读的丰富标记，但也可以生成外观漂亮且功能强大的 HTML
PEP 12 包含用于 reStructuredText PEP 的指令和模板 [4]

Python 脚本自动将 PEP 转换为 HTML 以便在 Web 上查看 [5]
reStructuredText PEP 的转换由 Docutils_ 模块处理;
同一脚本还在内部呈现 PEP 的传统纯文本格式，以支持 pre-reST 文档

PEP Header Preamble
===================

每个 PEP 必须以 RFC 822 样式头部前言开始, 标题必须按以下顺序显示
标有 "*" 的标题是可选的，如下所述, 所有其他头部都是必需的, ::
    PEP: <pep number>
    Title: <pep title>
    Author: <list of authors' real names and optionally, email addrs>
  * BDFL-Delegate: <PEP czar's real name>
  * Discussions-To: <email address>
    Status: <Draft | Active | Accepted | Deferred | Rejected |
             Withdrawn | Final | Superseded>
    Type: <Standards Track | Informational | Process>
  * Content-Type: <text/x-rst | text/plain>
  * Requires: <pep numbers>
    Created: <date created on, in dd-mmm-yyyy format>
  * Python-Version: <version number>
    Post-History: <dates of postings to python-ideas and/or python-dev>
  * Replaces: <pep number>
  * Superseded-By: <pep number>
  * Resolution: <url>

Author 标题列出了 PEP 的所有作者或所有者的姓名和电子邮件地址
Author 头部值的格式必须是 Random J. User <address@dom.ain>

如果包含电子邮件地址，只是 Random J. User

如果没有给出地址, 由于历史原因, 格式 "address@dom.ain（Random J. User）" 可能出现在 PEP 中,
但是新的 PEP 必须使用上面的强制格式, 并且当 PEP 更新时可以更改为此格式

如果有多个作者，则每个作者应遵循 RFC 2822 延续行约定
请注意，PEP 中的个人电子邮件地址将被隐藏，以防止垃圾邮件收集者

BDFL-Delegate 字段用于记录批准或拒绝 PEP 的最终决定权由 BDFL 以外的其他人决定的情况
（由于 reStructuredText PEP 的电子邮件地址屏蔽限制，当前省略了代理人的电子邮件地址）

*Note：Resolution 头部仅适用于 Standards Track PEP
它包含一个URL, 该 URL 应该指向发送有关 PEP 的声明的电子邮件或其他 Web 资源

对于将在除 python-dev 之外的列表上进行最终声明的 PEP, Discussions-To 标题将指示将发生声明的邮件列表或 URL
在提交声明之前讨论 PEP 草案时, 也可以使用临时的 Discussions-To 标题
如果正在与作者私下讨论 PEP，或者在 python-list, python-ideas 或 python-dev 邮件列表上讨论, 则不需要讨论头部
请注意, Discussions-To 标题中的电子邮件地址不会被遮盖

Type 头部指定 PEP 的类型: Standards Track，Informational 或 Process

PEP 的格式使用 Content-Type 头部指定, 可接受的值是明文 PEP 的 "text/plain" ( 参见 PEP 9 [3] )
和 reStructuredText PEP 的 "text/x-rst" ( 参见 PEP 12 [4] )
reStructuredText 是首选, 但为了向后兼容, 如果不存在 Content-Type 头部, 则纯文本当前仍然是默认文本

Created 头部记录了 PEP 分配号码的日期, 而 Post-History 用于记录新版本 PEP 发布到 python-ideas 或 python-dev 的日期
两个标题都应该是 dd-mmm-yyyy 格式, 例如 14-Aug-2001

Standards Track PEPs 通常会有一个 Python-Version 头部，表示该功能将随之发布的 Python 版本
Standards Track PEPs 没有 Python-Version 头部的 PEP 表示最初将通过外部库和工具支持的互操作性标准, 然后由后来的 PEP 补充以添加对标准库的支持
Informational and Process PEPs 不需要 Python-Version 头部

PEP 可能有一个 Requires 头部，表示该 PEP 所依赖的 PEP 编号

PEP 还可以具有 Superseded-By 头部, 指示 PEP 已被之后的文档废弃; 该值是替换当前文档的 PEP 的编号
更新的 PEP 必须有一个 Replaces 头部, 其中包含它过时的 PEP 编号

Auxiliary Files
===============

PEP 可以包括辅助文件，例如图表
这些文件必须命名为 "pep-XXXX-Y.ext", 其中 "XXXX" 是PEP编号，"Y" 是序列号 (从1开始), "ext" 由实际文件扩展名替换 (例如: "png")

Reporting PEP Bugs, or Submitting PEP Updates
=============================================

如何报告错误或提交 PEP 更新取决于几个因素, 例如: PEP 的成熟度，PEP 作者的偏好以及评论的性质
对于 PEP 的早期草案阶段, 最好将您的意见和更改直接发送给 PEP 作者
对于更成熟或已完成的 PEP, 您可能需要向 Python "issue tracker" 提交更正，以便您的更改不会丢失
如果 PEP 作者是 Python 开发人员, 请将错误/补丁分配给他们, 否则将其分配给 PEP 编辑者

如果对发送更改的位置有疑问, 请先咨询 PEP作者 或 PEP 编辑者

具有 PEP 存储库的 git 推送权限的 PEP 作者可以通过使用 "git push" 提交他们的更改来更新 PEP 本身

Transferring PEP Ownership
==========================

偶尔有必要将 PEP 的所有权转让给新的拥护者
一般来说，最好保留原作者作为转移的 PEP 的共同作者，但这完全取决于原作者
转让所有权的一个很好的理由是因为原作者不再有时间或兴趣更新它或完成 PEP 流程，或者已经脱离了 python 网络（即无法访问或不响应电子邮件）
转让所有权的一个不好的理由是因为作者不同意 PEP 的方向
PEP 流程的一个目标是尝试围绕 PEP 建立共识，但如果不可能，作者可以随时提交对立的 PEP

如果您有兴趣获得 PEP 的所有权，您也可以通过拉取请求执行此操作
分出 `PEP repository` 分支, 进行所有权修改, 并提交拉取请求
您还应该发送一条消息, 要求接管原始作者和 PEP 编辑 <peps@python.org>
如果原作者没有及时回复电子邮件, PEP 编辑将做出单方面的决定（这样的决定不能逆转)

PEP Editor Responsibilities & Workflow
======================================

PEP 编辑者必须订阅 <peps@python.org> 列表，并且必须查看 `PEP repository`_
关于 PEP 管理的大多数通信可以通过 GitHub 问题和拉取请求来处理，但您也可以使用 <peps@python.org> 进行半私人讨论
请不要重复发帖！

对于编辑者中的每个新的 PEP，执行以下操作：

* 阅读 PEP 以检查它是否准备就绪：声音和完整。 这些想法必须具有技术意义，即使它们似乎不太可能被接受
* 标题应该准确描述内容
* 编辑 PEP 语言（拼写，语法，句子结构等），标记（用于 reST PEP），代码样式（示例: 应该与 PEP8 和 PEP7 匹配）

如果 PEP 没有准备好，编辑者会将其发回给作者进行修改，并附上具体说明

一旦 PEP 准备好存储库，PEP 编辑者将:

* 分配一个 PEP 编号（几乎总是只是下一个可用号码，但有时它是一个特殊或搞笑号码，如 666 或 3141）
 （ 澄清：对于 Python 3, 3000s 中的数字用于特定于 Py3k 的提议
  但是现在所有新功能仅用于 Python 3, 过程又回到了使用 100s 中的数字
  请记住, 低于100的数字是 meta-PEPs ）
* 将 PEP 添加到 PEP 存储库的本地分支
  有关工作流程说明，请参阅 "Python开发人员指南" <http://docs.python.org/devguide>
  peps 的 git repo 是 https://github.com/python/peps
* 运行 "./ genpepindex.py" 和 "./pep2html.py <PEP Number>" 以确保生成它们没有错误
  如果任何一个触发错误，则不会更新网站以反映 PEP 更改
* 提交并推送新的（或更新的）PEP
* 监控 python.org 用来确保 PEP 正确添加到站点
  如果它不能出现，运行 "make" 将构建所有当前的 PEP
  如果其中任何一个触发错误，必须在任何 PEP 在站点上更新之前纠正它们
* 使用后续步骤将电子邮件发回给 PEP 作者（发布到 python-list＆-dev）

对现有 PEP 的更新应该使用 "GitHub pull request" 提交, 可以将问题发送到 <peps@python.org>

许多 PEP 由具有 Python 代码库写入权限的开发人员编写和维护
PEP 编辑者监视 python-checkins 列表以查找 PEP 更改，并更正他们看到的任何结构，语法，拼写或标记错误

PEP 编辑者不会对 PEP 做出判断, 他们只是做管理和编辑部分（通常是一个小批量的任务）

Resources:

* Python 增强建议索引 <http://www.python.org/dev/peps/>
* 遵循 Python 的开发 <http://docs.python.org/devguide/communication.html>
* Python 开发人员指南 <http://docs.python.org/devguide/>
* 针对开发人员的常见问题解答 <http://docs.python.org/devguide/faq.html>

References and Footnotes
========================

.. [1] 此历史记录可通过常规 git 命令获取, 用于检索旧版本, 也可以通过 HTTP 在此处浏览: https://github.com/python/peps
.. [2] PEP 2，添加新模块的程序, Faassen （http://www.python.org/dev/peps/pep-0002）
.. [3] PEP 9，样本明文PEP模板, Warsaw （http://www.python.org/dev/peps/pep-0009）
.. [4] PEP 12，样本 reStructuredText PEP 模板, Goodger, Warsaw （http://www.python.org/dev/peps/pep-0012）
.. [5] 这里提到的脚本是 pep2pyramid.py, 它是 pep2html.py 的继承者，它们都与 hpe repo 中的 PEPs 目录位于同一目录中
       试试 "pep2html.py --help" 了解详情, 用于在 Web 上查看 PEP 的 URL: http://www.python.org/dev/peps/

.. _issue tracker: http://bugs.python.org/
.. _Open Publication License: http://www.opencontent.org/openpub/
.. _reStructuredText: http://docutils.sourceforge.net/rst.html
.. _Docutils: http://docutils.sourceforge.net/
.. _PEP repository: https://github.com/python/peps
.. _`GitHub pull request`: https://github.com/python/peps/pulls

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
