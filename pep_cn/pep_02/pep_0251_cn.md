
PEP: 251
Title: Python 2.2 Release Schedule
Version: $Revision$
Last-Modified: $Date$
Author: barry@python.org (Barry Warsaw), guido@python.org (Guido van Rossum)
Status: Final
Type: Informational
Content-Type: text/x-rst
Created: 17-Apr-2001
Python-Version: 2.2
Post-History: 14-Aug-2001


Abstract
========

本文档描述了 Python 2.2 开发和发布计划. 该计划主要关注 PEP 大小的项目.
小错误修复和更改将在第一个测试版发布之前发生.

下面的时间表代表 Python 2.2 的实际发布日期.
请注意, Python 2.2 的任何后续维护版本都应该由单独的 PEP 涵盖.


Release Schedule
================

暂定未来的发布日期. 请注意, 与 2.2a1 版本发布的时间表相比, 我们已经下滑了.

* 21-Dec-2001: 2.2   [Released] (final release)
* 14-Dec-2001: 2.2c1 [Released]
* 14-Nov-2001: 2.2b2 [Released]
* 19-Oct-2001: 2.2b1 [Released]
* 28-Sep-2001: 2.2a4 [Released]
* 7-Sep-2001: 2.2a3 [Released]
* 22-Aug-2001: 2.2a2 [Released]
* 18-Jul-2001: 2.2a1 [Released]


Release Manager
===============

Barry Warsaw 是 Python 2.2 的发布经理.


Release Mechanics
=================

我们尝试了一种新的发布机制: 在每个 alpha, beta 或其他版本发布前一周,
我们分派了一个分支, 它成为发布版本. 对分支机构的更改仅限于发布经理及其指定的机器人.
该实验被认为是成功的, 应该在未来的版本中观察到. 有关实际的释放机制, 请参见 PEP 101 [1]_.


New features for Python 2.2
===========================

Python 2.2 中引入了以下新功能. 有关更详细的说明,
请参阅 Python 发行版中的 Misc/NEWS [2]_
或 Andrew Kuchling 的 "Python 2.2中的新功能" 文档 [3]_.

- iterators (PEP 234)
- generators (PEP 255)
- unifying long ints and plain ints (PEP 237)
- division (PEP 238)
- unification of types and classes (PEP 252, PEP 253)


References
==========

.. [1] PEP 101, 进行 Python 发布 101
       http://www.python.org/dev/peps/pep-0101/

.. [2] 来自 CVS 的 Misc/NEWS 文件
       http://cvs.sourceforge.net/cgi-bin/viewcvs.cgi/python/python/dist/src/Misc/NEWS?rev=1.337.2.4&content-type=text/vnd.viewcvs-markup

.. [3] Andrew Kuchling, Python 2.2 中的新功能
       http://www.python.org/doc/2.2.1/whatsnew/whatsnew22.html


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
