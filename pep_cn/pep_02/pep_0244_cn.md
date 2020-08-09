
PEP: 244
Title: The ``directive`` statement
Version: $Revision$
Last-Modified: $Date$
Author: martin@v.loewis.de (Martin von Löwis)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 20-Mar-2001
Python-Version: 2.1
Post-History:


Motivation
==========

Python 不时会对核心语言结构的宣传语义进行不兼容的更改,
或者以某种方式更改其意外 (依赖于实现的) 特性.
虽然这种情况从来没有反复无常, 而且总是以改善长期语言为目的,
但在短期内它会引起争议和破坏.

PEP 1, 语言进化指南 [1]_ 建议缓解疼痛的方法, 本 PEP 引入了一些机制来支持这一点.

PEP 2, 静态嵌套范围 [2]_ 是第一个应用程序, 将在这里用作示例.

当添加新的, 可能不兼容的语言功能时, 某些模块和库可能会选择使用它, 而其他模块和库可能不会.
本规范引入了一种语法, 其中模块作者可以表示模块中是否使用了某种语言特性.

在讨论这个 PEP 时, 读者评论说有两种 "可设置" 的语言特征:

- 那些旨在最终成为唯一选择的, 在此时不再需要使用它们. "回到 ``__future__``" PEP 236,
  返回到 ``__future__`` [3]_ 的语法被提出的特征属于这一类. 此 PEP 支持声明此类功能,
  并支持逐步淘汰在新功能下语义已更改的构造的 "旧" 含义. 但是, 它没有定义最终必须逐步淘汰哪些功能的策略.

- 那些旨在永久保持可选的, 例如, 如果他们更改了解释器中的某些默认设置.
  此类设置的示例可能是始终为某个模块发出行号指令的请求;
  在本说明书中没有提出这种特定的标志.

由于此 PEP 的主要目标是支持新的语言结构而不立即破坏旧库,
因此特别注意不要通过引入新语法来破坏旧库.


Syntax
======

directive_statement 是表单的声明 ::

   directive_statement: 'directive' ``NAME`` [atom] [';'] NEWLINE

指令中的名称表示指令的种类; 它定义了是否可以存在可选原子, 以及是否存在对原子的进一步语法或语义限制.
此外, 根据指令的名称, 可能会对指令施加某些额外的语法或语义限制 (例如, 指令在模块中的位置可能仅限于模块的顶部).

在 directive_statement 中, ``directive`` 是一个新的关键字. 根据 [1]_,
此关键字最初仅在用于指令语句时被视为关键字, 请参阅下面的 "向后兼容性".


Semantics
=========

指令语句指示 Python 解释器以不同的方式处理源文件; 该处理的具体细节取决于指令名称.
通常在处理源代码时解释可选原子; 该解释的细节取决于指令.


Specific Directives: transitional
=================================

如果向 Python 添加了不兼容的语法或语义变化, [1]_ 要求语言的过渡演变,
其中新特征最初与旧特征一起提供. 通过过渡指令可以实现这种转变.

在过渡指令中, "NAME" 是"过渡性". 原子必须存在, 它必须是一个 "名字".
定义语言更改时, 将定义该名称的可能值. 这种指令的一个例子是 ::

   directive transitional nested_scopes

过渡指令必须发生在模块中的任何其他语句之前, 除了文档字符串
(即, 只有当第一个语句是 ``STRING+`` 时, 它才可能显示为模块的第二个语句).


Backwards Compatibility
=======================

将 ``directive`` 作为新关键字引入可能会导致与现有代码不兼容. 遵循 [1]_ 中的指导原则,
在本规范的初始实现中, 指令只有在有效的 directive_statement 中使用时才是新关键字
(即, 如果它作为模块中的第一个非字符串标记出现).


Unresolved Problems:  directive as the first identifier
=======================================================

在模块中使用指令作为 ::

    directive = 1

(即, name 指令显示为模块中的第一件事) 将其视为关键字, 而不是标识符.
可以使用额外的前瞻标记将其分类为 "NAME", 但是 Python 标记器中没有这种前瞻功能.


Questions and Answers
=====================

**Q:** 看起来这个 PEP 是为了允许定义源代码字符集而编写的. 真的吗?

**A:** 不可以. 即使指令工具可以扩展为允许源代码编码, 也没有提出具体的指令.

**Q:** 那为什么要写这个 PEP 呢?

**A:** 它充当 [3]_ 的反提议, 它建议以新的含义重载 import 语句. 该 PEP 允许以更一般的方式解决该问题.

**Q:** 但不是混合源码和语言变化, 如混合苹果和橘子?

**A:** 也许. 为了解决这种差异, 已经定义了预定义的 "过渡" 指令.


References and Footnotes
========================

.. [1] PEP 5, 语言进化指南, Prescod
       http://www.python.org/dev/peps/pep-0005/

.. [2] PEP 227, 静态嵌套范围, Hylton
       http://www.python.org/dev/peps/pep-0227/

.. [3] PEP 236, 返回到 ``__future__``, Peters
       http://www.python.org/dev/peps/pep-0236/


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
