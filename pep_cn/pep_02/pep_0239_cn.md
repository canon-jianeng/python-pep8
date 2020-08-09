
PEP: 239
Title: Adding a Rational Type to Python
Version: $Revision$
Last-Modified: $Date$
Author: Christopher A. Craig <python-pep@ccraig.org>, Moshe Zadka <moshez@zadka.site.co.il>
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 11-Mar-2001
Python-Version: 2.2
Post-History: 16-Mar-2001


Abstract
========

Python 没有数字类型, 其语义是无限精确的有理数. 该提议解释了这种类型的语义,
并建议内置函数和字面量来支持这种类型. 这个 PEP 建议有理数没有字面量的情况留给另一个 PEP [1]_.


BDFL Pronouncement
==================

该 PEP 被拒绝. 理论部分概述的需求在某种程度上通过接受 PEP 327 进行十进制算术来解决.
Guido 还指出, "有理数算法是 ABC 中的默认 '精确' 算术, 并没有像预期的那样成功".
请参阅 2005年6月17日的 python-dev 讨论 [2]_.

*后记:* 接受 PEP 3141, "数字类型层次", 在 'fractions' 模块中
添加了一个 'Rational' 数字抽象基类和一个具体实现.


Rationale
=========

虽然有时较慢且内存密集程度较高 (通常情况下, 无限制), 但有理数算术更接近于数学的数学理想,
并且往往具有对新手来说不那么令人惊讶的特性. 尽管已经编写了许多有理数的 Python 实现,
但这些实现都不存在于核心中, 或以任何方式记录. 这使得那些不太了解 Python 的人更难以访问它们.


RationalType
============

将添加一个名为 ``RationalType`` 的新数字类型. 它的一元运算符将做明显的事情.
二元运算符将强制整数和长整数归结为有理数, 以及对浮点数和复数的有理数.

将支持以下属性: ``.numerator`` 和 ``.denominator``. 语言定义将保证 ::

    r.denominator * r == r.numerator

分子的 GCD 和分母是1, 分母是正的.

方法 ``r.trim(max_denominator)`` 将最接近的有理数 ``s`` 返回到 ``r``,
这样 ``abs(s.denominator) <= max_denominator``.


The rational() Builtin
======================

这个函数将具有签名 ``rational(n, d=1)``. ``n`` 和 ``d`` 必须是整数, 长整数或有理数.
保证是这样的 ::

    rational(n, d) * d == n


Open Issues
===========

- 也许这种类型应该被称为 rat 而不是 rational. 有人提出我们有 "抽象" 的纯数学类型,
  名为复数, 实数, 有理数, 整数和 "具体" 表示类型名称如 float, rat, long, int.

- 是否应该允许带有整数值的有理数作为序列索引? 例如, 应该 ``[5/3 - 2/3]`` 相当于 ``s[1]``?

- 有理数应该允许 ``shift`` 和 ``mask`` 运算符吗? 对于具有整数值的有理数?

- Marcin 'Qrczak' Kowalczyk 在 c.l.py 上总结了支持和反对统一整数的理由.

  用有理数统一整数的论证:

  - 由于 ``2 == 2/1`` 和 ``str(2/1) == '2'``, 它减少了对象看起来相同但特性不同的惊讶.

  - 当我知道没有余数时, ``/`` 可以自由地用于整数除法 (如果我错了, 有余数, 以后可能会有一些异常).

  反对的论点:

  - 当我使用 ``/`` 的结果作为序列索引时, 它通常是一个错误,
    不应该通过使程序为某些数据工作来隐藏它, 因为它会破坏其他数据.

  - (这假设在统一之后 int 和 rational 将是不同的类型:) 类型应该很少依赖于值. 当知道变量的类型时, 更容易推理:
    我知道如何使用它. 我可以确定某个东西是一个 int, 并期望在这个地方使用的其他对象也将是 int.

  - (这假定它们的类型相同:) int 本身就是一个好类型, 不能与有理数混合使用.
     事实上, 什么是一个整数应该表达为它的类型声明.
	许多操作需要整数并且不接受有理数. 将它们视为不同类型是很自然的.


References
==========

.. [1] PEP 240, 将有理数字面量添加到 Python, Zadka,
       http://www.python.org/dev/peps/pep-0240/

.. [2] Raymond Hettinger, 建议拒绝 PEP 239 和 240 - 一个内置的有理数类型和有理数字面量
       https://mail.python.org/pipermail/python-dev/2005-June/054281.html

Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
