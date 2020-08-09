
PEP: 207  
Title: Rich Comparisons  
Version: $Revision$  
Last-Modified: $Date$  
Author: guido@python.org (Guido van Rossum), DavidA@ActiveState.com (David Ascher)  
Status: Final  
Type: Standards Track  
Content-Type: text/x-rst  
Created:  
Python-Version: 2.1  
Post-History:  


Abstract
========

该 PEP 提出了几个用于比较的新功能:

- 允许在类和 C 扩展中单独重载 <, >, <=, >=, ==, !=.

- 除布尔结果外, 允许任何重载运算符返回其他内容.


Motivation
==========

主要动机来自 NumPy, 其用户同意 A<B 应该返回数组逐个元素比较结果;
他们目前必须将其拼写为 less(A, B), 因为 A<B 只能返回布尔结果或引发异常.

另一个动机是, 类型通常不具有自然顺序, 但仍需要进行相等性比较.
目前这种类型 *必须* 实现比较, 从而定义任意顺序, 利于可以测试相等性.

此外, 对于某些对象类型, 可以比指令测试更有效地实现相等性测试;
例如, 长度不同的列表和词典是不相等的, 但是排序需要检查一些 (可能是所有) 项.


Previous Work
=============

之前提出了丰富的比较; 特别是 David Ascher, 在使用 Numerical Python 之后:

  http://starship.python.net/crew/da/proposals/richcmp.html

它也包含在下面作为附录. 本 PEP 中的大多数材料都来自 David 的提议.


Concerns
========

