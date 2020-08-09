
PEP: 228
Title: Reworking Python's Numeric Model
Version: $Revision$
Last-Modified: $Date$
Author: moshez@zadka.site.co.il (Moshe Zadka), guido@python.org (Guido van Rossum)
Status: Withdrawn
Type: Standards Track
Content-Type: text/x-rst
Created: 4-Nov-2000
Post-History:


Withdrawal
==========

该 PEP 已被撤回, 有利于 PEP 3141.


Abstract
========

今天, Python 的数值模型类似于 C 数值模型: 有几种不相关的数值类型, 当请求数值类型之间的操作时,
就会发生强制. 虽然数值模型的 C 基本原理是它与硬件级别的结果非常相似, 但该基本原理并不适用于 Python.
因此, 虽然 C 程序员可以接受 ``2/3 == 0``, 但许多 Python 程序员都会感到惊讶.

注意: 根据新闻组最近的讨论, 需要扩展本 PEP (和细节) 的动机.


Rationale
=========

在可用性研究中, Python 最不可用的方面之一是整数除法返回除法的最低点. 这使得很难正确编程,
需要通过代码在各个部分中转换为 ``float()``. Python 的数值模型源于 C,
而可能更容易使用的模型可以基于对数字的数学理解.


Other Numerical Models
======================

Perl 的数值模型是有一种类型的数字 - 浮点数. 虽然它一致且表面上并不令人惊讶, 但它往往会有微妙的陷阱.
其中之一是打印数字非常棘手, 需要正确的舍入. 在 Perl 中, 还有一种模式, 其中所有数字都是整数.
这种模式也存在一些问题, 这些问题是由于甚至没有一种近似的方法来划分数字和得到有意义的答案.


Suggested Interface For Python's Numerical Model
================================================

虽然强制规则将保留为附加类型和类, 但内置类型系统将只有一个 Python 类型 - 一个数字.
有几件事可以被认为是 "数字方法":

1. ``isnatural()``
2. ``isintegral()``
3. ``isrational()``
4. ``isreal()``
5. ``iscomplex()``
6. ``isexact()``

显然, 对于1到5的问题, 回答真实的数字也将回答任何后续问题.
如果 ``isexact()`` 不是真的, 那么任何答案都可能是错误的.
(但并非可怕的错误: 它接近事实.)

现在, 模型约定为字段操作符提供两件事
(``+``, ``-``, ``/``, ``*``):

- 如果两个操作数都满足 ``isexact()``, 则结果满足 ``isexact()``.

- 所有字段规则都是正确的, 除了对于非 ``isexact()`` 数字, 它们可能只是近似为真.

这两个规则的一个结果是所有精确的计算都是以 (复杂的) 有理数完成的:
因为字段法则必须适用 ::

    (a/b)*b == a

必须坚持.

有内置函数, ``inexact()`` 它接受一个数字并返回一个不精确的数字, 这是一个很好的近似值.
不精确的数字必须至少与使用 IEEE-754 一样准确.

即使给出不精确的数字, 一些经典的 Python 函数也会返回确切的数字: 例如, ``int()``.


Coercion
========

数字类型没有定义 ``nb_coerce`` 任何数字操作槽, 当接收到其他东西, 然后 ``PyNumber`` 拒绝实现它.


Inexact Operations
==================

``math`` 模块中的函数将被允许返回精确值的不精确结果. 但是, 它们永远不会返回非实数.
``cmath`` 模块中的函数也允许返回精确参数的不精确结果, 并且还允许为真实参数返回复杂结果.


Numerical Python Issues
=======================

使用 Numerical Python 的人这样做是为了进行高性能矢量操作.
因此, NumPy 应该保留其基于硬件的数字模型.


Unresolved Issues
=================

哪个数字字面量将是准确的, 哪些不准确?

我们如何处理 IEEE 754 操作? ( 可能, isnan/isinf 应该是方法 )

在 64 位计算机上, 当比较涉及转换为浮点数时, 可能会破坏整数和浮点数之间的比较.
同样可以用于 longs 和 floats 之间的比较. 这可以通过避免转换为浮点来处理. 
 (由于 Andrew Koenig.)


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
