# @Pep     : 0002
# @Date    : 2018-04-15 09:00:00
# @Author  : Canon
# @Link    : https://www.python.org
# @Version : 3.6.1


PEP: 2
Title: Procedure for Adding New Modules
Version: $Revision$
Last-Modified: $Date$
Author: Martijn Faassen <faassen@infrae.com>
Status: Final
Type: Process
Content-Type: text/x-rst
Created: 07-Jul-2001
Post-History: 07-Jul-2001, 09-Mar-2002

PEP Replacement
===============

此 PEP 已被 Python 开发人员指南中的更新资料所取代 [1]

Introduction
============

Python 标准库为 Python 的成功做出了重大贡献
该语言附带 "batteries included", 因此, 仅使用标准库就可以轻松提高人们的工作效率
因此, 这个库随着语言的发展而变得非常重要, 并且支持和鼓励这种增长

对库的许多贡献不是由核心开发人员创建的, 而是来自 Python 社区的人员, 他们是特定领域的专家
此外, 社区成员也是标准库的用户, 在各种各样的环境中应用它
这使得社区有能力检测和报告 python 标准库的差距, 哪些缺少但应该添加的东西

新功能通常以新模块的形式添加到库中
该 PEP 将描述新模块的 *addition* 的过程。 PEP 4 涉及模块弃用程序;
从标准库中 *removal* 旧的和未使用的模块
最后还有 *changing* 现有模块的问题，以使图书馆演变完整
PEP 3 和PEP 5 就此提供了一些指导, 继续维护现有模块是决定是否将新模块添加到标准库的决定的一个组成部分
因此, 本 PEP 还介绍了与维护问题相关的概念（集成商，维护人员）

Integrators
===========

该系统集成商的一群人有以下职责:

* 他们确定拟议的贡献是否应该成为标准库的一部分
* 他们将接受的贡献整合到标准库中
* 他们制作标准库版本

这群人将是由 Guido 领导的 PythonLabs

Maintainer(s)
=============

对标准库的所有贡献都需要一个或多个维护者
这可以是个人，但它通常是一组人，例如 XML-SIG, 可以在它们之间细分维护任务
一个或多个维护者应该是 *head maintainer* (通常这也是主要的开发人员)
如果想要解决特定问题 (例如, 本文档后面详述的那些问题), 则头部维护人员是集成商可以解决的方便人员

Developers(s)
=============

一个或多个开发人员已经开发了对标准库的贡献
最初的维护者是原始的开发人员, 除非有特殊情况 (应该在 PEP 中详细说明提出贡献)

Acceptance Procedure
====================

当开发人员希望在标准库中接受一个贡献时, 他们将首先形成一组维护者 (通常最初由他们自己组成)

然后, 该组将产生一个称为 library PEP 的 PEP
library PEP 是 standards track PEP 的一种特殊形式
library PEP 概述了拟议的贡献, 以及作为参考实施的拟议贡献
library PEP 还应该包含为何该贡献应成为标准库的一部分的动机

一个或多个维护者应该作为 PEP 拥护者向前迈进 (作者领域中列出的人员是拥护者)
PEP 拥护者将是最初的 head maintainer(s)

如 PEP 1 中所述, standards track PEP 应该包括设计文档和参考实现
library PEP 与普通 standard track PEP 的不同之处在于,
在 PEP 被审查以供集成商纳入并由社区评论之前, 参考实施应该始终已经写好;
参考实施 *is* 拟议的贡献

出于以下原因存在这种不同的要求:

* 当有源代码和文档要查看时, 集成商只能正确评估对标准库的贡献, 即参考实施总是必要的, 以帮助人们研究 PEP
* 即使被拒绝的贡献在标准库之外也是有用的, 因此, 开发人员浪费工作的风险会降低
* 它会给集成商留下严肃的贡献, 并有助于防止他们不得不评估太多无聊的建议

一旦提交了 library PEP 进行审核, 集成商就会对其进行评估
library PEP 将遵循 PEP 1 中描述的正常 PEP 工作流程
如果 PEP 被接受, 他们将通过负责人维护人员为整合做好准备

Maintenance Procedure
=====================

在接受了贡献之后, 整合者和维护者的工作都没有结束
集成商会将标准库中的任何错误报告转发给相应的 head maintainers

在功能冻结准备发布标准库之前, 集成商将与主管维护人员核实所有贡献, 用来查看是否有任何更新要包含在下一版本中
集成商将针对诸如向后兼容性等问题评估任何此类更新, 如果认为更改很大, 则可能需要 PEPs

head maintainers 应该积极参与 Python 开发过程, 如果 head maintainer 无法用这种方式运行,
他应该打算向集成商和其他维护人员宣布休息，以便替换人员可以继续
集成商应该始终能够通过电子邮件联系 head maintainers

在没有找到 head maintainer 的情况下(可能因为没有维护者), 集成商将向整个社区发出呼叫, 要求新的维护人员继续
如果没有人, 则集成商可以决定按照 PEP 4 中的描述申报弃用的贡献

Open issues
===========

需要有一些程序, 以便集成商可以随时联系维护人员 (或至少是 head maintainers)
这可以通过邮件列表来完成，所有 head maintainers 都应该订阅 (这可能是python-dev)
另一种可能在任何情况下都有用的可能性是维护类似于 PEP 列表的列表, 列出所有贡献及其 head maintainers 的联系信息
事实上, 这可能是 PEP 列表的一部分, 因为新的贡献需要 PEP
但是, 由于 PEP 引入新模块的作者或所有者, 最终可能与维护它的人不同, 因此这还不能解决所有问题

是否应该列出集成商用于评估贡献的标准?
(源代码以及文档和测试套件等内容，以及诸如 "维护者的可靠性" 之类的模糊内容)

这涉及所有技术问题: 签入权限，编码样式要求，文档要求，测试套件要求, 这些优选是另一个 PEP 的一部分

当前的标准库是否应该细分为维护者?
许多部分已经拥有（非正式）维护者, 使这更明确可能是好的

或许对 "贡献" 有更好的说法, "贡献" 一词可能并不意味着在文稿被接受并整合到存储库之后, (开发和维护)过程不会停止

与虚构目录的关系?

References
==========

.. [1] 添加到 Stdlib （http://docs.python.org/devguide/stdlibchanges.html）

Copyright
=========

本文档已置于公共域名

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   fill-column: 70  
   coding: utf-8  
   End:  
