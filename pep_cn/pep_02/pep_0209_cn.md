
PEP: 209  
Title: Multi-dimensional Arrays  
Version: $Revision$  
Last-Modified: $Date$  
Author: barrett@stsci.edu (Paul Barrett), oliphant@ee.byu.edu (Travis Oliphant)  
Status: Withdrawn  
Type: Standards Track  
Content-Type: text/x-rst  
Created: 03-Jan-2001   
Python-Version: 2.2  
Post-History:  


Abstract
========

该 PEP 建议重新设计和重新实现多维阵列模块 Numeric, 以便更容易向模块添加新特性和功能.
将受到特别关注的数字2的方面是有效访问超过千兆字节的阵列, 并且由不均匀的数据结构或记录组成.
提出的设计使用了四个 Python 类: ArrayType, UFunc, Array 和 ArrayView;
和一个低级 C扩展模块_ufunc, 可以有效地处理数组操作. 此外, 每个数组类型都有自己的 C扩展模块,
该模块定义了该类型的强制规则, 操作和方法. 此设计可以以模块化方式添加新类型, 特征和功能.
新版本将引入与当前数字的一些不兼容性.


Motivation
==========

多维数组通常用于存储和操作科学, 工程和计算中的数据.
Python 目前有一个扩展模块, 名为 Numeric (以下称为 Numeric 1),
它为用户操作中等大小 (10 MB 的数据) 的同类数据阵列提供了一组令人满意的功能.
为了访问可能不均匀数据的更大阵列 (100 MB 或更多指令), 数字 1 的实现效率低且麻烦.
将来, Numerical Python 社区对其他功能的请求也可能是 PEP 211:
向 Python 添加新的线性运算符, 225: Elementwise/Objectwise 运算符说明.


Proposal
========

该提议建议重新设计和重新实现 Numeric 1, 此后称为 Numeric 2,
它将以简单和模块化的方式添加新类型, 特征和功能.
Numeric 2 的初始设计应该专注于为操作各种类型的数组提供通用框架,
并且应该能够提供一种直接的机制来添加新的数组类型和 UFunc.
然后, 可以在这个核心之上分层对各种学科更具体的功能方法.
此新模块仍将被称为 Numeric, 并且将保留在 Numeric 1 中找到的大多数特性.

提出的设计使用了四个 Python 类: ArrayType, UFunc, Array 和 ArrayView;
以及一个低级 C 扩展模块, 可以有效地处理数组操作. 此外,
每个数组类型都有自己的 C 扩展模块, 该模块定义了该类型的强制规则, 操作和方法.
在以后, 当核心功能稳定时, 一些 Python 类可以转换为 C 扩展类型.

一些计划的功能是 :

1.  改善内存使用

    在处理大型阵列时, 此功能尤为重要, 并且可以显着提高性能和内存使用率.
	我们已经确定了可以改进内存使用的几个方面:

    a.  使用局部强制模型

        Numeric 2 (如 Numeric 1) 不是使用创建临时数组的 Python 全局强制模型,
		而是实现 PEP 208 中描述的局部强制模型, 该模型将强制的责任推迟到运算符.
		通过使用内部缓冲区, 如果需要, 可以在操作时对每个阵列 (包括输出阵列) 进行强制操作.
		基准测试 [1]_ 表明, 在内部缓冲区小于 L2 高速缓存大小且处理器负载不足的情况下,
		性能最多只会略有降低. 为了完全避免数组强制, 在 Numeric 2 中允许具有混合类型参数的 C 函数.

    b.  避免创建临时数组

        在复杂的数组表达式中 (即具有多个操作), 每个操作将创建一个临时数组,
		该数组将被后续操作使用然后删除. 更好的方法是识别这些临时数组并尽可能重用它们的数据缓冲区,
		即当数组的形状和类型与正在创建的临时数组相同时. 这可以通过检查临时数组的引用计数来完成.
		如果它是 1, 那么一旦操作完成它将被删除并且是重用的候选者.

    c.  可选地使用内存映射文件

        数字用户有时需要从非常大的文件访问数据或处理大于可用内存的数据.
		内存映射数组提供了一种机制, 通过将数据存储在磁盘上使其显示在内存中来实现此目的.
		内存映射阵列应该通过在文件访问期间消除两个重复步骤之一来改进对所有文件的访问.
		数字应该能够透明地访问内存和内存映射的数组.

    d.  记录访问权限

        在某些科学领域, 数据作为二进制记录存储在文件中. 例如, 在天文学中,
		光子数据按照到达时间的顺序存储为一维光子列表.
		这些记录或类 C 结构包含有关检测到的光子的信息,
		例如其到达时间, 其在探测器上的位置及其能量. 每个字段可以是不同的类型, 
		例如 char, int 或 float. 这样的阵列引入了必须处理的新问题,
		特别是可能需要执行字节对齐或字节交换, 利于正确访问数值
		(尽管字节交换也是存储器映射数据的问题).
		Numeric 2 设计用于在访问或操作数据时自动处理对齐和表示问题.
		实施记录有两种方法; 作为派生数组类或特殊数组类型, 具体取决于您的观点.
		我们将此讨论推迟到 "未解决问题" 部分.


