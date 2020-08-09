
PEP: 257
Title: Docstring Conventions
Version: $Revision$
Last-Modified: $Date$
Author: David Goodger <goodger@python.org>,
        Guido van Rossum <guido@python.org>
Discussions-To: doc-sig@python.org
Status: Active
Type: Informational
Content-Type: text/x-rst
Created: 29-May-2001
Post-History: 13-Jun-2001


Abstract
========

此 PEP 记录了与 Python 文档字符串相关的语义和约定.


Rationale
=========

此 PEP 的目的是标准化文档字符串的高级结构:
它们应该包含的内容以及如何表达它 (不涉及文档字符串中的任何标记语法).
PEP 包含惯例, 而不是法律或语法.

    "通用惯例也提供了所有可维护性, 清晰度, 一致性以及良好编程习惯的基础.
	它不做的是坚持你违背自己的意愿. 那是 Python!"

    -- Tim Peters on comp.lang.python, 2001-06-16

如果你违反这些惯例, 你会得到的最糟糕的是一些肮脏的外表.
但是一些软件 (例如 Docutils_ docstring 处理系统 [1]_ [2]_)
会知道这些约定, 所以遵循它们会得到最好的结果.


Specification
=============

什么是 Docstring?
--------------------

docstring 是一个字符串文字, 作为模块, 函数, 类或方法定义中的第一个语句出现.
这样的 docstring 成为该对象的 ``__doc__`` 特殊属性.

所有模块通常都应该有文档字符串, 模块导出的所有函数和类也应该有文档字符串.
公共方法 (包括 ``__init__`` 构造函数) 也应该有 docstrings.
包可以记录在包目录中 ``__init __.py`` 文件的模块 docstring 中.

Python 代码中其他地方出现的字符串文字也可以作为文档.
它们不被 Python 字节码编译器识别, 并且不能作为运行时对象属性访问
(即未分配给 ``__doc__``), 但软件工具可以提取两种类型的额外文档字符串:

1. 在模块, 类或 ``__init__`` 方法的顶层进行简单赋值后,
   立即出现的字符串文字称为 "attribute docstrings".

2. 在另一个 docstring 之后立即出现的字符串文字被称为 "additional docstrings".

有关属性和其他文档字符串的详细说明, 请参阅 PEP 258, "Docutils 设计规范" [2]_.

为了保持一致性, 请始终在 docstrings 周围使用 ``"""triple double quotes"""``.
如果你在文档字符串中使用任何反斜杠, 请使用 ``r"""raw triple double quotes"""``
对于 Unicode 文档字符串, 请使用 ``u"""Unicode triple-quoted strings"""``.

有两种形式的文档字符串: 单行和多行文档字符串.


One-line Docstrings
--------------------

单行是非常明显的示例. 他们应该真的适合一条线. 例如::

    def kos_root():
        """Return the pathname of the KOS root directory."""
        global _kos_root
        if _kos_root: return _kos_root
        ...

注意:

- 即使字符串适合一行, 也会使用三重引号. 这使得以后扩展它变得容易.

- 开始引用与结束引用相同. 对于单行来说, 这看起来更好.

- 在 docstring 之前或之后没有空行.

- docstring 是一个以句点结尾的短语. 它将函数或方法的效果规定为命令
  ("Do this", "Return that"), 而不是描述; 例如, 不要写 "返回路径名 ...".

- 单行文档字符串不应该是重复函数/方法参数的 "签名" (可以通过自我检查获得).
  不要这样做::

      def function(a, b):
          """function(a, b) -> list"""

  这种类型的 docstring 仅适用于 C 函数 (例如, 内置函数), 其中自我检查是不可能的.
  但是, *返回值* 的性质不能通过自我检查来确定, 因此应该提及.
  这种文档字符串的首选形式是类似的::

      def function(a, b):
          """Do X and return a list."""

  (当然, "Do X" 应该被有用的描述所取代!)


Multi-line Docstrings
----------------------

多行文档字符串由一个摘要行组成, 就像一行文档字符串, 后跟一个空行,
后面是更详细的描述. 摘要行可以由自动索引工具使用;
重要的是它适合一行并通过空行与文档字符串的其余部分分开.
摘要行可以与开始引用在同一行或在下一行. 整个 docstring 缩进与第一行的引号相同
(参见下面的示例).

在记录类的所有文档字符串 (单行或多行) 之后插入一个空行 -- 一般来说,
类的方法通过单个空行彼此分隔, 并且 docstring 需要从第一个方法偏移一个空白行.

脚本 (独立程序) 的 docstring 应该可用作其 "用法" 消息, 当使用不正确或缺少的参数
(或者可能使用 "-h" 选项, "help") 调用脚本时打印. 这样的 docstring 应该记录脚本的功能和命令行语法,
环境变量和文件. 用法消息可以相当复杂 (几个屏幕已满), 并且应该足以让新用户正确使用该命令,
以及对复杂用户的所有选项和参数的完整快速参考.

