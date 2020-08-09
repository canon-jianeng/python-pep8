
PEP: 8
Title: Style Guide for Python Code
Version: $Revision$
Last-Modified: $Date$
Author: Guido van Rossum <guido@python.org>,
        Barry Warsaw <barry@python.org>,
        Nick Coghlan <ncoghlan@gmail.com>
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 05-Jul-2001
Post-History: 05-Jul-2001, 01-Aug-2013


Introduction
============

本文档提供了 Python 代码的编码规则, 包括 Python 发行版中的标准库
请参阅在 Python 的 C 实现 C 代码的伴随信息 PEP 描述风格指南 [1]_

本文和 PEP 257（Docstring Conventions）改编自 Guido 的原始 Python 风格指南文章,
还有一些来自 Barry 风格指南的补充 [2]_

随着时间的推移, 随着语言本身的变化, 过去的约定被淘汰, 这种风格指南随着时间的推移而不断发展

A Foolish Consistency is the Hobgoblin of Little Minds
======================================================

Guido 的一个关键见解是代码的读取频率远高于编写代码
此处提供的准则旨在提高代码的可读性, 并使其在各种 Python 代码中保持一致
正如 PEP 20 所说, "可读性很重要"

风格指南是关于一致性的
与此风格指南的一致性非常重要, 项目内的一致性更为重要
一个模块或功能内的一致性是最重要的

但是, 知道何时不一致 - 有时风格指南建议不适用
如有疑问, 请使用您的最佳判断, 查看其他示例并确定最佳效果, 并且不要犹豫, 不要问!

特别是: 不要为了遵守这个 PEP 而破坏向后兼容性!

忽略特定指南的其他一些好理由:

1. 在应用指南时, 即使是习惯于遵循此 PEP 阅读代码的人, 也会使代码的可读性降低
2. 为了与周围的代码保持一致（也许是出于历史原因） - 虽然这也是一个清理别人的混乱的机会（真正的XP风格）
3. 因为有问题的代码早于指南的引入, 所以没有其他理由可以修改该代码
4. 当代码需要与不支持风格指南推荐的功能的旧版 Python 兼容时

Code lay-out
============

Indentation
-----------

每个缩进级别使用4个空格

延续线应该垂直对齐包装元素, 使用 Python 的隐含线连接 ()，[] 和 {} 内，
或使用 *Hanging indentation* [＃fn-hi]_ 使用悬挂式缩进时, 应该考虑以下因素;
第一行应该没有参数, 应该使用进一步的缩进来明确区分自己作为延续线

Yes::

    # Aligned with opening delimiter.
    foo = long_function_name(var_one, var_two,
                             var_three, var_four)

    # More indentation included to distinguish this from the rest.
    def long_function_name(
            var_one, var_two, var_three,
            var_four):
        print(var_one)

    # Hanging indents should add a level.
    foo = long_function_name(
        var_one, var_two,
        var_three, var_four)

No::

    # Arguments on first line forbidden when not using vertical alignment.
    foo = long_function_name(var_one, var_two,
        var_three, var_four)

    # Further indentation required as indentation is not distinguishable.
    def long_function_name(
        var_one, var_two, var_three,
        var_four):
        print(var_one)

对于连续线, 4个空格规则是可选的

Optional::

    # Hanging indents *may* be indented to other than 4 spaces.
    foo = long_function_name(
      var_one, var_two,
      var_three, var_four)

.. _`多行 if 语句`:

当 ``if`` 语句的条件部分长到足以要求它跨多行写入时, 值得注意的是两个字符关键字的组合 (即``if``)，
加上一个空格, 加上一个左括号为多行条件的后续行创建一个自然的 4个空格缩进,
这可能会与嵌套在``if``语句中的缩进代码集产生视觉冲突, 这些代码自然也会缩进到4个空格
这个 PEP 没有明确地说明如何(或是否)在``if``语句中进一步在视觉上区分这些条件行和嵌套套件
在这种情况下可接受的选择包括但不限于::

    # No extra indentation.
    if (this_is_one_thing and
        that_is_another_thing):
        do_something()

    # Add a comment, which will provide some distinction in editors
    # supporting syntax highlighting.
    if (this_is_one_thing and
        that_is_another_thing):
        # Since both conditions are true, we can frobnicate.
        do_something()

    # Add some extra indentation on the conditional continuation line.
    if (this_is_one_thing
            and that_is_another_thing):
        do_something()

(另请参阅下面关于是否在二元运算符之前或之后中断的讨论)

多行结构上的 ()/[]/{} 可以在列表最后一行的第一个非空白字符下排列，如::

    my_list = [
        1, 2, 3,
        4, 5, 6,
        ]
    result = some_function_that_takes_arguments(
        'a', 'b', 'c',
        'd', 'e', 'f',
        )

或者它可以排在启动多行结构的行的第一个字符下面，如::

    my_list = [
        1, 2, 3,
        4, 5, 6,
    ]
    result = some_function_that_takes_arguments(
        'a', 'b', 'c',
        'd', 'e', 'f',
    )

Tabs or Spaces?
---------------

空格是首选的缩进方法

Tabs 应该仅用于与已使用 Tabs 缩进的代码保持一致

Python 3 不允许混合使用制表符和空格来缩进

使用制表符和空格的混合缩进的 Python 2 代码应该转换为仅使用空格

