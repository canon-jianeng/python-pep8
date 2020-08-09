
PEP: 100
Title: Python Unicode Integration
Version: $Revision$
Last-Modified: $Date$
Author: mal@lemburg.com (Marc-AndrÃ© Lemburg)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 10-Mar-2000
Python-Version: 2.0
Post-History:


Historical Note
===============

本文档最初由 Marc-Andre 在 PEP 前期编写,
最初在 Python 发行版中以 Misc/unicode.txt 的形式发布, 并包含在 Python 2.1 中.
该位置的提案的最新修订版本标记为 1.7 版 (CVS 修订版 3.10).
由于该文件在 PEP 后时代显然符合信息化 PEP 的目的,
因此已将其移至此处并重新格式化以符合 PEP 指南.
未来将对本文档进行修订, 而 Misc/unicode.txt 将包含指向此 PEP 的指针.

-Barry Warsaw, PEP editor


Introduction
============

这个提议的想法是以一种尽可能简单地使用 Unicode 字符串的方式,
向 Python 添加本机 Unicode 3.0支持, 而不会在此过程中引入太多陷阱.

由于这个目标不容易实现 -- 字符串是 Python 中最基本的对象之一 --
我们希望这个提议经过一些重要的改进.

请注意, 由于 Unicode-Python 集成的许多不同方面, 此提案的当前版本仍然有点未分类.

本文档的最新版本始终可以在以下位置获得:
http://starship.python.net/~lemburg/unicode-proposal.txt

旧版本可用作:
http://starship.python.net/~lemburg/unicode-proposal-X.X.txt

[ed. 注意: 应该对本 PEP 文件进行新的修订, 而版本 1.7 之前的历史记录应该是
从 MAL 的网址或 Misc/unicode.txt 中检索]


Conventions
===========

- In examples we use u = Unicode object and s = Python string

- 'XXX' markings indicate points of discussion (PODs)


General Remarks
===============

- Unicode 编码名称在输出时应该为小写, 在输入时不区分大小写
  (所有 API 都将编码名称作为输入转换为小写).

- 编码名称应该遵循 Unicode Consortium 使用的名称约定: 空格转换为连字符,
  例如: 'utf 16' 写成 'utf-16'.

- 编解码器模块应该使用相同的名称, 但使用连字符转换为下划线,
  例如 ``utf_8``, ``utf_16``, ``iso_8859_1``.


Unicode Default Encoding
========================

Unicode 实现必须对传递给它的8位字符串的编码进行一些假设,
以及在没有给出特定编码时将 Unicode 转换为字符串的默认编码.
此编码在本文中称为 <default encoding>.

为此, 实现维护一个可以在 site.py Python 启动脚本中设置的全局, 无法进行后续更改
可以使用两个 sys 模块 API 设置和查询 <default encoding>:

``sys.setdefaultencoding(encoding)``
    设置 Unicode 实现使用的 <default encoding>.
    encoding 必须是 Python 安装支持的编码, 否则会引发 LookupError.

    注意: 此 API 仅在 site.py 中可用! 使用后, 它将通过 site.py 从 sys 模块中删除.

``sys.getdefaultencoding()``
    返回当前的 <default encoding>.

如果没有另外定义或设置, <default encoding> 默认为 'ascii'.
此编码也是 Python 的启动默认值 (在执行 site.py 之前有效).

请注意, 默认的 site.py 启动模块包含已禁用的可选代码,
可以根据当前语言环境定义的编码设置 <default encoding>.
语言环境模块用于从 OS 环境定义的语言环境默认设置中提取编码 (请参阅 locale.py).
如果无法确定编码, 未知或不支持, 则代码默认将 <default encoding> 设置为 "ascii".
想要启用此代码, 请编辑 site.py 文件或将相应的代码放入 Python 安装的 sitecustomize.py 模块中.


Unicode Constructors
====================

Python 应该为 Unicode 字符串提供内置构造函数
可以通过 ``__builtins__`` ::

    u = unicode(encoded_string[,encoding=<default encoding>][,errors="strict"])

    u = u'<unicode-escape encoded Python string>'

    u = ur'<raw-unicode-escape encoded Python string>'

将 'unicode-escape' 编码定义为:

- 所有非转义字符都表示为 Unicode 序数 (例如, 'a' -> U+0061).

- 所有现有的已定义的 Python 转义序列都被解释为 Unicode 序列;
  请注意, ``\ xXXXX`` 可以表示所有 Unicode 序数,
  而 ``\OOO`` (八进制) 可以表示直到 U+01FF 的 Unicode 序号.

- 一个新的转义序列, ``\uXXXX`` 代表 U+XXXX;
  在 ``\ u`` 之后, 少于4位的语法错误.

