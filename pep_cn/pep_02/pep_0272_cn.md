
PEP: 272
Title: API for Block Encryption Algorithms v1.0
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Final
Type: Informational
Content-Type: text/x-rst
Created: 18-Sep-2001
Post-History: 17-Apr-2002, 29-May-2002


Abstract
========

本文档为 DES 或 Rijndael 等保密密钥块加密算法指定了标准 API, 使得在不同算法和实现之间切换更加容易.


Introduction
============

加密算法以某种方式转换它们的输入数据 (称为明文), 这种方式依赖于变量密钥, 从而产生密文.
当且仅当知道密钥的转换时, 才能轻易地反转. 关键是从一些非常大的可能键空间中选择的一系列位.
有两类加密算法: 块密码和流密码.

块密码加密固定大小的多字节输入 (通常为8或16字节长), 并且可以在各种反馈模式下操作.
本规范支持的反馈模式是:

======  ========  =====================
Number  Constant  Description
======  ========  =====================
1       MODE_ECB  Electronic Code Book
2       MODE_CBC  Cipher Block Chaining
3       MODE_CFB  Cipher Feedback
5       MODE_OFB  Output Feedback
6       MODE_CTR  Counter
======  ========  =====================

这些模式将按照 NIST 发行版 SP 800-38A [1]_ 中所述实施.
前三种反馈模式的描述也可以在 Bruce Schneier 的书中找到 *Applied Cryptography* [2]_.

(数值4保留给 MODE_PGP, 这是 RFC 2440 中描述的 CFB 的变体: "OpenPGP 消息格式" [3]_.
这种模式被认为不够重要, 不足以使它对所有块加密密码都要求它, 尽管支持它是一个很好的额外功能.)

在严格的形式意义上, 流密码逐位加密数据; 实际上, 流密码在逐个字符的基础上工作.
此 PEP 仅旨在为块密码指定接口, 尽管流密码可以通过将 'block_size' 固定为1来支持此处描述的接口.
反馈模式对流密码也没有意义, 因此唯一合理的反馈模式是 ECB 模式.


Specification
=============

加密模块可以添加除本 PEP 中描述的函数, 方法和属性之外的其他功能,
但是此模块中描述的所有功能必须存在才能使模块声明符合它.

密钥加密模块应该定义一个功能::

    new(key, mode, [IV], **kwargs)

返回一个加密对象, 使用字符串 'key' 中包含的密钥, 并使用反馈模式 'mode',
它必须是上表中的常量之一.

如果 'mode' 是 MODE_CBC 或 MODE_CFB, 则必须提供 'IV',
并且必须是与块大小相同长度的字符串. 不提供 'IV' 值将导致引发 ``ValueError`` 异常.

根据算法, 模块可能支持此函数的其他关键字参数. 此 PEP 指定了一些关键字参数,
模块可以自由添加其他关键字参数. 如果没有为给定关键字提供值, 则应该使用安全默认值.
例如, 如果算法在1到16之间具有可选择的 rounds  次数, 并且1轮加密是不安全的,
并且8轮加密被认为是安全的, 则 'rounds' 的默认值应该是8或更多.
(模块实现者也可以选择一个非常慢但安全的值, 例如, 本例中的16个. 这个决定由实现者决定.)

下表列出了此 PEP 定义的关键字参数:

============  ============================================
Keyword       Meaning
============  ============================================
counter       Callable object that returns counter blocks
              (see below; CTR mode only)

rounds        Number of rounds of encryption to use

segment_size  Size of data and ciphertext segments,
              measured in bits (see below; CFB mode only)
============  ============================================

计数器反馈模式需要一系列输入块, 称为计数器, 用于产生输出. 当 'mode' 是 MODE_CTR 时,
必须提供 'counter' 关键字参数, 并且其值必须是可调用对象, 例如, 函数或方法.
对此可调用对象的连续调用必须返回一个长度为 'block_size' 并且从不重复的字符串序列.
(NIST 发行版的附录 B 给出了生成这样一个序列的方法, 但这超出了本 PEP 的范围.)

