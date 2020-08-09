
PEP: 261
Title: Support for "wide" Unicode characters
Version: $Revision$
Last-Modified: $Date$
Author: Paul Prescod <paul@prescod.net>
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 27-Jun-2001
Python-Version: 2.2
Post-History: 27-Jun-2001


Abstract
========

Python 2.1 unicode 字符的序数最多只能达到 ``2**16 - 1``.
此范围对应于 Unicode 中称为基本多语言平面的范围.
现在 Unicode 中的字符存在于其他 "平面" 上.
Unicode 中最大的可寻址字符有序数 "17 * 2**16 - 1`` (``0x10ffff``).
为了便于阅读, 我们将调用此 TOPCHAR 并调用此范围内的字符 "宽字符".


Glossary
========

Character
   单独使用, 表示 Python Unicode 字符串的可寻址单元.

Code point
   代码点是 0 到 TOPCHAR 之间的整数. 如果您将 Unicode 想象为从整数到字符的映射,
   则每个整数都是一个代码点. 但是 0 和 TOPCHAR 之间没有映射到字符的整数也是代码点.
   有些人有一天会被用于字符. 有些保证永远不会用于字符.

Codec
   用于在物理编码 (例如, 在磁盘上或从网络进入) 到逻辑 Python 对象之间进行转换的一组函数.

Encoding
   用物理位和字节表示抽象字符的机制.
   编码允许我们在磁盘上存储 Unicode 字符,
   并以与其他 Unicode 软件兼容的方式通过网络传输它们.

Surrogate pair
   两个表示单个逻辑字符的物理字符.
   用两个16位代码点表示32位代码点的约定的一部分.

Unicode string
   表示具有 "字符串语义" 的代码点序列的 Python 类型
   (例如, 大小写转换, 正则表达式兼容性等) 使用 ``unicode()`` 函数构造.


Proposed Solution
=================

一种解决方案是仅将最大序数增加到更大的值. 不幸的是, 这个想法的唯一直接实现是每个字符使用4个字节.
这会使大多数 Unicode 字符串的大小加倍. 为了避免将这个成本强加给每个用户,
Python 2.2 将允许4字节实现作为构建时选项. 用户可以选择是否关心宽字符或更喜欢保留内存.

4字节选项称为 "wide ``Py_UNICODE``".
2字节选项称为 "narrow ``Py_UNICODE``".

大多数事物在广阔和狭隘的世界中都会表现得相同.

* ``unichr(i)`` for 0 <= i < ``2**16`` (``0x10000``) 总是返回一个长度为一个字符串.

* ``unichr(i)`` for ``2**16`` <= i <= TOPCHAR 将在宽阔的 Python 构建上返回长度为1的字符串.
   在狭窄版本上, 它会引发 ``ValueError``.

  **ISSUE**

     Python 目前允许 ``\U`` 文字不能表示为单个 Python 字符.
	 它生成两个称为 "代理对" 的 Python 字符.
	 在未来的 Python 构建中是否应该禁止这种情况?

  **Pro:**

     Python 已经为大型 unicode 文字字符转义序列构建了一个代理对.
	 这基本上被设计为即使在狭窄的 Python 构建中构造 "宽字符" 的简单方法.
	 考虑到 Unicode-literal 语法基本上是一种调用 unicode-escape 编解码器的简短方法,
	 这也有点合乎逻辑.

  **Con:**

     代理可以通过这种方式轻松创建, 但用户仍然需要注意切片, 索引, 打印等.
	 因此, 有些人认为 Unicode 文字不应该支持代理.

  **ISSUE**

     Python 应该允许构造与 Unicode 代码点不对应的字符吗?
	 未分配的 Unicode 代码点显然应该是合法的 (因为它们可以随时分配).
	 但是保证 TOPCHAR 以上的代码点永远不会被 Unicode 使用.
	 我们是否应该允许访问它们?

  **Pro:**

     如果 Python 用户认为他们知道他们正在做什么,
	 为什么我们应该尝试阻止他们违反 Unicode 规范呢?
	 毕竟, 我们不会阻止8位字符串包含非 ASCII 字符.

  **Con:**

     编解码器和其他使用 Unicode 的代码必须注意 Unicode 规范不允许的这些字符.

* ``ord()`` 总是与 ``unichr()`` 相反

