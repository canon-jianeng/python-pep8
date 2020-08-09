
PEP: 229
Title: Using Distutils to Build Python
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 16-Nov-2000
Post-History:


Introduction
============

``Modules/Setup`` 机制有一些缺陷:

* 人们必须记住取消注释 ``Modules/Setup`` 来获得所有可能的模块.

* 将 ``Setup`` 移动到新版本的 Python 是乏味的;
   新模块已添加, 因此您不能只复制旧版本, 但必须协调这两个版本.

* 用户必须弄清楚所需库的位置, 例如 ``zlib``.


Proposal
========

使用 Distutils 构建 Python 附带的模块.

这些变化可以分为几个部分:

1. Distutils 需要一些 Python 模块才能构建模块.
   目前我认为最小的列表是 ``posix``, ``_sre`` 和 ``string``.

   这些模块必须在使用 Distutils 之前构建,
   因此它们只需硬连线到 ``Modules/Makefile`` 并自动构建.

2. 将编写一个顶级 setup.py 脚本, 用于检查系统上安装的库并编译尽可能多的模块.

3. 将保存 ``Modules/Setup`` 并且其中的设置将覆盖 ``setup.py`` 的常规行为,
   因此您可以禁用已知有错误的模块, 或指定特定的编译或链接器标志.
   但是, 在 ``setup.py`` 正常工作的常见情况下, ``Setup`` 中的所有内容都将被注释掉.
   另一个 ``Setup.*`` 变得不必要, 因为什么都不会自动生成 ``Setup``.

该补丁已签入 Python 2.1, 随后进行了修改.


Implementation
==============

SourceForge 上的补丁 #102588 包含建议的补丁. 目前, 补丁尝试保守并尽可能少地更改文件,
利于简化补丁. 例如, 没有尝试删除现有的构建机制. 这种简化可以在测试周期的后期等待,
当我们确定将保留补丁时, 或者他们可以等待 Python 2.2.

该补丁进行了以下更改:

* 对 distutils/sysconfig 进行一些必要的更改 (这些将单独检查)

* 在顶级 ``Makefile.in`` 中, "sharedmods" 目标只运行 ``"./python setup.py build"``,
   而 "sharedinstall" 运行 ``"./python setup.py install"``.
   "clobber" 目标还会删除 Distutils 输出的 ``build/`` 子目录.

* ``Modules/Setup.config.in`` 只包含 ``gc`` 和 ``thread`` 模块的条目;
   删除 ``readline``, ``curses`` 和 ``db`` 模块, 因为它现在是 ``setup.py`` 的工作来处理它们.

* ``Modules/Setup.dist`` 现在只包含3个模块的条目 - ``_sre``, ``posix`` 和 ``strop``.

* ``configure`` 脚本从 ``setup.cfg.in`` 构建 ``setup.cfg``.
   这需要有两个原因: 在子目录中构建工作, 以及获取配置的安装前缀.

* 将 ``setup.py`` 添加到源树的顶级目录. ``setup.py`` 是难题中最大的一块, 虽然不是最复杂的难题.
``setup.py`` 包含 ``BuildExt`` 类的子类, 并使用 ``detect_modules()`` 方法扩展它,
该方法可以确定何时可以编译模块, 并将它们添加到 'exts' 列表.


Unresolved Issues
=================

我们是否需要在不手动破解 Makefile 的情况下禁用3个硬连线模块?  [答案: No.]

Distutils 总是将模块编译为共享库. 我们如何支持将它们静态编译到生成的 Python 二进制文件中?

[答案: 使用 Distutils 构建 Python 二进制文件应该是可行的, 尽管还没有人实现它.
这应该在某一天完成, 但不是一个紧迫的优先事项, 因为搞乱顶层的 ``Makefile.pre.in`` 就足够了.]


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
