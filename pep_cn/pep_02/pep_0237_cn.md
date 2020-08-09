
PEP: 237
Title: Unifying Long Integers and Integers
Version: $Revision$
Last-Modified: $Date$
Author: Moshe Zadka, Guido van Rossum
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 11-Mar-2001
Python-Version: 2.2
Post-History: 16-Mar-2001, 14-Aug-2001, 23-Aug-2001


Abstract
========

Python 目前区分两种整数 (整数): 常规或短整数, 受 C long (通常为32或64位) 的大小限制,
以及长整数, 仅受可用内存的限制. 当短整数的操作产生不适合 C long 的结果时, 它们会引发错误.
还有其他一些区别. 这个 PEP 建议消除语义上的大多数差异, 从 Python 用户的角度统一这两种类型.


Rationale
=========

许多程序发现需要在事后处理更大的数字, 并且稍后更改算法是麻烦的.
在正常情况下, 当使用长整数执行所有算术时, 无论是否需要它, 都会妨碍性能.

使机器字大小暴露于该语言会妨碍可移植性. 例如, Python 源文件和 .pyc 不能在32位和64位机器之间移植.

当 Python 用户与大多数应用程序无关时, 通常也希望隐藏不必要的细节. 一个例子是内存分配,
它在 C 中是显式的, 但在 Python 中是自动的, 为我们提供了在字符串, 列表等上无限大小的便利.
将这种便利扩展到数字是有意义的。.

它会给新的 Python 程序员 (无论他们是否是一般的编程新手), 在他们开始使用该语言之前, 应该少学一点.


Implementation
==============

最初, 提出了两个替代实现 (每个作者一个):

1. 用于 C long 的 ``PyInt`` 类型的槽将变为 a ::

       union {
           long i;
           struct {
               unsigned long length;
               digit digits[1];
           } bignum;
       };

   只有 ``long`` 的 ``n-1`` 低位才有意义; 最高位始终设置. 这区分了 ``union``.
   在确定要使用哪种类型的操作之前, 所有 ``PyInt`` 函数都将检查此位.

2. 现有的 short 和 long int 类型仍然存在, 但是当结果不能表示为 short int 时,
    操作返回 long int 而不是引发 ``OverflowError``. 可以引入一种新类型 ``integer``,
	它是一种抽象基类型, 其中 ``int`` 和 ``long`` 实现类型都是子类. 这很有用,
	因此程序可以通过单个测试来检查整数 ::

       if isinstance(i, integer): ...

经过一番考虑后, 选择了第二个实施计划, 因为它更容易实现, 在 C API 级别向后兼容,
此外可以部分实施为过渡措施.


Incompatibilities
=================

对于短整数和长整数, 以下操作 (通常是巧妙地) 具有不同的语义, 并且必须以某种方式改变一个或另一个.
这是一份详尽的清单. 如果您知道结果不同的任何其他操作, 具体取决于是否通过了具有相同值的 short 或 long int,
请写第二个作者.

- 目前, 如果结果不能表示为 short int, 那么除了 ``<<`` 之外的所有算术运算符都会引发 ``OverflowError``.
   这将被更改为返回一个 long int. 以下运算符当前可以引发 ``OverflowError``: ``x+y``, ``xy``, ``x*y``, ``x**y``,
   ``divmod(x, y) ``, ``x/y``, ``x％y`` 和 ``-x``. (当涉及到值 ``-sys.maxint-1`` 时, 最后四个只能溢出.)

- 目前, ``x << n`` 可以丢失用于短整数的位. 这将被更改为返回包含所有移出位的 long int,
   如果返回 short int 将丢失位 (其中更改符号被视为丢失位的特殊情况).

- 目前, 短整数的十六进制和八进制文字可以指定负值; 例如32位机器上的 ``0xffffffff == -1``.
   这将改为等于 ``0xffffffffL`` (``2**32-1``).

