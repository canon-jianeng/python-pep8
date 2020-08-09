
PEP: 263
Title: Defining Python Source Code Encodings
Version: $Revision$
Last-Modified: $Date$
Author: mal@lemburg.com (Marc-André Lemburg),
  martin@v.loewis.de (Martin von Löwis)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 06-Jun-2001
Python-Version: 2.3
Post-History:


Abstract
========

该 PEP 建议引入一种语法来声明 Python 源文件的编码. 然后,
Python 解析器使用编码信息来使用给定的编码来解释文件.
最值得注意的是, 这增强了源代码中 Unicode 字面量的解释,
并且可以使用, 例如 Unicode 来编写 Unicode 字面量.
UTF-8 直接在 Unicode 识别编辑器中.


Problem
=======

在 Python 2.1 中, Unicode 字面量只能使用基于 Latin-1 的编码 "unicode-escape" 编写.
这使编程环境对在 non-Latin-1 语言环境中生活和工作的 Python 用户非常不友好,
例如, 许多亚洲国家. 程序员可以使用喜欢的编码编写他们的8位字符串,
但是绑定到 Unicode 字面量的 "unicode-escape" 编码.


Proposed Solution
=================

我建议在每个源文件的基础上使 Python 源代码编码可见和可更改,
方法是使用文件顶部的特殊注释来声明编码.

为了使 Python 了解此编码声明, 在处理 Python 源代码数据时需要进行许多概念更改.


Defining the Encoding
=====================

如果没有给出其他编码提示, Python 将默认为 ASCII 作为标准编码.

要定义源代码编码, 必须将魔法注释作为文件中的第一行或第二行放入源文件中,
例如::

    # coding=<encoding name>

或 (使用流行编辑认可的格式)::

    #!/usr/bin/python
    # -*- coding: <encoding name> -*-

或::

    #!/usr/bin/python
    # vim: set fileencoding=<encoding name> :

更确切地说, 第一行或第二行必须与以下正则表达式匹配::

    ^[ \t\v]*#.*?coding[:=][ \t]*([-_.a-zA-Z0-9]+)

然后将此表达式的第一组解释为编码名称. 如果 Python 不知道编码, 则在编译期间会引发错误.
在包含编码声明的行上不能有任何 Python 语句. 如果第一行匹配, 则忽略第二行.

为了帮助将诸如 Windows 之类的平台添加到 Unicode 文件的开头,
UTF-8 签名 ``\xef\xbb\xbf`` 也将被解释为 'utf-8' 编码 (即使, 没有给出魔法编码注释).

如果源文件同时使用 UTF-8 BOM 标记签名和魔法编码注释,
则唯一允许的注释编码为 "utf-8". 任何其他编码都会导致错误.


Examples
========

这些是一些示例, 用于阐明在 Python 源文件顶部定义源代码编码的不同样式:

1. 使用解释器二进制和使用 Emacs 样式文件编码注释::

       #!/usr/bin/python
       # -*- coding: latin-1 -*-
       import os, sys
       ...

       #!/usr/bin/python
       # -*- coding: iso-8859-15 -*-
       import os, sys
       ...

       #!/usr/bin/python
       # -*- coding: ascii -*-
       import os, sys
       ...

2. 没有解释器行, 使用纯文本::

       # This Python file uses the following encoding: utf-8
       import os, sys
       ...

3. 文本编辑器可能有不同的方式来定义文件的编码, 例如::

       #!/usr/local/bin/python
       # coding: latin-1
       import os, sys
       ...

4. 如果没有编码注释, Python 的解析器将采用 ASCII 文本::

       #!/usr/local/bin/python
       import os, sys
       ...

5. 编码不起作用的注释:

   1. 缺少 "coding:" 前缀::

          #!/usr/local/bin/python
          # latin-1
          import os, sys
          ...

   2. 编码注释不在第1行或第2行::

          #!/usr/local/bin/python
          #
          # -*- coding: latin-1 -*-
          import os, sys
          ...

   3. 不支持的编码::

          #!/usr/local/bin/python
          # -*- coding: utf-42 -*-
          import os, sys
          ...


