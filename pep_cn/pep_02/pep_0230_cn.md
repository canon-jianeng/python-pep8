
PEP: 230
Title: Warning Framework
Version: $Revision$
Last-Modified: $Date$
Author: guido@python.org (Guido van Rossum)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created:
Python-Version: 2.1
Post-History: 05-Nov-2000


Abstract
========

这个 PEP 提出了一个 C 和 Python 级别的 API, 以及命令行标志, 用于发出警告消息并控制它们发生的情况.
这主要基于 GvR 在2000年11月5日发布到 python-dev 的提案, 其中一些想法 (例如使用类别分类警告) 
与 Paul Prescod 在同一天发布的反提案合并. 此外, 试图实施该提案引起了一些小的调整.


Motivation
==========

随着 Python 3000 的出现, 除了错误之外, 还必须开始发布有关使用过时或已弃用功能的警告.
在编译时和运行时, 还有许多其他原因可以从 C 和 Python 代码发出警告.

警告不是致命的, 因此程序可能会在一次执行期间多次触发相同的警告.
如果程序发出无穷无尽的相同警告, 那将会很烦人.
因此, 需要一种抑制多个相同警告的机制.

还希望用户控制打印哪些警告. 虽然通常一直看到所有警告很有用,
但有时候在生产程序中立即修复代码是不切实际的.
在这种情况下, 应该有一种方法来抑制警告.

在程序开发期间能够抑制特定警告也是有用的, 例如, 当一段第三方代码生成警告时, 无法立即修复,
或者无法修复代码时 (可能会为完美的代码生成警告消息). 在这种情况下提供抑制所有警告是不明智的:
开发人员会错过有关其余代码的警告.

另一方面, 也存在可以想到的情况, 其中一些或所有警告被更好地视为错误. 例如, 它可能是本地编码标准,
不应该使用特定的弃用特征. 为了强制执行此操作, 能够将有关此特定功能的警告转换为错误, 引发异常
(不必将所有警告都转换为错误).

因此, 我建议引入一个灵活的 "警告过滤器", 它可以过滤掉警告或将其更改为异常, 基于:

- 代码中的位置 (每个包, 模块或函数)

- 警告类别 (警告类别将在下面讨论)

- 一条特定的警告信息

警告筛选器必须可以从命令行和 Python 代码进行控制.


APIs For Issuing Warnings
=========================

- 从 Python 发出警告::

      import warnings
      warnings.warn(message[, category[, stacklevel]])

  如果给出, category 参数必须是警告类别类 (见下文); 它默认为 warnings.UserWarning.
  如果警告过滤器将发出的特定警告更改为错误, 则可能会引发异常.
  stacklevel 可以由 Python 编写的包装函数使用, 就像这样::

      def deprecation(message):
          warn(message, DeprecationWarning, level=2)

  这使警告引用 deprecation() 的调用者, 而不是 deprecation() 本身的源
  (因为后者会破坏警告消息的目的).

- 从 C 发出警告::

    int PyErr_Warn(PyObject *category, char *message);

  正常返回 0, 如果引发异常则返回 1, (因为警告已转换为异常, 或者由于实现中的故障, 例如内存不足).
  category 参数必须是警告类别类 (见下文) 或 ``NULL``, 在这种情况下, 它默认为 ``PyExc_RuntimeWarning``.
  当 ``PyErr_Warn()`` 函数返回1时, 调用者应该进行正常的异常处理.

  ``PyErr_Warn()`` 的当前 C 实现导入警告模块 (用 Python 实现) 并调用它的 ``warn()`` 函数.
  这最小化为实现警告功能而需要添加的 C 代码量.

  [XXX 公开问题: 如何在 lexing 或 parsing 发出警告, 这些警告没有可用的异常机制?]


Warnings Categories
===================

有许多内置异常代表警告类别. 此分类对于能够过滤掉警告组非常有用.
目前定义了以下警告类别类:

- ``Warning`` -- 这是所有警告类别类的基类, 它本身也是 Exception 的子类

