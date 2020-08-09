
PEP: 215
Title: String Interpolation
Version: $Revision$
Last-Modified: $Date$
Author: ping@zesty.ca (Ka-Ping Yee)
Status: Superseded
Type: Standards Track
Content-Type: text/x-rst
Created: 24-Jul-2000
Python-Version: 2.1
Post-History:
Superseded-By: 292


Abstract
========

本文档提出了 Python 的字符串插值功能, 以便更轻松地进行字符串格式化, 建议的语法更改是引入一个'$'前缀,
触发字符串中'$'字符的特殊解释, 其方式与 Unix shell, awk, Perl 或 Tcl 中的变量插值相似.


Copyright
=========

本文档属于公共域名.


Specification
=============

字符串前面可以加上前缀为单引号或双引号 (或三元组) 的'$'前缀, 并且在任何其他字符串前缀 ('r'或'u')之前.
在对其内容中的反斜杠转义进行正常解释之后, 处理这样的字符串以进行插值. 每次按下字符串时,
就会在将字符串压入值堆栈之前进行处理. 简而言之, Python 的行为就像'$'是应用于字符串的一元运算符一样.
执行的操作如下:

字符串从头到尾扫描'$'字符 (8位字符串中的``\x24``或 Unicode 字符串中的``\u0024``).
如果没有'$'字符, 则返回不变的字符串.

在字符串中找到的任何'$', 后跟下面描述的两种表达式之一, 将替换为当前名称空间中计算的表达式的值.
如果包含的字符串是8位字符串, 则使用``str()``转换该值; 如果是 Unicode 字符串, 则使用``unicode()``转换.

1.  Python 标识符可选地后跟任意数量的 trailer, 其中 trailer 包含:
    - 一个点和一个标识符,
    - 用方括号括起来的表达式, 或
    - 括在括号中的参数列表
    (这正是 Python 语法中用 "``NAME trailer*``" 表达的模式, 使用``Grammar/Grammar``中的定义.)

2.  用大括号括起来的任何完整的 Python 表达式.

两个美元符号 ("$$") 被替换为单个 "$".


Examples
========

以下是展示此功能的预期行为的交互式会话的示例. ::

   >>> a, b = 5, 6
   >>> print $'a = $a, b = $b'
   a = 5, b = 6
   >>> $u'uni${a}ode'
   u'uni5ode'
   >>> print $'\$a'
   5
   >>> print $r'\$a'
   \5
   >>> print $'$$$a.$b'
   $5.6
   >>> print $'a + b = ${a + b}'
   a + b = 11
   >>> import sys
   >>> print $'References to $a: $sys.getrefcount(a)'
   References to 5: 15
   >>> print $"sys = $sys, sys = $sys.modules['sys']"
   sys = <module 'sys' (built-in)>, sys = <module 'sys' (built-in)>
   >>> print $'BDFL = $sys.copyright.split()[4].upper()'
   BDFL = GUIDO


Discussion
==========

为了熟悉, '$' 被选为字符串中的插值字符, 因为它已经在许多其他语言和上下文中用于此目的.

然后选择'$'作为前缀是很自然的, 因为它是插值字符的助记符.

Trailers 允许这种插值机制比大多数其他语言中的插值功能更强大,
而要插值的表达式仍然清晰可见且没有花括号.

'$'的工作方式类似于运算符, 可以作为运算符实现, 但这会阻止编译时优化并出现安全问题.
因此, 它只允许作为字符串前缀.


Security Issues
===============

"$" 具有评估的能力, 但只能评估文字. 如此处所述 (字符串前缀而不是运算符),
它不会引入新的安全性问题, 因为要评估的表达式必须在字面上存在于代码中.


Implementation
==============

[1]_ 处的``Itpl``模块提供了此功能的原型. 它使用``tokenize``模块来查找要插值的表达式的结尾,
然后在每次需要值时对表达式调用``eval()``. 在原型中, 每次计算表达式时都会对表达式进行解析和编译.

作为优化, 插值字符串可以直接编译成相应的字节码; 那是 ::

   $'a = $a, b = $b'

可以编译, 好像它是表达式 ::

   ('a = ' + str(a) + ', b = ' + str(b))

所以它只需要编译一次.


References
==========

.. [1] http://www.lfw.org/python/Itpl.py


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
