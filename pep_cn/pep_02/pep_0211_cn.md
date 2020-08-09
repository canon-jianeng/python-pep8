
PEP: 211
Title: Adding A New Outer Product Operator
Version: $Revision$
Last-Modified: $Date$
Author: gvwilson@ddj.com (Greg Wilson)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 15-Jul-2000
Python-Version: 2.1
Post-History:
Resolution: https://www.python.org/dev/peps/pep-0465/#rejected-alternatives-to-adding-a-new-operator


Introduction
============

本 PEP 描述了一个将 "@" (发音为 "across") 定义为 Python 2.2 中的新外部产品运算符的提议.
当应用于序列 (或其他可迭代对象) 时, 此运算符将组合它们的迭代器, 为了::

    for (i, j) in S @ T:
        pass

将等同于::

    for i in S:
        for j in T:
            pass

类可以使用特殊方法 ``__across__``, ``__racross__`` 和 ``__across__`` 重载此运算符.
特别是, 新的数值模块 (PEP 209) 将为多维数组重载此运算符以实现矩阵乘法.


Background
==========

数字运算现在只是计算的一小部分, 但许多程序员 - 包括许多 Python 用户 - 仍然需要在代码中表达复杂的数学运算.
因此, 大多数数字语言, 如 APL, Fortran-90, MATLAB, IDL 和 Mathematica, 都提供两种形式的通用算术运算符.
一种形式逐个元素地工作, 例如将矩阵参数的相应元素相乘. 另一个实现该操作的 "数学" 定义, 例如, 执行行列矩阵乘法.

Zhu 和 Lielens 提出以这种方式加倍 Python 的运算符 [1]_.
他们的提议将创建六个新的二进制中间运算符和六个新的原位运算符.

该提案的原始版本更为保守.
作者咨询了 GNU Octave [2]_ 的开发人员, 这是一个 MATLAB 的开源克隆.
它的开发人员一致认为为矩阵乘法提供中间运算符很重要:
数值程序员确实关心是否必须编写 ``mmul(A, B)`` 而不是 ``A op B``.

另一方面, 当被问及为矩阵解决方案和其他操作设置中间运算符的重要性时,
James Rawlings 教授回答说 [3]_:

    我不认为这是必须的, 我做了很多矩阵求逆.
	我不记得它是 ``A\b`` 还是 ``b\A``, 所以我总是写 ``inv(A)*b``.
	我建议删掉 ``\``.

基于此讨论以及美国国家实验室和其他地方学生的反馈,
我们建议只为 Python 添加一个用于矩阵乘法的新运算符.


Iterators
=========

计划在 Python 2.2 中添加迭代器为此提案开辟了更广阔的范围. 作为讨论 PEP 201,
Lockstep Iteration [4]_ 的一部分, 该提案的作者进行了一次非正式的可用性实验 [5]_.
结果显示, 用户在心理上接受 "跨产品" 循环语法. 例如, 大多数用户都期望::

    S = [10, 20, 30]
    T = [1, 2, 3]
    for x in S; y in T:
        print x+y,

打印 ``11 12 13 21 22 23 31 32 33`` 我们相信用户会有同样的反应::

    for (x, y) in S @ T:
        print x+y

即, 他们自然会将此解释为编写循环嵌套的整洁方式.

这就是迭代器的用武之地. 实际上, 在执行循环之前构造两个 (或更多) 序列的交叉产品将非常昂贵.
另一方面, 可以定义 ``@`` 来获取它的参数的迭代器, 然后创建一个外部迭代器, 它返回内部迭代器返回的值的元组.


Discussion
==========

1. 与新的中间运算符相比, 添加 "跨越" 命名函数对 Python 的影响较小.
    然而, 这不会让 Python 对数字程序员更有吸引力, 
	他们真的关心他们是否可以使用运算符编写矩阵乘法,
	或者他们是否必须将其写为函数调用.

2. ``@`` 可以与比较运算符相同的方式链接, 即 ::

    (1, 2) @ (3, 4) @ (5, 6)

   必须返回 ``(1,3,5) ... (2,4,6)``, *not* ``((1,3), 5) ... ((2,4), 6)```.
   这不应该需要解析器的特殊支持, 因为第一个 ``@`` 创建的外部迭代器
   可以很容易地学习如何将它自己与普通的迭代器结合起来.

3. 必须有某种方法来区分可重启的迭代器和无法重启的迭代器. 例如,
   如果 ``S`` 是输入流 (例如文件), 而 ``L`` 是一个列表, 那么 ``S @ L`` 很简单,
   但 ``L @ S`` 不是, 因为通过流的迭代不能重复. 这可以被视为错误,
   或者通过让外部迭代器检测不可重新启动的内部迭代器并缓存它们的值.

4. 在三个新手 Python 用户面前 (所有经验丰富的程序员) 对白板进行测试表明用户会期望::

    "ab" @ "cd"

   返回四个字符串, 而不是四个字符对的元组. 意见分歧是什么::

    ("a", "b") @ "cd"

   应该 return...


Alternatives
============

1. 什么都不做 - 保持 Python 简单.

   这始终是默认选择.

2. 添加命名函数而不是运算符.

   Python 主要不是数字语言; 对于这种特殊情况, 它可能不值得复杂化. 但是,
   支持真实矩阵乘法  *is* 经常被要求, 并且内置序列类型的 ``@`` 的语义
   将简化一个非常常见的习惯用法 (嵌套循环) 的表达式.

3. 引入所有现有运算符的前缀形式, 如 ``~*`` 和 ``~+``, 如 PEP 225 中所提出的那样 [1]_.

   我们对此的反对意见是, 没有足够的要求来证明额外的复杂性 (参见 Rawlings 的评论 [3]_),
   并且所提出的语法未能通过 "low toner" 可读性测试.


Acknowledgments
===============

我很感谢朱怀宇发起这次讨论, 感谢 James Rawlings
和各种 Python 课程的学生讨论数字程序员真正关心的问题.


References
==========

.. [1] PEP 225, Elementwise/Objectwise Operators, Zhu, Lielens
       http://www.python.org/dev/peps/pep-0225/

.. [2] http://bevo.che.wisc.edu/octave/

.. [3] http://www.egroups.com/message/python-numeric/4

.. [4] PEP 201, Lockstep Iteration, Warsaw
       http://www.python.org/dev/peps/pep-0201/

.. [5] https://mail.python.org/pipermail/python-dev/2000-July/006427.html



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