- ``UserWarning`` -- ``warnings.warn()`` 的默认类别

- ``DeprecationWarning`` -- 有关已弃用功能的警告的基本类别

- ``SyntaxWarning`` -- 关于可疑句法特征的警告的基本类别

- ``RuntimeWarning`` -- 有关可疑运行时功能的警告的基类别

[XXX: 在本 PEP 的审查期间可能会提出其他警告类别.]

这些标准警告类别可从 C 获得为 ``PyExc_Warning``, ``PyExc_UserWarning`` 等.
从 Python 开始, 它们可以在 ``__builtin__`` 模块中使用, 因此无需导入.

用户代码可以通过继承其中一个标准警告类别来定义其他警告类别.
警告类别必须始终是 Warning 类的子类.


The Warnings Filter
===================

警告过滤器控制是否忽略, 显示警告或变成错误 (引发异常).

警告过滤器有三个方面:

- 用于有效确定特定 ``warnings.warn()`` 或 ``PyErr_Warn()`` 调用的处理的数据结构.

- 用于从 Python 源代码控制过滤器的 API.

- 命令行切换到控制过滤器.

警告过滤器分几个阶段工作. 它针对 (预期是常见的) 情况进行了优化,
其中一遍又一遍地从代码中的相同位置发出相同的警告.

首先, 警告过滤器收集发出警告的模块和行号; 这些信息可以通过 ``sys._getframe()`` 获得.

从概念上讲, 警告过滤器维护过滤规范的有序列表; 任何特定警告依次与列表中的每个过滤器规范匹配,
直到找到匹配为止; 匹配决定了匹配的配置. 每个入口都是如下的元组::

  (category, message, module, lineno, action)

- category 是一个类 (``warnings.Warning`` 的子类), 其警告类别必须是子类才能匹配

- message 是一个编译的正则表达式, 警告消息必须匹配 (匹配不区分大小写)

- module 是模块名称必须匹配的已编译正则表达式

- lineno 是一个整数, 发生警告的行号必须匹配, 或 0 表示所有行号

- action 是以下字符串之一:

  - "error" -- 将匹配警告转换为异常

  - "ignore" -- 永远不会打印匹配的警告

  - "always" -- 始终打印匹配警告

  - "default" -- 为发出警告的每个位置打印第一次匹配警告

  - "module" -- 为发出警告的每个模块打印第一次匹配警告

  - "once" -- 仅打印第一次出现的匹配警告

由于 ``Warning`` 类是从内置的 ``Exception`` 类派生的, 为了将警告变为错误,
我们只需要提出 ``category(message)``.


Warnings Output And Formatting Hooks
====================================

当警告过滤器决定发出警告时 (但不会在它决定引发异常时), 它会传递
有关函数 ``warnings.showwarning(message, category, filename, lineno)`` 的信息.
此函数的默认实现将警告文本写入 ``sys.stderr``, 并显示文件名的源代码行.
它有一个可选的第5个参数, 可用于指定与 ``sys.stderr`` 不同的文件.

警告的格式化由一个单独的函数 ``warnings.formatwarning(message, category, filename, lineno)`` 完成.
这将返回一个字符串 (可能包含换行符并以换行符结尾), 可以打印该字符串来获得 ``showwarning()`` 函数的相同效果.


API For Manipulating Warning Filters
====================================
::

    warnings.filterwarnings(message, category, module, lineno, action)

这将检查参数的类型, 编译消息和模块正则表达式, 并将它们作为元组插入警告过滤器前面.

::

    warnings.resetwarnings()

将警告过滤器重置为空.


Command Line Syntax
===================

应该有命令行选项来指定最常见的过滤操作, 我希望至少包括这些操作:

- 压制所有警告

- 到处抑制特定的警告信息

- 禁止特定模块中的所有警告

- 将所有警告变为异常

我提出以下命令行选项语法::

    -Waction[:message[:category[:module[:lineno]]]]

Where:

- 'action' 是允许操作之一的缩写
  ("error", "default", "ignore", "always", "once", or "module")