有关错误的可能值的说明, 请参阅下面的编解码器部分.

Examples::

    u'abc'          -> U+0061 U+0062 U+0063
    u'\u1234'       -> U+1234
    u'abc\u1234\n'  -> U+0061 U+0062 U+0063 U+1234 U+005c

'raw-unicode-escape' 编码定义如下:

- 当且仅当前导反斜杠的数量为奇数时, ``\uXXXX`` 序列表示 U+XXXX Unicode 字符

- 所有其他字符都表示为 Unicode 序数 (例如, 'b' -> U+0062)

请注意, 您应该在源文件的前几行注释行 (例如, '#source file encoding: latin-1')
中的一个注释行中提供用于将程序编写为编译指示行的编码的一些提示.
如果您只使用7位 ASCII, 那么一切都很好, 并且不需要这样的通知,
但如果您包含未在 ASCII 中定义的 Latin-1 字符, 那么包括提示可能是值得的
因为其他国家的人们希望能够也读取你的源字符串


Unicode Type Object
===================

Unicode 对象应具有类型名称为 "unicode" 的 UnicodeType 类型, 可以通过标准类型模块使用.


Unicode Output
==============

Unicode 对象有一个方法 .encode ([encoding=<default encoding>]),
它使用给定的方案返回编码 Unicode 字符串的 Python 字符串 (参见编解码器).

::

    print u := print u.encode()   # using the <default encoding>

    str(u)  := u.encode()         # using the <default encoding>

    repr(u) := "u%s" % repr(u.encode('unicode-escape'))

另请参阅内部参数分析和缓冲区接口,
来获取有关如何使用 C 语言编写的其他 API 处理 Unicode 对象的详细信息.


Unicode Ordinals
================

由于 Unicode 3.0 具有32位序数字符集, 因此实现应该提供32位可识别序数转换 API ::

    ord(u[:1]) (this is the standard ord() extended to work with Unicode
                objects)
      --> Unicode ordinal number (32-bit)

    unichr(i)
        --> Unicode object for character i (provided it is 32-bit);
            ValueError otherwise
        
两个 API 都应该进入 ``__builtins__``, 就像它们的字符串对应 ``ord()`` 和 ``chr()`` 一样.

请注意, Unicode 为私有编码提供了空间.
使用这些可能会导致不同机器上的输出表示不同.
此问题不是 Python 或 Unicode 问题, 而是机器设置和维护问题.


Comparison & Hash Value
=======================

在将这些其他对象强制转换为 Unicode 之后, Unicode 对象应与其他对象进行比较.
对于字符串, 这意味着它们使用 <default encoding> 解释为 Unicode 字符串.

Unicode 对象应返回与其 ASCII 等效字符串相同的哈希值.
保留非 ASCII 值的 Unicode 字符串不保证返回与默认编码的等效字符串表示形式相同的散列值.

当使用 ``cmp()`` (或 ``PyObject_Compare()``) 进行比较时,
实现应该屏蔽转换期间引发的 ``TypeErrors`` 来保持与字符串行为同步.
在将字符串强制转换为 Unicode 期间引发的所有其他错误 (例如, "ValueErrors") 不应该被屏蔽并传递给用户.

在容量测试中 (u'abc'中的 'a' 和 'abc' 中的 u'a') 在应用测试之前,
双方都应该被强制转换为 Unicode. 在强制期间发生的错误 (例如, u'abc'中的 None) 不应该被掩盖.


Coercion
========

使用 Python 字符串和 Unicode 对象来形成新对象应该始终强制使用更精确的格式, 即 Unicode 对象.

::

    u + s := u + unicode(s)

    s + u := unicode(s) + u

所有字符串方法都应该将调用委托给等效的 Unicode 对象方法调用,
方法是将所有涉及的字符串转换为 Unicode,
然后将参数应用于同名的 Unicode 方法, 例如:

::

    string.join((s,u),sep) := (s + sep) + u

    sep.join((s,u)) := (s + sep) + u

有关 ％-formatting w/r 到 Unicode 对象的讨论, 请参阅格式化标记.


Exceptions
==========

``UnicodeError``在例外模块中定义为``ValueError``的子类
它可以通过``PyExc_UnicodeError``在 C 级获得
与 Unicode 编码/解码相关的所有异常都应该是 ``UnicodeError`` 的子类.


Codecs (Coder/Decoders) Lookup
==============================

编解码器 (参见编解码器接口定义) 搜索注册表应该由模块 "编解码器" 实现 ::

    codecs.register(search_function)

搜索函数应该采用一个参数, 所有小写字母的编码名称以及连字符和空格转换为下划线,
并返回一个函数元组 (编码器, 解码器, stream_reader, stream_writer) 以下参数:

