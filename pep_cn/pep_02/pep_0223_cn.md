
PEP: 223
Title: Change the Meaning of ``\x`` Escapes
Version: $Revision$
Last-Modified: $Date$
Author: tim.peters@gmail.com (Tim Peters)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 20-Aug-2000
Python-Version: 2.0
Post-History: 23-Aug-2000


Abstract
========

更改 ``\x`` 转义, 在8位和 Unicode 字符串中, 完全消耗后面的两个十六进制数字.
该提案将此视为纠正原始设计缺陷, 使所有字符串更清晰, 更清晰的 Unicode,
更好与 Perl 正则表达式兼容, 并且对现有代码的风险最小.


Syntax
======

在各种非原始字符串中, ``\x``的语法变为 ::

   \xhh

其中 h 是十六进制数字 (0-9, a-f, A-F). 1.5.2 中的确切语法未在参考手册中明确说明; 它说 ::

   \xhh...

暗示 "两个或更多" 十六进制数字, 但 1.5.2 编译器也接受一位数形式,
并且普通的 ``\x`` 被 "扩展" 为自身 (即反斜杠后跟字母 x).
目前还不清楚参考手册是指 1 位数或 0 位数的行为.


Semantics
=========

在一个8位非原始字符串中, ::

   \xij

扩展到字符 ::

   chr(int(ij, 16))

请注意, 这与 1.6 及之前的相同.

在 Unicode 字符串中,
::

   \xij

行为与以下形式相同 ::

   \u00ij

i.e. 它从 Unicode 空间的初始段落扩展为明显的 Latin-1 字符.

一个 ``\x`` 后面跟着至少两个十六进制数字是编译时错误,
特别是8位字符串中的 ``ValueError`` 和 ``UnicodeError``
(``ValueError`` 的子类) 在 Unicode 字符串中.
请注意, 如果 ``\x`` 后面跟着两个以上的十六进制数字, 则只有前两个被 "消耗".
在 1.6 之前, 除了 *last* 之外, 我们都默默地被忽略了.


Example
=======

In 1.5.2::

    >>> "\x123465"  # same as "\x65"
    'e'
    >>> "\x65"
    'e'
    >>> "\x1"
    '\001'
    >>> "\x\x"
    '\\x\\x'
    >>>

In 2.0::

    >>> "\x123465" # \x12 -> \022, "3456" left alone
    '\0223456'
    >>> "\x65"
    'e'
    >>> "\x1"
    [ValueError is raised]
    >>> "\x\x"
    [ValueError is raised]
    >>>


History and Rationale
=====================

在 C 中引入了 ``\x`` 转义符作为指定可变宽度字符编码的方法.
究竟是哪些编码, 以及它们需要多少个十六进制数字, 留给每个实现.
该语言简单地声明 ``\x`` "消耗" *所有* 十六进制数字跟随, 并将意义留给每个实现.
因此, 实际上, C 中的 ``\x`` 是提供平台定义行为的标准钩子.

因为 Python 明确地旨在实现平台独立性, 所以 Python 中的 ``\x`` 转义
(直到并包括 1.6) 在所有平台上都被视为相同: 除了 *最后两个十六进制数字之外的所有* 都被默默地忽略.
因此 Python 中 ``\x`` 转义的唯一实际用途是使用十六进制表示法指定单个字节.

Larry Wall 似乎已经意识到这是在平台无关的语言中 ``\x`` 转义的唯一真正用途,
因为 Python 2.0 的提议规则实际上是 Perl 从一开始就做的
(尽管你需要在 Perl -w 模式下运行以获得关于 ``\x`` 转义的警告,
其后面的数字少于2个十六进制数字 -- 显然更多 Pythonic 一直坚持2).

当 Unicode 字符串被引入 Python 时, ``\x`` 被一般化, 以便忽略 Unicode 字符串中
除最后的 *四个*十六进制数字之外的所有数字. 这对新的正则表达式引擎造成了技术上的困难:
SRE 非常努力地允许以直观的方式混合8位和 Unicode 模式和字符串,
并且它不再有任何方法可以猜出什么, 例如, ``r"\x123456"`` 应该表示一种模式:
是否要求匹配8位字符 ``\x56`` 或 Unicode 字符 ``\u3456``?

有猜测的黑客方法, 但它并没有结束. ISO C99 标准还引入了8位 ``\U12345678`` 转义来覆盖整个 ISO 10646 字符空间,
并且还希望 Python 2 从一开始就支持它. 但那么 ``\x`` 脱离应该是什么意思呢?
他们会忽略除最后的 *八位* 十六进制数字之外的所有数字吗? 如果在 Unicode 字符串中少于8个,
那么除了最后4个之外的所有字符串? 如果少于4, 除了最后 2?

这一点变得越来越混乱, 这个提议通过使 ``\x`` 更简单而不是更复杂来削减 Gordian.
请注意, Unicode 字符串中对 ``\xijkl`` 的4位泛化也是多余的, 因为它与 Unicode 字符串中的 ``\uijkl`` 完全相同.
只有一种明显的方法可以通过十六进制表示法指定 Unicode 字符, 这更像 Pythonic.


Development and Discussion
==========================

该提案是在 Guido van Rossum, Fredrik Lundh 和 Tim Peters 的电子邮件中制定的.
随后在主题为 "Go \x yourself" [1]_ 的 Python-Dev 中进行了解释和讨论,
从 2000-08-03 开始. 反应非常积极; 没有人反对.


Backward Compatibility
======================

更改 ``\ x`` 转义的含义确实存在破坏现有代码的风险, 但尚未发现任何不兼容的实例. 相信风险很小.

Tim Peters 证实, 除了标准测试套件故意引发最终案例之外, 在 Python CVS 开发树中,
没有 ``\xabcdef...`` 具有少于或多于2个十六进制数字的实例, 或者装在他的机器上的各种 Python 包.

不太可能有少于2, 因为参考手册暗示他们不合法 (虽然这是有争议的!).
如果有超过2的任何一个, Guido 已经准备好争辩他们还有错误 <0.9 wink>.

Guido 报道称, O'Reilly Python 书籍 *已经* 证明 Python 以提议的方式工作,
可能是由于他们的 Perl 发布遗产 (如上所述, Perl 从一开始就工作 (非常接近) 提议的方式).

Finn Bock 报道说, JPython 对 ``\x`` 逃脱的做法今天无法预测.
该提议提供了一个明确的含义, 可以在所有 Python 实现中始终如一地轻松实现.


Effects on Other Tools
======================

被认为是 none, 破损的候选者主要是解析工具, 但作者知道没有人担心 Python 字符串的内部结构超出了近似
"当有反斜杠, 吞下下一个字符时". Tim Peters 检查了 ``python-mode.el``, 标准 ``tokenize.py`` 和 
``pyclbr.py``, 以及 IDLE 语法着色子系统, 并认为没有必要改变它们中的任何一个.
像 ``tabnanny.py`` 和 ``checkappend.py`` 这样的工具继承了 ``tokenize.py`` 的免疫力.


Reference Implementation
========================

代码更改非常简单, 不会生成单独的修补程序.
Fredrik Lundh 正在编写代码, 是该领域的专家, 并将在 2.0b1 发布之前检查更改.


BDFL Pronouncements
===================

是的, ``ValueError``, 而不是 ``SyntaxError``.
"文字解释的问题传统上会引发 'runtime' 异常, 而不是语法错误."


References
==========

.. [1] Tim Peters, Go \x yourself
       https://mail.python.org/pipermail/python-dev/2000-August/007825.html


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