- 'message' 是一个消息字符串; 匹配其消息文本是 'message' 的初始子字符串的警告
  (匹配不区分大小写)

- 'category' 是标准警告类别类名的缩写, *or*
  [package.]module.classname 形式的用户定义警告类型类的完全限定名称.

- 'module' 是一个模块名称 (可能是 package.module)

- 'lineno' 是一个整数行号

除了 'action' 之外的所有部分都可以省略,
其中剥离空格后的空值与省略的值相同.

解析 Python 命令行的 C 代码将所有 -W 选项的主体保存在字符串列表中,
并将其作为 sys.warnoptions 提供给警告模块. 警告模块在首次导入时解析这些.
在解析 sys.warnoptions 期间检测到的错误并不是致命的; 消息将写入 sys.stderr 并继续处理该选项.

Examples:

``-Werror``
    将所有警告变为错误

``-Wall``
    显示所有警告

``-Wignore``
   忽略所有警告

``-Wi:hello``
    忽略其消息文本以 "hello" 开头的警告

``-We::Deprecation``
    将弃用警告变为错误

``-Wi:::spam:10``
   忽略模块垃圾邮件第10行的所有警告

``-Wi:::spam -Wd:::spam:10``
    忽略模块垃圾邮件中的所有警告, 但第10行除外

``-We::Deprecation -Wd::Deprecation:spam``
    将弃用警告转换为错误, 模块垃圾邮件除外


Open Issues
===========

一些悬而未决的问题脱颖而出:

- 如何在 lexing 或 parsing 期间发出警告, 没有可用的异常机制?

- 建议的命令行语法有点难看 (尽管简单的情况并不是那么糟糕: ``-Werror``, ``-Wignore`` 等).
  任何人都有更好的主意?

- 我有点担心滤波器规格太复杂了. 也许只过滤类别和模块 (而不是消息文本和行号) 就足够了?

- 模块名称和文件名之间存在一些混淆. 报告使用文件名, 但过滤器规范使用模块名称.
  也许它应该允许文件名?

- 我完全不相信包处理得当.

- 我们是否需要更多标准警告类别?  更少?

- 为了最小化启动开销, 警告模块通过第一次调用 ``PyErr_Warn()`` 导入.
  它在导入时执行命令行解析 ``-W`` 选项. 因此, 无警告程序可能不会抱怨无效的 ``-W`` 选项.


Rejected Concerns
=================

Paul Prescod, Barry Warsaw 和 Fred Drake 提出了一些我认为并不重要的问题.
我在这里解决他们 (问题被解释, 而不是他们的话):

- Paul: ``warn()`` 应该是内置的或声明, 使得其易于使用.

  Response: "from warnings import warn" 很容易.

- Paul: 如果我有一个 speed-critical 模块在内循环中触发警告怎么办?
   应该可以禁用检测警告的开销 (而不仅仅是抑制警告).

  Response: 重写内循环来避免触发警告.

- Paul: 如果我想查看警告的完整上下文, 该怎么办?

  Response: 使用 ``-Werror`` 将其变成异常.

- Paul: 我更喜欢 ":\*:\*:" 到 ":::" 来留下警告规范的部分内容.

  Response: 我不喜欢.

- Barry: 如果 lineno 可以是 range  格式, 那将是很好的.

  Response: 太复杂了.

- Barry: 我想添加自己的警告操作. 也许如果 'action' 可以是可调用的, 也可以是字符串.
  然后在我的 IDE 中, 我可以将其设置为 "mygui.popupWarningsDialog".

  Response: 为此, 你将覆盖 ``warnings.showwarning()``.

- Fred: 为什么警告类别类必须在 ``__builtin__`` 中?

  Response: 这是最简单的实现, 因为警告类别必须在第一个 ``PyErr_Warn()`` 调用之前在 C 中可用,
  它会导入警告模块. 我认为将它们作为内置函数是可用的, 没有问题.


Implementation
==============

Here's a prototype implementation:
http://sourceforge.net/patch/?func=detailpatch&patch_id=102715&group_id=5470

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
