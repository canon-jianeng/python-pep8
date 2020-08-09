
PEP: 240
Title: Adding a Rational Literal to Python
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

一个不同的 PEP [1]_ 建议在 Python 中添加一个内置的有理数类型.
这个 PEP 建议在 Python 中将 ddd.ddd 浮点数字面量更改为有理数,
并修改非整数除法来返回它.


BDFL Pronouncement
==================

该 PEP 被拒绝. 理论部分概述的需求在某种程度上通过接受 PEP 327 进行十进制算术来解决.
Guido 还指出, "有理数算法是 ABC 中的默认 '精确' 算术, 并没有像预期的那样成功".
请参阅 2005年6月17日的 python-dev 讨论 [2]_.


Rationale
=========

有理数用于精确和不令人惊讶的算术. 他们给出了人们在各种数学课上教授的正确结果.
使具有更多可预测语义的 "明显" 非整数类型将使新程序员感到惊讶, 而不是使用浮点数.
由于 c.l.py 上和 tutor@python.org 上发布了很多帖子, 人们常常会得到浮点数的奇怪语义:
例如, ``round(0.98, 2)`` 仍然给出了 0.97999999999999998.


Proposal
========

符合正则表达式 ``'\d*.\d*'`` 的字面量将是有理数.


Backwards Compatibility
=======================

唯一向后兼容的问题是上面提到的字面量类型. 建议进行以下迁移:

1. 批准后的下一个 Python 将允许 ``from __future__ import rational_literals``
   使所有这些字面量被视为有理数.

2. 在缺少 ``__future__`` 语句的情况下, Python 3.0 将默认启用关于此类字面量的警告.
    警告消息将包含有关 ``__future__`` 语句的信息, 并指示要获得浮点数字面量, 它们应该以 "e0" 为后缀.

3. Python 3.1 默认会关闭警告. 此警告将保留24个月, 此时字面量将是合理的, 警告将被删除.


Common Objections
=================

有理数是缓慢和内存密集的!
(放松, 我不会带走浮点数, 我只是再加两个字符. ``1e0`` 仍然是浮点数)

有理数必须将自己呈现为十进制浮点数, 否则对于期望小数的用户来说它们会很糟糕
(即 ``str(.5)`` 应该返回 ``'.5'`` 而不是 ``'1/2'``). 这意味着必须在某些时候截断许多有理数,
这给我们带来了新的精度损失.



References
==========

.. [1] PEP 239, 将有理数类型添加到 Python, Zadka,
       http://www.python.org/dev/peps/pep-0239/
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