当使用``-t``选项调用 Python 2 命令行解释器时, 它会发出有关非法混合制表符和空格的代码的警告
使用``-tt``时, 这些警告会出错, 强烈推荐这些选项！

Maximum Line Length
-------------------

将所有行限制为最多79个字符

对于具有较少结构限制（文档字符串或注释）的长文本块, 行长度应限制为72个字符

限制所需的编辑器窗口宽度使得可以并排打开多个文件，并且在使用在相邻列中显示两个版本的代码审查工具时可以正常工作

大多数工具中的默认包装会破坏代码的可视化结构, 使其更难理解
选择限制是为了避免在窗口宽度设置为80的情况下包装在编辑器中, 即使工具在包装线条时在最终列中放置标记字形
某些基于 Web 的工具可能根本不提供动态换行

有些团队强烈倾向于更长的行字符长度
对于专门或主要由可以就此问题达成一致的团队维护的代码,
可以将行长度从80个字符增加到100个字符 (有效地将最大长度增加到99个字符),
前提是评论和文档字符串仍然包装 72个字符

Python 标准库是保守的, 需要将行限制为79个字符 (文档字符串/注释限制为72个)

包装长行的首选方法是在 (), [] 和 {} 内使用 Python 隐含的行连续
通过在括号中包装表达式, 可以在多行上分割长行, 这些应该优先使用反斜杠来延续行

反斜杠有时可能仍然合适, 例如, 多个长的``with``语句不能使用隐式延续, 因此可以接受反斜杠::

    with open('/path/to/some/file/you/want/to/read') as file_1, \
         open('/path/to/some/file/being/written', 'w') as file_2:
        file_2.write(file_1.read())

(参见前面关于`multiline if-statements`_的讨论, 以进一步思考这种多行``with``语句的缩进)

另一个这样的情况是``assert``语句

确保适当地缩进续行

Should a line break before or after a binary operator?
------------------------------------------------------

几十年来, 推荐的风格是在二元运算符之后打破
但这会以两种方式损害可读性:
操作员倾向于分散在屏幕上的不同列上, 并且每个操作符都会从其操作数移到前一行
在这里, 眼睛必须做额外的工作来分辨哪些项目被添加以及哪些项目被减去::

    # No: operators sit far away from their operands
    income = (gross_wages +
              taxable_interest +
              (dividends - qualified_dividends) -
              ira_deduction -
              student_loan_interest)

为了解决这个可读性问题, 数学家和他们的出版商遵循相反的惯例
Donald Knuth 在他的 * Computers and Typesetting * 系列中解释了传统规则:
"虽然段落中的公式总是在二元操作和关系之后中断, 但显示的公式总是在二元操作之前中断" [3] _

遵循数学传统通常会产生更易读的代码::

    # Yes: easy to match operators with operands
    income = (gross_wages
              + taxable_interest
              + (dividends - qualified_dividends)
              - ira_deduction
              - student_loan_interest)

在 Python 代码中, 只要约定在本地一致, 就允许在二元运算符之前或之后中断
对于新代码, 建议使用 Knuth 的样式

Blank Lines
-----------

使用两个空行环绕顶级函数和类定义

类中的方法定义由单个空行包围

可以使用额外的空白行(谨慎地)来分离相关功能组
在一堆相关的单行之间可以省略空行 (例如, 一组虚拟实现)

在函数中使用空行, 谨慎地指示逻辑部分

Python 接受 control-L (即^L) 换页符作为空格;
许多工具将这些字符视为页面分隔符, 因此您可以使用它们来分隔文件相关部分的页面
请注意, 某些编辑器和基于 Web 的代码查看器可能无法将 control-L 识别为换页符, 并且会在其位置显示另一个字形

Source File Encoding
--------------------

核心 Python 发行版中的代码应该始终使用 UTF-8 (或 Python 2 中的 ASCII)

使用 ASCII (在Python 2中) 或 UTF-8 (在Python 3中) 的文件不应该具有编码声明

在标准库中, 非默认编码应该仅用于测试目的, 或者当注释或文档字符串需要提及包含非ASCII字符的作者名称时;
否则, 使用``\ x``，``\ u``，``\ U``或``\ N``转义是在字符串文字中包含非ASCII数据的首选方法

对于Python 3.0及更高版本, 标准库规定了以下策略 (参见PEP 3131):
Python 标准库中的所有标识符必须使用仅ASCII标识符, 并且应该尽可能使用英语单词 (在许多情况下, 缩写和技术使用的术语不是英语)
此外, 字符串文字和注释也必须是ASCII格式
唯一的例外是 (a) 测试非ASCII功能的测试用例, 以及 (b) 作者姓名
名称不是基于拉丁字母 (latin-1，ISO/IEC 8859-1 字符集) 的作者必须在此字符集中提供其名称的音译

鼓励全球受众的开源项目采用类似的策略

Imports
-------

- Imports 通常应该分开, 例如::

      Yes: import os
      import sys

      No:  import sys, os

    虽然可以这样说:

      from subprocess import Popen, PIPE

- 导入总是放在文件的顶部, 就在任何模块注释和文档字符串之后, 以及模块全局变量和常量之前

    应该按以下顺序对 Imports 进行分组:

    1. 标准库导入
    2. 第三方库导入
    3. 本地应用程序/库特定导入

    您应该在每组导入之间添加一个空行