编码器和解码器
    这些函数或方法必须与 Codec 实例的 ``.encode`` 或 ``.decode`` 方法具有相同的接口
    (参见 Codec 接口). 期望函数或方法在无状态模式下工作.

stream_reader和stream_writer
    这些必须是具有以下功能的工厂函数
    接口 ::

        factory(stream,errors='strict')
    
    工厂函数必须返回提供由 ``StreamWriter`` 或 ``StreamReader`` resp 定义的接口的对象
    (参见编解码器接口). 流编解码器可以维护状态.

    可能的错误值在下面的 "编解码器" 部分中定义.

如果搜索功能无法找到给定的编码, 则应该返回 None.

别名对编码的支持留给搜索功能来实现.

由于性能原因, 编解码器模块将维护编码缓存.
首先在缓存中查找编码. 如果未找到, 则扫描已注册的搜索功能列表.
如果未找到编解码器元组, 则引发 LookupError.
否则, 编解码器元组存储在缓存中并返回给调用者.

想要查询 Codec 实例, 应该使用以下 API ::

    codecs.lookup(encoding)

这将返回找到的编解码器元组或引发 ``LookupError``.


Standard Codecs
===============

标准编解码器应该位于标准 Python 代码库中的 encodings/ 包目录中.
该目录的 ``__init __.py`` 文件应该包括与 Codec Lookup 兼容的搜索功能,
该功能实现基于惰性模块的编解码器查找.

Python 应该为最相关的编码提供一些标准编解码器, 例如:

::

    'utf-8':              8位可变长度编码
    'utf-16':             16位可变长度编码 (little/big endian)
    'utf-16-le':          utf-16 但明确是 little endian
    'utf-16-be':          utf-16 但明确是 big endian
    'ascii':              7位 ASCII 内码表
    'iso-8859-1':         ISO 8859-1 (Latin 1) 内码表
    'unicode-escape':     有关定义, 请参阅 Unicode 构造函数
    'raw-unicode-escape': 有关定义, 请参阅 Unicode 构造函数
    'native':             Python 使用的内部格式转储

默认情况下也应该提供常见别名, 例如:
'latin-1' 代表 'iso-8859-1'.

注意:
'utf-16' 应该通过使用并要求文件输入或输出的字节顺序标记 (BOM) 来实现.

所有其他编码 (例如, 支持亚洲脚本的 CJK 编码) 应该在单独的包中实现,
这些包不包含在核心 Python 发行版中, 也不是此提议的一部分.


Codecs Interface Definition
===========================

应该在模块 "codecs" 中定义以下基类.
它们不仅提供供编码模块实现者使用的模板, 还提供 Unicode 实现所期望的接口.

请注意, 此处定义的编解码器接口非常适合更广泛的应用
Unicode 实现期望输入的 ``.encode()`` 和 ``.write()`` 上的 Unicode 对象
和 ``.decode()`` 的输入上的字符缓冲兼容对象 ``.encode()``
和 ``.read()`` 的输出应该是一个 Python 字符串, ``.decode()`` 必须返回一个 Unicode 对象.

首先, 我们有无状态编码器或解码器. 它们不像流编解码器 (见下文) 那样在块中工作,
因为所有组件都应该在内存中可用.

::

    class Codec:

        """Defines the interface for stateless encoders/decoders.

           The .encode()/.decode() methods may implement different
           error handling schemes by providing the errors argument.
           These string values are defined:

             'strict'  - raise an error (or a subclass)
             'ignore'  - ignore the character and continue with the next
             'replace' - replace with a suitable replacement character;
                         Python will use the official U+FFFD
                         REPLACEMENT CHARACTER for the builtin Unicode
                         codecs.
        """

        def encode(self,input,errors='strict'):

            """Encodes the object input and returns a tuple (output
               object, length consumed).

               errors defines the error handling to apply.  It
               defaults to 'strict' handling.

               The method may not store state in the Codec instance.
               Use StreamCodec for codecs which have to keep state in
               order to make encoding/decoding efficient.
            """

        def decode(self,input,errors='strict'):

            """Decodes the object input and returns a tuple (output
               object, length consumed).

               input must be an object which provides the
               bf_getreadbuf buffer slot.  Python strings, buffer
               objects and memory mapped files are examples of objects
               providing this slot.

               errors defines the error handling to apply.  It
               defaults to 'strict' handling.

               The method may not store state in the Codec instance.
               Use StreamCodec for codecs which have to keep state in
               order to make encoding/decoding efficient.

            """

``StreamWriter`` 和 ``StreamReader`` 定义了在流上工作的有状态编码器或解码器的接口
这些允许以块的形式处理数据以有效地使用存储器.
如果你在内存中有大字符串, 你可能想用 ``cStringIO`` 对象包装它们,
然后在它们上使用这些编解码器也可以进行块处理, 例如, 向用户提供进度信息.

