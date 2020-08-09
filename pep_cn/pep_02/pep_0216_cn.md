
PEP: 216
Title: Docstring Format
Version: $Revision$
Last-Modified: $Date$
Author: moshez@zadka.site.co.il (Moshe Zadka)
Status: Rejected
Type: Informational
Content-Type: text/x-rst
Created: 31-Jul-2000
Post-History:
Superseded-By: 287


Notice
======

该 PEP 被作者拒绝. 它已被 PEP 287 取代.


Abstract
========

命名的 Python 对象, 例如模块, 类和函数, 有一个名为``__doc__``的字符串属性.
如果定义中的第一个表达式是文字字符串, 则该字符串将分配给``__doc__``属性.

``__doc__``属性称为文档字符串或 docstring. 它通常用于概括模块, 类或函数的接口.
但是, 由于文档字符串没有通用格式, 因此用于提取文档字符串并将其转换为标准格式
(例如, DocBook) 的文档的工具并未大量涌现, 而那些确实存在的工具大部分都是未维护和未使用的.


Perl Documentation
==================

在 Perl 中, 大多数模块都以称为 POD - Plain Old Documentation 的格式记录.
这是一种易于类型, 非常低级的格式, 可与 Perl 解析器很好地集成.
存在许多将 POD 文档转换为其他格式的工具: 信息, HTML 和手册页等.
但是, 在 Perl 中, 信息在运行时不可用.


Java Documentation
==================

在 Java 中, 类和函数之前的特殊注释用于记录代码.
提取这些文件并将其转换为 HTML 文档的程序称为 javadoc, 它是标准 Java 发行版的一部分.
但是, 唯一支持的输出格式是 HTML, JavaDoc 与 HTML 有非常密切的关系.


Python Docstring Goals
======================

Python 文档字符串在解析过程中很容易被发现, 并且也可供运行时解释器使用.
这个双重目的有点问题, 有时候: 例如, 有些人不愿意有太长的文档字符串,
因为他们不想在运行时占用太多空间. 此外, 由于目前缺乏工具, 人们通过 "print" 来阅读对象的文档字符串,
因此，使它们简短且没有标记的趋势如雨后春笋般涌现. 这种趋势阻碍了编写更好的文档提取工具,
因为它会导致文档字符串包含很少的信息, 这很难解析.


High Level Solutions
====================

为了抵制字符串在正在运行的程序中占据的异议,
建议文档提取工具将连接出现在定义开头的字符串文字的最大前缀.
其中第一个也将在交互式解释器中提供, 因此它应包含一些摘要行.


Docstring Format Goals
======================

这些是 docstring 格式的目标, 正如 doc-sig 中讨论的那样令人作呕.

1. 使用任何标准文本编辑器输入都必须很容易.
2. 对于不经意的检查者来说, 它必须是可读的.
3. 它不能包含可以从 parsing 模块中推断出的信息.
4. 它必须包含足够的信息, 以便可以转换为任何合理的标记格式.
5. 必须可以在文档字符串中编写模块的整个文档, 而不会受到标记语言的妨碍.


Docstring Contents
==================

对于上面的要求 5. 需要指定 docstrings 中必须包含的内容.

至少必须提供以下内容:

a. 一个标签, 意思是 "这是一个 Python *something*, 猜什么"

   Example: 在句子 "POP3 class" 中, 我们需要标记 "POP3".
   解析器将能够猜测它是一个来自``poplib``模块内容的类, 但我们需要猜测它.

b. 标签表示 "this is a Python class/module/class var/instance var..."

   Example: 单例类``A``的常用 Python 习语是将``_A``作为类, 而``A``是一个返回``_A``对象的函数.
   尽管如此, 通常将课程记录为``A``. 这需要有力量说 "The class``A``" 并且将``A``超链接并标记为类.

c. 包含 Python 源代码或 Python 交互式会话的简单方法

d. 重点或加粗

e. 列表或表


Docstring Basic Structure
=========================

文档字符串将位于 StructuredTextNG 中 (http://www.zope.org/Members/jim/StructuredTextWiki/StructuredTextNG)
由于 StructuredText 尚不足以处理上述 (a) 和 (b), 我们需要扩展它.
我建议使用 ``[<optional description>：python identifier]``.
例如: ``[class：POP3]``, ``[：POP3.list]``等. 如果缺少描述, 将从文本中猜测.


Unresolved Issues
=================

有没有办法逃脱 ST 的角色? 如果是这样, 怎么样?
(example: * 在一行的开头没有 bullet 符号)

我的上述建议是否符合与 ST-NG 兼容的 Python 符号?
扩展 ST-NG 来支持它有多难?

我们如何描述函数的输入和输出类型?

我们对每个文档字符串强制执行哪些附加约束?
(module/class/function)?

什么是猜测规则?


Rejected Suggestions
====================

XML -- 打字非常困难, 而且太杂乱无法轻松阅读.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
