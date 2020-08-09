
PEP: 227
Title: Statically Nested Scopes
Version: $Revision$
Last-Modified: $Date$
Author: jeremy@alum.mit.edu (Jeremy Hylton)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 01-Nov-2000
Python-Version: 2.1
Post-History:

Abstract
========

此 PEP 描述了为 Python 2.2 添加静态嵌套作用域 (词法作用域),
以及作为 python 2.1 的源级别选项. 此外, Python 2.1 将发出有关构造的警告,
这些构造的含义可能会在启用此功能时发生更改.

旧语言定义 (2.0及之前) 定义了三个用于解析名称的名称空间
-- 本地名称空间, 全局名称空间和内置名称空间.
添加嵌套作用域允许在封闭函数中解析未绑定的本地名称命名空间.

这种变化最明显的结果是 lambdas (和其他嵌套函数) 可以引用周围命名空间中定义的变量.
目前, lambdas 必须经常使用默认参数在 lambda 的命名空间中显式创建绑定.

Introduction
============

此提议更改了在 Python 函数中解析自由变量的规则. 新的名称解析语义将在 Python 2.2 中生效.
通过将 "from __future__ import nested_scopes" 添加到模块的顶部,
这些语义也将在 Python 2.1 中提供.  (See PEP 236.)

Python 2.0 定义确切地指定了三个名称空间来检查每个名称 -- 本地名称空间, 全局名称空间和内置名称空间.
根据此定义, 如果函数 A 在函数 B 中定义, 则 B 中绑定的名称在 A 中不可见.
提议更改规则, 以便在 B 中绑定的名称在 A 中可见 (除非 A 包含名称绑定在 B) 中隐藏绑定.

本规范引入了类似 Algol 语言中常见的词法作用域规则.
词法范围与现有的一流功能支持相结合, 让人想起 Scheme.

更改的范围规则解决了两个问题 -- lambda 表达式的有限效用 (以及一般的嵌套函数)，
以及熟悉支持嵌套词法范围的其他语言的新用户的频繁混淆, 例如: 除模块级别外无法定义递归函数.

lambda 表达式生成一个未命名的函数, 用于计算单个表达式.
它通常用于回调函数. 在下面的示例中 (使用 Python 2.0 规则编写),
lambda 主体中使用的任何名称都必须作为默认参数显式传递给 lambda.

::

    from Tkinter import *
    root = Tk()
    Button(root, text="Click here",
           command=lambda root=root: root.test.configure(text="..."))

这种方法很麻烦, 特别是当 lambda 主体中使用了几个名称时. 长列表的默认参数掩盖了代码的目的.
粗略地说, 所提出的解决方案自动实现默认参数方法. 可以省略 "root=root" 参数.

新的名称解析语义将导致某些程序的行为与在 Python 2.0 下的行为不同. 在某些情况下, 程序将无法编译.
在其他情况下, 先前使用全局命名空间解析的名称将使用封闭函数的本地命名空间进行解析.
在 Python 2.1 中, 将针对行为不同的所有语句发出警告.

Specification
=============

在传统的 Algol 中, Python 是一种具有块结构的静态范围语言.
代码块或区域 (例如, 模块, 类定义或函数体) 是程序的基本单元.

名称涉及对象. 名称由名称绑定操作引入.
程序文本中每次出现的名称都是指在包含使用的最里面的功能块中建立的名称的绑定.

名称绑定操作是参数声明, 赋值, 类和函数定义, import 语句, for 语句和 except 子句.
每个名称绑定发生在由类或函数定义或模块级别 (顶级代码块) 定义的块中.

如果名称绑定在代码块中的任何位置, 则块中名称的所有使用都将被视为对当前块的引用.
(注意: 在绑定之前在块中使用名称时, 这可能会导致错误.)

如果全局语句发生在块中, 则语句中指定的名称的所有使用都将引用该名称在顶级名称空间中的绑定.
通过搜索全局命名空间 (即包含代码块的模块的命名空间) 和内置命名空间 (即``__builtin__``模块的命名空间),
在顶级命名空间中解析名称. 首先搜索全局命名空间. 如果在那里找不到名称, 则搜索内置命名空间.
全局语句必须在名称的所有使用之前.

如果在代码块中使用了名称, 但它未绑定在那里并且未声明为全局,
则将该用法视为对最近的封闭函数区域的引用.
(注意: 如果某个区域包含在类定义中, 则类块中出现的名称绑定对于封闭的函数是不可见的.)

