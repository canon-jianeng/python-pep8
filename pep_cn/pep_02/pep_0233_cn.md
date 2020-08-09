
PEP: 233
Title: Python Online Help
Version: $Revision$
Last-Modified: $Date$
Author: paul@prescod.net (Paul Prescod)
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 11-Dec-2000
Python-Version: 2.1
Post-History:


Abstract
========

本 PEP 描述了一个用于 Python 的命令行驱动的在线帮助工具.
该工具应该能够建立在现有的文档工具上, 例如 Python 文档和文档字符串.
对于新类型和模块, 它也应该是可扩展的.


Interactive use
===============

只需输入 ``help`` 描述帮助功能 ( 通过 ``repr()`` 重载 ).

``help`` 也可以用作函数.

该函数采用以下形式的输入:

* ``help( "string" )`` -- 内置主题或全局
* ``help( <ob> )`` -- 来自对象或类型的 docstring
* ``help( "doc:filename" )`` -- Python 文档中的文件名

如果您要求全局, 它可以是完全限定的名称, 例如 ::

    help("xml.dom")

您还可以从命令行使用该功能 ::

    python --help if

在任何一种情况下, 输出都会执行类似于 ``more`` 命令的分页.


Implementation
==============

帮助功能在需要加载的 ``onlinehelp`` 模块中实现.

应该有通过 ``onlinehelp`` 模块从命令行以外的环境获取帮助信息的选项 ::

    onlinehelp.gethelp(object_or_string) -> string

还应该可以通过赋予 ``onlinehelp.displayhelp(object_or_string)`` 来覆盖帮助显示功能.

该模块应该能够从 Python 文档的 HTML 或 LaTeX 版本中提取模块信息.
链接应该以 "lynx-like" 的方式进行.

随着时间的推移, 它还应该能够识别 docstrings 何时采用结构化文本,
HTML 和 LaTeX 等 "特殊" 语法并对其进行适当解码.

Python 源代码发布的原型实现为 ``nondist/sandbox/doctools/onlinehelp.py``.


Built-in Topics
===============

* ``help( "intro" )`` -- 什么是 Python? 先读一下!

* ``help( "keywords" )`` -- 关键字是什么?

* ``help( "syntax" )`` -- 什么是整体语法?

* ``help( "operators" )`` -- 有哪些运算符可用?

* ``help( "builtins" )`` -- 内置了哪些函数, 类型等?

* ``help( "modules" )`` -- 标准库中有哪些模块?

* ``help( "copyright" )`` -- 谁拥有 Python?

* ``help( "moreinfo" )`` -- 哪里有更多的信息?

* ``help( "changes" )`` -- Python 2.0 中发生了哪些变化?

* ``help( "extensions" )`` -- 安装了哪些扩展?

* ``help( "faq" )`` -- 经常提出什么问题?

* ``help( "ack" )`` -- 谁最近在 Python 上做过工作?


Security Issues
===============

此模块将尝试导入与所请求主题具有相同名称的模块.
如果您不确定 ``PYTHONPATH`` 中的所有内容都来自可靠来源,
请不要使用这些模块.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