1. 向后兼容性, 在 Python 级别 (使用 ``__cmp__`` 的类不需要更改)
    和 C级 (定义 ``tp_comparea`` 的扩展不需要更改, 使用 ``PyObject_Compare()`` 的代码必须工作,
	即使比较对象使用新的丰富比较方案）.

2. 当 A<B 返回元素比较矩阵时, 一个容易犯的错误就是在布尔上下文中使用这个表达式.
   没有特别的预防措施, 它总是如此. 此用法应该引发异常.

3. 如果一个类重写 x == y 但没有别的, 那么 x != y 应该计算为 not(x == y), 还是失败?
    那么 < 和 >= 之间, 或者 > 和 <= 之间的相似关系怎么样?

4. 同样, 我们是否应该允许从 y>x 计算 x<y?
    并且 x<=y 来自 not(x>y)?
	x==y 来自 y==x,
	或 x!=y 来自 y!=x?

5. 当比较运算符返回元素比较时, 如何处理快速运算符,
    如 A<B<C, ``A<B and C<D``, ``A<B or C<D``?

6. 如何处理 ``min()`` 和 ``max()``, 'in' 和 'not in' 运算符,
    ``list.sort()``, 通过内置操作字典键比较和其他比较用法？


Proposed Resolutions
====================

1. 完全向后兼容性可以如下实现. 当一个对象定义 ``tp_compare()``
    但不是 ``tp_richcompare()``, 并且请求一个丰富的比较时,
	``tp_compare()`` 的结果用明显的方式使用.
	例如. 如果请求 "<", 如果 ``tp_compare()`` 引发异常则会出现异常,
	如果 ``tp_compare()`` 为负, 结果为1, 如果为零或为正, 则结果为0. 等等.

   可以如下实现完全向前兼容性. 当对一个实现 ``tp_richcompare()`` 的对象请求经典比较时,
   最多使用三个比较: 首先, 尝试==, 如果返回 true, 则返回 0;
   接下来, 尝试 <, 如果返回 true, 则返回 -1;
   接下来, 尝试  >, 如果返回 true, 则返回+1.
   如果尝试任何运算符, 返回非布尔值 (见下文), 则转换为布尔值引发的异常将通过.
   如果没有尝试运算符, 返回 true, 则接下来尝试经典的比较回退.

   (我认为应该尝试三种比较的顺序是漫长而艰难的. 根据循环数据结构比较的特性,
    我有一个令人信服的论据, 即按此顺序进行. 但由于该代码再次发生变化, 我不太确定它会再有所作为.)

2. 任何返回布尔值集合而不是单个布尔值的类型都应该定义 ``nb_nonzero()`` 来引发异常.
    这种类型被认为是非布尔型的.

3. == 和 != 运算符不认为是对方的补码 (例如, IEEE 754 浮点数不满足此要求).
    如果需要, 可以根据类型来实现. 类似于 < 和 >=, 或 > 和 <=;
	有很多这些假设不正确的例子 (例如, tabnanny)

4. 自反性规则 *是* 由Python 假设. 因此, 解释器可以将 y>x 与 x<y, y>=x 交换,
    其中 x<=y, 并且可以交换 x==y 和 x!=y 的参数.
	(注意: Python 目前假设 x==x 始终为真且 x!=x 永远不为真; 不应该假设这一点.)

5. 在当前的提议中, 当 A<B 返回元素比较数组时, 此结果被视为非布尔值,
   并且快速操作符将其解释为布尔值会引发异常. David Ascher 的提议试图解决这个问题;
   我认为这是不值的, 代码会导致额外复杂性. 你可以写 (A<B)&(B<C), 而不是 A<B<C.

6. ``min()`` 和 ``list.sort()`` 操作只使用 < 运算符; max() 只会使用 > 运算符.
    'in' 和 'not in' 运算符和字典查找仅使用 == 运算符.


Implementation Proposal
=======================

这密切遵循 David Ascher 的提议.

C API
-----

- 新函数::

      PyObject *PyObject_RichCompare(PyObject *, PyObject *, int)

  这将执行请求的丰富比较, 返回 Python 对象或引发异常.
  第三个参数必须是以下其中之一
  Py_LT, Py_LE, Py_EQ, Py_NE, Py_GT 或者 Py_GE.

  ::

      int PyObject_RichCompareBool(PyObject *, PyObject *, int)

  这将执行请求的丰富比较, 返回布尔值: -1表示异常, 0表示false, 1表示true.
  第三个参数必须是 Py_LT, Py_LE, Py_EQ, Py_NE, Py_GT 或 Py_GE 之一.
  请注意, 当 ``PyObject_RichCompare()`` 返回非布尔对象时, ``PyObject_RichCompareBool()`` 将引发异常.

- 新的 typedef::

      typedef PyObject *(*richcmpfunc) (PyObject *, PyObject *, int);

- 类型对象中的新槽, 替换备用的 tp_xxx7::

      richcmpfunc tp_richcompare;

  这应该是一个与 ``PyObject_RichCompare()`` 具有相同签名的函数,
  并执行相同的比较. 至少有一个参数是使用 tp_richcompare 槽的类型,
  但另一个可能具有不同的类型. 如果函数无法比较特定的对象组合,
  则应返回对 ``Py_NotImplemented`` 的新引用。.

- ``PyObject_Compare()`` 被更改为尝试丰富的比较 (如果它们被定义)
   (但仅限于未定义经典比较).

Changes to the interpreter
--------------------------

- 每当调用 ``PyObject_Compare()`` 来获取特定比较的结果
  (例如, 在 ``list.sort()`` 中, 当然对于 ceval.c 中的比较运算符) 时,
  代码是更改为调用 ``PyObject_RichCompare()`` 或 ``PyObject_RichCompareBool()``;
  如果 C 代码需要知道比较的结果, 则在结果上调用 ``PyObject_IsTrue()``
  (可能引发异常).

- 目前定义比较的大多数内置类型将被修改来定义丰富的比较. 
  (这是可选的; 到目前为止, 我已经转换了列表, 元组, 复数和数组,
  但我不确定是否会转换其他数据.)

Classes
-------

- 类可以定义新的特殊方法 ``__lt__``, ``__le__``, ``__eq__``, ``__ne__``，,
  ``__gt__``, ``__ge__`` 来覆盖相应的运算符. 
  (即, <, <=, ==, !=, >, >=. 你必须喜欢 Fortran 遗产.)
  如果一个类定义了 ``__cmp__``, 它只在 ``__lt__`` 等时使用,
  已经尝试过并返回 ``NotImplemented``.


Copyright
=========

This document has been placed in the public domain.

Appendix
========

以下是 David Ascher 的大部分原始提案
(版本 0.2.1, 日期为1998年7月22日星期三 16:49:28;
我已经将内容, 历史和补丁部分排除在外).
它几乎解决了上述所有问题.


Abstract
========

提出了一种新机制, 允许比较 Python 对象返回除 -1, 0 或 1 之外的值 (或引发异常).
这种机制完全向后兼容, 可以在 C ``PyObject`` 类型或 Python 类定义的级别进行控制.
建议机制有三个合作部分:

- 使用类型对象结构中的最后一个槽来存储指向丰富比较函数的指针

- 为类添加特殊方法

- 在内置 ``cmp()`` 函数中添加了一个可选参数.


Motivation
==========

Python 对象的当前比较协议假设可以比较任何两个 Python 对象
(从 Python 1.5 开始, 对象比较可以引发异常), 并且任何比较的返回值应该是 -1, 0 或 1.
-1 表示 比较函数的第一个参数小于右边的一个, +1 表示对立面, 0 表示两个对象相等.
虽然这种机制允许建立一个顺序关系 (例如, 由列表对象的 ``sort()`` 方法使用),
但它已被证明在数字 Python (NumPy) 的上下文中受到限制。.

具体来说, NumPy 允许创建支持大多数数字运算符的多维数组::

     x = array((1,2,3,4))        y = array((2,2,4,4))

是两个 NumPy 数组. 虽然它们可以按元素添加,::

     z = x + y   # z == array((3,4,7,8))

它们无法在当前框架中进行比较 - NumPy 的发布版本比较指针 (因此产生垃圾信息),
这是最近添加能力 (在 1.5 中) 在比较函数中引发异常之前的唯一解决方案.

即使具有引发异常的能力, 当前协议也使得数组比较无用. 为了解决这个问题,
NumPy 包含了几个执行比较的函数: ``less()``, ``less_equal()``, ``greater()``,
``greater_equal()``, ``equal()``, ``not_equal()``.
这些函数返回与其参数 (取模广播) 具有相同形状的数组, 填充 0 和 1,
具体取决于每个元素对的比较是否为真. 因此, 例如, 使用上面定义的数组 x 和 y ::

     less(x,y)

将是一个包含数字的数组 (1,0,0,0).

目前的提议是修改 Python 对象接口来允许 NumPy 包使其成为 x<y 返回与 less(x, y) 相同的东西.
确切的返回值取决于 NumPy 包 - 这个提议真正要求的是改变 Python 核心,
利于扩展对象能够返回除 -1, 0, 1之外的其他东西, 如果他们的作者选择这样做的话.

Current State of Affairs
========================

当前协议在 C 级别, 每个对象类型定义一个 ``tp_compare`` 槽, 它是一个指向一个函数的指针,
该函数接受两个 ``PyObject *`` 引用并返回 -1, 0 或 1. 该函数由 C API 中定义的 ``PyObject_Compare()`` 函数调用.
``PyObject_Compare()`` 也被内置函数 ``cmp()`` 调用, 它带有两个参数.

Proposed Mechanism
------------------

1. 对类型对象的 C 结构的更改

   ``PyTypeObject`` 中的最后一个可用槽, 保留到现在为将来的扩展,
   用于可选地存储指向新的比较函数的指针, 类型为 richcmpfunc, 由::

      typedef PyObject *(*richcmpfunc)
           Py_PROTO((PyObject *, PyObject *, int));

   该函数有三个参数. 前两个是要比较的对象, 第三个是对应于操作码的整数 (LT, LE, EQ, NE, GT, GE 之一).
   如果此槽保留为 NULL, 则不支持该对象类型的丰富比较 (除了类提供下面描述的特殊方法的类实例).

   上述操作码需要添加到已发布的 Python/C API 中
   (可能名称为 Py_LT, Py_LE 等)

2. 增加了类的特殊方法

   希望支持丰富的比较机制的类必须添加以下一个或多个新的特殊方法 ::

        def __lt__(self, other):
           ...
        def __le__(self, other):
           ...
        def __gt__(self, other):
           ...
        def __ge__(self, other):
           ...
        def __eq__(self, other):
           ...
        def __ne__(self, other):
           ...

   当类实例位于相应运算符 (<, <=, >, >=, == 和 != 或 <>) 的左侧时,
   将调用其中的每一个. 参数 other 设置为运算符右侧的对象.
   这些方法的返回值取决于类实现者 (毕竟, 这是提案的全部内容).

   如果运算符左侧的对象没有定义适当的富比较运算符
   (在 C 级别或使用其中一种特殊方法, 则比较相反, 右侧运算符与相反的运算符一起调用,
   并且这两个对象是交换的. 这假设 a<b 和 b>a 是等价的, 因为 a<=b 和 b>=a,
   并且 == 和 != 是可交换的 (例如, a==b 当且仅当 b==a).

   例如, 如果 obj1 是一个支持丰富的比较协议的对象, 而 x 和 y 是不支持丰富的比较协议的对象,
   那么 obj1 < x 将调用 obj1 的 ``__lt__`` 方法, 其中 x 作为第二个参数.
   x < obj1 将调用 obj1 的 ``__gt__`` 方法, x 为第二个参数, x < y 将只使用现有 (非丰富) 比较机制.

   上面的机制使得类可以不执行 ``__lt__``  和  ``__le__`` 或 ``__gt__`` 和 ``__ge__``.
   可以在比较机制中添加更多智能, 但是选择了这组有限的允许 "交换",
   因为它不需要基础设施对返回值进行任何处理 (否定).
   通过单个 (例如, ``__richcmp__``) 方法选择六种特殊方法,
   来允许在 C 实现级别而不是用户定义的方法上执行操作码的调度.

3. 在内置 ``cmp()`` 中添加了一个可选参数

   内置的 ``cmp()`` 仍然用于简单的比较. 对于丰富的比较, 它用第三个参数调用,
   一个是 "<", "<=", ">", ">=", "==", "!=", "<>" (最后两个具有相同的含义).
   当使用其中一个字符串作为第三个参数调用时, ``cmp()`` 可以返回任何 Python 对象.
   否则, 它只能像以前一样返回 -1, 0 或 1.

Chained Comparisons
-------------------

Problem
'''''''

允许比较返回除 -1, 0 或 1 以外的对象的对象在链式比较中使用会很好, 例如::

     x < y < z

目前, 这被 Python 解释为::

     temp1 = x < y
     if temp1:
       return y < z
     else:
       return temp1

请注意, 这需要测试比较结果的真值, 以及右侧比较测试的潜在 "简化".
换句话说, 比较结果的结果的真值确定链式操作的结果.
这在数组的情况下是有问题的, 因为如果 x, y 和 z 是三个数组, 那么用户期望 ::

    x < y < z

成为 0 和 1 的数组, 其中 1 对应于 y 的元素的位置, 这些元素位于 x 和 z 中的对应元素之间.
换句话说, 无论 x<y 的结果如何, 都必须评估右侧, 这与解析器当前使用的机制不兼容。.

Solution
''''''''

Guido 提到, 一种可能的方式是更改链式比较生成的代码, 来允许对数组进行链接 - 智能比较.
以下是他的想法和我的建议的混合. 为 x < y < z 生成的代码将等效于::

     temp1 = x < y
     if temp1:
       temp2 = y < z
       return boolean_combine(temp1, temp2)
     else:
       return temp1

其中 boolean_combine 是一个新函数, 它执行类似下面的操作::

     def boolean_combine(a, b):
         if hasattr(a, '__boolean_and__') or \
            hasattr(b, '__boolean_and__'):
             try:
                 return a.__boolean_and__(b)
             except:
                 return b.__boolean_and__(a)
         else: # standard behavior
             if a:
                 return b
             else:
                 return 0

其中 ``__boolean_and__`` 特殊方法是通过 richcmp 函数的第三个参数的另一个值为 C 级类型实现的.
此方法将执行数组的布尔比较 (当前在 umath 模块中实现为 logical_and ufunc).

因此, 富比较返回的对象应该始终测试为 true, 但应该定义另一个特殊方法, 该方法创建它们的布尔组合及其参数.

这种解决方案的优点是允许链式比较适用于数组, 但缺点是它需要比较数组总是返回 true
(在理想的世界中, 我会让它们总是在真值测试中引发异常, 因为测试的意义 "if a>b:" 是非常含糊不清的.

处理整数比较的内联已经存在仍然适用, 导致最常见情况没有性能成本.

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
