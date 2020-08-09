
PEP: 235
Title: Import on Case-Insensitive Platforms
Version: $Revision$
Last-Modified: $Date$
Author: Tim Peters <tim.peters@gmail.com>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created:
Python-Version: 2.1
Post-History: 16 February 2001


Note
====

这本质上是一个追溯性的 PEP: 在2.1版本发布过程中出现的问题为时尚晚,
利于在决定做什么之前征求广泛的意见, 并且在不延迟 Cygwin 和 MacOS X 端口的情况下也不能推迟到 2.2.


Motivation
==========

文件系统在不同平台上的区别在于它们是否保留文件名的大小写,
以及平台 C 库文件打开函数是否包含区分大小写的匹配项 ::

                         case-preserving     case-destroying
                     +-------------------+------------------+
    case-sensitive   | most Unix flavors | brrrrrrrrrr      |
                     +-------------------+------------------+
    case-insensitive | Windows           | some unfortunate |
                     | MacOSX HFS+       | network schemes  |
                     | Cygwin            |                  |
                     |                   | OpenVMS          |
                     +-------------------+------------------+

在左上角的框中, 如果你创建 "fiLe", 它存储为 "fiLe", 只有 ``open("fiLe")`` 将打开它
(``open("file")`` 既不会, 也不会该主题的其他14个变种）.

在右下角的框中, 如果你创建了 "fiLe", 那就不知道它存储的是什么 - 但很可能是
"FILE" - 而且 ``open("FilE")`` 的16个明显的变化中的任何一个打开它.

左下方的框是混合: 创建 "fiLe" 在平台目录中存储 "fiLe", 但是打开它时不必匹配大小写;
``open("FILe")`` 工作的16个明显变体中的任何一个.

没有改变! Python 将继续遵循 w.r.t 的平台约定. 创建文件时是否保留大小写,
以及 w.r.t. 是否 ``open()`` 需要区分大小写的匹配. 在实践中, 您应该始终编写代码,
就像匹配区分大小写一样, 否则您的程序将无法移植.

什么是建议要改变的 Python 的 "import" 语句语义, 并有 *only* 在左下框.


Current Lower-Left Semantics
============================

支持 MacOSX HFS+ 和 Cygwin 是 2.1 中的新功能, 因此没有任何改变.
Windows 特性正在发生变化. 以下是 Windows 上导入的当前规则:

1. 尽管文件系统不区分大小写, 但 Python 坚持使用区分大小写的匹配.
   但不是左上方框的工作方式: 如果你有两个文件, ``FiLe.py`` 和 ``sys.path`` 上的 ``file.py``, 并且 ::

       import file

   然后, 如果 Python 首先找到 ``FiLe.py``, 它会引发一个 ``NameError``. 它 *not* 继续找到 ``file.py``;
   实际上, 除了在 ``sys.path`` 上的第一个不区分大小写的匹配之外, 不可能导入任何内容,
   并且只有在大小写与第一个不区分大小写的匹配时才匹配

2. 一个丑陋的异常: 如果对 ``sys.path`` 的第一个不区分大小写的匹配是一个名字完全是大写的文件
    (``FILE.PY`` 或 ``FILE.PYC`` 或 ``FILE .PYO``), 然后导入默默地抓住, 无论在 import 语句中使用了什么示例混合.
	这显然是为了迎合真正适合右下方框架的悲惨的旧文件系统. 但是, 由于可能存在或可能不存在的原因, 此异常是 Windows 独有的.

3. 另一个异常: 如果环境变量 ``PYTHONCASEOK`` 存在, Python 默默地抓住任何类型的第一个不区分大小写的匹配.

因此, 这些 Windows 规则非常复杂, 既不符合 Unix 规则, 也不为本机文件系统提供自然的语义.
这使得他们很难向 Unix *or* Windows 用户解释. 尽管如此, 他们多年来一直工作得很好,
而且没有任何令人信服的理由可以改变它们.

然而, 那是在 MacOSX HFS+ 和 Cygwin 端口到来之前. 它们还具有保留大小写的不区分大小写的文件系统,
但是这些端口的人鄙视 Windows 规则. 事实上, 一个使 HFS+ 像 Unix 一样用于导入的补丁已经过了审稿人并进入了代码库,
顺便说一下, Cygwin 也像 Unix 一样
(但这符合 Cygwin 人员的无限批准, 所以他们肯定没有抱怨 - - 他们有自己的补丁等待这样做, 但那些评论者阻止).

在更高的层次上, 我们希望通过在 *all* 平台上遵循相同的规则来保持 Python 的一致性,
并保留大小写不区分大小写的文件系统.


Proposed Semantics
==================

建议左下方框的新语义:

A. 如果 ``PYTHONCASEOK`` 环境变量存在, 与之前相同:
    静默接受任何类型的第一个不区分大小写的匹配;
	如果找不到, 则引发 ``ImportError``.

B. 否则搜索 ``sys.path`` 进行第一个区分大小写的匹配;
    如果找不到, 则引发 ``ImportError``.

#B 与 Unix 上使用的规则相同, 因此这将提高跨平台的可移植性. 那很好.
#B 也是 Mac 和 Cygwin 人们想要的规则 (并且想要多次实现自己, 这是 PythonLand 中强大的参数).
它不会导致任何现有的非异常 Windows 导入失败,
因为任何现有的非异常 Windows 导入都会在路径中首先找到区分大小写的匹配 - 它仍然会.
一个异常的 Windows 导入目前会出现一个 ``NameError`` 或 ``ImportError``, 在后一种情况下它仍然会,
或者前一种情况将继续搜索, 并成功或爆发 ``ImportError``.

#A 需要迎合安装在 Windows 上的破坏案例的文件系统, 并且 *may* 也被那些迷恋 "natural" Windows 特性的人使用,
他们愿意设置环境变量来获取它. 我也不打算为 Unix 实现 #A, 但那只是因为我不清楚我如何有效地做到这一点
(我不会因为理论上的纯正而放慢 Unix 下的导入速度).

潜在的损害在于: #2 (匹配 ``ALLCAPS.PY``) 建议被删除. 破坏示例的文件系统是一个消失的品种, 对它们的支持是丑陋的.
我们已经支持 (并将继续支持) ``PYTHONCASEOK`` 为他们的利益, 但他们在2001年没有更多的攻击.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