2.  其他数组类型

    Numeric 1 有11种定义的类型: char, ubyte, sbyte, short, int, long, float, double, cfloat, cdouble 和 object.
	没有 ushort, uint 或 ulong 类型, 也没有更复杂的类型, 例如位类型, 它可用于某些科学领域,
	也可能用于实现 masked-arrays. Numeric 1 的设计使这些和其他类型的添加, 成为一个困难且容易出错的过程.
	为了能够轻松添加 (和删除) 新的数组类型, 例如, 下面描述的位类型, 需要重新设计数值.

    a.  位类型

        数组之间丰富比较的结果是布尔值数组. 结果可以存储在 char 类型的数组中, 但这是不必要的内存浪费.
		更好的实现将使用位或布尔类型, 将数组大小压缩8倍. 目前正在为 Numeric 1 (由 Travis Oliphant 执行) 实施,
		并且应该包含在 Numeric 2 中.

3.  增强的数组索引语法

    扩展切片语法被添加到 Python 中, 通过允许步长大于1来操作数值数组时提供更大的灵活性.
	这种语法很适合作为常规间隔索引列表的简写. 对于需要不规则间隔索引列表的情况, 
	增强型数组索引语法将允许1-D数组作为参数.

4.  丰富的比较

    PEP 207 的实现: Python 2.1 中的丰富的比较在操作数组时提供了额外的灵活性.
	我们打算在 Numeric 2 中实现此功能.

5. 数组广播规则

   当标量和数组之间的操作完成时, 此隐式特性是创建一个新的数组,
   其形状与包含标量值的数组操作数相同. 这称为数组广播.
   它也适用于较小等级的数组, 例如向量. 此隐式特性在 Numeric 1 中实现,
   也将在 Numeric 2 中实现.


Design and Implementation
=========================

Numeric 2 的设计有四个主要类:

1.  ArrayType:

    这是一个简单的类, 它描述了数组类型的基本属性,
	例如: 它的名称, 大小 (以字节为单位), 与其他类型的强制关系, 等等, 例如.

    ::

        Int32 = ArrayType('Int32', 4, 'doc-string')

    当导入该类型的 C 扩展模块时, 将定义其与其他类型的关系.
	相应的 Python 代码是::

        Int32.astype[Real64] = Real64

    这表示 Real64 数组类型的优先级高于 Int32 数组类型.

    为核心实现提出了以下属性和方法. 可以在个体基础上添加附加属性,
	例如, 位类型的 .bitsize 或 .bitstrides.

    Attributes::

        .name:                  e.g. "Int32", "Float64", etc.
        .typecode:              e.g. 'i', 'f', etc.
                                (for backward compatibility)
        .size (in bytes):       e.g. 4, 8, etc.
        .array_rules (mapping): rules between array types
        .pyobj_rules (mapping): rules between array and python types
        .doc:                   documentation string

    Methods::

        __init__():             initialization
        __del__():              destruction
        __repr__():             representation

    C-API: 这仍然需要充实.