::

    class StreamWriter(Codec):

        def __init__(self,stream,errors='strict'):

            """Creates a StreamWriter instance.

               stream must be a file-like object open for writing
               (binary) data.

               The StreamWriter may implement different error handling
               schemes by providing the errors keyword argument.
               These parameters are defined:

                 'strict' - raise a ValueError (or a subclass)
                 'ignore' - ignore the character and continue with the next
                 'replace'- replace with a suitable replacement character
            """
            self.stream = stream
            self.errors = errors

        def write(self,object):

            """Writes the object's contents encoded to self.stream.
            """
            data, consumed = self.encode(object,self.errors)
            self.stream.write(data)

        def writelines(self, list):

            """Writes the concatenated list of strings to the stream
               using .write().
            """
            self.write(''.join(list))

        def reset(self):

            """Flushes and resets the codec buffers used for keeping state.

               Calling this method should ensure that the data on the
               output is put into a clean state, that allows appending
               of new fresh data without having to rescan the whole
               stream to recover state.
            """
            pass

        def __getattr__(self,name, getattr=getattr):

            """Inherit all other methods from the underlying stream.
            """
            return getattr(self.stream,name)


    class StreamReader(Codec):

        def __init__(self,stream,errors='strict'):

            """Creates a StreamReader instance.

               stream must be a file-like object open for reading
               (binary) data.

               The StreamReader may implement different error handling
               schemes by providing the errors keyword argument.
               These parameters are defined:

                 'strict' - raise a ValueError (or a subclass)
                 'ignore' - ignore the character and continue with the next
                 'replace'- replace with a suitable replacement character;
            """
            self.stream = stream
            self.errors = errors

        def read(self,size=-1):

            """Decodes data from the stream self.stream and returns the
               resulting object.

               size indicates the approximate maximum number of bytes
               to read from the stream for decoding purposes.  The
               decoder can modify this setting as appropriate.  The
               default value -1 indicates to read and decode as much
               as possible.  size is intended to prevent having to
               decode huge files in one step.

               The method should use a greedy read strategy meaning
               that it should read as much data as is allowed within
               the definition of the encoding and the given size, e.g.
               if optional encoding endings or state markers are
               available on the stream, these should be read too.
            """
            # Unsliced reading:
            if size < 0:
                return self.decode(self.stream.read())[0]

            # Sliced reading:
            read = self.stream.read
            decode = self.decode
            data = read(size)
            i = 0
            while 1:
                try:
                    object, decodedbytes = decode(data)
                except ValueError,why:
                    # This method is slow but should work under pretty
                    # much all conditions; at most 10 tries are made
                    i = i + 1
                    newdata = read(1)
                    if not newdata or i > 10:
                        raise
                    data = data + newdata
                else:
                    return object

        def readline(self, size=None):

            """Read one line from the input stream and return the
               decoded data.

               Note: Unlike the .readlines() method, this method
               inherits the line breaking knowledge from the
               underlying stream's .readline() method -- there is
               currently no support for line breaking using the codec
               decoder due to lack of line buffering.  Subclasses
               should however, if possible, try to implement this
               method using their own knowledge of line breaking.

               size, if given, is passed as size argument to the
               stream's .readline() method.
            """
            if size is None:
                line = self.stream.readline()
            else:
                line = self.stream.readline(size)
            return self.decode(line)[0]

        def readlines(self, sizehint=0):

            """Read all lines available on the input stream
               and return them as list of lines.

               Line breaks are implemented using the codec's decoder
               method and are included in the list entries.

               sizehint, if given, is passed as size argument to the
               stream's .read() method.
            """
            if sizehint is None:
                data = self.stream.read()
            else:
                data = self.stream.read(sizehint)
            return self.decode(data)[0].splitlines(1)

        def reset(self):

            """Resets the codec buffers used for keeping state.

               Note that no stream repositioning should take place.
               This method is primarily intended to be able to recover
               from decoding errors.

            """
            pass

        def __getattr__(self,name, getattr=getattr):

            """ Inherit all other methods from the underlying stream.
            """
            return getattr(self.stream,name)

流编解码器实现者可以自由地将 ``StreamWriter`` 和 ``StreamReader`` 接口组合成一个类.
即使将所有这些与 Codec 类相结合也应该是可能的.

实现者可以自由添加其他方法来增强编解码器功能或提供工作所需的额外状态信息.
但是, 内部编解码器实现仅使用上述接口.

Unicode 实现不要求使用这些基类, 只有接口必须匹配; 这允许将编解码器编写为扩展类型.

作为指导原则, 应该使用单独 (共享) 扩展模块中的静态 C 数据来实现大型映射表.
这样多个进程可以共享相同的数据.

