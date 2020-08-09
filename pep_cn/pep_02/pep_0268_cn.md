
PEP: 268
Title: Extended HTTP functionality and WebDAV
Version: $Revision$
Last-Modified: $Date$
Author: gstein@lyra.org (Greg Stein)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 20-Aug-2001
Python-Version: 2.x
Post-History: 21-Aug-2001


Rejection Notice
================

该 PEP 已被拒绝. 自提案以来的六年里, 它未能产生足够的社区支持.


Abstract
========

本 PEP 讨论了 Python 的 HTTP 支持的新模块和扩展功能. 值得注意的是,
添加了经过身份验证的请求, 代理支持, 经过身份验证的代理使用以及 WebDAV_ 功能.


Rationale
=========

由于其 "内置电池" 定位, Python 一直非常受欢迎. HTTP (参见 RFC 2616) 是最常用的协议之一,
已经包含在 Python 中多年 (``httplib``). 但是, 这种支持无法满足许多基于 HTTP 的应用程序
和系统的全部需求和要求. 此外, 基于 HTTP 的新协议 (如 WebDAV 和 XML-RPC) 正变得越来越有用,
并且越来越多地使用它们. 提供此功能符合 Python 的 "内置电池" 角色, 同时也使 Python 处于新技术的前沿.

虽然身份验证和代理支持是 Python 核心 HTTP 处理中缺少的两个非常值得注意的特性,
但它们作为 Python 的 URL 处理 (``urllib`` 和 ``urllib2``) 的一部分处理得最少. 但是,
需要细粒度或复杂 HTTP 处理的应用程序在驻留在 urllib 中时无法使用这些功能.
将这些功能重构到可以直接与 HTTP 连接关联的位置, 可以提高 urllib 和复杂应用程序的实用性.

这个 PEP 的动机来自直接请求这些功能的几个人, 以及 SourceForge 上的一些功能请求.
由于要提供的模块的确切形式和所使用的类别或架构可能会受到争议,
因此本 PEP 旨在为这些讨论提供一个焦点.


Specification
=============

标准库中将添加两个模块: ``httpx`` (HTTP 扩展功能) 和 ``davlib`` (WebDAV 库).

[ 欢迎提出模块名称的建议; ``davlib`` 有一些优先权, 但可能需要像 ``webdav`` 这样的东西 ]


HTTP Authentication
-------------------

``httpx`` 模块将提供一个 mixin, 用于执行 HTTP 身份验证 (用于代理和源服务器身份验证).
这个 mixin (``httpx.HandleAuthentication``) 可以与 ``HTTPConnection``
和 ``HTTPSConnection`` 类结合使用 (mixin 可能适用于 HTTP 和 HTTPS 兼容类, 但这不是必需的).

mixin 将身份验证过程委托给一个或多个 "身份验证器" 对象, 允许多个连接共享身份验证器.
使用单独的对象允许与认证系统 (例如 LDAP) 的长期连接. 将提供基本和摘要机制的验证器 (参见 RFC 2617).
用户提供的身份验证器子类可以由连接注册和使用.

"凭证" 对象 (``httpx.Credentials``) 也与 mixin 相关联, 并存储验证者所需的凭证 (例如, 用户名和密码).
可以创建凭据的子类来保存其他信息 (例如, NT域).

mixin 覆盖 ``getresponse()`` 方法来检测 ``401 (Unauthorized)`` 和 ``407 (需要代理身份验证)`` 响应.
找到此项后, 响应对象, 连接和凭据将传递给与响应中指定的身份验证方案相对应的身份验证器
(如果响应中有多个方案, 则会按安全性的降序尝试多个身份验证器). 每个身份验证器都可以检查响应标题头,
并决定是否以及如何使用正确的身份验证标题头重新发送请求. 如果没有身份验证器可以成功处理身份验证, 则会引发异常.

