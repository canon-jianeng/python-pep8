
PEP: 3
Title: Guidelines for Handling Bug Reports
Version: $Revision$
Last-Modified: $Date$
Author: jeremy@alum.mit.edu (Jeremy Hylton)
Status: Withdrawn
Type: Process
Content-Type: text/x-rst
Created: 25-Sep-2000
Post-History:


Introduction
============

此 PEP 包含在 Python 错误跟踪器中处理错误报告的指南
它已被开发人员指南描述的问题分类所取代

:: https://docs.python.org/devguide/triaging.html

人们提交 Python 错误的指南是

:: http://docs.python.org/bugs.html

Original Guidelines
===================

1. 确保错误类别和错误组正确无误, 如果它们是正确的, 那么有兴趣帮助找出所有开放的 Tkinter 错误的人会更容易
2. 如果这是您不打算立即解决的次要功能请求, 请将其添加到 PEP 42 或要求所有者为您添加
   如果您将错误添加到 PEP 42, 请将错误标记为 "feature request", "later" 和 "closed"; 并在 bug 中添加注释, 说明情况就是这样 (明确提到 PEP)
   ::
        我们更喜欢 the tracker 还是 PEP 42?
3. 为错误分配合理的优先级, 我们还不清楚每个优先级应该意味着什么
   但是, 有一条规则是, 必须在下一次发布之前修复的优先级 "urgent" 或更高的错误
4. 如果错误报告没有足够的信息来允许您重现或诊断它, 请向原始提交者询问更多信息
   如果原始报告非常薄, 并且您的电子邮件在合理的等待期后没有收到回复, 则可以关闭该错误
5. 如果您修复了错误, 请将状态标记为 "Fixed" 并将其关闭, 在注释中, 包括提交的 SVN 修订号,
   在 SVN 签入消息中, 包括问题编号 **and** 正常的更改描述, 如果应用了修补程序, 则提及贡献者
6. 如果您被分配了一个您无法处理的错误, 如果您认为他们能够处理它, 请将其分配给其他人, 否则最好取消分配它

References
==========

.. [1] http://bugs.python.org/