- 建议使用绝对导入, 因为它们通常更具可读性, 并且如果导入系统配置不正确
  (例如, 当包中的目录最终位于``sys.path``时), 它们往往表现得更好 (或至少提供更好的错误消息) ::

    import mypkg.sibling
    from mypkg import sibling
    from mypkg.sibling import example

  但是, 显式相对导入是绝对导入的可接受替代方法,
  尤其是在处理复杂的包布局时, 使用绝对导入会增加不必要地冗长 ::

    from . import sibling
    from .sibling import example

  标准库代码应该避免复杂的包布局, 并始终使用绝对导入

  隐式相对导入应该永远不会被使用, 并且已经在 Python 3 中删除了

- 从包含类的模块导入类时，通常可以拼写这个 ::

      from myclass import MyClass
      from foo.bar.yourclass import YourClass

  如果此拼写导致本地名称冲突, 则拼写它们 ::

      import myclass
      import foo.bar.yourclass

  并使用 "myclass.MyClass" 和 "foo.bar.yourclass.YourClass"

- 应该避免使用通配符导入 (``from <module> import *``),
因为它们不清楚命名空间中存在哪些名称, 使读者和许多自动化工具混淆
通配符导入有一个可防御的用例, 即将内部接口重新发布为公共 API 的一部分
(例如, 使用可选加速器模块中的定义覆盖接口的纯 Python 实现, 以及确切的定义将是 被覆盖的事先不知道)

  以这种方式重新发布名称时, 以下有关公共和内部接口的指南仍然适用

Module level dunder names
-------------------------

模块级别 "dunders" (即具有两个前导和两个尾部下划线的名称),
例如``__ all__``, ``__ author__``, ``__ version__``等应该放在模块 docstring 之后
但是在任何 import 语句之前 *除了* ``from __future__`` imports
Python 要求 future-imports 必须在除 docstrings 之外的任何其他代码之前出现在模块中

For example::

    """This is the example module.

    This module does stuff.
    """

    from __future__ import barry_as_FLUFL

    __all__ = ['a', 'b', 'c']
    __version__ = '0.1'
    __author__ = 'Cardinal Biggles'

    import os
    import sys

String Quotes
=============

在 Python 中, 单引号字符串和双引号字符串是相同的
该 PEP 不会对此提出建议, 选择规则并坚持下去
但是, 当字符串包含单引号或双引号字符时, 请使用另一个字符串以避免字符串中出现反斜杠, 它提高了可读性

对于三引号字符串, 始终使用双引号字符与 PEP 257 中的 docstring 约定一致

Whitespace in Expressions and Statements
========================================

Pet Peeves
----------

在以下情况下避免无关的空格:

- 紧靠在 (), [] 或 {} 内 ::

      Yes: spam(ham[1], {eggs: 2})
      No:  spam( ham[ 1 ], { eggs: 2 } )

- 在尾随逗号和后续括号之间 ::

      Yes: foo = (0,)
      No:  bar = (0, )

- 紧接着逗号，分号或冒号 ::

      Yes: if x == 4: print x, y; x, y = y, x
      No:  if x == 4 : print x , y ; x , y = y , x

- 但是, 在切片中, 冒号的行为类似于二元运算符, 并且两侧的数量应该相等 (将其视为具有最低优先级的运算符)
  在扩展切片中, 两个冒号必须具有相同的间距, 异常: 省略切片参数时, 省略空格

    Yes::

      ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
      ham[lower:upper], ham[lower:upper:], ham[lower::step]
      ham[lower+offset : upper+offset]
      ham[: upper_fn(x) : step_fn(x)], ham[:: step_fn(x)]
      ham[lower + offset : upper + offset]

    No::

      ham[lower + offset:upper + offset]
      ham[1: 9], ham[1 :9], ham[1:9 :3]
      ham[lower : : upper]
      ham[ : upper]

- 紧接在左圆括号之前, 它启动函数调用的参数列表 ::

      Yes: spam(1)
      No:  spam (1)

- 紧接在开始索引或切片的左括号之前 ::

      Yes: dct['key'] = lst[index]
      No:  dct ['key'] = lst [index]

- 赋值(或其他)运算符周围有多个空格, 以使其与另一个运算符对齐

  Yes::

      x = 1
      y = 2
      long_variable = 3

  No::

      x             = 1
      y             = 2
      long_variable = 3

Other Recommendations
---------------------

- 避免在任何地方尾随空格, 因为它通常是看不见的, 所以可能会造成混淆
  例如: 反斜杠后跟空格和换行符不算作行继续标记
  有些编辑器不保留它, 许多项目 (如 CPython 本身) 都有预先提交的拒绝它的钩子

- 总是围绕这些二元运算符, 两边都有一个空格:
  赋值 (``=``), 扩充赋值 (``+ =``, ``-=``等), 比较 (``==`` ，``<``, ``>``, ``!=``, ``<>``, ``<=``,
  ``>=``, ``in``, ``not in``, ``is``, ``is not``), 布尔 (``and``, ``or``, ``not``)

- 如果使用具有不同优先级的运算符, 请考虑在具有最低优先级的运算符周围添加空格, 使用你自己的判断;
  但是, 永远不要使用多个空格, 并且在二元运算符的两边始终具有相同数量的空格

  Yes::

      i = i + 1
      submitted += 1
      x = x*2 - 1
      hypot2 = x*x + y*y
      c = (a+b) * (a-b)

  No::

      i=i+1
      submitted +=1
      x = x * 2 - 1
      hypot2 = x * x + y * y
      c = (a + b) * (a - b)

