
PEP: 243
Title: Module Repository Upload Mechanism
Version: $Revision$
Last-Modified: $Date$
Author: jafo-pep@tummy.com (Sean Reifschneider)
Discussions-To: distutils-sig@python.org
Status: Withdrawn
Type: Standards Track
Content-Type: text/x-rst
Created: 18-Mar-2001
Python-Version: 2.1
Post-History: 20-Mar-2001, 24-Mar-2001


Abstract
========

要使模块存储库系统 (例如 Perl 的 CPAN) 成功, 模块作者必须尽可能容易地提交他们的工作.
在成功创建分发存档之后, Distutils 工具中显示此提交发生的明显位置. 例如,
在模块作者测试了他们的软件 (验证 ``setup.py sdist`` 的结果) 后, 他们可能会输入 ``setup.py sdist --submit``.
这将标记 Distutils 将源分发提交到存档服务器来包含和分发到镜像.

此 PEP 仅处理将软件分发提交到存档的机制, 而不处理实际的存档或目录服务器.


Upload Process
==============

上传将包括 Distutils ``PKG-INFO`` 元数据信息 (如 PEP 241 [1]_ 中所述), 实际软件分发和其他可选信息.
此信息将作为多部分表单上载, 其编码方式与常规 HTML 文件上载请求相同.
这个表单使用 ``ENCTYPE = "multipart/form-data"`` 编码进行发布 [2]_.

上传将在端口 80/tcp 上发送给主机 "www.python.org"
(``POST http://www.python.org:80/pypi``). 表格将包含以下字段:

- ``distribution`` -- 包含模块软件的文件 (例如, ``.tar.gz`` 或 ``.zip`` 文件).

- ``distmd5sum`` -- 上传分布的 MD5 哈希值, 用 ASCII 编码, 表示摘要的十六进制表示形式
  (``for digest in digest: s = s +（'％02x'％ord(byte))``).

- ``pkginfo`` (optional) -- 包含分布元数据的文件 (在 PEP 241 [1]_ 中指定). 请注意,
  如果未包含此内容, 则分发文件应该为 ".tar" 格式 (允许 gzip 压缩和 bzipped 压缩)
  或 ``.zip`` 格式, 并带有 ``PKG-INFO`` 文件在它提取的顶级目录中 (``package-1.00/PKG-INFO``).

- ``infomd5sum`` (required if pkginfo field is present) -- 上传的元数据的 MD5 哈希值, 用 ASCII 编码,
  表示摘要的十六进制表示形式 (``for byte in digest: s = s + ('%02x' % ord(byte))``).

- ``platform`` (optional) -- 表示此分发的目标平台的字符串. 这仅适用于二进制分发. 它被编码为
  ``<os_name>-<os_version>-<platform architecture>-<python version>``.

- ``signature`` (optional) -- 与作者签名的上传分发版的 OpenPGP 兼容签名 [3]_
  编目系统可以使用它来自动接受上传.

- ``protocol_version`` -- 一个字符串, 指示客户端支持的协议版本. 本文档描述了协议版本 "1".


Return Data
===========

将使用 HTTP 非标准 (``X-*``) 标头报告上传状态. ``X-Swalow-Status`` 标题可以具有以下值:

- ``SUCCESS`` -- 表示上传成功.

- ``FAILURE`` -- 由于某种原因, 上传无法处理.

- ``TRYAGAIN`` -- 此时服务器无法接受上传, 但客户端应该在以后再次尝试.
  造成这种情况的可能原因是服务器上的资源短缺, 管理停机时间, 等等...

可选地, 可能存在 ``X-Swalow-Reason`` 标题, 其包括人类可读的字符串,
其提供关于 ``X-Swalow-Status`` 的更详细信息.

如果没有 ``X-Swalow-Status`` 标题, 或者它不包含上面三个字符串中的一个,
则应该将其视为临时故障.

例如 ::

    >>> f = urllib.urlopen('http://www.python.org:80/pypi')
    >>> s = f.headers['x-swalow-status']
    >>> s = s + ': ' + f.headers.get('x-swalow-reason', '<None>')
    >>> print s
    FAILURE: Required field "distribution" missing.


Sample Form
===========

上传客户端必须以与 Netscape Navigator 版本 4.76 相同的形式提交页面,
利于在出现以下表单时生成 Linux ::

    <H1>Upload file</H1>
    <FORM NAME="fileupload" METHOD="POST" ACTION="pypi"
        ENCTYPE="multipart/form-data">
    <INPUT TYPE="file" NAME="distribution"><BR>
    <INPUT TYPE="text" NAME="distmd5sum"><BR>
    <INPUT TYPE="file" NAME="pkginfo"><BR>
    <INPUT TYPE="text" NAME="infomd5sum"><BR>
    <INPUT TYPE="text" NAME="platform"><BR>
    <INPUT TYPE="text" NAME="signature"><BR>
    <INPUT TYPE="hidden" NAME="protocol_version" VALUE="1"><BR>
    <INPUT TYPE="SUBMIT" VALUE="Upload">
    </FORM>


Platforms
=========

以下是有效的操作系统名称::

    aix beos debian dos freebsd hpux mac macos mandrake netbsd
    openbsd qnx redhat solaris suse windows yellowdog

以上包括许多不同类型的 Linux 发行版. 由于版本控制问题, 必须将它们分开,
并且当一个系统使用在其他类似系统上进行的分发时, 预期当下载客户端将区分时.

Version 是供应商为特定版本指定的正式版本字符串. 例如,
"2000" 和 "nt" (Windows), "9.04" (HP-UX), "7.0" (RedHat, Mandrake).

以下是有效的结构 ::

    alpha hppa ix86 powerpc sparc ultrasparc


Status
======

我目前有一个概念验证客户端和服务器实现. 我打算为 2.1 版本准备好 Distutils 补丁.
结合 Andrew 的 PEP 241 [1]_ 来指定分布元数据, 我希望有一个平台, 它将允许我们收集真实数据,
来完成 2.2 版本的目录系统.


References
==========

.. [1] Python 软件包的元数据, Kuchling,
       http://www.python.org/dev/peps/pep-0241/

.. [2] RFC 1867, HTML 格式的基于表单的文件上传
       http://www.faqs.org/rfcs/rfc1867.html

.. [3] RFC 2440, OpenPGP 消息格式
       http://www.faqs.org/rfcs/rfc2440.html


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