2.  UFunc:

    这个类是 Numeric 2 的核心. 它的设计类似于 ArrayType 的设计,
	因为 UFunc 创建一个单独的可调用对象, 其属性是名称, 参数的总数和输入数,
	文档字符串和空的 CFunc 字典; 例如.

    ::

        add = UFunc('add', 3, 2, 'doc-string')

    定义时, add 实例没有与之关联的 C 函数, 因此无法正常工作.
	稍后在导入数组类型的 C 扩展模块时填充或注册 CFunc 字典.
	register 方法的参数是: 函数名, 函数描述符和 CUFunc 对象.
	相应的 Python 代码是

    ::

        add.register('add', (Int32, Int32, Int32), cfunc-add)

    在数组类型模块的初始化函数中, 例如, Int32, 有两个 C API 函数:
	一个用于初始化强制规则, 另一个用于注册 CFunc 对象.

    当一个操作应用于某些数组时, 调用 ``__call__`` 方法. 它获取每个数组的类型
	(如果没有特定输出数组, 它是根据强制规则创建的) 并检查 CFunc 字典中是否有与参数类型匹配的键.
	如果存在则立即执行操作, 否则使用强制规则来搜索相关操作和一组转换函数.
	然后 ``__call__`` 方法调用用 C 编写的计算方法迭代每个数组的切片, 即::

        _ufunc.compute(slice, data, func, swap, conv)

    'func' 参数是 CFuncObject, 而 'swap' 和 'conv' 参数是需要预处理或后处理的
	那些数组的 CFuncObjects 列表, 否则使用 None. data 参数是缓冲区对象的列表,
	slice 参数给出每个维度的迭代次数以及每个数组和每个维度的缓冲区偏移量和步长.

   我们预定义了几个 UFuncs 提供给 ``__call__`` 方法使用: cast, swap, getobj 和 setobj.
   cast 和 swap 函数分别进行强制和字节交换, getobj 和 setobj 函数在 Numeric 数组和 Python 序列之间进行强制转换.

    为核心实现提出了以下属性和方法.

    Attributes::

        .name:                  e.g. "add", "subtract", etc.
        .nargs:                 number of total arguments
        .iargs:                 number of input arguments
        .cfuncs (mapping):      the set C functions
        .doc:                   documentation string

    Methods::

        __init__():             initialization
        __del__():              destruction
        __repr__():             representation
        __call__():             look-up and dispatch method
        initrule():             initialize coercion rule
        uninitrule():           uninitialize coercion rule
        register():             register a CUFunc
        unregister():           unregister a CUFunc

    C-API: This still needs to be fleshed-out.

3.  Array:

    这个类包含有关数组的信息, 例如数据的形状, 类型, 字节序等.
	它的运算符, '+', ' - ' 等只是调用相应的 UFunc 函数, 例如.

    ::

        def __add__(self, other):
            return ufunc.add(self, other)

    为核心实现提出了以下属性, 方法和功能.

    Attributes::

        .shape:                 shape of the array
        .format:                type of the array
        .real (only complex):   real part of a complex array
        .imag (only complex):   imaginary part of a complex array

    Methods::

        __init__():             initialization
        __del__():              destruction
        __repr_():              representation
        __str__():              pretty representation
        __cmp__():              rich comparison
        __len__():
        __getitem__():
        __setitem__():
        __getslice__():
        __setslice__():
        numeric methods:
        copy():                 copy of array
        aslist():               create list from array
        asstring():             create string from array

    Functions::

        fromlist():             create array from sequence
        fromstring():           create array from string
        array():                create array with shape and value
        concat():               concatenate two arrays
        resize():               resize array

    C-API: 这仍然需要充实.

