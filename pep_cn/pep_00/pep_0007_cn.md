
PEP: 7
Title: Style Guide for C Code
Version: $Revision$
Last-Modified: $Date$
Author: Guido van Rossum <guido@python.org>, Barry Warsaw <barry@python.org>
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 05-Jul-2001
Post-History:


Introduction
============

本文档给出了包含 Python 的 C 实现的 C 代码的编码约定
请参阅描述 Python 代码样式指南的配套信息 PEP [1]_

注意, 规则是有缺陷的, 打破特定规则的两个充分理由:

1. 应用规则会使代码的可读性降低, 即使是习惯于阅读遵循规则的代码的人也是如此
2. 为了与周围的代码保持一致（也许是出于历史原因） - 虽然这也是一个清理别人的混乱的机会（真正的 XP 风格）

C dialect
=========

* Python 3.6 之前的 Python 版本使用 ANSI/ISO 标准C (1989版标准)
  这意味着（在许多其他事情中）所有声明必须位于代码块的顶部（不一定在函数的顶部）

* 大于或等于 3.6 的 Python 版本使用 C89 和几个选择的 C99 功能

  - "<stdint.h>" 和 "<inttypes.h>" 中的标准整数类型, 我们需要固定宽度的整数类型
  - "static inline" 函数
  - 指定的初始值设定项（特别适用于类型声明）
  - 混合声明
  - 布尔
  - C ++ - 样式行注释

  未来的 C99 功能可能会在未来添加到此列表中, 具体取决于编译器支持 (主要是 MSVC )

* 不要使用 GCC 扩展 (例如, 不要在没有尾部反斜杠的情况下编写多行字符串)

* 所有函数声明和定义必须使用完整原型 (即指定所有参数的类型)

* 仅在 Python 3.6 或更高版本中使用 C++ 样式 //单行注释

* 主要的编译器没有编译器警告 (gcc, VC++, 其他几个)

Code lay-out
============

* 使用 4个空格缩进且没有标签

* 任何行都不应该超过79个字符, 如果这和前面的规则一起没有给你足够的代码空间,
  那么你的代码就太复杂了 - 考虑使用子程序

* 任何行都不应该以空格结尾, 如果您认为需要重要的行尾空格, 请再想一想 - 某人的编辑可能会将其作为常规问题删除

* 函数定义样式：第1列中的函数名，第1列中最外面的花括号，局部变量声明后的空行 ::

    static int
    extra_ivars(PyTypeObject *type, PyTypeObject *base)
    {
        int t_size = PyType_BASICSIZE(type);
        int b_size = PyType_BASICSIZE(base);

        assert(t_size >= b_size); /* type smaller than base! */
        ...
        return 1;
    }

* 代码结构：关键字之间的一个空格, 如 "if", "for" 和下面的左括号; 在括号里面没有空间;
  在任何地方都需要括号, 即使 C 允许省略它们, 但不要将它们添加到您没有修改的代码中
  所有新的 C 代码都需要大括号, 大括号应格式如下 ::

    if (mro != NULL) {
        ...
    }
    else {
        ...
    }

* return 语句不应该得到多余的括号 ::

    return albatross; /* correct */
    return(albatross); /* incorrect */
  
* 函数和宏调用样式: "foo(a, b, c)" - 左括号之前没有空格, 括号里面没有空格, 逗号前没有空格, 每个逗号后面有一个空格

* 始终在赋值, 布尔和比较运算符周围放置空格, 在使用大量运算符的表达式中, 在最外层（最低优先级）运算符周围添加空格

* 打破长行: 如果可以的话, 在最外面的参数列表中用逗号分隔, 始终适当地缩进延续行, 例如 ::

    PyErr_Format(PyExc_TypeError,
                "cannot create '%.100s' instances",
                type->tp_name);

* 当您在二元运算符处断开长表达式时, 运算符将在前一行的末尾处运行, 并且圆括号应该如图所示进行格式化, 例如 ::

    if (type->tp_dictoffset != 0 && base->tp_dictoffset == 0 &&
        type->tp_dictoffset == b_size &&
        (size_t)t_size == b_size + sizeof(PyObject *))
    {
        return 0; /* "Forgive" adding a __dict__ only */
    }

* 在函数, 结构定义和函数内的主要部分周围放置空行

* 在描述的代码之前进行注释

* 所有函数和全局变量都应该声明为 static, 除非它们是已经发布接口的一部分

* 对于外部函数和变量, 我们总是在 "Include" 目录中的相应头文件中有一个声明, 它使用 "PyAPI_FUNC" 宏, 就像这样 ::

    PyAPI_FUNC(PyObject *) PyObject_Repr(PyObject *);

Naming conventions
==================

* 公共函数使用 "Py" 前缀; 从不用于静态功能
  "Py_" 前缀保留给全局服务程序, 如 "Py_FatalError";
  特定的程序组 (例如, 特定对象类型 API) 使用更长的前缀, 例如, "PyString_" 用于字符串函数

* 公共函数和变量使用具有下划线的混合词, 如下所示: "PyObject_GetAttr", "Py_BuildValue", "PyExc_TypeError"

* 有时，加载器必须能够看到 "internal" 功能;
  我们使用 "_Py" 前缀, 例如: "_PyObject_Dump"

* 宏应该有一个混合词前缀, 然后使用大写, 例如: "PyString_AS_STRING", "Py_PRINT_RAW"

Documentation Strings
=====================

* 对文档字符串使用 "PyDoc_STR()" 或 "PyDoc_STRVAR()" 宏来支持在没有 docstrings 的情况下构建 Python ("./configure --without-doc-strings")

  对于需要支持 2.3 以上版本的 Python 的 C 代码,
  你可以在包含 "Python.h" :: 之后包含这个内容 ::

      #ifndef PyDoc_STR
      #define PyDoc_VAR(name)         static char name[]
      #define PyDoc_STR(str)          (str)
      #define PyDoc_STRVAR(name, str) PyDoc_VAR(name) = PyDoc_STR(str)
      #endif

* 每个函数 docstring 的第一行应该是一个 "签名行", 它给出了参数和返回值的简要概要, 例如 ::

    PyDoc_STRVAR(myfunction__doc__,
      "myfunction(name, value) -> bool\n\n\
      Determine whether name and value make a valid pair.");

  始终在签名行和说明文本之间包含一个空行

  如果函数的返回值始终为 None (因为没有有意义的返回值), 请不要包含返回类型的指示

* 在编写多行文档字符串时, 请务必始终使用反斜杠延续, 如上例所示, 或字符串文字串联 ::

    PyDoc_STRVAR(myfunction__doc__,
    "myfunction(name, value) -> bool\n\n"
    "Determine whether name and value make a valid pair.");

  尽管一些 C 编译器接受字符串文字没有任何 ::

    /* BAD -- don't do this! */
    PyDoc_STRVAR(myfunction__doc__,
    "myfunction(name, value) -> bool\n\n
    Determine whether name and value make a valid pair.");

  不是全部都做; 众所周知, MSVC 编译器会抱怨这一点

References
==========

.. [1] PEP 8, "Python 代码的样式指南", van Rossum, Warsaw (http://www.python.org/dev/peps/pep-0008)

Copyright
=========

本文档已置于公共域名

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   sentence-end-double-space: t  
   fill-column: 70  
   coding: utf-8  
   End:  
