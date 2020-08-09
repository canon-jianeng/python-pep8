
PEP: 160
Title: Python 1.6 Release Schedule
Version: $Revision$
Last-Modified: $Date$
Author: Fred L. Drake, Jr. <fdrake@acm.org>
Status: Final
Type: Informational
Content-Type: text/x-rst
Created: 25-Jul-2000
Python-Version: 1.6
Post-History:


Introduction
============

此 PEP 描述了 Python 1.6 发布计划. 此文件的 CVS 修订历史记录包含确定的历史记录.

此版本将由 BeOpen PythonLabs 员工为国家研究计划公司 (CNRI) 制作.


Schedule
========

* August 1: 1.6 beta 1 release (planned).
* August 3: 1.6 beta 1 release (actual).
* August 15: 1.6 final release (planned).
* September 5: 1.6 final release (actual).


Features
========

Python 1.6 需要许多功能才能实现已经做出的各种承诺.
以下内容必须完全可操作, 记录并向前兼容 Python 2.0 的计划:

* Unicode 支持: 必须提供为 Python 2.0 定义的 Unicode 对象, 包括所有方法和编解码器支持.

* SRE: Fredrik Lundh 的新正则表达式引擎将用于支持8位字符串和 Unicode 字符串.
  它必须通过用于 re 模块的基于 pcre 的版本的回归测试.

* curses 模块正在转换为包, 因此采用了最终形式.


Mechanism
=========

该版本将作为 2001年5月16日 CNRI 业务结束时开发树的分支创建.
最近签入所需的补丁将通过尽可能在单个文件上移动分支标记来合并,
以减少邮件列表的混乱并避免分歧和不兼容的实现.

分支标签是 "cnri-16-start".

修补程序和功能将合并到通过 2000年5月16日 生效的回归测试所需的范围内.

测试版在 CVS 存储库中标记为 "r16b1", 最终的 Python 1.6 版本在存储库中标记为 "release16".


Copyright
=========

本文档已置于公共域名.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
