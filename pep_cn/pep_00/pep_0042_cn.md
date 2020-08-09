
PEP: 42
Title: Feature Requests
Version: $Revision$
Last-Modified: $Date$
Author: Jeremy Hylton <jeremy@alum.mit.edu>
Status: Rejected
Type: Process
Content-Type: text/x-rst
Created: 12-Sep-2000
Post-History:
Resolution: https://github.com/python/peps/pull/108#issuecomment-249603204


.. 注意:: 此 PEP 已被废弃.
   对于非常简单的请求, 所有新的功能请求都应该转到 `Python bug tracker`_,
   或者为其他所有请求转到 `python-ideas`_ 邮件列表. 本文档的其余部分仅用于历史目的


Introduction
============

此 PEP 包含可用于未来 Python 版本的功能请求列表
此处不应该包含大型功能请求, 但应该在单独的 PEP 中进行描述;
但是, 在创建自己的 PEP 之前, 可以在此处列出没有自己的 PEP 的大型功能请求
有关详细信息, 请参阅 PEP 0.

创建此 PEP 是为了允许我们关闭真正的功能请求的错误报告
标记为开放, 他们分散了真正的错误列表 (理想情况下应该少于一页).
标记为已关闭, 他们往往被遗忘
现在的过程是: 如果错误报告确实是一个功能请求, 请将功能请求添加到此 PEP;
将错误标记为 "feature request", "later" 和 "closed";
并在 bug 中添加注释, 说明情况就是这样 (明确提到 PEP).
将大型功能请求直接从错误数据库移动到单独的 PEP 也是可以接受的

该 PEP 应该真正分为四个不同的类别 (Laura Creighton的类别):

1. BDFL 拒绝这是一个坏主意, 别回来吧.

2. 如果有人写代码, BDFL 会输入.
  (或者无论如何, BDFL 会说 '改变这一点, 如果你出现代码, 我会把它改成'.)

   可能分为:

       a）BDFL 真的很想看到一些代码!

       b）BDFL 永远不会对此充满热情, 但是当它很容易时, 它会工作.

3. 如果你出现代码, BDFL 会发表声明. 它可能是 ICK.

4. 此 PEP 太模糊. 此 PEP 被拒绝, 但仅仅是基于含糊不清的理由
   如果您喜欢此增强功能, 请制作新的 PEP.


Core Language / Builtins
========================

* 解析器应该处理更深层次的嵌套解析树.

  后面代码将是失败 -- ``eval("["*50`` + ``"]"*50)`` -- 因为解析器对堆栈大小有硬编码限制.
  应该提高或删除此限制. 删除会很困难, 因为如果嵌套太深, 当前编译器可能会溢出 C 堆栈.

  http://www.python.org/sf/215555

* 非偶然的 IEEE-754 支持 (Infs, NaNs, settable traps等).
  大项目.

* Windows: 尝试创建 (甚至访问) 具有某些魔法名称的文件可能会导致 Windows 系统挂起或崩溃
  这实际上是操作系统中的一个错误, 但有些应用程序试图保护用户不受此限制
  当它发生时, 症状非常混乱.

  Hang using files named prn.txt, etc http://www.python.org/sf/481171

* eval 和自由变量:
  如果有一种方法可以在传递带有自由变量的代码对象时将自由变量的绑定传递给 eval,
  这可能很有用. http://www.python.org/sf/443866


Standard Library
================

* urllib 模块应该支持需要身份验证的代理, 有关信息, 请参阅 SourceForge 错误＃210619:

  http://www.python.org/sf/210619

* 应该修改 os.rename() 以处理不允许 rename() 跨文件系统边界操作的平台上的 EXDEV 错误,
  方法是复制文件并删除原始文件. Linux 是一个需要这种处理的系统.

  http://www.python.org/sf/212317

* 信号处理并不总是按预期工作. 例如, 如果 sys.stdin.readline() 被(返回)信号处理程序中断,
  则返回 "". 最好让它引发异常 (对应于 EINTR) 或重启.
  但是这些更改必须应用于所有可以阻止可中断 I/O 的地方. 所以这是一个很大的项目.

  http://www.python.org/sf/210599

* 扩展 Windows utime 来接受目录路径

  http://www.python.org/sf/214245

* 将 copy.py 扩展为模块和函数类型.

  http://www.python.org/sf/214553

* 更好地检查 ``marshal.load*()`` 的错误输入.

  http://www.python.org/sf/214754

* rfc822.py 应该比它解析的地址字段类型中的规范更宽松
  具体而言, 应该正确解析 "From: Amazon.com <deliver-news2@amazon.com>" 形式的无效地址.

  http://www.python.org/sf/210678

* 面对大型二进制文件上传, cgi.py 的 FieldStorage 类应该更加保守内存.

  http://www.python.org/sf/210674

  这里有两个问题: 首先, 因为 read_lines_to_outerboundary() 使用 readline(),
  所以可能会将大量数据读入内存来进行二进制文件上载.
  这可能应该查看该部分的 Content-Type 标头, 如果它是二进制类型, 则执行分块读取.

  第二个问题与 self.lines 属性有关, 该属性在 cgi.py 的修订版 1.56 中被删除 (另请参阅):

  http://www.python.org/sf/219806

* urllib 应该支持仅包含主机和端口的代理定义

  http://www.python.org/sf/210849

* 应该更新 urlparse 来符合 RFC 2396, 后者为路径的每个段定义可选参数.

  http://www.python.org/sf/210834