- 当用于表示关键字参数或默认参数值时, 不要在``=``符号周围使用空格

  Yes::

      def complex(real, imag=0.0):
          return magic(r=real, i=imag)

  No::

      def complex(real, imag = 0.0):
          return magic(r = real, i = imag)

- 函数注释应该使用冒号的常规规则, 并且如果存在的话, ``->``箭头周围总是有空格
  (有关函数注释的更多信息, 请参阅下面的`Function Annotations`_)

  Yes::

      def munge(input: AnyStr): ...
      def munge() -> AnyStr: ...

  No::

      def munge(input:AnyStr): ...
      def munge()->PosInt: ...

- 将参数注释与默认值组合时, 请使用 ``=`` 符号周围的空格 (但仅适用于同时具有注释和默认值的参数)

  Yes::

      def munge(sep: AnyStr = None): ...
      def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...

  No::

      def munge(input: AnyStr=None): ...
      def munge(input: AnyStr, limit = 1000): ...

- 通常不鼓励使用复合语句 (同一行上的多个语句)

  Yes::

      if foo == 'blah':
          do_blah_thing()
      do_one()
      do_two()
      do_three()

  Rather not::

      if foo == 'blah': do_blah_thing()
      do_one(); do_two(); do_three()

- 虽然有时可以将 if/for/while 与主体代码放在同一行上, 但是不要对多子句语句执行此操作
  还要避免折叠如此长的行!

  Rather not::

      if foo == 'blah': do_blah_thing()
      for x in lst: total += x
      while t < 10: t = delay()

  Definitely not::

      if foo == 'blah': do_blah_thing()
      else: do_non_blah_thing()

      try: something()
      finally: cleanup()

      do_one(); do_two(); do_three(long, argument,
                                   list, like, this)

      if foo == 'blah': one(); two(); three()

When to use trailing commas
===========================

尾随逗号通常是可选的, 除了它们在创建一个元素的元组时是必需的 (并且在 Python 2 中它们具有``print``语句的语义)
为清楚起见, 建议将后者括在 (技术上冗余的) 括号中

Yes::

    FILES = ('setup.cfg',)

OK, but confusing::

    FILES = 'setup.cfg',

当尾随逗号是多余的时, 它们通常在使用版本控制系统时有用, 当预期值列表, 参数或导入项目随时间延长时
模式是将每个值（等）单独放在一行上, 始终添加一个尾随逗号, 并在下一行添加 ()/[]/{}
但是, 在与结束分隔符相同的行上使用尾随逗号是没有意义的 (除了上面的单例元组的情况)

Yes::

    FILES = [
        'setup.cfg',
        'tox.ini',
        ]
    initialize(FILES,
               error=True,
               )

No::

    FILES = ['setup.cfg', 'tox.ini',]
    initialize(FILES, error=True,)


Comments
========

与代码相矛盾的注释比没有注释更糟糕
始终优先考虑在代码更改时保持注释的最新状态!

注释应该是完整的句子
第一个单词应该大写, 除非它是一个以小写字母开头的标识符 (永远不会改变标识符的大小写!)

块注释通常由完整句子构成的一个或多个段落组成, 每个句子以句点结束

除了最后一句之外, 你应该在句子结束之后的多句话注释中使用两个空格

写英文时，请遵循 Strunk 和 White

来自非英语国家的 Python 程序员: 请用英语撰写您的注释,
除非您 120％ 确定将不会有不懂您的语言的人阅读该代码

Block Comments
--------------

块注释通常适用于跟随它们的一些(或所有)代码, 并且缩进到与该代码相同的级别
块注释的每一行以 ``＃`` 和单个空格开头 (除非它是注释中的缩进文本)

块注释中的段落由包含单个``＃``的行分隔

Inline Comments
---------------

谨慎使用内联注释

内联注释是与语句在同一行上的注释
内联注释应该与语句至少分隔两个空格
它们应该以 ＃ 和单个空格开头

内联注释是不必要的, 如果他们状态明显的话, 实际上会分散注意力, 不要这样做 ::

    x = x + 1                 # Increment x

但有时, 这很有用 ::

    x = x + 1                 # Compensate for border

Documentation Strings
---------------------

编写良好文档字符串的惯例 (a.k.a. "docstrings") 在 PEP 257 中永远存在的

- 为所有公共模块, 函数, 类和方法编写文档字符串
  对于非公共方法, 文档字符串不是必需的, 但是您应该有一个注释来描述该方法的作用
  此注释应该出现在``def``行之后