- 目前, ``%u``, ``％x``, ``％X`` 和 ``％o`` 字符串格式化操作符和 ``hex()`` 和 ``oct()`` 内置函数对负数的行为有所不同:
   负短整数被格式化为无符号 C long, 而负长整数则用负号格式化. 这将被改为在所有情况下使用 long int 语义
   (但是没有尾随 *L*, 它目前区分 ``hex()`` 和 ``oct()`` 的输出为 long int.
   请注意, 这意味着 ``％u`` 成为 ``％d`` 的别名. 它最终将被删除.

- 目前, long int 的 ``repr()`` 返回以 *L* 结尾的字符串, 而 short int 的 ``repr()`` 则不返回.
   *L* 将被删除; 但不是在 Python 3.0 之前.

- 目前, 具有长操作数的操作永远不会返回短整数. 这个可能会改变, 因为它允许一些优化.
   (此区域尚未进行任何更改, 也没有计划.)

- 表达式 ``type(x).__name__`` 取决于 *x* 是 short 还是 long int. 由于选择了实施方案2,
  这种差异将保持不变. (在 Python 3.0 中, 我们可能部署一个技巧来隐藏差异, 因为它令人烦恼,
  来揭示用户代码的差异, 更使两种类型之间的差异不太明显.)

- ``marshal`` 模块以及 ``pickle`` 和 ``cPickle`` 模块处理的是长和短的整数.
  这种差异将保持不变 (至少在 Python 3.0 之前).

- 具有较小值 (通常介于 -1 和 99 之间) 的短整数是 *interned* - 只要结果具有这样的值,
  就返回具有相同值的现有 short int. 对于具有相同值的长整数, 不会这样做. 这种差异将保持不变. 
  (由于无法保证这种实习, 这是否存在语义差异是有争议的 - 但是可能存在使用 ``is`` 进行短整数比较的代码,
  并且由于这种实习而恰好起作用. 如果用于长整数, 这样的代码可能会失败.)


Literals
========

整数文字末尾的尾随 *L* 将停止具有任何含义, 并最终将变为非法. 编译器将仅根据值选择适当的类型.
(在 Python 3.0 之前, 它会强制文字很长; 但是没有尾随 *L* 的文字也可能很长, 如果它们不能表示为短整数.)


Built-in Functions
==================

函数 ``int()`` 将根据参数值返回 short 或 long int. 在 Python 3.0 中,
函数 ``long()`` 将调用函数 ``int()``; 在此之前, 它将继续强制结果为 long int,
但其他方式与 ``int()`` 的工作方式相同. 内置名称 ``long`` 将保留在语言中来表示长实现类型
(除非在 Python 3.0 中完全废止), 但仍然建议使用 ``int()`` 函数, 因为它将需要时自动返回很长时间.


C API
=====

C API 保持不变; C 代码仍然需要注意 short 和 long int 之间的区别.
(Python 3.0 C API 可能完全不兼容.)

``PyArg_Parse*()`` API 已经接受了长整数, 只要它们在 C int 或 longs 可表示的范围内,
因此使用 C int 或 long 参数的函数不必担心处理 Python longs.


Transition
==========

过渡有三个主要阶段:

1. 当前引发 ``OverflowError`` 的短整数运算返回一个 long int 值. 这是这一阶段的唯一变化.
    文字仍将区分短类型和长类型的整数. 上面列出的其他语义差异 (包括 ``<<`` 的特性) 将保留.
	因为此阶段仅更改当前引发 ``OverflowError`` 的情况, 所以假设这不会破坏现有代码.
	(依赖于此异常的代码必须过于复杂而无法关注它.) 对于那些担心极端向后兼容性的人,
	命令行选项 (或对警告模块的调用) 将允许警告或错误此时发布, 但默认情况下已关闭.

2. 解决了剩余的语义差异. 在所有情况下, long int 语义都将占上风. 由于这将引入向后兼容性,
    这将破坏一些旧代码, 此阶段可能需要将来的声明或警告, 以及延长的过渡阶段.
	尾随 *L* 将继续用作输入的 long 和 ``repr()``.

   A. 关于将在阶段 2B 中更改其数字结果的操作启用警告, 特别是 ``hex()`` 和 ``oct()``, ``％u``, ``％x``, ``％X``
       和 ``％o``, ``hex`` 和 ``oct`` 字面量在 (包括) 范围 ``[sys.maxint+1, sys.maxint*2+1]``, 以及左移失去了位.

   B. 实现了这些操作的新语义. 给出与以前不同结果的操作将不发出警告.

3. 尾随 *L* 从 ``repr()`` 中删除, 并在输入时变为非法. (如果可能的话, ``long`` 类型完全消失.)
    尾随的 *L* 也从 ``hex()`` 和 ``oct()`` 中删除.

第1阶段将在 Python 2.2 中实现.

第2阶段将逐步实施, Python 2.3 中的 2A 和 Python 2.4 中的 2B.

第3阶段将在 Python 3.0 中实现 (Python 2.4 发布后至少两年).


OverflowWarning
===============

以下是指导在当前引发 ``OverflowError`` 的情况下生成警告的规则. 这适用于过渡阶段1.历史记录:
尽管第1阶段已在 Python 2.2 中完成, 第2阶段在 Python 2.3 中完成,
但没有人注意到仍然在 Python 2.3 中生成了 OverflowWarning. 它最终在 Python 2.4 中被禁用.
Python 内置的 ``OverflowWarning`` 和相应的 C API ``PyExc_OverflowWarning`` 不再在 Python 2.4 中生成或使用,
但在 Python 2.5 之前将保留 (不太可能) 用户代码的情况.

- 引入了一个新的警告类别, ``OverflowWarning``. 这是一个内置名称.

- 如果 int 结果溢出, 则发出 ``OverflowWarning`` 警告, 并带有指示操作的消息参数, 例如: "整数加法".
  这可能会也可能不会导致在 ``sys.stderr`` 上显示警告消息, 或者可能导致引发异常, 所有这些都在 ``-W`` 命令行和警告模块的控制之下.

- 默认情况下会忽略 ``OverflowWarning`` 警告.

- 可以通过 ``-W`` 命令行选项或通过 ``warnings.filterwarnings()`` 调用来控制 ``OverflowWarning`` 警告,
   就像所有警告一样.
   例如 ::

      python -Wdefault::OverflowWarning

  导致 ``OverflowWarning`` 在第一次出现在特定源码行时显示, 并且 ::

      python -Werror::OverflowWarning

  导致 ``OverflowWarning`` 在发生时变成异常. 以下代码启用程序内部的警告 ::

      import warnings
      warnings.filterwarnings("default", "", OverflowWarning)

  查看 ``-W`` 选项的 python ``man`` 页面和 ``filterwarnings()`` 的 ``warnings`` 模块文档.

- 如果 ``OverflowWarning`` 警告变成错误, 则替换 ``OverflowError``.
  这是向后兼容性所必需的.

- 除非将警告转换为异常, 否则在将参数转换为 long int 后重新计算操作的结果
  (例如, ``x + y``).


Example
=======

如果将 long int 传递给 C 函数或带有整数的内置操作, 只要该值适合, 它就会被视为短整数
(由于 ``PyArg_ParseTuple()`` 的实现方式). 如果 long 值不合适, 它仍然会引发 ``OverflowError``.
For example::

    def fact(n):
        if n <= 1:
        return 1
    return n*fact(n-1)

    A = "ABCDEFGHIJKLMNOPQ"
    n = input("Gimme an int: ")
    print A[fact(n)%17]

对于 ``n >= 13``, 这当前引发 ``OverflowError`` (除非用户输入尾随 *L* 作为其输入的一部分),
即使计算的索引总是在 ``range(17)``. 使用新方法, 此代码将执行正确的操作:
索引将计算为 long int, 但其值将在范围内.


Resolved Issues
===============

以前公开的这些问题已经解决.

- ``hex()`` 和 ``oct()`` 应用于 longs 将继续产生尾随 *L* 直到 Python 3000.
  上面的原始文本不清楚这个, 但因为它没有发生在 Python 2.4 最好不要管它.
  BDFL 在这里发表声明:

  https://mail.python.org/pipermail/python-dev/2006-June/065918.html

- 怎么做 ``sys.maxint``? 保留它, 因为只要 short 和 long int 之间的区别仍然相关
  (例如检查值的类型时), 它仍然相关.

- 我们应该完全删除 ``％u`` 吗? 删除它.

- 我们应该警告 ``<<``, 不截断整数? 是的.

- 溢出警告应该是可移植的最大大小吗? 没有.


Implementation
==============

完成了 Python 2.x 系列的实现工作; 第1阶段是使用 Python 2.2 发布的,
第2A阶段是使用 Python 2.3 发布的, 第2B阶段将使用 Python 2.4 发布
(已经在 CVS 中).


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  