4.  ArrayView

    此类类似于 Array 类, 除了 reshape 和 flat 方法将引发异常,
	因为不能使用指针和步长信息对非连续数组进行重新修正或展平.

    C-API: 这仍然需要充实.

5.  C-extension modules:

    Numeric2 将具有多个 C 扩展模块.

    a.  _ufunc:

        该集的主要模块是 _ufuncmodule.c. 该模块的目的是尽量减少,
		即使用指定的 C 函数迭代数组. 这些功能的接口与数字1相同, 即.

        ::

            int (*CFunc)(char *data, int *steps, int repeat, void *func);

        并且它们的功能预计是相同的, 即它们在最内层维度上迭代.

        为核心实现提出了以下属性和方法.

        Attributes:

        Methods::

            compute():

        C-API: 这仍然需要充实.

    b.  _int32, _real64, etc.:

        每种数组类型也会有 C 扩展模块, 例如: _int32module.c, _real64module.c 等.
		如前所述, 当这些模块由 UFunc 模块导入时, 它们将自动注册其功能和强制规则.
		可以轻松实现和使用这些模块的新版本或改进版本, 而不会影响 Numeric 2 的其余部分.


Open Issues
===========

1.  切片语法是否默认为复制或查看特性?

    Python 的默认行为是在使用切片语法时返回子列表或元组的副本, 而 Numeric 1 将视图返回到数组中.
	为 Numeric 1 做出的选择显然是出于性能原因: 开发人员希望避免在每个数组操作期间分配和复制数据缓冲区的代价,
	并且认为对数组的深层副本的需求很少. 然而, 有些人认为 Numeric 的切片表示法也应该具有与 Python 列表一致的复制行为.
	在这种情况下, 可以通过实现写​​时复制来最小化与复制特性相关的性能损失.
	该方案使两个数组共享一个数据缓冲区 (如在视图特性中), 直到为任一数组分配新数据, 此时制作数据缓冲区的副本.
	然后, 视图特性将由 ArrayView 类实现, 其特性类似于 Numeric 1 数组, 即 .shape 不能为非连续数组设置.
	使用 ArrayView 类还可以明确表示数组包含的数据类型.

2.  项语法是否默认为复制或查看特性?

    项语法也出现了类似的问题. 例如, 如果 ``a = [[0,1,2], [3,4,5]]`` 和 ``b = a[0]``,
	那么改变 ``b[0]`` 改变 ``a[0][0]``, 因为 ``a[0]`` 是 a 的第一行的引用或视图.
	因此, 如果 c 是一个二维数组, 那么 ``c[i]`` 应该返回一个 1-d 数组, 这是一个视图,
	而不是 c 的副本, 以保持一致性. 然而, ``c[i]`` 可以被认为只是 ``c[i,：]`` 的简写,
	这意味着复制特性假设切片语法返回一个副本. Numeric 2 的行为应该与列表相同并返回视图,
	还是应该返回副本.

3.  如何实现标量强制?

    Python 的数字类型比 Numeric 少, 这会导致强制问题. 例如, 当将 float 类型的 Python 标量
	与 float 类型的 Numeric 数组相乘时, Numeric 数组将转换为 double, 因为 Python float 类型实际上是 double.
	这通常不是理想的特性, 因为数字数组的大小将加倍, 这可能很烦人, 特别是对于非常大的数组.
	我们更喜欢数组类型胜过同一类型的 python 类型, 即整数, 浮点数和复数. 因此,
	Python 整数和 Int16 (short) 数组之间的操作将返回 Int16 数组. 而 Python float 和 Int16 数组之间的操作
	将返回 Float64 (double) 数组. 两个数组之间的操作使用正常的强制规则.

4.  如何处理整数除法?

    在未来的 Python 版本中, 整数除法的行为将发生变化. 操作数将转换为浮点数, 因此结果将是浮点数.
	如果我们实现建议的标量强制规则, 其中数组优先于 Python 标量, 那么将数组除以整数将返回一个整数数组,
	并且与将返回 double 类型数组的 Python 的未来版本不一致. 科学程序员熟悉整数和浮点除法之间的区别,
	因此 Numeric 2 应该继续这种行为?