应该提供一种将 Unicode 映射文件自动转换为映射模块的工具,
来简化对其他映射的支持 (请参阅参考资料)


Whitespace
==========

``.split（）`` 方法必须知道 Unicode 中的空白符是什么.


Case Conversion
===============

由于存在许多不同的条件, 因此使用 Unicode 数据进行大小写转换非常复杂. 看到

    http://www.unicode.org/unicode/reports/tr13/

有关实施案例转换的一些准则.

对于 Python, 我们应该只实现 Unicode 中包含的 1-1 转换. 依赖于区域设置和其他特殊情况的转换
(请参阅 Unicode 标准文件 SpecialCasing.txt) 应该留给用户的例程, 而不是进入核心解释器.

方法 ``.capitalize()`` 和 ``.iscapitalized()`` 应该尽可能地遵循上述技术报告中定义的案例映射算法.


Line Breaks
===========

应该对具有 B 属性的所有 Unicode 字符以及 CRLF, CR, LF (按此顺序解释)
和标准定义的其他特殊行分隔符进行换行.

Unicode 类型应该提供一个 ``.splitlines()`` 方法, 它根据上面的规范返回一个行列表.
请参阅 Unicode 方法.


Unicode Character Properties
============================

单独的模块 "unicodedata" 应该为标准的 UnicodeData.txt 文件中定义的所有 Unicode 字符属性提供紧凑的接口.

除此之外, 这些属性提供了识别数字，数字，空格，空白符等的方法.

由于此模块必须提供对所有 Unicode 字符的访问, 因此最终必须包含来自 UnicodeData.txt 的数据,
该数据占用大约 600 kB. 因此, 数据应该存储在静态 C 数据中.
这使得编译成为底层操作系统可以在进程之间共享的共享模块 (与普通的 Python 代码模块不同).

应该有一个标准的 Python 接口来访问这些信息, 以便其他实现者可以插入他们自己可能的增强版本,
例如, 那些在运行中解压缩数据的人.


Private Code Point Areas
========================

对这些的支持留给用户登陆编解码器, 而不是明确地集成到核心中.
请注意, 由于实现了内部格式, 只有 ``\uE000`` 和 ``\uF8FF`` 之间的区域可用于私有编码.


Internal Format
===============

Unicode 对象的内部格式应该使用特定于 Python 的固定格式 <PythonUnicode>,
实现为 "unsigned short" (或另一种具有16位的无符号数字类型).
字节顺序取决于平台.

此格式将保存相应 Unicode 序数的 UTF-16 编码.
Python Unicode 实现将解决这些值, 就好像它们是 UCS-2 值一样.
所有当前定义的 Unicode 字符点的 UCS-2 和 UTF-16 都相同.
没有代理的 UTF-16 提供对大约 64k 字符的访问, 并覆盖 Unicode 的基本多语言平面 (BMP) 中的所有字符.

编解码器有责任确保传递给 Unicode 对象构造函数的数据遵循这一假设.
构造函数不检查数据是否符合 Unicode 或使用代理.

未来的实现可以将32位限制扩展到所有 UTF-16 可寻址字符的整个集合 (大约1M个字符).

Unicode API 应该提供从 <PythonUnicode> 到编译器的 wchar_t 的接口例程,
它可以是16位或32位, 具体取决于所使用的编译器或 libc 或平台.

Unicode 对象应该有一个指向缓存的 Python 字符串对象 <defenc> 的指针,
该对象使用 <default encoding> 保存对象的值.
这是性能和内部解析 (请参阅内部参数解析) 原因所必需的.
在对象上发出对 <default encoding> 的第一个转换请求时, 将填充缓冲区.

由于 Python 标识符仅定义为 ASCII, 因此不需要驻留 (暂时).

``codecs.BOM`` 应该返回内部使用格式的字节顺序标记 (BOM).
编解码器模块应该提供以下附加常量以方便和引用
(``codecs.BOM`` 将是 ``BOM_BE`` 或 ``BOM_LE``, 具体取决于平台) ::

    BOM_BE: '\376\377'
      (对应到 Unicode U + 0000FEFF 在 UTF-16 上的 big endian 平台
       == ZERO WIDTH NO-BREAK SPACE)

    BOM_LE: '\377\376'
      (对应到 Unicode U+0000FFFE 在 UTF-16 上的 little endian 平台
       == defined as being an illegal Unicode character)

    BOM4_BE: '\000\000\376\377'
      (对应于 UCS-4 中的 Unicode U + 0000FEFF)

    BOM4_LE: '\377\376\000\000'
      (对应于 UCS-4 中的 Unicode U + 0000FFFE)

请注意, Unicode 将  big endian 字节顺序视为 "正确".
交换的指令被视为 "错误" 格式的指示符, 因此是非法字符定义.

