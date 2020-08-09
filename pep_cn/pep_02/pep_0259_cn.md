
PEP: 259
Title: Omit printing newline after newline
Version: $Revision$
Last-Modified: $Date$
Author: guido@python.org (Guido van Rossum)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 11-Jun-2001
Python-Version: 2.2
Post-History: 11-Jun-2001


Abstract
========

目前, ``print`` 语句总是附加换行符, 除非使用尾随逗号.
这意味着如果我们要打印已经以换行符结尾的数据,
除非采取特殊预防措施, 否则我们会获得两个换行符.

我建议在跟随来自数据的换行符时, 跳过打印换行符.

为了避免必须向文件对象添加另一个魔术变量, 我建议给现有的 'softspace' 变量一个额外的含义:
负值意味着 "最后写入的数据在换行符中结束, 因此没有空格或换行符是必须的."


Problem
=======

当使用简单循环打印类似于从文件读取的行的数据时,
除非特别小心, 否则会出现双倍间距::

    >>> for line in open("/etc/passwd").readlines():
    ... print line
    ...
    root:x:0:0:root:/root:/bin/bash

    bin:x:1:1:bin:/bin:

    daemon:x:2:2:daemon:/sbin:

    (etc.)

    >>>

虽然有简单的解决方法, 但这通常仅在测试期间被注意到,
并且需要额外的编辑测试往返; 固定代码更难以维护.


Proposed Solution
=================

在 ``ceval.c`` 中的 ``PRINT_ITEM`` 操作码中, 当打印一个字符串对象时,
已经进行了检查来查看该字符串的最后一个字符. 目前,
如果最后一个字符是空格以外的空白字符, 则软空间标志重置为零;
如果第一个项目是以换行符, 制表符等结尾的字符串 (但不是在空格中结束时),
则会抑制两个项目之间的空间. 否则, softspace 标志设置为 1.

该提议稍微更改此测试, 利于将 softspace 设置为:

- ``-1`` -- 如果写的最后一个对象是以换行符结尾的字符串

- ``0`` -- 如果写的最后一个对象是一个以空白字符结尾的字符串, 既不是空格也不是换行符

- ``1`` -- 在所有其他情况下 (包括最后一个对象写入空字符串或不是字符串的情况)

然后, ``PRINT_NEWLINE`` 操作码, 如果 softspace 的值为负, 则抑制换行的打印;
在任何情况下, softspace 标志都被重置为零.


Scope
=====

这仅影响8位字符串的打印. 它不会影响 Unicode,
尽管这可能被视为 Unicode 实现中的错误.
它不会影响其字符串表示形式恰好以换行符结尾的其他对象.


Risks
=====

此更改会破坏一些现有代码. 例如::

    print "Subject: PEP 259\n"
    print message_body

在当前的 Python 中, 这会产生一个空白行, 将主题与消息体分开;
随着提议的改变, body 在主题的正下方开始. 无论如何, 这不是非常强大的代码;
它写得更好::

    print "Subject: PEP 259"
    print
    print message_body

在测试套件中, 只有 ``test_StringIO`` (显式测试此功能) 会中断.


Implementation
==============

这里有一个与当前 CVS 相关的补丁::

    http://sourceforge.net/tracker/index.php?func=detail&aid=432183&group_id=5470&atid=305470


Rejected
========

用户社区一致拒绝了这一点, 所以我不再进一步追求这个想法.
经常听到反对的论点:

- 它可能会破坏成千上万的 CGI 脚本.

- 已经足够的魔法 (因此: 请不再修补 'print').


Copyright
=========

This document has been placed in the public domain.


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
