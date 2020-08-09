
PEP: 236
Title: Back to the __future__
Version: $Revision$
Last-Modified: $Date$
Author: Tim Peters <tim.peters@gmail.com>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 26-Feb-2001
Python-Version: 2.1
Post-History: 26-Feb-2001


Motivation
==========

Python 不时会对核心语言结构的公布语义进行不兼容的更改, 或者以某种方式更改其意外 (依赖于实现的) 特性.
虽然这种情况从来没有反复无常, 而且总是以改善长期语言为目的, 但在短期内它会引起争议和破坏。.

PEP 5, 语言进化指南 [1]_ 提出了缓解烦恼的方法, 本 PEP 引入了一些机制来支持.

PEP 227, Statically Nested Scopes [2]_ 是第一个应用程序, 将在这里用作示例.


Intent
======

[注意: 这是策略, 因此最终应该进入 PEP 5 [1]_]

当对核心语言语法或语义进行不兼容的更改时:

1. 引入更改的版本 C 默认情况下不会更改语法或语义

2. 确定将来的版本 R, 其中将强制执行新的语法或语义.

3. PEP 3, Warning Framework [3]_ 中描述的机制用于在可能的情况下生成关于构造或操作的警告,
    其意义可能 [4]_ 在版本 R 中发生变化.

4. 新的 future_statement (见下文) 可以明确地包含在模块 M 中,
    来请求模块 M 中的代码使用当前版本 C 中的新语法或语义.

因此, 对于至少一个版本, 旧代码默认继续工作, 尽管它可能会开始生成新的警告消息.
在此期间可以继续迁移到新的语法或语义, 使用 future_statement 使包含它的模块
就像新的语法或语义已经被强制执行一样.

请注意, 除非可以破坏现有代码, 否则不需要将 future_statement 机器包含在新功能中;
完全向后兼容的添加可以 - 而且应该 - 在没有相应的 future_statement 的情况下引入.


Syntax
======

future_statement 只是一个使用保留模块名称 ``__future__`` 的 from/import 语句 ::

    future_statement: "from" "__future__" "import" feature ["as" name]
                      (","feature ["as" name])*

    feature: identifier
    name: identifier

此外, 所有 future_statments 必须出现在模块顶部附近.
在 future_statement 之前可以出现的唯一行 :

+ 模块 docstring (如果有的话).
+ 注释.
+ 空白行.
+ 其他 future_statements.

例子::

    """This is a module docstring."""

    # This is a comment, preceded by a blank line and followed by
    # a future_statement.
    from __future__ import nested_scopes

    from math import sin
    from __future__ import alabaster_weenoblobs  # compile-time error!
    # That was an error because preceded by a non-future_statement.


Semantics
=========

在编译时特别识别和处理 future_statement: 通常通过生成不同的代码来实现对核心构造的语义的更改.
甚至可能是新功能引入了新的不兼容语法 (例如新的保留字), 在这种情况下, 编译器可能需要以不同方式解析模块.
这些决定直到运行时才能被推迟.

对于任何给定的版本, 编译器知道已定义了哪些功能名称, 并且如果 future_statement 包含其未知的功能,
则会引发编译时错误 [5]_.

直接运行时语义与任何 ``import`` 语句相同: 有一个标准模块 ``__future__.py``,
稍后描述, 它将在执行 future_statement 时以通常的方式导入.

*interesting* 运行时语义取决于模块中出现的 future_statement "imported" 的特定功能.

请注意, 该声明没有什么特别之处 ::

    import __future__ [as name]

这不是一个 future_statement; 它是一个普通的 import 语句, 没有特殊的语义或语法限制.


Example
=======

在文件 scope.py 中考虑此代码 ::

    x = 42
    def f():
        x = 666
        def g():
            print "x is", x
        g()
    f()

在 2.0 下, 它打印::

    x is 42

嵌套范围 [2]_ 正在 2.1 中引入. 但在 2.1 下, 它仍然打印 ::

    x is 42

并且还会生成警告.

在 2.2 中, 以及 2.1 *if* ``from __future__ import nested_scopes``
包含在 ``scope.py`` 的顶部, 它打印 ::

    x is 666


Standard Module __future__.py
=============================