* sys 模块中有一个整数值, 用于描述当前解释器上 Unicode 字符串中字符的最大序数.
  ``sys.maxunicode`` 在 Python 的窄版本和宽版本的 TOPCHAR 上是 ``2**16-1`` (``0xffff``).

  **ISSUE:**

     是否应该有不同的常量来访问 TOPCHAR 和 ``unichr`` 域的真正上限 (如果它们不同)?
	 还有一个建议是 ``sys.unicodewidth``, 它可以取值 ``'wide'`` 和 ``'narrow'``.

* 每个 Python Unicode 字符都代表一个 Unicode 代码点
  (即 Python Unicode Character = Abstract Unicode character）.

* 编解码器将升级为支持 "宽字符" (直接在 UCS-4 中表示, 以及 UTF-8 和 UTF-16 中的可变长度序列).
  这是尚未完成的实施的主要部分.

* Unicode 世界中有一种约定, 用于根据两个16位代码点对32位代码点进行编码.
  这些被称为 "代理对". Python 的编解码器将采用这种约定, 并将32位代码点编码为窄版本 Python 上的代理对.

  **ISSUE**

     是否有办法告诉编解码器不要生成代理, 而是将宽字符视为错误?

  **Pro:**

     我可能想编写仅适用于固定宽度字符的代码, 而不必担心代理.

  **Con:**

     没有关于如何与编解码器通信的明确建议.

* 构造使用代码点 "保留代理" 保留不当的字符串没有任何限制. 这些被称为 "isolated surrogates".
  编解码器应该禁止从文件中读取它们, 但是你可以使用字符串文字或 ``unichr()`` 来构造它们.


Implementation
==============

有一个新的定义::

    #define Py_UNICODE_SIZE 2

要测试是否正在使用 UCS2 或 UCS4, 应该使用派生宏 ``Py_UNICODE_WIDE``,
这是在使用 UCS-4 时定义的.

有一个新的配置选项:

=====================  ============================================
--enable-unicode=ucs2  configures a narrow ``Py_UNICODE``, and uses
                       wchar_t if it fits
--enable-unicode=ucs4  configures a wide `Py_UNICODE``, and uses
                       wchar_t if it fits
--enable-unicode       same as "=ucs2"
--disable-unicode      entirely remove the Unicode functionality.
=====================  ============================================

还建议有一天 ``--enable-unicode`` 将默认为你的平台 ``wchar_t`` 的宽度.

基于对宽字符的请求很少这一事实, Windows 构建将会缩短一段时间,
这些请求主要来自能够购买自己的 Python 的硬核程序员,
而 Windows 本身则强烈偏向于16位字符.


Notes
=====

此 PEP 并不意味着使用 Unicode 的人需要对磁盘上的文件使用4字节编码或通过网络发送.
它只允许他们这样做. 例如, ASCII 仍然是合法的 (7位) Unicode 编码.

有人提出应该有一个模块来处理程序员的窄 Python 构建中的代理.
如果有人想要实现它, 那将是另一个 PEP. 它还可以与允许其他类型的基于字符,
字和行的索引的功能相结合.


Rejected Suggestions
====================

或多或少的现状

   我们可以正式地说 Python 字符是16位的, 并且要求程序员通过
   组合代理对来在他们的应用程序逻辑中实现宽字符. 这是一个沉重的负担,
   因为如果完全用 Python 编码, 模拟32位字符的效率可能非常低.
   另外, 这些抽象的伪字符串作为正则表达式引擎的输入是不合法的.

"Space-efficient Unicode" 类型

   另一类解决方案是在内部使用一些有效的存储, 但向程序员提供宽字符的抽象.
   任何这些都需要比接受的解决方案更复杂的实现. 例如, 考虑对正则表达式引擎的影响.
   理论上, 我们可以在不破坏 Python 代码的情况下转向此实现.
   未来的 Python 可以在狭窄的 Python 上 "emulate" 宽泛的 Python 语义.
   Guido 现在不愿意接受实施.

Two types

   我们可以在16位类型的同时引入32位 Unicode 类型.
   有很多代码期望只有一种 Unicode 类型.

该 PEP 代表了最省力的解决方案. 在接下来的几年中, 32位 Unicode 字符将变得更加普遍,
这可能使我们相信我们需要更复杂的解决方案, 或者 (另一方面) 说服我们只需强制使用
宽 Unicode 字符就是一个合适的解决方案. 现在, 表格上的两个选项什么也不做或者这样做.


References
==========

Unicode Glossary: http://www.unicode.org/glossary/


Copyright
=========

This document has been placed in the public domain.


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