类定义是一个可执行语句, 可以包含名称的用法和定义.
这些引用遵循名称解析的常规规则. 类定义的名称空间成为类的属性字典.

以下操作是名称绑定操作. 如果它们出现在一个块中, 它们会在当前块中引入新的本地名称, 除非还有一个全局声明.

::

    函数定义: def name ...
    参数声明: def f(...name...), lambda ...name...
    类定义: class name ...
    赋值语句: name = ...
    导入语句: import name, import module as name,
        	 from module import name
    隐式赋值: names are bound by for statements and except clauses

在某些情况下, 当与包含自由变量的嵌套作用域一起使用时, Python 语句是非法的.

如果在封闭范围中引用变量, 则删除名称是错误的. 编译器将为 'del name' 引发 ``SyntaxError``.

如果在函数中使用了通配符形式的 import (``import *``),
并且该函数包含一个带有自由变量的嵌套块, 编译器将引发一个 ``SyntaxError``.

如果在函数中使用了 exec 并且函数包含带有自由变量的嵌套块,
则编译器将引发 ``SyntaxError``, 除非 exec 显式指定了 exec 的本地名称空间.
(换句话说, "exec obj" 将是非法的, 但 "exec obj in ns" 将是合法的.)

如果函数作用域中绑定的名称也是模块全局名称或标准内置名称的名称,
并且该函数包含引用该名称的嵌套函数作用域, 则编译器将发出警告.
名称解析规则将导致 Python 2.0 下的绑定不同于 Python 2.2 下的绑定.
警告表明程序可能无法与所有版本的 Python 一起正确运行.

Discussion
==========

指定的规则允许在使用该函数定义的任何嵌套函数中引用函数中定义的名称.
名称解析规则是静态范围语言的典型规则, 有三个主要例外:

- 无法访问类范围中的名称.
- 全局声明使正常规则缩短.
- 变量未声明.

无法访问类范围中的名称. 名称在最里面的封闭函数范围内解析.
如果类定义出现在嵌套作用域链中, 则解析过程将跳过类定义.
此规则可防止类属性与本地变量访问之间的奇怪交互.
如果在类定义中发生名称绑定操作, 则会在生成的类对象上创建属性.
要在方法中或在嵌套在方法中的函数中访问此变量, 必须使用属性引用,
可以通过 self 或通过类名.

另一种方法是允许类范围中的名称绑定与函数范围中的名称绑定完全相同.
此规则允许通过属性引用或简单名称引用类属性. 此选项被排除,
因为它将与所有其他形式的类和实例属性访问不一致, 后者始终使用属性引用.
使用简单名称的代码应该是模糊的.

全局声明使正常规则缩短. 根据该提案, 全局语句与 Python 2.0 具有完全相同的效果.
它也值得注意, 因为它允许在一个块中执行的名称绑定操作来更改另一个块 (模块) 中的绑定.

变量未声明. 如果名称绑定操作发生在函数中的任何位置, 则该名称将被视为函数的本地名称,
并且所有引用都引用本地绑定. 如果在绑定名称之前发生引用, 则引发 NameError.
唯一的声明是全局语句, 它允许使用可变全局变量编写程序.
因此, 无法重新绑定封闭范围中定义的名称. 赋值操作只能绑定当前范围或全局范围中的名称.
缺乏声明和无法在封闭范围内重新标记名称对于词汇范围的语言来说是不寻常的;
通常有一种机制来创建名称绑定 (例如, lambda 和 let in Scheme) 和一种更改绑定的机制 (在 Scheme 中设置!).

XXX Alex Martelli 建议与 Java 进行比较, Java 不允许名称绑定隐藏早期绑定.

Examples
========

包含一些示例来说明规则的工作方式.

XXX 解释这些例子