CFB 模式对明文和密文的段落进行操作, 这些段落是 'segment_size' 位长. 因此, 使用此模式时,
输入和输出字符串的长度必须是 'segment_size' 位的倍数. 'segment_size' 必须是1到 block_size\*8 之间的整数,
包括1和 block_size\*8. (因子8来自 'block_size', 以字节为单位而不是以位为单位).
此参数的默认值应该为 block_size\*8. 为简单起见, 允许实现者将 'segment_size' 约束为8的倍数,
但鼓励它们支持任意值以保持通用性.

密钥加密模块应该定义两个变量:

- block_size

  整数值; 由此模块加密的块的大小, 以字节为单位. 对于所有反馈模式,
  传递给 encrypt() 和 decrypt() 的字符串长度必须是块大小的倍数.

- key_size

  整数值; 此模块所需的密钥大小, 以字节为单位. 如果 key_size 为 None,
  则算法接受可变长度键. 这可能意味着模块接受任何随机长度的密钥,
  或者存在几种不同的可能长度, 例如, 16, 24 或 32 个字节.
  您不能将长度为0的键 (即空字符串'') 作为可变长度键传递.


密码对象应该有两个属性:

- block_size

  一个整数值, 等于此对象加密的块的大小. 对于具有可变块大小的算法,
  此值等于为此对象选择的块大小.

- IV

  包含将用于启动密码反馈模式的初始值; 它总是一个字符串, 长度恰好是一个块.
  在加密或解密字符串后, 将更新此值来反映修改后的反馈文本. 它是只读的, 不能分配新值.


密码对象需要以下方法:

- decrypt(string)

  解析 'string', 使用对象中依赖于键的数据并使用适当的反馈模式.
  字符串的长度必须是算法块大小的精确倍数, 或者在 CFB 模式下,
  是段落大小的精确倍数. 返回包含明文的字符串.

- encrypt(string)

  使用对象中与密钥相关的数据以及相应的反馈模式加密非空字符串.
  字符串的长度必须是算法块大小的精确倍数, 或者在 CFB 模式下,
  是段落大小的精确倍数. 返回包含密文的字符串.

这是一个例子, 使用名为 'DES' 的模块::

    >>> import DES
    >>> obj = DES.new('abcdefgh', DES.MODE_ECB)
    >>> plaintext = "Guido van Rossum is a space alien."
    >>> len(plaintext)
    34
    >>> obj.encrypt(plaintext)
    Traceback (innermost last):
      File "<stdin>", line 1, in ?
    ValueError: Strings for DES must be a multiple of 8 in length
    >>> ciphertext = obj.encrypt(plain+'XXXXXX')   # Add padding
    >>> ciphertext
    '\021,\343Nq\214DY\337T\342pA\372\255\311s\210\363,\300j\330\250\312\347\342I\3215w\03561\303dgb/\006'
    >>> obj.decrypt(ciphertext)
    'Guido van Rossum is a space alien.XXXXXX'


References
==========

.. [1] NIST publication SP 800-38A, "Recommendation for Block Cipher
       Modes of Operation" (http://csrc.nist.gov/encryption/modes/)

.. [2] Applied Cryptography

.. [3] RFC2440: "OpenPGP Message Format" (http://rfc2440.x42.com,
       http://www.faqs.org/rfcs/rfc2440.html)


Changes
=======

2002-04: 删除了对流密码的引用; 重新命名的 PEP; 带有 ``MODE_`` 的前缀反馈模式常量;
删除了 PGP 反馈模式; 增加了 CTR 和 OFB 反馈模式; 澄清了以字节为单位测量数字的位置以及位数的位置.

2002-09: 通过使用 "可变长度键" 而不是 "任意长度" 来阐明键长度的讨论.


Acknowledgements
================

Thanks to the readers of the python-crypto list for their comments on
this PEP.


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  