- PEP 257 描述了良好的文档字符串约定
  请注意, 最重要的是, 结束多行文档字符串的`` """ ``应该在一行上, 例如 ::

      """Return a foobang

      Optional plotz says to frobnicate the bizbaz first.
      """

- 对于一个文档字符串, 请在同一行保持结束 `` """ ```

Naming Conventions
==================

Python 库的命名约定有点混乱, 所以我们永远不会完全一致 - 尽管如此, 这是目前推荐的命名标准
新模块和包 (包括第三方框架) 应该写入这些标准, 但是现有库具有不同的样式, 内部一致性是首选

Overriding Principle
--------------------

作为 API 的公共部分对用户可见的名称应该遵循反映使用而非实现的约定

Descriptive: Naming Styles
--------------------------

有很多不同的命名方式, 它有助于识别正在使用的命名样式, 与它们的用途无关

通常会区分以下命名样式:

- ``b`` (single lowercase letter)
- ``B`` (single uppercase letter)
- ``lowercase``
- ``lower_case_with_underscores``
- ``UPPERCASE``
- ``UPPER_CASE_WITH_UNDERSCORES``

- ``CapitalizedWords`` （或 CapWords, 或 CamelCase - 因其字母凹凸不平[4] _而得名
  这有时也被称为 StudlyCaps)

  Note: 在 CapWords 中使用首字母缩略词时, 请将首字母缩略词的所有字母大写
  因此 HTTPServerError 优于 HttpServerError

- ``mixedCase`` (与大写字母的不同之处在于初始小写字符! - 驼峰式大小写)
- ``Capitalized_Words_With_Underscores`` (这个样式是丑的!)

还有使用简短唯一前缀将相关名称组合在一起的风格
这在 Python 中并不常用, 但为了完整性而提到它
例如, ``os.stat()``函数返回一个元组, 其元素传统上有``st_mode``, ``st_size``, ``st_mtime``等名称
(这样做是为了强调与 POSIX 系统调用 struct 的字段的对应关系, 这有助于程序员熟悉它)

X11 库为其所有公共函数使用前导 X
在 Python 中, 这种样式通常被认为是不必要的,
因为属性和方法名称以对象为前缀, 函数名称以模块名称为前缀

此外, 还会识别使用前导或尾随下划线的以下特殊形式 (这些形式通常可与任何示例约定结合使用):

- ``_single_leading_underscore``: 弱 "内部使用" 指针
  例如, ``indicator. E.g. ``from M import *``不会导入名称以下划线开头的对象

- ``single_trailing_underscore_``: 按惯例用于避免与 Python 关键字冲突, 例如: ::

      Tkinter.Toplevel(master, class_='ClassName')

- ``__double_leading_underscore``: 在命名一个class属性时, 调用name mangling
  (在类 FooBar 中, ``__ boo`` 变成 ``_FooBar__boo``; 见下文)

- ``__double_leading_and_trailing_underscore__``: 位于用户控制的命名空间中的 "magic" 对象或属性
  例如, ``__init__``, ``__ import__`` 或 ``__file__`` 不要发明这样的名字; 仅按记录使用它们

Prescriptive: Naming Conventions
--------------------------------

Names to Avoid
~~~~~~~~~~~~~~

切勿将字符 'l'(小写字母 el), 'O'(大写字母 oh) 或 'I'(大写字母 eye) 用作单个字符变量名称

在某些字体中, 这些字符与数字 1 和 0 无法区分, 当试图使用 'l' 时，请使用 'L'

ASCII Compatibility
~~~~~~~~~~~~~~~~~~~

标准库中使用的标识符必须与 ASCII 兼容,
如 PEP 3131 的 `policy section <https://www.python.org/dev/peps/pep-3131/#policy-specification>`_ 中所述

Package and Module Names
~~~~~~~~~~~~~~~~~~~~~~~~

模块应该有简短的全小写名称
如果提高可读性, 可以在模块名称中使用下划线
Python 包也应该有简短的全小写名称, 但不鼓励使用下划线

当用 C 或 C++ 编写的扩展模块具有提供更高级别 (例如更多面向对象) 的接口的 Python 模块时,
C/C++ 模块具有前置下划线 (例如 ``_socket``)

Class Names
~~~~~~~~~~~

类名通常应使用 CapWords 约定

在接口被记录并主要用作可调用的情况下, 可以使用函数的命名约定

请注意, 内置名称有一个单独的约定:
大多数内置名称是单个单词 (或两个单词一起运行), CapWords 约定仅用于异常名称和内置常量

Type variable names
~~~~~~~~~~~~~~~~~~~

在 PEP 484 中引入的类型变量的名称通常应该使用 CapWords, 而不是短名称: ``T``, ``AnyStr``, ``Num``
建议在用于相应声明协变或逆变行为的变量中添加后缀 ``_co`` 或 ``_contra``, 例子 ::

  from typing import TypeVar

  VT_co = TypeVar('VT_co', covariant=True)
  KT_contra = TypeVar('KT_contra', contravariant=True)

Exception Names
~~~~~~~~~~~~~~~

因为异常应该是类, 所以类命名约定适用于此处
但是, 您应该在异常名称上使用后缀 "Error" (如果异常实际上是错误)

Global Variable Names
~~~~~~~~~~~~~~~~~~~~~

(我们希望这些变量仅用于一个模块内) 约定与函数的约定大致相同

设计为通过``from M import *``使用的模块应该使用``__all__``机制来防止输出全局变量,
或者使用旧的约定为这些全局变量加上下划线 (你可能希望用它来表示这些全局变量是 "module non-public")

Function Names
~~~~~~~~~~~~~~

函数名称应该为小写, 并根据需要用下划线分隔, 以提高可读性

只有在已经是主流风格 (例如, threading.py) 的上下文中才允许使用 mixedCase, 以保持向后兼容性

Function and method arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

总是使用``self``作为实例方法的第一个参数

总是使用``cls``作为类方法的第一个参数