* pickle 和 cPickle 提出的例外情况目前不同;
  这些应该统一 (可能是异常应该在由两者导入的辅助模块中定义).
  [没有错误报告; 我只想到了这个.]

* 更多标准库例程应该支持 Unicode.
  例如, urllib.quote() 可以将 Unicode 字符串转换为 UTF-8,
  然后执行通常的 ％HH 转换. 但这不是唯一的!

  http://www.python.org/sf/216716

* 应该有一种方法可以说你不介意 ``str()`` 或 ``__str __()``返回一个 Unicode 字符串对象.
  或者提出了一个不同的函数 - “ustr()``. 或者其他的东西...

  http://sf.net/patch/?func=detailpatch&patch_id=101527&group_id=5470

* 从另一个线程中杀死一个线程, 或者可能发送信号. 或者可能引发异步异常.

  http://www.python.org/sf/221115

* 调试器 (pdb) 应该了解包.

  http://www.python.org/sf/210631

* Jim Fulton (人名) 建议如下:

  ::

    我想知道是否有一种新的临时文件将数据存储在内存中是个好主意, 除非:

    - 数据超出一定的规模, 或

    - 有人要求 fileno.

    然后 cgi 模块 (和其他应用程序) 可以统一的方式使用这个东西.

  http://www.python.org/sf/415692

* Jim Fulton 指出, binascii 的 b2a_base64() 函数有这样的情况,
  即不附加换行符或附加除换行符之外的其他内容.

  建议:

  - 添加一个可选参数, 给出要附加的分隔符字符串, 默认为 "\\n"

  - 可能特殊情况无作为分隔符字符串, 以避免添加填充字节???

  http://www.python.org/sf/415694

* pydoc 应该与 HTML 文档集成, 或者至少能够链接到它们.

  http://www.python.org/sf/405554

* Distutils 应该推断出 .c 和 .h 文件的依赖关系.

  http://www.python.org/sf/472881

* 面对多线程, asynchat 是错误的.

  http://www.python.org/sf/595217

* 如果更高级别的模块 (httplib, smtplib, nntplib等) 具有设置套接字超时的选项, 那将是很好的.

  http://www.python.org/sf/723287

* curses 库缺少两个重要的调用: newterm() 和 delscreen()

  http://www.python.org/sf/665572, http://bugs.debian.org/175590

* 如果内置 SSL 套接字类型可用于非阻塞 SSL I/O, 那将是很好的.
  目前使用 SSL 实现异步服务器的 Twisted 等软件包必须要求第三方软件包, 如 pyopenssl.

* reST 作为标准库模块

* 导入锁可以使用一些重新设计.

  http://www.python.org/sf/683658

* 一个更好的 API 来打开文本文件, 取代丑陋 (在某些人的眼中) "U" 模式标志
  有一个提议有一个新的内置类型文本文件 (文件名, 模式, 编码).
  (它不应该有一个 bufsize 参数吗?)

* 支持 Tkinter 的新的小部件或参数

* 对于在另一个类中定义的类, __name__ 应该是 "outer.inner", 并且 pickling 应该起作用.
  (GvR 不再确定这很容易, 甚至是正确的.)

  http://www.python.org/sf/633930

* 决定更明确的弃用政策 (尤其是模块) 并采取行动.

  https://mail.python.org/pipermail/python-dev/2002-April/023165.html

* 为类型模块的常见用途提供替代方案; Skip Montanaro 已经为这个想法发布了一个 proto-PEP:

  https://mail.python.org/pipermail/python-dev/2002-May/024346.html

* 对类型和字符串模块使用挂起的弃用.
  这需要为尚未覆盖的部分提供替代方案 (例如, string.whitespace 和 types.TracebackType).
  我们似乎无法对此达成共识.

* 懒惰跟踪元组?

  https://mail.python.org/pipermail/python-dev/2002-May/023926.html
  http://www.python.org/sf/558745

* 将'as'设为关键字. 它一直是伪关键词.
  (它已在 2.5 中弃用, 并将成为 2.6 中的关键字.)


C API wishes
============

* 添加 C API 函数以帮助构建嵌入式应用程序的 Windows 用户,
  其中 FILE \* 结构与编译解释器的 FILE \* 不匹配.

  http://www.python.org/sf/210821

  有关特定建议的信息, 请参阅此错误报告,
  该建议将允许 Borland C++ 构建器应用程序与使用 MSVC 的 python.dll 构建进行交互.


Tools
=====

* Python 可以使用 GUI 构建器.

  http://www.python.org/sf/210820


Building and Installing
=======================

* Modules/makesetup 应该确保它从各种 Setup 文件生成的 'config.c' 文件是有效的C.
  它当前接受的模块名称包含 Python 或 C 标识符中不允许的字符.

  http://www.python.org/sf/216326

* 从源代码构建不应该尝试覆盖 Include/graminit.h 和 Parser/graminit.c 文件,
  至少对于下载源代码而不是使用 Subversion 或 snapshots 的人来说.
  有些人发现这在异常构建环境中存在问题.

  http://www.python.org/sf/219221

* 配置脚本可能随着年龄的增长而变得有点苛刻, 可能无法很好地跟踪 autoconf 的最新功能.
  它应该被看到并且可能被清理干净.

  https://mail.python.org/pipermail/python-dev/2004-January/041790.html

* 使 Python 符合 FHS (文件系统层次结构标准)

  http://bugs.python.org/issue588756

.. _`Python bug tracker`: https://bugs.python.org
.. _`python-ideas`: https://mail.python.org/mailman/listinfo/python-ideas

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