配置脚本应该帮助决定 Python 是否可以使用本机 ``wchar_t`` 类型 (它必须是16位无符号类型).


Buffer Interface
================

使用 <defenc> Python 字符串对象作为 ``bf_getcharbuf`` 的基础
和 ``bf_getreadbuf`` 的内部缓冲区来实现缓冲区接口.
如果请求 ``bf_getcharbuf`` 并且 <defenc> 对象尚不存在, 则首先创建它.

请注意, 作为特殊情况, 解析器标记 "s＃" 不会返回原始的 Unicode UTF-16 数据
(``bf_getreadbuf`` 返回), 而是尝试使用默认编码对 Unicode 对象进行编码,
然后返回指针生成的字符串对象 (或在转换失败时引发异常).
这样做是为了防止意外地将二进制数据写入另一端可能无法识别的输出流.

这具有能够写入输出流 (通常使用该接口) 而无需额外规范要使用的编码的优点.

如果需要访问 Unicode 对象的读缓冲区接口, 请使用 ``PyObject_AsReadBuffer()`` 接口.

也可以使用 'unicode-internal' 编解码器访问内部格式,
例如, 通过 ``u.encode('unicode-internal')``.


Pickle/Marshalling
==================

应该具有本机 Unicode 对象支持. 应该使用独立于平台的编码对对象进行编码.

Marshal 应该使用 UTF-8 和 Pickle 应该选择 Raw-Unicode-Escape (在文本模式下)
或 UTF-8 (在二进制模式下) 作为编码. 使用 UTF-8 而不是 UTF-16 具有无需存储 BOM 标记的优势.


Regular Expressions
===================

Secret Labs AB 正在开发一种支持 Unicode 的正则表达式机制.
它适用于普通的8位, UCS-2 和(可选) UCS-4 内部字符缓冲区.

Also see

    http://www.unicode.org/unicode/reports/tr18/

关于如何处理 Unicode RE 的一些备注.


Formatting Markers
==================

格式标记用于 Python 格式字符串.
如果 Python 字符串用作格式字符串, 则以下解释应该有效 ::

    '%s': 对于 Unicode 对象, 这将导致整个格式字符串强制转换为 Unicode.
          请注意, 出于性能原因, 您应该使用 Unicode 格式字符串.

如果格式字符串是 Unicode 对象, 则首先将所有参数强制转换为 Unicode,
然后将它们放在一起并根据格式字符串进行格式化. 数字首先转换为字符串, 然后转换为 Unicode.

::

    '%s': Python 字符串使用 <default encoding> 解释为 Unicode 字符串.
          Unicode 对象按原样使用.

所有其他字符串格式器应该相应地工作.

Example::

    u"%s %s" % (u"abc", "abc")  ==  u"abc abc"


Internal Argument Parsing
=========================

这些标记由 ``PyArg_ParseTuple()`` API使用:

"U"
    检查 Unicode 对象并返回指向它的指针

"s"
   对于 Unicode 对象: 返回指向对象的 <defenc> 缓冲区 (使用 <default encoding>) 的指针.

"s#"
   访问 Unicode 对象的默认编码版本 (请参阅缓冲区接口);
   请注意, 长度与默认编码字符串的长度有关, 而不是 Unicode 对象长度.

"t#"
    与 "s#" 相同.

"es"
   采用两个参数: encoding (``const char *``) 和 buffer (``char **``).

   输入对象首先用通常的方式强制转换为 Unicode, 然后使用指定的编码将其编码为字符串.

   在输出时, 分配所需大小的缓冲区并通过 ``* buffer`` 作为以 NULL 结尾的字符串返回.
   编码可能不包含嵌入的 NULL 字符. 调用者负责调用 ``PyMem_Free()`` 以在使用后释放分配的 ``* buffer``.

"es#"
    采用三个参数: encoding (``const char *``), buffer (``char **``)
    和 buffer_len (``int *``).

    输入对象首先用通常的方式强制转换为 Unicode, 然后使用指定的编码将其编码为字符串.

    如果 ``* buffer`` 是非 NULL, 则 ``* buffer_len`` 必须在输入时设置为 ``sizeof(buffer)``.
    然后将输出复制到 ``* buffer``.

    如果 ``* buffer`` 为 NULL, 则分配所需大小的缓冲区并将输出复制到其中.
    然后更新 ``* buffer`` 以指向分配的内存区域.
    调用者负责调用 ``PyMem_Free()`` 以在使用后释放分配的 ``* buffer``.

    在这两种情况下, ``* buffer_len`` 更新为写入的字符数 (不包括尾随的 NULL 字节).
    确保输出缓冲区以 NULL 结尾.

Examples:

在自动分配中使用 "es#" ::

    static PyObject *
    test_parser(PyObject *self,
                PyObject *args)
    {
        PyObject *str;
        const char *encoding = "latin-1";
        char *buffer = NULL;
        int buffer_len = 0;

        if (!PyArg_ParseTuple(args, "es#:test_parser",
                              encoding, &buffer, &buffer_len))
            return NULL;
        if (!buffer) {
            PyErr_SetString(PyExc_SystemError,
                            "buffer is NULL");
            return NULL;
        }
        str = PyString_FromStringAndSize(buffer, buffer_len);
        PyMem_Free(buffer);
        return str;
    }

使用带有自动分配的 "es" 返回以 NULL 结尾的字符串 ::

    static PyObject *
    test_parser(PyObject *self,
                PyObject *args)
    {
        PyObject *str;
        const char *encoding = "latin-1";
        char *buffer = NULL;

        if (!PyArg_ParseTuple(args, "es:test_parser",
                              encoding, &buffer))
            return NULL;
        if (!buffer) {
            PyErr_SetString(PyExc_SystemError,
                            "buffer is NULL");
            return NULL;
        }
        str = PyString_FromString(buffer);
        PyMem_Free(buffer);
        return str;
    }

将 "es#" 与预先分配的缓冲区一起使用 ::

    static PyObject *
    test_parser(PyObject *self,
                PyObject *args)
    {
        PyObject *str;
        const char *encoding = "latin-1";
        char _buffer[10];
        char *buffer = _buffer;
        int buffer_len = sizeof(_buffer);

        if (!PyArg_ParseTuple(args, "es#:test_parser",
                              encoding, &buffer, &buffer_len))
            return NULL;
        if (!buffer) {
            PyErr_SetString(PyExc_SystemError,
                            "buffer is NULL");
            return NULL;
        }
        str = PyString_FromStringAndSize(buffer, buffer_len);
        return str;
    }


File/Stream Output
==================

由于 file.write(object) 和大多数其他流编辑器使用 "s#" 或 "t#" 参数解析标记来查询要写入的数据,
因此, Unicode 对象的默认编码字符串版本将写入流中 (请参阅缓冲接口).

想要使用 Unicode 显式处理文件, 应该使用通过编解码器模块提供的标准流编解码器.

编解码器模块应该提供可用的快捷方式 (文件名, 模式, 编码), 这也确保模式在需要时包含 "b" 字符.



File/Stream Input
=================

只有用户知道输入数据使用的编码, 因此, 不会应用任何特殊的魔法
用户必须根据需要显式地将字符串数据转换为 Unicode 对象,
或者使用编解码器模块中定义的文件包装器 (请参阅文件或流输出).


Unicode Methods & Attributes
============================

所有 Python 字符串方法, 附加 ::

    .encode([encoding=<default encoding>][,errors="strict"])
       --> 请参阅 Unicode 输出

    .splitlines([include_breaks=0])
       --> 将 Unicode 字符串分成 (Unicode) 行列表;
           如果 include_breaks 为 true, 则返回包含换行符的行.
           有关如何完成换行的规范, 请参阅换行符.


Code Base
=========

我们应该使用 Fredrik Lundh 的 Unicode 对象实现作为基础.
它已经实现了所需的大多数字符串方法, 并提供了一个我们可以构建的编写良好的代码库.

应该删除 Fredrik 实现中实现的对象共享.


Test Cases
==========

测试用例应该遵循 Lib/test/test_string.py 中的测试用例,
并包括对 Codec Registry 和 Standard Codecs 的附加检查.


References
==========

* Unicode Consortium: http://www.unicode.org/

* Unicode FAQ: http://www.unicode.org/unicode/faq/

* Unicode 3.0: http://www.unicode.org/unicode/standard/versions/Unicode3.0.html

* Unicode-TechReports: http://www.unicode.org/unicode/reports/techreports.html

* Unicode-Mappings: ftp://ftp.unicode.org/Public/MAPPINGS/

* Introduction to Unicode (a little outdated by still nice to read):
  http://www.nada.kth.se/i18n/ucs/unicode-iso10646-oview.html

* For comparison:
  Introducing Unicode to ECMAScript (aka JavaScript) --
  http://www-4.ibm.com/software/developer/library/internationalization-support.html

* IANA Character Set Names:
  ftp://ftp.isi.edu/in-notes/iana/assignments/character-sets

* Discussion of UTF-8 and Unicode support for POSIX and Linux:
  http://www.cl.cam.ac.uk/~mgk25/unicode.html

