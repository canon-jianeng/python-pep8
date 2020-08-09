
PEP: 271
Title: Prefixing sys.path by command line option
Version: $Revision$
Last-Modified: $Date$
Author: fred@arakne.com (Frédéric B. Giacometti)
Status: Rejected
Type: Standards Track
Content-Type: text/x-rst
Created: 15-Aug-2001
Python-Version: 2.2
Post-History:


Abstract
========

目前, 设置 ``PYTHONPATH`` 环境变量是定义其他 Python 模块搜索目录的唯一方法.

这个 PEP 在 python 命令中引入了 '-P' 值选项, 作为 ``PYTHONPATH`` 的替代.


Rationale
=========

在 Unix 上::

    python -P $SOMEVALUE

将等同于::

   env PYTHONPATH=$SOMEVALUE python

在 Windows 2K 上::

    python -P %SOMEVALUE%

将 (几乎) 等同于::

   set __PYTHONPATH=%PYTHONPATH% && set PYTHONPATH=%SOMEVALUE%\
      && python && set PYTHONPATH=%__PYTHONPATH%


Other Information
=================

此选项等同于 'java -classpath' 选项.


When to use this option
=======================

例如, 该选项旨在简化 Python 并在测试或构建脚本中使用 Python 更加健壮.


Reference Implementation
========================

SourceForge 提供了一个实现此功能的补丁::

    http://sourceforge.net/tracker/download.php?group_id=5470&atid=305470&file_id=6916&aid=429614

随着补丁讨论::

    http://sourceforge.net/tracker/?func=detail&atid=305470&aid=429614&group_id=5470


Copyright
=========

This document has been placed in the public domain.



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  sentence-end-double-space: t  
  fill-column: 70  
  coding: utf-8  
  End:  