Concepts
========

PEP 基于以下概念, 必须实施这些概念才能使用这样的魔法注释:

1. 完整的 Python 源文件应该使用单个编码. 不允许嵌入不同编码的数据,
   并且在编译 Python 源代码期间将导致解码错误.

   允许以上述方式处理前两行的任何编码作为源代码编码, 这包括 ASCII 兼容编码以及某些多字节编码,
   例如 Shift_JIS. 它不包括对所有字符使用两个或多个字节的编码, 例如 UTF-16.
   这样做的原因是使编码检测算法在标记化器中保持简单.

2. 转义序列的处理应该像现在一样继续工作, 但是对于所有可能的源代码编码, 标准字符串字面量 (8位和 Unicode)
   都要进行转义序列扩展, 而原始字符串字面量只扩展转义序列的一个非常小的子集.

3. Python 的 tokenizer/compiler 组合需要更新才能工作, 如下:

   1. 读取文件

   2. 假设每个文件编码固定, 将其解码为 Unicode

   3. 将其转换为 UTF-8 字节字符串

   4. 标记化 UTF-8 内容

   5. 编译它, 从给定的 Unicode 数据创建 Unicode 对象,
      并通过首先使用给定的文件编码将 UTF-8 数据重新编码为8位字符串数据,
	  从 Unicode 字面量数据创建字符串对象

请注意, Python 标识符仅限于编码的 ASCII 子集, 因此在步骤4之后无需进一步转换.


Implementation
==============

为了向后兼容当前在字符串字面量中使用非 ASCII, 而不声明编码的现有代码,
将在两个阶段引入实现:

1. 通过将缺少的编码声明内部视为 "iso-8859-1" 的声明, 允许在字符串字面量和注释中使用非 ASCII.
   这将导致任意字节字符串在处理的第2步和第5步之间正确地往返,
   并为包含非 ASCII 字节的 Unicode 字面量提供与 Python 2.2 的兼容性.

   如果在输入中找到非 ASCII 字节, 则会发出警告, 每个编码输入文件编码不正确.

2. 删除警告, 并将默认编码更改为 "ascii".

内置的 ``compile()`` API 将被增强, 来接受 Unicode 作为输入.
如上所述, 8位字符串输入符合用于编码检测的标准过程.

如果带有编码声明的 Unicode 字符串传递给 ``compile()``, 则会引发 ``SyntaxError``.

SUZUKI Hisao 正在制作补丁; 详见 [2]_. 在 [1]_ 处可以获得仅实施阶段1的补丁.


Phases
======

除了将默认编码更改为 "ascii" 之外, 上述步骤1和2的实现在 2.3 中完成.

版本 2.5 中的默认编码设置为 "ascii".


Scope
=====

该 PEP 旨在提供从当前 (或多或少) 未定义的源代码编码情况到更健壮和可移植的定义的升级路径.


References
==========

.. [1] 段落 1 实现:
       https://bugs.python.org/issue526840

.. [2] 段落 2 实现:
       https://bugs.python.org/issue534304

History
=======

- 1.10 及以上: 见 CVS 历史
- 1.8: 添加 '.' 到编码 RE.
- 1.7: 为第1阶段实施添加了警告. 用解释器的默认编码替换 Latin-1 默认编码. 添加了对 ``compile()`` 的调整.
- 1.4 - 1.6: 小调整
- 1.3: Martin v. Loewis 的注释工作:
  UTF-8 BOM 标记检测, Emacs 风格魔法注释, 两个阶段实现方法


Copyright
=========

This document has been placed in the public domain.


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  sentence-end-double-space: t  
  fill-column: 70  
  coding: utf-8  
  End:  
