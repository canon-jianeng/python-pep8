
PEP: 264
Title: Future statements in simulated shells
Version: $Revision$
Last-Modified: $Date$
Author: Michael Hudson <mwh@python.net>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Requires: 236
Created: 30-Jul-2001
Python-Version: 2.2
Post-History: 30-Jul-2001


Abstract
========

正如在 PEP 236 中所指出的, "模拟交互式 shell" 没有明确的方法来
模拟 "真实" 交互式 shell 中 ``__future__`` 语句的特性,
即具有 ``__future__`` 语句的效果可以持续使用 shell.

PEP 还借此机会清理 PEP 236 中提到的其他未解决的问题,
无法停止 ``compile()`` 继承影响代码调用 ``compile()`` 的未来语句的效果.

本 PEP 建议通过向内置函数 "compile" 添加可选的第四个参数来解决第一个问题,
将信息添加到 ``__future __.py`` 中定义的 ``_Feature`` 实例,
并将机制添加到标准库模块中 "codeop" 和 "code" 使这种 shell 的构造变得容易.

第二个问题是通过简单地将 *another* 可选的参数添加到 ``compile()`` 来处理的,
如果非零, 则禁止继承未来语句的效果.


Specification
=============

我建议在内置的 "compile" 函数中添加第四个可选的 "flags" 参数.
如果省略此参数, 则 Python 2.1 的特性不会发生变化.

如果它存在, 则它应该是一个整数, 表示各种可能的编译时选项作为位域.
位域将具有与 Python 解释器的 C 部分已经使用的 ``CO_*`` 标志相同的值, 来引用未来的语句.

``compile()`` 如果它不能识别提供的标志中设置的任何位,
则会引发 ``ValueError`` 异常.

除非新的第五个可选参数是一个非零整数, 否则提供的标志将按位 "或" 加上将要设置的标志,
在这种情况下, 提供的标志将完全是使用的集合.

上面提到的标志目前没有暴露给 Python. 我建议在 ``__future__.py`` 中的 ``_Feature`` 对象中
添加 ``.compiler_flag`` 属性, 这些对象包含必要的位, 因此可以编写代码如下::

    import __future__
    def compile_generator(func_def):
        return compile(func_def, "<input>", "suite",
                    __future__.generators.compiler_flag)

最近的更改意味着可以使用这些相同的位来判断代码对象是否使用给定的功能进行编译;
例如 ::

   codeob.co_flags & __future__.generators.compiler_flag``

当且仅当代码对象 "codeob" 是在允许生成器的环境中编译时才为非零.

我还将在 ``__future__`` 模块中添加一个 ``.all_feature_flags`` 属性,
这样可以省力地枚举正在运行的解释器支持的所有 ``__future__`` 选项。.

我还建议在标准库模块 codeop 中添加一对类.

一个 - ``Compile`` - 将运行一个 ``__call__`` 方法, 它的行为很像 2.1 的内置 "compile",
不同之处在于它编译了一个 ``__future__`` 语句后, 它 "记住" 它并使用 ``__future__`` 选项编译所有后续代码.

它将通过使用上面提到的 ``__future__`` 模块的新功能来实现.

添加到 codeop 的另一个类的对象 - ``CommandCompiler`` - 将完成
现有 ``codeop.compile_command`` 函数的工作, 但是以 ``__future__`` 的方式工作.

最后, 我建议修改标准库模块代码中的 ``InteractiveInterpreter`` 类,
使用 ``CommandCompiler`` 来更接近地模拟默认 Python shell 的特性.


Backward Compatibility
======================

应该很少或没有; 编译的更改对现有代码没有任何影响, 也不会向 codeop 添加新的函数或类.
使用 ``code.InteractiveInterpreter`` 的现有代码可能会改变特性, 但只是为了更好,
因为 "真正的" Python shell 将被更好地模仿.


Forward Compatibility
=====================

在添加 ``__future__`` 功能时, 需要对 ``Lib/__future__.py`` 进行的改动将会更加复杂.
其他一切都应该起作用.


Issues
======

我希望上面的接口对于 Jython 实现起来不会太具破坏性.


Implementation
==============

一系列初步实施在 [1]_.

经过 Tim Peters 的轻微修改后, 他们现在已经接受检查.


References
==========

.. [1] http://sourceforge.net/tracker/?func=detail&atid=305470&aid=449043&group_id=5470

Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  

