
PEP: 267
Title: Optimized Access to Module Namespaces
Version: $Revision$
Last-Modified: $Date$
Author: jeremy@alum.mit.edu (Jeremy Hylton)
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 23-May-2001
Python-Version: 2.2
Post-History:


Deferral
========

虽然这个 PEP 是一个不错的主意, 但是还没有一个人开始研究这个 PEP,
PEP 266 和 PEP 280 之间的差异. 因此, 它正在推迟.


Abstract
========

此 PEP 提出了全局模块命名空间的新实现以及加速名称解析的内置命名空间.
该实现将使用一组对象指针来处理这些命名空间中的大多数操作.
编译器将在编译时为全局变量和模块属性分配索引.

当前实现将这些名称空间表示为字典. 全局名称每次使用时都会产生字典查找;
内置名称会导致两个字典查找, 全局命名空间中的查找失败以及内置命名空间中的第二个查找.

此实现应该加速使用模块级函数和变量的 Python 代码. 它还应该消除笨拙的编码风格,
这些风格已经发展到可以加速访问这些名称.

实现很复杂, 因为全局和内置命名空间可以以编译器无法检测的方式动态修改.
(示例: 导入模块后, 脚本会修改模块的命名空间.)
因此, 实现必须维护多个辅助数据结构来保留这些动态功能.


Introduction
============

该 PEP 为模块对象提出了一种新的属性访问实现, 它优化了对编译时已知的模块变量的访问.
该模块将这些变量存储在一个数组中, 并提供一个使用数组偏移查找属性的接口.
对于全局变量, 内置变量和导入模块的属性, 编译器将生成使用数组偏移量进行快速访问的代码.

[描述设计的关键部分: dlict, 编译器支持, 愚蠢的名称技巧解决方法, 优化其他模块的全局变量]

该实现将保留模块名称空间的现有语义,
包括在运行时以影响内置名称可见性的方式修改模块名称空间的能力.


DLict design
============

命名空间是使用有时名为 ``dlict`` 的数据结构实现的. 它是一个字典, 为某些字典条目编号.
必须在 C 中实现该类型才能获得可接受的性能. 新的类型统一工作应该使这相当容易.
``DLict`` 可能是字典的子类, 有一些备用存储模块用于某些键.

这里包含一个 Python 实现来说明基本设计::

    """A dictionary-list hybrid"""

    import types

    class DLict:
        def __init__(self, names):
            assert isinstance(names, types.DictType)
            self.names = {}
            self.list = [None] * size
            self.empty = [1] * size
            self.dict = {}
            self.size = 0

        def __getitem__(self, name):
            i = self.names.get(name)
            if i is None:
                return self.dict[name]
            if self.empty[i] is not None:
                raise KeyError, name
            return self.list[i]

        def __setitem__(self, name, val):
            i = self.names.get(name)
            if i is None:
                self.dict[name] = val
            else:
                self.empty[i] = None
                self.list[i] = val
                self.size += 1

        def __delitem__(self, name):
            i = self.names.get(name)
            if i is None:
                del self.dict[name]
            else:
                if self.empty[i] is not None:
                    raise KeyError, name
                self.empty[i] = 1
                self.list[i] = None
                self.size -= 1

        def keys(self):
            if self.dict:
                return self.names.keys() + self.dict.keys()
            else:
                return self.names.keys()

        def values(self):
            if self.dict:
                return self.names.values() + self.dict.values()
            else:
                return self.names.values()

        def items(self):
            if self.dict:
                return self.names.items()
            else:
                return self.names.items() + self.dict.items()

        def __len__(self):
            return self.size + len(self.dict)

        def __cmp__(self, dlict):
            c = cmp(self.names, dlict.names)
            if c != 0:
                return c
            c = cmp(self.size, dlict.size)
            if c != 0:
                return c
            for i in range(len(self.names)):
                c = cmp(self.empty[i], dlict.empty[i])
            if c != 0:
                return c
            if self.empty[i] is None:
                c = cmp(self.list[i], dlict.empty[i])
                if c != 0:
                    return c
            return cmp(self.dict, dlict.dict)

        def clear(self):
            self.dict.clear()
            for i in range(len(self.names)):
                if self.empty[i] is None:
                    self.empty[i] = 1
                    self.list[i] = None

        def update(self):
            pass

        def load(self, index):
            """dlict-special method to support indexed access"""
            if self.empty[index] is None:
                return self.list[index]
            else:
                raise KeyError, index # XXX might want reverse mapping

        def store(self, index, val):
            """dlict-special method to support indexed access"""
            self.empty[index] = None
            self.list[index] = val

        def delete(self, index):
            """dlict-special method to support indexed access"""
            self.empty[index] = 1
            self.list[index] = None