如果函数参数的名称与保留关键字冲突, 通常最好附加单个尾随下划线而不是使用缩写或拼写损坏
因此``class_``比``clss``更好, (或许更好的方法是通过使用同义词来避免这种冲突)

Method Names and Instance Variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

使用函数命名规则: 小写, 必要时用下划线分隔, 以提高可读性

仅对非公共方法和实例变量使用一个前置下划线

为避免与子类的名称冲突, 请使用两个前置下划线来调用 Python 的名称修改规则

Python 使用类名来破坏这些名称: 如果类 Foo 具有名为``__a``的属性, 则不能通过``Foo .__a``访问它
(一个始终坚持的用户仍然可以通过调用 ``Foo._Foo__a`` 获得访问权限)
通常, 两个前置下划线应该仅用于避免名称与设计为子类的类中的属性冲突

注意: 关于 __names 的使用存在一些争议 (见下文)

Constants
~~~~~~~~~

常量通常在模块级别定义, 并以全部大写字母书写, 下划线分隔单词
例子包括 ``MAX_OVERFLOW`` 和 ``TOTAL``

Designing for inheritance
~~~~~~~~~~~~~~~~~~~~~~~~~

始终决定一个类的方法和实例变量 (统称为 "属性") 是公共的还是非公共的
如有疑问, 请选择非公共; 将公共属性设为非公共更容易公开

公共属性是指您希望类的无关客户端使用的属性, 您承诺避免向后不兼容的更改
非公共属性是指不打算由第三方使用的属性; 您不能保证非公共属性不会更改甚至不会被删除

我们在这里不使用术语 "private", 因为在 Python 中没有属性是私有的 (不工作的通常是不必要的)

另一类属性是属于 "子类 API" 的属性 (通常在其他语言中称为 "protected")
某些类旨在从中继承, 以扩展或修改类的行为方面
在设计这样的类时, 请注意明确决定哪些属性是公共的, 哪些是子类 API 的一部分, 哪些属性真正只能由基类使用

考虑到这一点, 这是 Pythonic 指南:

- 公共属性应该没有前置下划线

- 如果公共属性名称与保留关键字冲突, 请在属性名称后附加单个尾随下划线, 这比缩写或损坏的拼写更可取
  (但是, 尽管有这个规则, 'cls' 是任何已知为类的变量或参数的首选拼写, 尤其是类方法的第一个参数)

  注1: 有关类方法, 请参阅上面的参数名称建议

- 对于简单的公共数据属性, 最好只公开属性名称, 而不使用复杂的 accessor/mutator 方法
  请记住, 如果您发现简单的数据属性需要增加功能行为, Python 提供了一条简单的未来增强路径
  在这种情况下, 使用属性隐藏简单数据属性访问语法背后的函数实现

  注1: 属性仅适用于新式类

  注2: 尽量保持功能行为无副作用, 例如, 缓存等的副作用一般是很好的

  注3: 避免使用属性进行计算成本高昂的操作; 属性符号使调用者相信访问（相对）便宜

- 如果您的类要进行子类化, 并且您具有不希望使用子类的属性, 请考虑使用双前置下划线和没有尾随下划线来命名它们
  这将调用 Python 的名称修改算法, 其中类的名称被修改为属性名称
  如果子类无意中包含具有相同名称的属性, 这有助于避免属性名称冲突

  注1: 请注意, 在修改的名称中只使用简单的类名,
  如果子类同时选择相同的类名和属性名, 则仍然会获得名称冲突

  注2: 名称修改可以使某些用途, 例如, 调试和``__getattr__()``, 不太方便
  但是, 名称修改算法已有详细记录, 并且易于手动执行

  注3: 不是每个人都喜欢名称错误
  尽量平衡避免意外名称冲突与高级调用者潜在使用的需要

Public and internal interfaces
------------------------------

任何向后兼容性保证仅适用于公共接口, 因此, 用户能够清楚地区分公共和内部接口是很重要的

记录的接口被认为是公共的, 除非文档明确声明它们是临时的或内部接口免于通常的向后兼容性保证
应该假定所有未记录的接口都是内部接口

为了更好地支持内省, 模块应该使用``__all__``属性在其公共 API 中显式声明名称
将``__all__``设置为空列表表示该模块没有公共 API

即使适当地设置``__all__``, 内部接口 (包, 模块, 类, 函数, 属性或其他名称) 仍应该以单个前置下划线为前缀

如果任何包含名称空间 （包, 模块或类) 的内容被视为内部接口, 则该接口也被视为内部接口

应该始终将导入的名称视为实现细节
其他模块不能依赖于对这些导入名称的间接访问, 除非它们是包含模块的 API 的明确记录部分,
例如 ``os.path`` 或包的 ``__init__`` 模块, 它从子模块中公开功能

Programming Recommendations
===========================

- 代码应该以不影响 Python 的其他实现 (PyPy, Jython, IronPython, Cython, Psyco等) 的方式编写

  例如, 不要依赖 CPython 为 ``a += b`` 或 ``a = a + b`` 形式的语句有效实现字符串连接
  即使在 CPython 中, 这种优化也很脆弱 (它只适用于某些类型), 并且在不使用引用计数的实现中根本不存在
  在库的性能敏感部分, 应该使用 `` ".join() `` 形式, 这将确保在各种实现中以线性时间进行连接