5.  如何实现记录?

    根据您的观点, 有两种方法可以实现记录. 第一种是将两个除法数组分成不同的类, 具体取决于它们的类型特性.
	例如, 数字数组是一个类, 字符串是第二个, 并记录第三个, 因为每个类的操作范围和类型不同.
	因此, 记录数组不是新类型, 而是更灵活的数组形式的机制. 为了方便地访问和操作这样的复杂数据,
	该类由具有不同字节偏移的数字数组组成数据缓冲区. 例如, 一个人可能有一个由 Int16, Real32 值数组组成的表.
	两个数字数组, 一个具有0字节的偏移量和一个6字节的步长被解释为 Int16,
	一个具有2字节的偏移量和一个6字节的步长被解释为 Real32 将代表记录数组.
	两个数值数组都引用相同的数据缓冲区, 但具有不同的偏移和步幅属性, 以及不同的数字类型.

    第二种方法是将记录视为许多数组类型之一, 尽管数组操作比数字数组少, 可能不同.
	此方法将数组类型视为固定长度字符串的映射. 映射可以是简单的, 如整数和浮点数,
	也可以是复数, 如复数, 字节串和 C 结构. 记录类型有效地将 struct 和 Numeric 模块合并为一个多维 struct 数组.
	此方法意味着对数组接口进行某些更改. 例如, 'typecode' 关键字参数应该更改为更具描述性的 'format' 关键字.

    a.  如何定义和实现记录语义?

        对于记录采用哪种实现方法, 如果希望访问记录的子字段, 则必须确定如何访问和操作它们的语法和语义.
		在这种情况下, 记录类型基本上可以被认为是非同类列表, 就像 struct 模块的 unpack 方法返回的元组一样;
		并且一维记录数组可以被解释为二维数组, 第二维度是字段列表的索引.
		这种增强的数组语义使得访问一个或多个字段的数组变得简单直接. 它还允许用户以自然直观的方式对场进行数组操作.
		如果我们假设记录是作为数组类型实现的, 那么最后一个维度默认为0, 因此可以忽略由简单类型组成的数组, 如数字.

6.  如何实现掩码数组?

    Numeric 1 中的掩码数组实现为单独的数组类. 通过向 Numeric 2 添加新数组类型的能力,
	可以将 Numeric 2 中的掩码数组实现为新数组类型而不是数组类.

7.  如何处理数字错误 (特别是 IEEE 浮点错误)?

    提议者 (Paul Barrett 和 Travis Oliphant) 不清楚处理错误的最佳或首选方法是什么. 由于大多数 C 函数执行操作,
	因此迭代数组的最内 (最后) 维. 该维度可以包含具有一个或多个不同类型的错误的一千个或更多个项目,
	例如除零, 下溢和溢出. 此外, 跟踪这些错误可能会牺牲性能. 因此, 我们建议几种选择:

    a.  打印最严重错误的消息, 将其留给用户以查找错误.

    b.  打印出现的所有错误和发生次数的消息, 并将其留给用户以查找错误.

    c.  打印出现的所有错误的消息以及它们发生的位置列表.

    d.  或者使用混合方法, 仅打印最严重的错误, 同时跟踪错误发生的位置和位置.
	    这将允许用户在保持错误消息简短的同时定位错误.

8.  简化 FORTRAN 库和代码集成需要哪些功能?

在这个阶段考虑如何简化 Numeric 2 中 FORTRAN 库和用户代码的集成是一个好主意.


Implementation Steps
====================