Compiler issues
===============

编译器当前收集模块中所有全局变量的名称. 这些名称绑定在模块级别或绑定在类或函数体中,
声明它们是全局的.

编译器将为每个全局名称分配索引, 并将全局变量的名称和索引添加到模块的代码对象中.
然后, 每个代码对象将不可撤销地绑定到它定义的模块. (不确定是否存在一些微妙的问题.)

对于导入模块的属性, 模块将存储间接记录. 在内部,
模块将存储指向定义模块的指针以及属性在定义模块的全局变量数组中的偏移量.
在第一次查找名称时, 将初始化偏移量.


Runtime model
=============

PythonVM 将使用新的操作码进行扩展, 来通过模块级数组访问全局变量和模块属性.

函数对象需要指向定义它的模块, 利于提供对模块级全局数组的访问.

对于存储在 ``dlict`` 中的模块属性 (称为静态属性), get/delattr 实现需要使用
旧的 by-name 接口跟踪对这些属性的访问. 如果静态属性是动态更新的, 例如.::

   mod.__dict__["foo"] = 2

实现需要更新阵列槽, 而不是备份字典.


Backwards compatibility
=======================

``dlict`` 需要维护关于当前是否使用槽的元信息. 它还需要维护一个指向内置命名空间的指针.
当名称当前未在全局命名空间中使用时, 查找将必须故障转移到内置命名空间.

在相反的情况下, 每个模块可能需要内置命名空间的特殊访问器函数,
该函数检查是否已动态添加内置映射的内置命名空间.
只有在对模块的 ``dlict`` 进行动态更改时才会进行此检查, 即当绑定的名称未在编译时发现时.

对于常见情况, 这些机制几乎没有成本, 无论模块的全局命名空间在运行时是否以奇怪的方式被修改.
它们会为使用全局名称做出不寻常事情的模块增加开销, 但这是一种不常见的做法, 可能值得令人沮丧.

可能需要在 Python 的某个未来版本中禁用对全局命名空间的动态添加.
如果是这样, 新的实施可以提供警告.


Related PEPs
============

PEP 266, 优化全局变量或属性访问, 提出了一种不同的机制, 用于优化对全局变量的访问以及对象的属性.
该机制使用两个新的操作码 ``TRACK_OBJECT`` 和 ``UNTRACK_OBJECT`` 在局部变量数组中创建一个槽,
该槽将全局或对象属性别名化. 如果作为别名的对象被反弹, 则重新绑定操作负责更新别名.

对象跟踪方法适用于比模块更广泛的对象. 它也可能具有更高的运行时成本,
因为使用全局或对象属性的每个函数必须执行额外的操作码以在对象中注册其兴趣
并在退出时取消注册; 注册成本尚不清楚, 但可能涉及动态调整大小的数据结构来保存回调列表.

这里提出的实现避免了注册的需要, 因为它不会创建别名. 相反,
它允许引用全局变量或模块属性的函数保留指向存储原始绑定的位置的指针.
第二个优点是每个模块执行一次初始查找, 而不是每个函数调用执行一次.


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