使用适当的凭据重新发送请求是身份验证系统中较困难的部分之一. 在记录最初发送的内容时出现了困难: 请求行, 标题和正文.
通过重写 putrequest, putheader 和 endheaders, 我们可以捕获除 body 之外的所有东西. 一旦调用了 endheaders 方法,
那么我们捕获所有对 send() 的调用 (直到下一个 putrequest 方法调用) 来保存正文内容.
mixin 将具有以这种方式保持的数据量的可配置限制 (例如, 仅容纳高达100k的正文内容). 假设已存储整个正文,
那么我们可以使用适当的身份验证信息重新发送请求.

如果主体太大而无法存储, 则 ``getresponse()`` 只返回响应对象, 指示 401 或 407 错误. 由于身份验证信息已经计算并缓存
(进入 Credentials 对象; 见下文), 因此调用者可以简单地重新生成请求. mixin 将附加相应的凭据.

"保护空间" (参见 RFC 2617, 第1.2节) 被定义为主机, 端口和认证领域的元组. 当请求最初发送到 HTTP 服务器时,
我们不知道身份验证领域 (只有在身份验证失败时才返回领域). 但是, 我们确实有来自 URL 的路径,
这对于确定要发送到服务器的凭据非常有用. 基本身份验证方案通常是分层次设置的: ``/path`` 的凭据可以用于 ``/path/subpath``.
摘要式身份验证方案明确支持分层设置. ``httpx.Credentials`` 对象将存储多个保护空间的凭证, 并且可以通过两种不同的方式查找:

1. 查找使用 ``(host, port, path)`` -- 在生成对我们不知道认证领域的路径的请求时使用此查找方案.

2. 查找使用 ``(host, port, realm)`` -- 当服务器指定 Request-URI 位于特定领域内时, 在身份验证过程中使用此机制.

如果可用, ``HandleAuthentication`` mixin 将覆盖 ``putrequest()`` 来自动插入凭证.
putrequest 中的 URL 用于确定要使用的相应身份验证信息.

同样重要的是要注意使用两组凭证, 并由 mixin 存储. 一组可用于任何代理, 一组用于目标源服务器.
由于代理没有路径, 代理凭证中的保护空间将始终使用 "/" 来存储和通过路径查找.


Proxy Handling
--------------

``httpx`` 模块将提供一个 mixin, 用于使用代理执行 HTTP(S) 操作. 这个 mixin (``httpx.UseProxy``)
可以与 ``HTTPConnection`` 和 ``HTTPSConnection`` 类结合使用
(mixin 可能适用于 HTTP 和 HTTPS 兼容类, 但这不是必需的).

mixin 将记录要使用的代理的 ``(host, port)``. 将覆盖 XXX 来使用此主机和端口组合进行连接,
并将请求 URL 重写为引用源服务器的 absoluteURI (这些 URI 将传递给代理服务器).

代理身份验证由 ``httpx.HandleAuthentication`` 类处理,
因为用户可以直接使用 ``HTTP(S) Connection`` 与代理进行通信.


WebDAV Features
---------------

``davlib`` 模块将提供一个 mixin, 用于将 WebDAV 请求发送到启用 WebDAV 的服务器.
这个 mixin (``davlib.DAVClient``) 可以与 ``HTTPConnection`` 和 ``HTTPSConnection`` 类结合使用
(mixin 可能适用于 HTTP 和 HTTPS 兼容类, 但这不是必需的).

mixin 提供了执行 RFC 2616 中的 HTTP 定义的各种 HTTP 方法的方法, 以及 RFC 2518 中的 WebDAV.

自定义响应对象用于解码 ``207 (多状态)`` 响应. 响应对象将使用标准库的 xml 包来解析多状态 XML 信息,
从而生成一个简单的对象结构来保存多状态数据. 将按照速度降低的顺序尝试和使用多种解析方案.


Reference Implementation
========================

实际 (未来/最终) 实现正在 ``/nondist/sandbox/Lib`` 目录中开发, 直到它被接受并移动到主 Lib 目录中.


References
==========

.. _WebDAV: http://www.webdav.org/


Copyright
=========

This document has been placed in the public domain.



..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   fill-column: 70  
   sentence-end-double-space: t  
   End:  