1.  实现基本的 UFunc 功能

    a.  最小数组类:

        必要的类属性和方法 例如 .shape, .data,
        .type, 等等.

    b.  最小 ArrayType 类:

        Int32, Real64, Complex64, Char, Object

    c.  最小的 UFunc 类:

        UFunc 实例化, CFunction 注册, 对一维数组的 UFunc 调用,
		包括进行对齐, 字节交换和强制的规则.

    d.  最小 C 扩展模块:

        _UFunc, 它在 C 中执行最里面的数组循环.

        此步骤实现了所需的任何操作: 'c = add(a, b)' 其中 a, b 和 c 是 1-D 数组.
		它告诉我们如何添加新的 UFunc, 强制数组, 将必要的信息传递给 C 迭代器方法并进行实际计算.

2.  继续增强 UFunc 迭代器和 Array 类

    a.  为 Array 类实现一些访问方法:
        print, repr, getitem, setitem, etc.

    b.  实现多维数组

    c.  使用 UFunc 实现一些基本的 Array 方法:
        +, -, \*, /, etc.

    d.  启用 UFuncs 来使用 Python 序列.

3.  完成标准的 UFunc 和 Array 类特性

    a.  实现 getslice 和 setslice 行为

    b.  处理数组广播规则

    c.  实现记录类型

4.  添加其他功能

    a.  添加更多 UFunc

    b.  实现缓冲区或 mmap 访问


Incompatibilities
=================

以下是 Numeric 1 和 Numeric 2 之间特性不兼容的列表.

1.  标量强制规则

    Numeric 1 具有针对数组和 Python 数字类型的单组强制规则. 在计算数组表达式时,
	这可能会导致意外和恼人的问题. Numeric 2 打算通过两套强制规则克服这些问题:
	一组用于数组和 Python 数值类型, 另一组用于数组.

2.  没有 savepace 属性

    Numeric 1 中的 savedpace 属性使得具有此属性集的数组优先于未设置该数组的数组.
	Numeric 2 将不具有这样的属性, 因此正常的数组强制规则将生效.

3.  切片语法返回一个副本

    Numeric 1 中的切片语法将视图返回到原始数组中. Numeric 2 的切片行为将是一个副本.
	您应该使用 ArrayView 类来获取数组视图.

4.  布尔比较返回一个布尔数组

    由于 Python 中的当前限制, Numeric 1 中的数组之间的比较导致布尔标量.
	Python 2.1 中 Rich Comparisons 的出现将允许返回一组布尔值.

5.  不推荐使用类型字符

    Numeric 2 将具有由 Type 实例组成的 ArrayType 类, 例如 Int8, Int16, Int32 和 Int 用于有符号整数.
	Numeric 1 中的 typecode 方案可用于向后兼容, 但不推荐使用.


Appendices
==========

A.  隐式子数组迭代

    计算机动画由许多 2-D 图像或相同形状的帧组成. 通过将这些图像堆叠到单个存储器块中, 创建了 3-D 阵列.
	然而, 要执行的操作不是针对整个 3-D 阵列, 而是针对 2-D 子阵列的集合. 在大多数数组语言中, 必须提取, 操作每个帧,
	然后使用类似 for 循环将其重新插入输出数组. J 语言允许程序员通过具有帧和数组的等级来隐式地执行这样的操作.
	默认情况下, 在创建数组期间, 这些等级将是相同的. Numeric 1 开发人员的目的是实现此功能,
	因为它基于语言 J.Nomeric 1 代码具有实现此行为所需的变量, 但从未实现过. 我们打算在 Numeric 2 中实现隐式子数组迭代,
	如果在 Numeric 1 中找到的数组广播规则不完全支持这种行为.


Copyright
=========

This document is placed in the public domain.


Related PEPs
============

* PEP 207: Rich Comparisons
  by Guido van Rossum and David Ascher

* PEP 208: Reworking the Coercion Model
  by Neil Schemenauer and Marc-Andre' Lemburg

* PEP 211: Adding New Linear Algebra Operators to Python
  by Greg Wilson

* PEP 225: Elementwise/Objectwise Operators
  by Huaiyu Zhu

* PEP 228: Reworking Python's Numeric Model
  by Moshe Zadka


References
==========

.. [1] P. Greenfield 2000. private communication.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
