
PEP: 269
Title: Pgen Module for Python
Version: $Revision$
Last-Modified: $Date$
Author: jriehl@spaceship.com (Jonathan Riehl)
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 24-Aug-2001
Python-Version: 2.2
Post-History:


Abstract
========

就像 ``parser`` 模块公开 Python 解析器一样, 这个 PEP 提出用于
创建 Python 解析器的解析器生成器 ``pgen``, 在 Python 中作为模块公开.


Rationale
=========

通过 Pythonic 历史的过程, 已经有很多关于 Python 编译器 [1]_ 的创建的讨论.
这些已经导致 Python 解析器的几个实现, 最值得注意的是当前在 Python 标准库 [2]_
和 Jeremy Hylton 的 ``compiler`` 模块 [3]_ 中提供的 ``parser`` 模块. 然而,
虽然已经提出了多种语言变化 [4]_ [5]_, 但是 Python 语法的实验缺乏 Python 绑定到
用于构建 Python 的实际解析器生成器的好处.

通过提供类似于 Fred Drake Jr. 的解析器包装器的 Python 包装器,
但是针对 ``pgen`` 库, 进行以下断言:

1. 语法更改的参考实现将更容易开发. 目前, 语法更改的参考实现需要开发人员使用命令行中的 ``pgen`` 工具.
    然后, 必须重新编写生成的解析器数据结构以与自定义 CPython 实现接口, 或者包装为 C 扩展模块.

2. 语法更改的参考实现将更容易分发. 由于解析器生成器将在 Python 中可用,
    因此应该遵循生成的解析器可以从 Python 访问. 因此, 参考实现应该是纯 Python 代码,
	而不是使用现有 CPython 发行版的自定义版本, 或者作为可编译的扩展模块.

3. 语法更改的参考实现将更容易与更多的受众讨论. 这有点落后于第二个断言,
   因为 Python 用户社区最有可能比 CPython 开发人员社区更大.

4. Python 中的小语言开发将得到进一步增强, 因为附加模块将是一个功能齐全的 LL(1) 解析器生成器.


Specification
=============

建议的模块将被称为 ``pgen``. ``pgen`` 模块将包含以下功能:


``parseGrammarFile (fileName) -> AST``
--------------------------------------

``parseGrammarFile()`` 函数将读取 fileName 指向的文件并创建一个 AST 对象.
AST 节点将包含解析器生成器元语法的非终结数值.
输出 AST 将是由 ``parser`` 模块提供的 AST 扩展类的实例.
输入文件中的语法错误将导致引发 SyntaxError 异常.


``parseGrammarString (text) -> AST``
------------------------------------

``parseGrammarString()`` 函数将遵循 ``parseGrammarFile()`` 的语义,
但接受语法文本作为输入字符串, 而不是文件名.


``buildParser (grammarAst) -> DFA``
-----------------------------------

``buildParser()`` 函数将接受一个 AST 对象进行输入并返回一个 DFA (确定性有限机器人) 数据结构.
DFA 数据结构将是一个 C 扩展类, 就像 ``parser`` 模块中提供的 AST 结构一样.
如果输入 AST 不符合为 ``pgen`` 元语法定义的非终结符号, ``buildParser()`` 将抛出 ``ValueError`` 异常.


``parseFile (fileName, dfa, start) -> AST``
-------------------------------------------

``parseFile()`` 函数本质上是 ``PyParser_ParseFile()`` C API 函数的包装器.
包装器代码将接受 DFA C 扩展类和文件名. 符合 "token" 模块中的词法值的 AST 实例
和 DFA 中包含的非终结值将被输出.


``parseString (text, dfa, start) -> AST``
-----------------------------------------

``parseString()`` 函数将以与 ``parseFile()`` 函数类似的方式运行, 但接受解析文本作为参数.
很像 ``parseFile()`` 将包装 ``PyParser_ParseFile()`` C API 函数,
``parseString()`` 将包装 ``PyParser_ParseString()`` 函数.


``symbolToStringMap (dfa) -> dict``
-----------------------------------

``symbolToStringMap()`` 函数将接受一个 DFA 实例并返回一个字典对象,
该对象从 DFA 的非终结符的数值映射到 DFA 的原始语法规范中的非终结符的字符串名称.


``stringToSymbolMap (dfa) -> dict``
-----------------------------------

``stringToSymbolMap()`` 函数输出一个字典, 将输入 DFA 的非终结名称映射到它们对应的数值.


如果 map 生成函数和解析函数也是 DFA 扩展类的方法, 则将获得额外的奖励.


Implementation Plan
===================

已经设计了一个狡猾的计划来实现这一增强:

1. 重命名 ``pgen`` 函数来符合 CPython 命名标准. 此操作可能涉及将一些头文件添加到 ``Include`` 子目录.

2. 将 Makefile.pre.in 中的 ``pgen`` C 模块从唯一的 ``pgen`` 元素移到 Python C 库.

3. 对 ``parser`` 模块进行任何必要的更改, 利于 AST 扩展类理解它可能无法理解的 AST 类型.
   粗略检查 AST 扩展类表明它跟踪树是套件还是表达式.

3. 在 `Modules` 目录中编写一个额外的 C 模块. C 扩展模块将实现 DFA 扩展类和上一节中概述的功能.

4. 将新模块添加到构建过程. 确实是黑魔法.


Limitations
===========

根据这个提议, Python 3000 的设计者仍将受限于 Python 的词汇约定.
Python 词法分析器的添加, 减少或修改超出了本 PEP 的范围.


Reference Implementation
========================

目前没有提供参考实施. 在
http://sourceforge.net/tracker/index.php func = detail＆aid = 599331＆group_id = 5470＆atid = 305470
的某个时刻提供了一个补丁, 但该补丁不再维护.


References
==========

.. [1] The (defunct) Python Compiler-SIG
       http://www.python.org/sigs/compiler-sig/

.. [2] Parser Module Documentation
       http://docs.python.org/library/parser.html

.. [3] Hylton, Jeremy.
       http://docs.python.org/library/compiler.html

.. [4] Pelletier, Michel. "Python Interface Syntax", PEP-245.
       http://www.python.org/dev/peps/pep-0245/

.. [5] The Python Types-SIG
       http://www.python.org/sigs/types-sig/


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  fill-column: 70  
  End:  
