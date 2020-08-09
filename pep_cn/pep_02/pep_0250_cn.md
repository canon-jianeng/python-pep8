
PEP: 250
Title: Using site-packages on Windows
Version: $Revision$
Last-Modified: $Date$
Author: p.f.moore@gmail.com (Paul Moore)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 30-Mar-2001
Python-Version: 2.2
Post-History: 30-Mar-2001


Abstract
========

标准的 Python 发行版包含一个目录 ``Lib/site-packages``,
它在 Unix 平台上用于保存本地安装的模块和包.
随 Python 一起发布的 ``site.py`` 模块包括支持在 site-packages 目录中定位其他模块.

此 PEP 建议以类似的方式在 Windows 平台上使用 site-packages 目录.


Motivation
==========

在 Windows 平台上, ``sys.path`` 的默认设置不包括适合用户安装本地开发模块的目录.
"预期" 位置似乎是包含 Python 可执行文件本身的目录.
这也是 distutils (和 distutils 生成的安装程序) 安装包的位置.
将本地开发的代码包含在与安装的可执行文件相同的目录中并不是一种好习惯.

显然, 用户可以在本地修改的 ``site.py`` 或适当的 ``sitecustomize.py``,
甚至是 ``.pth`` 文件中操作 ``sys.path``. 但是, 这些文件应该有一个标准位置,
而不是依赖于每个站点都必须设置自己的策略.

此外, 随着 distutils 作为分发模块的手段变得越来越普遍,
对分布式模块的标准安装位置的需求将变得更加普遍.
现在定义这样一个标准会更好, 而不是在以后存在需要重建的更多基于 distutils 的软件包时.

值得注意的是, 在 Python 2.1 之前, site-packages 目录未包含在 Macintosh 平台的 ``sys.path`` 中.
这已在 2.1 中更改, Macintosh 现在包含 ``sys.path``, 使 Windows 成为唯一没有特定于站点的模块目录的主要平台.


Implementation
==============

此功能的实现相当简单. 所需要的只是更改为 ``site.py``,
来更改部分设置 sitedirs. Python 2.1 版本有::

    if os.sep == '/':
        sitedirs = [makepath(prefix,
                            "lib",
                            "python" + sys.version[:3],
                            "site-packages"),
                    makepath(prefix, "lib", "site-python")]
    elif os.sep == ':':
        sitedirs = [makepath(prefix, "lib", "site-packages")]
    else:
        sitedirs = [prefix]

一个合适的改变是简单地用最后4行代替::

    else:
        sitedirs == [prefix, makepath(prefix, "lib", "site-packages")]

为了反映政策的这种变化, 还需要对 distutils 进行更改. Sourceforge 上提供了一个补丁,
补丁 ID 445744, 用于实现此更改. 请注意, 补丁检查 Python 版本,
并且仅从 2.2 开始调用 Python 版本的新行为. 这是为了确保 distutils 与早期版本的 Python 保持兼容.

最后, 实现 ``bdist_wininst`` 命令使用的 Windows 安装程序的可执行代码将需要更改来使用新位置.
目前有一个单独的补丁, 由 Thomas Heller 维护.


Notes
=====

- 此更改不会排除使用当前位置的包 - 更改只会将目录添加到 ``sys.path``, 它不会删除任何内容.

- 当前位置 (``sys.prefix``) 和新目录 (site-packages) 都包含在 sitedirs 中,
   因此 ``.pth`` 文件将在任一位置被识别.

- 此提案向 sitedirs 添加了一个额外的 site-packages 目录. 在 Unix 平台上,
  添加了两个目录, 一个用于与版本无关的文件 (Python 代码),
  另一个用于依赖于版本的代码 (C 扩展). 这在 Unix 上是必要的,
  因为 sitedirs 默认包含一个公共 (跨 Python 版本) 包位置, 在 ``/usr/local`` 中.
  由于 Windows 上没有这样的公共位置, 因此也不需要具有两个单独的包目录.

- 如果用户希望将 DLL 保存在 Windows 上的单个位置, 而不是将它们保存在包目录中,
  则 Python 安装目录的 DLLs 子目录已经可用于此目的. 不必为 DLL 添加额外的目录.


Open Issues
===========

- 来自 Unix 用户的评论表明, Unix 平台上的当前设置可能存在问题.
  该 PEP 不是涉及跨平台问题, 而是专门将自己限制在 Windows 平台上,
  其他平台的更改将在其他 PEP 中得到解决。.

- 嵌入 Python 的应用程序可能存在问题. 据作者所知,
  这种变化应该没有问题. 嵌入 Python 的用户没有注释 (支持或其他).


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