* Encodings:

  * Overview: http://czyborra.com/utf/

  * UCS-2: http://www.uazone.org/multiling/unicode/ucs2.html

  * UTF-7: Defined in RFC2152, e.g.
    http://www.uazone.org/multiling/ml-docs/rfc2152.txt

  * UTF-8: Defined in RFC2279, e.g.
    https://tools.ietf.org/html/rfc2279

  * UTF-16: http://www.uazone.org/multiling/unicode/wg2n1035.html


History of this Proposal
========================

[ed. 注意: 标准 Python 发行版的 Misc/unicode.txt 的
CVS 历史记录中提供了 1.7 之前的修订版.
所有后续历史记录均可通过此文件的 CVS 修订版获得.]

1.7
---

* 添加了有关 "s#" 更改行为的注释.

1.6
---

* 将 <defencstr> 更改为 <defenc>, 因为这是实现中使用的名称.
* 添加了有关缓冲协议实现中 <defense> 用法的说明.

1.5
---

* 添加了有关设置 <default encoding> 的注释.
* 修正了一些拼写错误 (感谢 Andrew Kuchling).
* 将 <defencstr> 更改为 <utf8str>.

1.4
---

* 添加了有关混合类型比较的注释, 并包含测试.
* 更改了格式字符串中 Unicode 对象的处理
  (如果与 ``'％s' ％ u`` 一起使用, 它们现在将导致格式字符串被强制转换为 Unicode,
  从而在返回时生成 Unicode 对象).
* 添加了 IANA 字符集名称的链接 (感谢 Lars Marius Garshol).
* 添加了新的编解码方法 ``.readline()``, ``.readlines()`` and ``.writelines()``.

1.3
---

* 添加了新的 "es" 和 "es#" 解析器标记

1.2
---

* 删除了关于 ``codecs.open()`` 的 POD

1.1
---

* 添加了有关比较和哈希值的注释.
* 添加了关于 case 映射算法的注释.
* 更改了流编解码器 ``.read()`` 和 ``.write()`` 方法以匹配标准的类文件对象方法
  (方法不再返回字节消耗的信息)

1.0
---

* 改变编码 Codec 方法与解码方法对称 (它们现在都返回 (对象，数据消), 因此可以互换);
* 删除了 Codec 类的 ``__init__`` 方法 (方法是无状态的) 并将 errors 参数移到方法中;
* 使得 Codec 设计更加通用 w/r 到输入和输出对象的类型;
* 将 ``StreamWriter.flush`` 更改为 ``StreamWriter.reset``
  用来避免覆盖流的 ``.flush()`` 方法;
* 将 ``.breaklines()`` 重命名为 ``.splitlines()``;
* 将模块 unicodec 重命名为 codecs;
* 修改了文件 I/O 部分来引用流编解码器.

0.9
---

* 更改错误关键字参数定义;
* 添加了 'replace' 错误处理;
* 更改了编解码器 API 来接受输入上的缓冲区;
* 一些小错字修复;
* 添加了空白符部分, 并包含具有空格和换行符特征的 Unicode 字符的引用;
* 补充说明搜索功能可以期待小写编码名称;
* 删除了编解码器 API 中的切片和偏移量

0.8
---

* 添加了编码包和原始 unicode 转义编码;
* 没有选择这个提议;
* 添加了关于 Unicode 格式字符串的注释;
* 添加了 ``.breaklines()`` 方法

0.7
---

* 添加了一整套新的编解码器 API;
* 添加了不同的编码器查找方案;
* 固定了一些名字

0.6
---

* 将 "s#" 改为 "t#";
* 将 <defencbuf> 更改为 <defencstr>, 保存一个真正的 Python 字符串对象;
* 更改了 Buffer Interface 来将请求委托给 <defencstr> 的缓冲区接口;
* 删除了对 unicodec.codecs 字典的显式引用 (模块可以实现这个目的);
* 删除了可设置的默认编码;
* 将 ``UnicodeError`` 从 unicodec 移到 exceptions;
* "s#" 不返回内部数据;
* 将 UCS-2/UTF-16 检查从 Unicode 构造函数传递给编解码器

0.5
---

* 将 ``sys.bom`` 移到 ``unicodec.BOM``;
* 添加了关于 case 映射的部分,
* 私有使用编码和 Unicode 字符属性

0.4
---

* 添加了 Codec 界面, 关于 ％-formatting 的注释,
* 改变了一些编码细节,
* 添加了关于流包装器的注释,
* 修正了一些讨论要点 (最重要的是: 内部格式),
* 阐明了 'unicode-escape' 编码, 添加了编码引用

0.3
---

* 添加了引用, 对编解码器模块的注释, 内部格式, bf_getcharbuffer 和 RE 引擎;
* 添加了由 Tim Peters 和固定的 repr(u) 提出的 "unicode-escape" 编码

0.2
---

* 集成了 Guido 的建议, 添加了流编解码器和文件包装

0.1
---

* 第一版


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
