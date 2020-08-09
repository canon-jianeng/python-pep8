
PEP: 247
Title: API for Cryptographic Hash Functions
Version: $Revision$
Last-Modified: $Date$
Author: A.M. Kuchling <amk@amk.ca>
Status: Final
Type: Informational
Content-Type: text/x-rst
Created: 23-Mar-2001
Post-History: 20-Sep-2001


Abstract
========

有几种不同的模块可用于实现加密散列算法, 如 MD5 或 SHA.
本文档为此类算法指定了标准 API, 利于在不同实现之间切换.


Specification
=============

所有散列模块都应该具有相同的接口. 可以添加其他方法或变量,
但应该始终存在本文档中描述的方法或变量.

散列函数模块定义一个函数:

| ``new([string])            (unkeyed hashes)``
| ``new([key] , [string])    (keyed hashes)``

   创建一个新的哈希对象并将其返回. 第一种形式用于未加密的哈希, 例如 MD5 或 SHA.
   对于带密钥散列 (如 HMAC), *key* 是一个必需参数, 包含一个字符串, 用于指定要使用的键.
   在这两种情况下, 可选的 *string* 参数 (如果提供) 将立即散列到对象的起始状态,
   就像调用 ``obj.update(string)`` 一样.

   在创建散列对象之后, 可以使用 ``update()`` 方法将任意字符串输入到对象中,
   并且可以通过调用对象的 ``digest()`` 方法随时获取散列值.

   可以向此函数添加任意其他关键字参数, 但如果未提供, 则应该使用合理的默认值.
   例如, 可以为散列函数添加 ``rounds`` 和 ``digest_size`` 关键字,
   它支持可变数量的次数和几种不同的输出大小, 并且它们应该默认为被认为是安全的值.

散列函数模块定义一个变量:

| ``digest_size``

   整数值; 由此模块创建的散列对象生成的摘要大小, 以字节为单位.
   您还可以通过创建示例对象并访问其 ``digest_size`` 属性来获取此值,
   但可以方便地从模块中获取此值. 具有可变输出大小的哈希将此变量设置为 ``None``.

散列对象需要单个属性:

| ``digest_size``

   此属性与模块级 "digest_size" 变量相同, 用于测量散列对象生成的摘要大小 (以字节为单位).
   如果散列具有可变输出大小, 则必须在创建散列对象时选择此输出大小,
   并且此属性必须包含所选大小. 因此, ``None`` 不是此属性的合法值.


散列对象需要以下方法:

| ``copy()``

   返回此散列对象的单独副本. 对此副本的更新不会影响原始对象.

| ``digest()``

   将此散列对象的散列值作为包含8位数据的字符串返回.
   该函数不会以任何方式改变对象; 您可以在调用此函数后继续更新对象.

| ``hexdigest()``

   将此散列对象的散列值作为包含十六进制数字的字符串返回.
   小写字母应该用于数字 ``a`` 到 ``f``. 与 ``.digest()`` 方法一样, 此方法不得更改对象.

| ``update(string)``

   散列 *string* 进入散列对象的当前状态. 在散列对象的生命周期中,
   ``update()`` 可以被调用任意次.

散列模块可以定义其他模块级函数或对象方法, 并且仍然符合此规范.

这是一个例子, 使用名为 ``MD5`` 的模块::

    >>> from Crypto.Hash import MD5
    >>> m = MD5.new()
    >>> m.digest_size
    16
    >>> m.update('abc')
    >>> m.digest()
    '\x90\x01P\x98<\xd2O\xb0\xd6\x96?}(\xe1\x7fr'
    >>> m.hexdigest()
    '900150983cd24fb0d6963f7d28e17f72'
    >>> MD5.new('abc').digest()
    '\x90\x01P\x98<\xd2O\xb0\xd6\x96?}(\xe1\x7fr'


Rationale
=========

摘要大小以字节为单位, 而不是位, 即使散列算法大小通常以位为单位引用;
例如, MD5 是128位算法, 而不是16字节算法. 这是因为, 在我看到的示例代码中,
通常需要以字节为单位的长度 (在文件中向前或向后搜索; 计算输出字符串的长度),
而很少使用以位为单位的长度. 因此, 负担将落在少数实际需要大小的人身上, 他们必须将 ``digest_size`` 乘以8.

有人建议将 ``update()`` 方法命名为 ``append()``. 但是, 该方法实际上是导致哈希对象的当前状态被更新,
并且 Python 中包含的 md5 和 sha 模块已经使用了 ``update()``, 所以单独保留名称 ``update()`` 似乎最简单.

关键字哈希的构造函数参数的顺序是一个棘手的问题. 目前尚不清楚 *key* 应该是第一个还是第二个.
它是一个必需的参数, 通常的惯例是首先放置所需的参数, 但这也意味着 *string* 参数从第一个位置移动到第二个位置.
可能会混淆并将单个参数传递给键控哈希, 认为您将初始字符串传递给未键入的哈希值,
但似乎不值得使键入哈希的接口更加模糊来避免这种潜力错误.


Changes
=======

2001-09-17: 将 ``clear()`` 重命名为 ``reset()``; 在对象中添加了 ``digest_size`` 属性;
添加了 ``.hexdigest()`` 方法.

2001-09-20: 完全删除了 ``reset()`` 方法.

2001-09-28: 对于可变大小的哈希, 将 ``digest_size`` 设置为 ``None``.


Acknowledgements
================

感谢 Aahz, Andrew Archibald, Rich Salz, Itamar Shtull-Trauring,
以及 python-crypto 列表的读者, 感谢他们对此 PEP 的评论.


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  