::

    >>> def make_adder(base):
    ...     def adder(x):
    ...         return base + x
    ...     return adder
    >>> add5 = make_adder(5)
    >>> add5(6)
    11

    >>> def make_fact():
    ...     def fact(n):
    ...         if n == 1:
    ...             return 1L
    ...         else:
    ...             return n * fact(n - 1)
    ...     return fact
    >>> fact = make_fact()
    >>> fact(7)
    5040L

    >>> def make_wrapper(obj):
    ...     class Wrapper:
    ...         def __getattr__(self, attr):
    ...             if attr[0] != '_':
    ...                 return getattr(obj, attr)
    ...             else:
    ...                 raise AttributeError, attr
    ...     return Wrapper()
    >>> class Test:
    ...     public = 2
    ...     _private = 3
    >>> w = make_wrapper(Test())
    >>> w.public
    2
    >>> w._private
    Traceback (most recent call last):
      File "<stdin>", line 1, in ?
    AttributeError: _private

Tim Peters 的一个例子展示了在没有声明的情况下嵌套作用域的潜在缺陷::

    i = 6
    def f(x):
        def g():
            print i
        # ...
        # skip to the next page
        # ...
        for i in x:  # ah, i *is* local to f, so this is what g sees
            pass
        g()

对 ``g()`` 的调用将引用 for 循环中 ``f()`` 绑定的变量.
如果在执行循环之前调用 ``g()``, 则会引发 NameError.

XXX 需要一些反例

Backwards compatibility
=======================

嵌套作用域导致两种兼容性问题. 在一种情况下, 由于嵌套作用域,
在早期版本中表现为单向的代码的行为会有所不同. 在其他情况下,
某些构造与嵌套作用域交互不良, 并在编译时触发 SyntaxErrors.

Skip Montanaro 的以下示例说明了第一种问题::

    x = 1
    def f1():
        x = 2
        def inner():
            print x
        inner()

在 Python 2.0 规则下, ``inner()`` 中的 print 语句引用全局变量 x,
如果调用 ``f1()`` 则打印1. 在新规则下, 它引用了 ``f1()`` 命名空间,
最近的封闭范围带有绑定.

仅当全局变量和局部变量共享同一名称并且嵌套函数使用该名称引用全局变量时, 才会出现此问题.
这是糟糕的编程习惯, 因为读者很容易混淆这两个不同的变量.
在嵌套作用域的实现过程中, 在 Python 标准库中发现了此问题的一个示例.

为了解决这个不太可能经常发生的问题, Python 2.1 编译器 (当嵌套作用域未启用时) 会发出警告.

另一个兼容性问题是由函数体中使用 ``import *`` 和 'exec' 引起的,
当该函数包含嵌套作用域并且包含的作用域具有自由变量时. 例如::

    y = 1
    def f():
        exec "y = 'gotcha'" # or from module import *
        def g():
            return y
        ...

在编译时, 编译器无法判断在本地命名空间上运行的 exec 或 ``import *`` 是否会引入影响全局 y 的名称绑定.
因此, 无法判断 ``g()`` 中对 y 的引用是否应该引用 ``f()`` 中的全局或本地名称.

在讨论 python-list 时, 人们争论两种可能的解释.
一方面, 有人认为 ``g()`` 中的引用应该绑定到本地 y (如果存在的话).
这种解释的一个问题是对人类来说是不可能的. 读者的代码, 通过本地检查确定 y 的绑定.
它似乎可能会引入微妙的错误. 另一种解释是将 exec 和 import * 视为不影响静态范围的动态特性.
根据这种解释, exec 和 import * 将引入本地名称, 但这些名称永远不会对嵌套作用域可见.
在上面的具体示例中, 代码的行为与早期版本的 Python 中的行为完全相同.

由于每个解释都存在问题且确切含义不明确, 因此编译器会引发异常.
当未启用嵌套作用域时, Python 2.1 编译器会发出警告.

对三个 Python 项目 (标准库, Zope 和 PyXPCOM 的 beta 版) 的简要回顾
在大约 200,000 行代码中发现了四个向后兼容性问题.
在标准库中有一个案例 #1 (细微行为改变) 和两个 ``import *`` 问题的例子.

(在 Python 2.1a2 中实现的 ``import *`` 和 exec 限制的解释更具限制性,
基于参考手册中从未强制执行的语言. 这些限制在发布后放宽了.)

Compatibility of C API
======================

该实现导致几个 Python C API 函数发生变化, 包括 ``PyCode_New()``.
因此, 可能需要更新 C 扩展来使用 Python 2.1 正常工作.

locals() / vars()
=================

这些函数返回包含当前范围的局部变量的字典. 对字典的修改不会影响变量的值.
在当前规则下, 使用 ``locals()`` 和 ``globals()`` 允许程序访问所有名称被解析的名称空间.

