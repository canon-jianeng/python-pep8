
PEP: 226
Title: Python 2.1 Release Schedule
Version: $Revision$
Last-Modified: $Date$
Author: Jeremy Hylton <jeremy@alum.mit.edu>
Status: Final
Type: Informational
Content-Type: text/x-rst
Created: 16-Oct-2000
Python-Version: 2.1
Post-History:


Abstract
========

本文档描述了 Python 2.0之后的开发和发布计划.
根据这个计划, Python 2.1 将于2001年4月发布.
该计划主要关注 PEP 大小的项目. 小错误修复和更改将在第一个测试版发布之前发生.


Release Schedule
================

暂定未来的发布日期

[漏洞修补发布日期在这里]

过去的发布日期:

- 17-Apr-2001: 2.1 final release
- 15-Apr-2001: 2.1 release candidate 2
- 13-Apr-2001: 2.1 release candidate 1
- 23-Mar-2001: Python 2.1 beta 2 release
- 02-Mar-2001: First 2.1 beta release
- 02-Feb-2001: Python 2.1 alpha 2 release
- 22-Jan-2001: Python 2.1 alpha 1 release
- 16-Oct-2000: Python 2.0 final release


Open issues for Python 2.0 beta 2
=================================

将默认单元测试框架添加到标准库.


Guidelines for making changes for Python 2.1
============================================

将根据 python-dev@python.org 邮件列表中的讨论修订指南和时间表.

PEP 系统是在 Python 2.0 开发周期的后期制定的, 许多变化并未遵循 PEP 1 中描述的过程.
然而, 2.1 的开发过程将遵循记录的 PEP 过程.

2.0 最终版的前八周将是设计和审核阶段. 到本期间结束时, 为 2.1 提议的任何 PEP 都应该准备好进行审查.
这意味着编写了 PEP 并在 python-dev@python.org 和 python-list@python.org 邮件列表上进行了讨论.

接下来的六周将用于审查 PEP 并实现和测试已接受的 PEP.
当这段时间停止时, 我们将会考虑结束任何不完整的 PEP.
在此期间即将结束时, 会有一个特性冻结在那里, PEP 的任何小的不值得的特征将不被接受.

在最终发布之前, 我们将进行六周的 beta 测试和一两个候选发布.


General guidelines for submitting patches and making changes
============================================================

提交更改时要有良好的判断力. 你应该知道我们的意思是什么,
或者我们不会给你提交权限 <0.5 wink>. 一些具有良好意义的具体例子包括:

- 做任何决策者告诉你的事情.

- 首先讨论 python-dev 上任何有争议的变化.
  如果您获得了很多+1票, 而且没有-1票, 请进行更改.
  如果你得到一些-1票, 请三思而后行; 考虑问 Guido 他的想法.

- 如果更改是您贡献的代码, 那么修复它可能是有意义的.

- 如果更改影响了其他人编写的代码, 那么首先询问他或她可能是有意义的.

- 您可以使用 SourceForge（SF）Patch Manager 提交补丁并将其分配给某人进行审核.

必须在 PEP 中描述任何重要的新功能, 并在检查之前获得批准.

任何重要的代码添加 (例如, 新模块或大型补丁) 都必须包含回归测试和文档的测试用例.
在测试和文档准备好之前, 不应该检查补丁.

如果您修复了一个错误, 您应该编写一个可以捕获该错误的测试用例.

如果您从 SF Patch Manager 提交补丁或修复 Jitterbug 数据库中的错误,
请务必在 CVS 日志消息中引用补丁或错误编号. 还要确保更改补丁管理器或错误数据库中的状态
(如果您有权访问错误数据库).

任何已检查的代码都不能使回归测试失败.
如果检查失败, 必须在24小时内修复, 否则将退回.

所有提供的 C 代码必须是 ANSI C. 如果可能, 请使用两个不同的编译器进行检查, 例如: gcc 和 MSVC.

所有贡献的 Python 代码都必须遵循 Guido 的 Python 风格指南.
http://www.python.org/doc/essays/styleguide.html

据了解, 任何贡献的代码都将在开源许可下发布.
如果无法以这种方式发布, 请不要贡献代码.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
