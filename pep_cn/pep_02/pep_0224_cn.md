
PEP: 224
Title: Attribute Docstrings
Version: $Revision$
Last-Modified: $Date$
Author: mal@lemburg.com (Marc-André Lemburg)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 23-Aug-2000
Python-Version: 2.1
Post-History:


Introduction
============

此 PEP 描述了 Python 2.0 的 "attribute docstring" 提议. 此 PEP 跟踪此功能的状态和所有权.
它包含功能的说明, 并概述了支持该功能所需的更改. 此文件的 CVS 修订历史记录包含确定的历史记录.


Rationale
=========

这个 PEP 对 Python 当前处理 Python 代码中嵌入的文档字符串的方式提出了一些补充.

Python 目前只处理文档字符串的情况, 它直接出现在类定义, 函数定义或模块中的第一个字符串文字之后.
字符串文字被添加到 ``__doc__`` 属性下的相关对象中, 从那时起可用于自我检查工具,
这些工具可以提取包含的信息以进行帮助, 调试和记录.

出现在上述位置以外的位置的文档字符串被忽略, 不会导致任何代码生成.

Here is an example::

    class C:
        "class C doc-string"

        a = 1
        "attribute C.a doc-string (1)"

        b = 2
        "attribute C.b doc-string (2)"

Python 字节码编译器当前忽略了文档字符串 (1) 和 (2),
但显然可以很好地用于记录它们之前的命名赋值.

该 PEP 还建议通过提出语义来使用这些情况, 以便将它们的内容添加到它们在新生成的属性名称下出现的对象中.

这个方法背后的原始思想也激发了上面的例子, 它启用了类属性的内联文档,
这些属性目前只能记录在类的文档字符串中或者使用不能用于自我检查的注释.


Implementation
==============

Docstrings 由字节码编译器作为表达式处理.
当前实现特殊情况上面提到的几个位置使用这些表达式, 但否则完全忽略字符串.

为了能够使用这些文档字符串来记录命名赋值 (这是定义类属性的自然方式),
编译器必须跟踪最后指定的名称, 然后使用此名称将 docstring 的内容分配给属性
包含对象的方法是将其作为常量存储, 然后在对象构造时将其添加到对象的命名空间中.

为了保留继承和隐藏 Python 的特殊属性 (具有前导和尾随双下划线的属性) 等功能,
必须应用一个特殊的名称修改, 它唯一地将文档字符串标识为属于名称赋值,
并允许稍后通过以下方式查找文档字符串. 检查命名空间.

The following name mangling scheme achieves all of the above::

    __doc_<attributename>__

为了跟踪最后指定的名称, 字节码编译器将此名称存储在编译结构的变量中.
此变量默认为 NULL. 当它看到 docstring 时, 它会检查变量并使用名称作为上述名称修改的基础,
以生成 docstring 到受损名称的隐式赋值. 然后它将变量重置为 NULL 以避免重复分配.

如果变量未指向名称 (即为 NULL), 则不进行任何分配. 这些将继续像以前一样被忽略.
所有经典文档都属于这种情况, 因此不会进行重复的赋值.

在上面的示例中, 这将导致创建以下新类属性 ::

    C.__doc_a__ == "attribute C.a doc-string (1)"
    C.__doc_b__ == "attribute C.b doc-string (2)"

在 SourceForge 上可以找到实现上述功能的当前 CVS 版本的 Python 2.0 的补丁 [1]_.


Caveats of the Implementation
=============================

由于实现在处理非表达式时不重置编译结构变量, 例如, 在函数定义中,
最后指定的名称保持活动状态, 直到下一个赋值或下一个 docstring 出现.

这可能导致 docstring 和赋值可能被其他表达式分隔的情况::

   class C:
       "C doc string"

       b = 2

       def x(self):
           "C.x doc string"
            y = 3
            return 1

       "b's doc string"

由于方法 "x" 的定义当前没有重置使用的赋值名称变量,
所以当编译器到达 docstring "b 的 doc 字符串" 时它仍然有效,
因此将字符串赋值给 ``__doc_b__``.

此问题的可能解决方案是重置编译器中所有非表达式节点的 name 变量.


Possible Problems
=================

即使极不可能, 属性文档字符串也可能意外地连接到属性的值::

   class C:
       x = "text" \
           "x's docstring"

尾部斜杠将导致 Python 编译器连接属性值和 docstring.

现代语法高亮编辑器很容易使这个事故可见, 并且只需在属性定义和文档字符串之间
插入 emtpy 线就可以完全避免可能的连接, 所以问题可以忽略不计.

另一个可能的问题是使用三引号字符串作为取消注释代码部分的方法.

如果在注释字符串开始之前恰好有一个赋值, 那么编译器会将注释视为 docstring 属性并将上述逻辑应用于它.

除了为其他未记录的属性生成文档字符串之外, 没有破损.


Comments from our BDFL
======================

早期对 Guido 的 PEP 发表评论:

    我 "有点" 喜欢有属性 docstrings 的想法 (这对我来说并不重要),
    但在你当前的提案中我有两件事我不喜欢:

    1. 你提出的语法太模糊了: 正如你所说, 独立字符串文字用于其他目的, 可能突然变成属性 docstrings.

    2. 我也不喜欢访问方法 (``__doc_<attrname>__``).

作者的回复:

    ::

        > 1. 你提出的语法太模糊了: 正如你所说, 独立
        >    字符串文字用于其他目的, 可能会突然
        >    成为属性 docstrings.


    这可以通过在编译器中引入一些额外的检查来修复, 以重置编译器结构中的 "doc attribute" 标志.

    ::

        > 2. 我也不喜欢访问方法 (``__doc_<attrname>__``).

    任何其他名称都可以. 它只需要符合这些标准:

    * 必须以两个下划线开头 (以匹配 ``__doc__``)
    * 必须使用某种形式的检查来提取 (例如, 使用包含某些固定名称部分的命名约定)
    * 必须与类继承兼容 (即应该存储为属性)

在 3月的晚些时候, Guido 在2001年3月 (在 python-dev 上) 宣读了这个 PEP.
以下是他在私人邮件中提到本 PEP 作者的拒绝理由:

    ...

    它可能有用, 但我真的很讨厌提出的语法.

    ::

        a = 1
        "foo bar"
        b = 1

    我真的无法知道 "foo bar" 是 a 还是 b 的文档字符串.

    ...

    您可以使用此约定::

        a = 1
        __doc_a__ = "doc string for a"

    这使它在运行时可用.

    ::

       > 您是否完全反对添加属性文档
       > Python 还是只是实现的方式? 
       > 我发现PEP中提出的语法非常直观
       > c.l.p 和私人电子邮件中的许多其他用户都支持它
       > 在我编写 PEP 时.

    这不是实现, 而是语法. 它没有在变量和 doc 字符串之间传达足够明确的耦合.


Copyright
=========

This document has been placed in the Public Domain.


References
==========

.. [1] http://sourceforge.net/patch/?func=detailpatch&patch_id=101264&group_id=5470



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