- 像 None 这样的单个对象的比较应该总是用 ``is`` 或 ``is not`` 来完成, 而不是等于运算符

  另外, 当你的意思是 ``if x is not None`` 时, 要小心写 ``if x`` - 例如,
  在测试默认为 None 的变量或参数是否设置为其他值时,
  另一个值可能有一个在布尔上下文中可能为 false 的类型 (如容器)!

- 使用 ``is``` 运算符而不是 ``not ... is``, 虽然两个表达式在功能上是相同的, 但前者更具可读性和首选性

  Yes::

      if foo is not None:

  No::

      if not foo is None:

- 当使用丰富的比较实现排序操作时, 最好实现所有六个操作 (``__eq__``, ``__ne__``,
  ``__lt__``, ``__le__``, ``__gt__``, ``__ge__``) 而不是依赖其他代码来进行特定的比较

  为了最大限度地减少所涉及的工作量, ``functools.total_ordering()`` decorator 提供了一个工具来生成缺少的比较方法

  PEP 207 表示 Python 假定反身性规则 *are*
  因此，解释器可以将 ``y > x`` 与 ``x < y``, ``y >= x`` 与 ``x <= y`` 交换,
  并且可以交换 ``x == y`` 和 ``x != y``的参数, ``sort()`` 和 ``min()`` 操作保证使用 ``<`` 运算符,
  ``max()`` 函数使用 ``>`` 运算符, 但是, 最好实施所有六个操作, 以免在其他情况下出现混淆

- 始终使用 def 语句, 而不是将 lambda 表达式直接绑定到标识符的赋值语句

  Yes::

      def f(x): return 2*x

  No::

      f = lambda x: 2*x

  第一种形式意味着生成的函数对象的名称特别是 'f' 而不是泛型 '<lambda>', 这对于一般的回溯和字符串表示更有用
  赋值语句的使用消除了 lambda 表达式在显式 def 语句上提供的唯一好处 (即它可以嵌入到更大的表达式中)

- 从 ``Exception`` 而不是 ``BaseException`` 派生异常
  来自 ``BaseException`` 的直接继承保留用于捕获它们的异常几乎总是错误的事情

  根据代码 *catching* 异常可能需要用来设计异常层次结构, 而不是引发异常的位置
  以编程方式在回答 "What went wrong?" 的问题, 而不是仅仅声明 "A problem occurred"
  (请参阅 PEP 3151, 了解本课程的示例是为内置异常层次结构学习的)

  类命名约定适用于此处, 但如果异常是一个错误, 则应该将后缀 "Error" 添加到异常类中
  用于非本地流控制或其他形式的信号的非错误异常不需要特殊后缀

- 适当地使用异常链接
  在 Python 3 中, "raise X from Y" 应该用于指示显式替换而不会丢失原始回溯

  当有意替换内部异常 （在 Python 2 中使用 "raise X" 或在 Python 3.3+ 中 "raise X from None"), 确保将相关细节传递给新异常
  (例如, 在将 KeyError 转换为 AttributeError 时, 保留属性名称, 或在新的异常消息中嵌入原始异常的文本）

- 在 Python 2 中引发异常时, 使用 ``raise ValueError('message')``,
  而不是旧形式 `` raise ValueError，'message' ``

  后一种形式不是合法的 Python 3 语法

  paren-using 表单还意味着当异常参数很长或包含字符串格式时, 由于包含括号, 您不需要使用行连续字符

- 捕获异常时, 尽可能提及特定的异常, 而不是使用一个简单的 ``except:`` 子句

  For example, use::

      try:
          import platform_specific_module
      except ImportError:
          platform_specific_module = None

  一个简单的 ``except:`` 子句将捕获 SystemExit 和 KeyboardInterrupt 异常,
  使得用 Control-C 中断程序变得更加困难，并且可以掩盖其他问题
  如果你想捕获所有表示程序错误的异常, 请使用 ``except Exception:``
  (暴露的异常等同于 ``except BaseException:``)

  一个好的经验法则是将暴露的 "except" 子句的使用限制为两种情况:

  1. 如果异常处理程序将打印出来或记录回溯; 至少用户会意识到发生了错误
  2. 如果代码需要做一些清理工作, 那么让异常用 ``raise`` 向上传播
     ``try ... finally`` 可以更好地处理这种情况

- 绑定捕获的名称异常时, 更喜欢 Python 2.6 中添加的显式名称绑定语法 ::

      try:
          process_data()
      except Exception as exc:
          raise DataProcessingFailedError(str(exc))

  这是 Python 3 中唯一支持的语法, 可以避免与旧的基于逗号的语法相关的歧义问题

- 在捕获操作系统错误时, 更喜欢 Python 3.3 中引入的显式异常层次结构, 而不是自我检查 ``errno`` 值

- 另外, 对于所有 try/except 子句, 将 ``try`` 子句限制为所需的绝对最小代码量, 同样, 这可以避免掩盖错误

  Yes::

      try:
          value = collection[key]
      except KeyError:
          return key_not_found(key)
      else:
          return handle_value(value)

  No::

      try:
          # Too broad!
          return handle_value(collection[key])
      except KeyError:
          # Will also catch KeyError raised by handle_value()
          return key_not_found(key)

- 当资源是特定代码段的本地资源时, 使用 ``with`` 语句确保在使用后立即可靠地清理它, try/finally 语句也是可以接受的

- 除了获取和释放资源之外, 还应该通过单独的函数或方法调用上下文管理器, 例如:

  Yes::

               with conn.begin_transaction():
                   do_stuff_in_transaction(conn)

  No::

               with conn:
                   do_stuff_in_transaction(conn)

  后一个例子没有提供任何信息来表明 ``__enter__`` 和 ``__exit__`` 方法正在做一些事情,
  而不是在交易后关闭连接, 在这种情况下, 明确是很重要的

- 在返回声明中保持一致
  函数中的所有 return 语句都应该返回一个表达式, 或者它们都不应该返回
  如果任何 return 语句返回一个表达式, 那么任何没有返回值的 return 语句都应该明确地将其声明为 ``return None``,
  并且在函数末尾应该有一个显式的 return 语句 (如果可以访问)

  Yes::

      def foo(x):
          if x >= 0:
              return math.sqrt(x)
          else:
              return None

      def bar(x):
          if x < 0:
              return None
          return math.sqrt(x)

  No::

      def foo(x):
          if x >= 0:
              return math.sqrt(x)

      def bar(x):
          if x < 0:
              return
          return math.sqrt(x)

- 使用字符串方法, 而不是字符串模块

字符串方法总是快得多, 并与 unicode 字符串共享相同的 API
如果需要向后兼容早于 2.0 的 Pythons, 则覆盖此规则

- 使用 `` ''.startswith() `` 和 `` ''.endswith() `` 而不是字符串切片来检查前缀或后缀

  startswith() 和 endswith() 更清晰，更不容易出错, 例如:

      Yes: if foo.startswith('bar'):
      No:  if foo[:3] == 'bar':

- 对象类型比较应该始终使用 isinstance() 而不是直接比较类型 ::

      Yes: if isinstance(obj, int):

      No:  if type(obj) is type(1):

  检查对象是否为字符串时, 请记住它也可能是一个 unicode 字符串!
  在 Python 2 中, str 和 unicode 有一个共同的基类 basetring, 所以你可以这样做 ::

      if isinstance(obj, basestring):

  请注意, 在 Python 3 中, ``unicode`` 和 ``basestring`` 不再存在(只有``str``),
  而 bytes 对象不再是一种字符串 (它是一个整数序列)

- 对于序列, (字符串, 列表, 元组), 请使用空序列为假的事实 ::

      Yes: if not seq:
           if seq:

      No: if len(seq):
          if not len(seq):

- 不要编写依赖于标志尾随空格的字符串文字, 这样的尾随空白在视觉上是难以区分的,
  一些编辑器 (或者最近 reindent.py) 将修整它们

- 不要使用 ``==`` 将布尔值与 True 或 False 进行比较 ::

      Yes:   if greeting:
      No:    if greeting == True:
      Worse: if greeting is True:

Function Annotations
--------------------

随着 PEP 484 的采用, 函数注释的样式规则正在发生变化

- 为了向前兼容, Python 3 代码中的函数注释应该优选使用 PEP 484 语法
  (上一节中有一些注释的格式化建议)

- 不再鼓励先前在 PEP 8 中推荐的注释样式

- 但是, 在 stdlib 之外, 现在鼓励在 PEP 484 规则内进行试验
  例如, 使用 PEP 484 样式类型注释标记大型第三方库或应用程序,
  查看添加这些注释是多么容易, 并观察它们的存在是否增加了代码可理解性

- Python 标准库在采用这样的注释时应该保守, 但是它们的使用允许用于新代码和大型重构

- 对于想要对函数注释进行不同使用的代码, 建议对表单进行注释 ::

    # type: ignore

  靠近文件顶部; 这告诉类型检查器忽略所有注释
  (在 PEP 484 中可以找到更细粒度的禁用类型检查器意见的方法)

- 像代码校验, 类型检查器是可选的, 单独的工具
  默认情况下, Python 解释器不应该由于类型检查而发出任何消息, 并且不应该基于注释更改其行为

- 不想使用类型检查器的用户可以自由地忽略它们
  但是, 预计第三方库包的用户可能希望在这些包上运行类型检查器
  为此, PEP 484 建议使用桩文件: .pyi 文件由类型检查程序读取, 而不是相应的 .py 文件
  桩文件可以与库一起发布, 也可以单独通过类型化的 repo [5]_（通过库作者的许可）

- 对于需要向后兼容的代码, 可以以注释的形式添加类型注释, 参见 PEP 484 的相关章节 [6]_

.. rubric:: 脚注

.. [#fn-hi] *Hanging indentation* 是一种类型设置样式, 其中段落中的所有行都是缩进的, 除了第一行
   在 Python 的上下文中, 该术语用于描述一种样式, 其中带括号的语句的左括号是该行的最后一个非空白字符, 后续行缩进直到右括号


References
==========

.. [1] PEP 7, C 代码风格指南, van Rossum

.. [2] Barry 的 GNU Mailman 风格指南
       http://barry.warsaw.us/software/STYLEGUIDE.txt

.. [3] Donald Knuth 的 *The TeXBook*, 第 195 页 和 第 196 页.

.. [4] http://www.wikipedia.com/wiki/CamelCase

.. [5] Typeshed repo
   https://github.com/python/typeshed

.. [6] Python 2.7 和 跨代码的建议语法
   https://www.python.org/dev/peps/pep-0484/#suggested-syntax-for-python-2-7-and-straddling-code

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