``Lib/__future__.py`` 是一个真实的模块, 有三个目的:

1. 避免混淆分析 import 语句的现有工具, 并期望找到他们导入的模块.

2. 确保 future_statements 在 2.1 之前的版本下运行至少产生运行时异常
    (导入 ``__future__`` 将失败, 因为在 2.1 之前没有该名称的模块).

3. 记录何时引入了不兼容的更改, 以及它们何时 - 或者是 - 必须进行更改. 这是一种可执行文档,
    可以通过导入 ``__future__`` 并检查其内容以编程方式进行检查.

``__future__.py`` 中的每个语句都是这种形式 ::

    FeatureName = "_Feature(" OptionalRelease "," MandatoryRelease ")"

通常, *OptionalRelease* < *MandatoryRelease*, 两者都是与 ``sys.version_info`` 相同形式的5元组::

    (PY_MAJOR_VERSION, # the 2 in 2.1.0a3; an int
     PY_MINOR_VERSION, # the 1; an int
     PY_MICRO_VERSION, # the 0; an int
     PY_RELEASE_LEVEL, # "alpha", "beta", "candidate" or "final"; string
     PY_RELEASE_SERIAL # the 3; an int )

*OptionalRelease* 记录其中的第一个版本::

    from __future__ import FeatureName

被接受了.

对于尚未发生的 *MandatoryReleases*, *MandatoryRelease* 预测该功能将成为该语言的一部分的版本.

当功能成为语言的一部分时, Else *MandatoryRelease* 记录;
在此时或之后的版本中, 模块不再需要 ::

    from __future__ import FeatureName

使用有问题的功能, 但可能会继续使用此类导入.

*MandatoryRelease* 也可能是 ``None``, 意味着计划的功能被删除.

``class_Feature`` 的实例有两个相应的方法, ``.getOptionalRelease()`` 和 ``.getMandatoryRelease()``.

从 ``__future__.py`` 中不会删除任何要素线.

示例行 ::

    nested_scopes = _Feature((2, 1, 0, "beta", 1), (2, 2, 0, "final", 0))

这意味着 ::

    from __future__ import nested_scopes

将在 2.1b1 或之后的所有版本中工作, 并且 nested_scopes 旨在从 2.2 版开始实施.


Resolved Problem:  Runtime Compilation
======================================

几个 Python 功能可以在模块的运行时编译代码:

1. The ``exec`` statement.
2. The ``execfile()`` function.
3. The ``compile()`` function.
4. The ``eval()`` function.
5. The ``input()`` function.

由于包含 future_statement 命名功能 F 的模块 M 明确地请求当前版本就像 F 的未来版本一样,
任何从 M 内传递给其中一个的文本动态编译的代码也应该使用相关的新语法或语义与 F.
2.1 版本确实这样做.

但是, 这并不总是需要的. 例如, ``doctest.testmod(M)`` 编译从 M 中的字符串中获取的示例,
这些示例应该使用 M 的选择, 而不一定是 doctest 模块的选择. 在 2.1 版本中, 这是不可能的,
并且还没有建议用于解决此问题的方案. 注意: PEP 264 后来通过向 ``compile()`` 添加可选参数以灵活的方式解决了这个问题.

在任何情况下, 由 ``exec``, ``execfile()`` 或 ``compile()`` 动态编译的文本出现的 "near the top"
(参见上面的语法) 的 future_statement 适用于生成的代码块, 但对执行这样一个 ``exec``, ``execfile()``
或 ``compile()`` 的模块没有进一步的影响. 这不能用于影响 ``eval()`` 或 ``input()``,
因为它们只允许表达式输入, 而 future_statement 不是表达式.


Resolved Problem:  Native Interactive Shells
============================================

有两种方法可以获得交互式 shell:

1. 通过从命令行调用 Python 而不使用脚本参数.

2. 通过命令行使用 ``-i`` 开关和脚本参数调用 Python.

交互式 shell 可以看作是运行时编译的极端情况 (见上文): 实际上,
在交互式 shell 提示符下输入的每个语句都运行一个新的 ``exec``, ``compile()``
或 ``execfile()``. 在交互式 shell 中键入的 future_statement 适用于 shell 会话的其余部分,
就像 future_statement 出现在模块的顶部一样.


Resolved Problem:  Simulated Interactive Shells
===============================================

"手工构建" 的交互式 shell (通过 IDLE 和 Emacs Python 模式等工具) 应该像本机交互式 shell 一样 (参见上文).
但是, 本机交互式 shell 内部使用的机制尚未公开, 并且工具构建自己的交互式 shell 来实现所需行为的方式并不明确.

注意: PEP 264 后来通过在标准 ``codeop.py`` 中添加智能来解决这个问题.
不使用标准库 shell 助手的模拟 shell 可以通过利用 PEP 264 添加的 ``compile()`` 的新可选参数来获得类似的效果。.


Questions and Answers
=====================

那么一个 "from __past__" 的版本, 可以回归旧特性?
-----------------------------------------------------------------

超出本 PEP 的范围. 但是, 作者似乎不太可能. 如果你想追求它, 写一个 PEP.

由于 Python 虚拟机的更改导致的不兼容性如何?
--------------------------------------------------------------------------

在 PEP 的范围之外, 尽管 PEP 5 [1]_ 也提出了宽限期, 而 future_statement 也可能在那里发挥作用.

由于 Python C API 的更改导致的不兼容性如何?
--------------------------------------------------------------

超出本 PEP 的范围.

我想将 future_statements 包装在 try/except 块中, 因此我可以使用不同的代码, 具体取决于我正在运行的 Python 版本. 为什么我不能?
-------------------------------------------------------------------------------------------------------------------------------------------------

Sorry!  ``try/except`` 是一个运行时功能; future_statements 主要是编译时的手法, 你的 ``try/except`` 在编译完成后很久就会发生.
也就是说, 当你执行 ``try/except`` 时, 对模块有效的语义已经完成了. 由于 ``try/except`` 不会完成它看起来它应该完成的东西,
所以根本不允许. 我们还希望保持这些特殊陈述非常容易找到和识别.

请注意, 您可以直接导入 ``__future__``, 并使用其中的信息以及 ``sys.version_info`` 来确定您在某个特定功能的状态下运行的版本的位置.

回到 nested_scopes 示例, 如果版本 2.2 出现并且我仍然没有更改我的代码怎么办? 那我怎么能保持 2.1 的特性呢?
----------------------------------------------------------------------------------------------------------------------------------------------------

继续使用 2.1, 而不是移动到 2.2, 直到您更改代码. future_statement 的目的是让那些及时掌握最新版本的人们的生活更轻松.
如果你不这样做, 我们不讨厌你, 但是你的问题要难以解决, 有问题的人需要写一个 PEP 来解决它们.
future_statement 面向不同的受众.

重载 ``import`` 很糟糕. 为什么不为此引入新的声明?
--------------------------------------------------------------------------

就像 ``lambda lambda nested_scopes``? 也就是说, 除非我们引入新关键字,
否则我们无法引入全新的声明. 但是如果我们引入一个新关键字, 那本身就会破坏旧代码.
这太具有讽刺意味了. 是的, 重载 ``import`` 确实很糟糕,
但不像备选方案那样精力充沛 - 因为 future_statements 是100％向后兼容的.


Copyright
=========

This document has been placed in the public domain.


References and Footnotes
========================

.. [1] PEP 5, 语言进化指南, Prescod
       http://www.python.org/dev/peps/pep-0005/

.. [2] PEP 227, 静态嵌套范围, Hylton
       http://www.python.org/dev/peps/pep-0227/

.. [3] PEP 230, 警告框架, Van Rossum
       http://www.python.org/dev/peps/pep-0230/

.. [4] 请注意, 这可能是 *may* 而不是 *will*: 防范于未然.
        当然, 如果以合理的成本可以避免, 就不会产生虚假警告.

.. [5] 这可以确保 future_statement 在第一个已知 （但是 >= 2.1) 的特定功能之前的版本下运行会引发编译时错误,
        而不是默默地做错事. 如果传送到 2.1 之前的版本, 则会因为无法导入 ``__future__`` 而引发运行时错误
		(2.1 版本之前标准发行版中不存在此类模块, 并且双下划线使其成为保留名称).



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