不会为嵌套作用域提供类似的功能.
根据此提议, 将无法获得对所有可见范围的字典式访问.

Warnings and Errors
===================

编译器将在 Python 2.1 中发出警告, 来帮助识别在未来版本的 Python 下无法编译或正确运行的程序.
在 Python 2.2 或 Python 2.1 下如果使用 ``nested_scopes`` future 语句
(本节统称为 "future semantics"), 编译器会在某些情况下发出 SyntaxErrors.

当包含具有自由变量的嵌套函数的函数时, 警告通常适用.
例如, 如果函数 F 包含函数 G, 而 G 使用内置的 ``len()``,
那么 F 是一个函数, 它包含一个带有自由变量 (len) 的嵌套函数 (G).
标签 "free-in-nested" 将用于描述这些功能.

import * used in function scope
-------------------------------

语言参考指定 ``import *`` 可能只出现在模块范围内.
(Sec.6.11) C Python 的实现在函数范围支持 ``import *``.

如果在一个自由嵌套函数的主体中使用 ``import *``, 编译器将发出警告.
在未来的语义中, 编译器将引发一个 ``SyntaxError``.

bare exec in function scope
---------------------------

exec 语句允许在关键字 "in" 后面的两个可选表达式指定用于 locals 和 globals 的名称空间.
省略这两个命名空间的 exec 语句是一个简单的 exec.

如果在免费嵌套函数的主体中使用了空的 exec, 编译器将发出警告.
在未来的语义中, 编译器将引发一个 ``SyntaxError``.

local shadows global
--------------------

如果嵌套函数具有对局部变量的绑定 (1) 在嵌套函数中使用,
(2) 与全局变量相同, 则编译器将发出警告.

Rebinding names in enclosing scopes
-----------------------------------

有一些技术问题使得难以支持在封闭范围内重新命名名称, 但是当前提案中不允许使用的主要原因是 Guido 反对它.
他的动机: 很难支持, 因为它需要一个新机制, 允许程序员指定一个块中的赋值应该重新绑定一个封闭块中的名称;
可能是关键字或特殊语法 (x := 3) 会使这成为可能. 鉴于这会鼓励使用局部变量来保存更好地存储在类实例中的状态,
所以不值得添加新语法来实现这一点 (在 Guido 看来).

建议的规则允许程序员实现重新绑定的效果, 尽管是笨拙的.
封闭函数将有效重新绑定的名称绑定到容器对象.
代替分配, 程序使用容器的修改来实现期望的效果::

    def bank_account(initial_balance):
        balance = [initial_balance]
        def deposit(amount):
            balance[0] = balance[0] + amount
            return balance
        def withdraw(amount):
            balance[0] = balance[0] - amount
            return balance
        return deposit, withdraw

支持嵌套作用域中的重新绑定将使此代码更清晰. 定义 ``deposit()`` 和 ``withdraw()`` 方法
以及将 balance 作为实例变量的类仍然更清晰. 由于类似乎用更直接的方式实现相同的效果, 因此它们是首选的.

Implementation
==============

XXX Jeremy, 情况仍然如此?

C Python 的实现使用平面闭包 [1]_. 如果函数体或任何包含的函数具有自由变量,
则执行的每个 def 或 lambda 表达式都将创建一个闭包.
使用平面闭合, 闭合的创建有点昂贵, 但查找很便宜.

该实现在代码对象中添加了几个新的操作码和两种新的名称.
变量可以是单元变量, 也可以是特定代码对象的自由变量.
包含范围引用单元格变量; 因此, 定义它的函数必须在每次调用时为其分配单独的存储.
通过函数的闭包引用自由变量.

免费关闭的选择基于三个因素. 首先, 假定嵌套函数不经常使用,
深度嵌套 (个别嵌套级别) 仍然不那么频繁. 其次, 在嵌套范围内查找名称应该很快.
第三, 嵌套作用域的使用, 特别是在返回访问封闭作用域的函数的情况下,
不应该阻止垃圾回收器回收未引用的对象.

XXX Much more to say here

References
==========

.. [1] Luca Cardelli.  Compiling a functional language.  In Proc. of
       the 1984 ACM Conference on Lisp and Functional Programming,
       pp. 208-217, Aug. 1984
       http://citeseer.ist.psu.edu/cardelli84compiling.html

Copyright
=========

XXX

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