模块的 docstring 通常应该列出模块导出的类, 异常和函数 (以及任何其他对象), 每个对应的摘要.
(这些摘要通常比对象的 docstring 中的摘要行提供的细节更少.) 包的 docstring
(即包的 ``__init __.py`` 模块的 docstring) 也应该列出包导出的模块和子包.

函数或方法的 docstring 应该总结其行为并记录其参数, 返回值, 副作用,
引发的异常以及何时可以调用它们的限制 (如果适用, 则全部).
应该指出可选参数. 应该记录关键字参数是否是接口的一部分.

类的 docstring 应该总结其行为并列出公共方法和实例变量. 如果该类要进行子类化,
并且具有子类的附加接口, 则应该单独列出此接口 (在 docstring 中).
类构造函数应该在 docstring 中记录为 ``__init__`` 方法.
各个方法应该由他们自己的 docstring 记录.

如果一个类是另一个类的子类, 并且它的行为主要是从该类继承的,
那么它的 docstring 应该提到这一点并总结这些差异.
使用动词 "override" 来表示子类方法替换超类方法, 而不调用超类方法;
使用动词 "extend" 来表示子类方法调用超类方法 (除了它自己的特性).

*Do not* 在运行文本时使用 Emacs 约定来提及大写的函数或方法的参数. Python 区分大小写,
参数名称可用于关键字参数, 因此 docstring 应该记录正确的参数名称.
最好在单独的行中列出每个参数. 例如::

    def complex(real=0.0, imag=0.0):
        """Form a complex number.

        Keyword arguments:
        real -- the real part (default 0.0)
        imag -- the imaginary part (default 0.0)
        """
        if imag == 0.0 and real == 0.0:
            return complex_zero
        ...

除非整个文档字符串适合一行, 否则将结束引号放在一行上.
这样, 可以在它上面使用 Emacs 的 ``fill-paragraph`` 命令.


Handling Docstring Indentation
------------------------------

文档字符串处理工具将从文档字符串的第二行和更多行中删除均匀数量的缩进,
等于第一行之后所有非空行的最小缩进. 文档字符串第一行中的任何缩进
(即, 直到第一个换行符) 都是无关紧要的并被删除. 保留 docstring 中后续行的相对缩进.
应该从 docstring 的开头和结尾删除空行.

由于代码比单词更精确, 因此这里是算法的实现::

    def trim(docstring):
        if not docstring:
            return ''
        # Convert tabs to spaces (following the normal Python rules)
        # and split into a list of lines:
        lines = docstring.expandtabs().splitlines()
        # Determine minimum indentation (first line doesn't count):
        indent = sys.maxint
        for line in lines[1:]:
            stripped = line.lstrip()
            if stripped:
                indent = min(indent, len(line) - len(stripped))
        # Remove indentation (first line is special):
        trimmed = [lines[0].strip()]
        if indent < sys.maxint:
            for line in lines[1:]:
                trimmed.append(line[indent:].rstrip())
        # Strip off trailing and leading blank lines:
        while trimmed and not trimmed[-1]:
            trimmed.pop()
        while trimmed and not trimmed[0]:
            trimmed.pop(0)
        # Return a single string:
        return '\n'.join(trimmed)

此示例中的 docstring 包含两个换行符, 因此长度为3行.
第一行和最后一行是空白的::

    def foo():
        """
        This is the second line of the docstring.
        """

为了显示::

    >>> print repr(foo.__doc__)
    '\n    This is the second line of the docstring.\n    '
    >>> foo.__doc__.splitlines()
    ['', '    This is the second line of the docstring.', '    ']
    >>> trim(foo.__doc__)
    'This is the second line of the docstring.'

修整后, 这些文档字符串是等效的::

    def foo():
        """A multi-line
        docstring.
        """

    def bar():
        """
        A multi-line
        docstring.
        """


References and Footnotes
========================

.. [1] PEP 256, 文档字符串处理系统框架, Goodger
   (http://www.python.org/dev/peps/pep-0256/)

.. [2] PEP 258, Docutils 设计规范, Goodger
   (http://www.python.org/dev/peps/pep-0258/)

.. _Docutils: http://docutils.sourceforge.net/

.. _Python 风格指南:
   http://www.python.org/dev/peps/pep-0008/

.. _Doc-SIG: http://www.python.org/sigs/doc-sig/


Copyright
=========

This document has been placed in the public domain.


Acknowledgements
================

The "Specification" text comes mostly verbatim from the `Python Style
Guide`_ essay by Guido van Rossum.

This document borrows ideas from the archives of the Python Doc-SIG_.
Thanks to all members past and present.



..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   fill-column: 70  
   sentence-end-double-space: t  
   End:  
