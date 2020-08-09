
PEP: 6
Title: Bug Fix Releases
Version: $Revision$
Last-Modified: $Date$
Author: aahz@pythoncraft.com (Aahz), anthony@interlink.com.au (Anthony Baxter)
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 15-Mar-2001
Post-History: 15-Mar-2001 18-Apr-2001 19-Aug-2004


Abstract
========

Python 历史上只有一个开发分支, 其中发布的目的是添加新功能和提供错误修复(这些类型的发布将被称为 "主要版本")
PEP 6 描述了如何保留分支, 修复bug, 旧版本的发布的主要目的是修复bug

PEP 6 保证 bug 修复版本的存在, 它只指定了一个程序
如果 Python 社区愿意完成这项工作, 那么需要修复bug

Motivation
==========

随着迁移到 SourceForge, Python 开发速度加快了
社区中有一种观点认为加速度太大, 许多人在升级到新版本以修复错误时感到不舒服, 因为添加了很多功能, 有时候在开发周期的后期

此问题的一个解决方案是维护以前的主要版本, 提供错误修复, 直到下一个主要版本
这应该使 Python 对企业开发更具吸引力, 因为 Python 可能需要在数百或数千台机器上安装

Prohibitions
============

Bug 修订版必须遵守以下限制：

1. 没有新功能, bug 修复版本的目的是修复 bug, 而不是从 CVS 根目录的 HEAD 中添加最新最好的 whizzo 功能
2. 是一个不会引起其他问题的升级, 用户应该确信从 2.x.y 升级到 2.x.(y+1) 不会破坏其运行系统
   这意味着, 除非有必要修复错误, 否则标准库不应该改变行为, 或者更糟糕的 API

Applicability of Prohibitions
=============================

上述禁止和非完全禁止适用于最终版本的错误修正版本(例如, 2.4到2.4.1) 和一个错误修正版本发布到系列中的下一个版本 (例如, 2.4.1到2.4.2)

遵循 PEP 6 中列出的禁令应该有助于让社区感到高兴的是, 错误修复版本是一种无痛且安全的升级

Helping the Bug Fix Releases Happen
===================================

这里有一些关于帮助 bug 修复发布过程的指示

1. Backport 错误修复, 如果您修复了一个错误, 并且看起来合适, 请将其移植到 CVS 分支以获取当前的错误修复版本
   如果您不愿意或无法自行向后移植, 请在提交消息中记下 "Bugfix candidate" 或 "Backport candidate" 等字样
2. 如果你不确定且他们认为特定的修复是合适的, 请询问管理当前错误修复版本的人
3. 如果有一个特定的错误, 你特别喜欢在 bug 修复版中修复, 跳上跳下并尝试完成它
   不要等到错误修复发布到期前48小时, 然后开始要求包含错误修复

Version Numbers
===============

从 Python 2.0 开始, 所有主要版本都需要具有 X.Y 形式的版本号, bugfix 版本将始终为 X.Y.Z 形式

目前正在开发的主要版本被称为版本N, 刚刚发布的主要版本称为N-1

在 CVS 中, 错误修复版本发生在分支上
对于版本 2.x, 分支名为 "release2x-maint"
例如, python 2.3 维护版本的分支是 release23-maint

Procedure
=========

管理 bugfix 版本的过程部分基于 Tcl 系统建模 [1]

Patch Czar 是针对错误修复版本的 BDFL 的对应物
然而, BDFL 和指定的任命人员对个别补丁保留否决权
Patch Czar 可能只关注一个开发分支, 很可能不同的人可能正在维护 python 2.3.x 和 2.4.x 版本

当单个补丁被贡献给 CVS 的当前主干时, 要求每个补丁提交者考虑补丁是否是适合包含在错误修复版本中的错误修复
如果认为补丁是合适的, 则提交者可以将释放提交到维护分支, 或者在提交消息中标记补丁

此外, Python 社区的任何人都可以自由地建议包含补丁, 可以专门为 bugfix 版本提交补丁, 他们应该遵循 PEP 3 [2] 中的指导原则
但是, 一般情况下, 特定版本中的错误也可能更好地修复在 HEAD 和分支上

Patch Czar 决定何时有足够数量的补丁来保证版本
该版本已打包, 包括 Windows 安装程序, 并公开发布
如果发现任何新错误, 必须立即修复它们并公布新的错误修复版本(增加版本号)
对于 python 2.3.x 周期, Patch Czar (Anthony) 大约每六个月尝试一次发布, 但不应将其视为对任何未来版本的任何约束

预计错误修复版本将在大约六个月的时间间隔内发布
然而, 这只是一个指导原则, 如果发现一个主要错误, 错误修复版本可能会更快
通常, 只有 N-1 版本可以随时进行主动维护, 也就是说, 在 Python 2.4 的开发过程中, Python 2.3 获得了 bugfix 版本
但是, 如果某人有资格继续工作以维持旧版本, 则应该鼓励他们

Patch Czar History
==================

Anthony Baxter 是 2.3.1 到 2.3.4 的补丁处理者
Barry Warsaw 是 2.2.3 的补丁处理者
Guido van Rossum 是 2.2.2 的补丁处理者
Michael Hudson 是 2.2.1 的补丁处理者
Anthony Baxter 是 2.1.2 和 2.1.3 的补丁处理者
Thomas Wouters 是 2.1.1 的补丁处理者
Moshe Zadka 是 2.0.1 的补丁处理者

History
=======

这个 PEP 最初是作为 comp.lang.python 的提案开始的
原始版本建议 N-1 版本的单个补丁与 N 版本同时发布
原始版本还认为坚持严格的错误修复策略

根据 BDFL 和其他人的反馈, PEP 草案包含一个扩展的错误修正发布周期, 允许任何以前的主要版本获得补丁并放宽严格的错误修复要求
(主要是由于 PEP 235 [3] 的例子, 这可以被认为是错误修复或功能)

然后讨论主要转移到 python-dev, 其中 BDFL 最终发布了一个基于 Tcl 的 Python 错误修复发布过程的公告,
它基本上只返回到原始提案, 只是版本 N-1 和只有错误修复但允许多个错误修复的发布版本 N 发布

然后, 根据 python 2.3 发布周期的经验, Anthony Baxter 接受了这个 PEP 并对其进行了修订

References
==========

.. [1] http://www.tcl.tk/cgi-bin/tct/tip/28.html


.. [2] PEP 3, 处理错误报告的指南, Hylton
    (http://www.python.org/dev/peps/pep-0003/)

.. [3] PEP 235, 在 Case-Insensitive 平台上导入, Peters
    (http://www.python.org/dev/peps/pep-0235/)

Copyright
=========

本文档已置于公共域名

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
