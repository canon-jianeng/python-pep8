
PEP: 5
Title: Guidelines for Language Evolution
Version: $Revision$
Last-Modified: $Date$
Author: paul@prescod.net (Paul Prescod)
Status: Active
Type: Process
Content-Type: text/x-rst
Created: 26-Oct-2000
Post-History:


Abstract
========

在编程语言的自然演化中，有时需要进行修改旧程序的行为
PEP 5 提出了一种策略, 用于以尊重已安装的 Python 用户群的方式实现这些更改

Implementation Details
======================

实施 PEP 5 需要增加一个正式的警告和弃用工具, 将在另一个提案中进行描述

Scope
=====

这些指南适用于引入向后不兼容行为的 Python 的未来版本
向后不兼容的行为是 Python 解释与 Python 标准文档中描述的早期行为的主要偏差, 删除特征也构成了行为的改变

PEP 5 不会替换或排除其他兼容性策略, 例如, 向后兼容解析器的动态加载
另一方面, 如果执行 "旧代码" 需要特殊的开关或编译指示, 那么从用户的角度来看确实是行为的改变, 并且应该根据这些指南实现改变

一般而言, 执行这些准则必须具备常识
例如, 更改 "sys.copyright" 并不构成向后兼容的行为变化!

Steps For Introducing Backwards-Incompatible Features
=====================================================

1. 在 PEP 中提出向后兼容的行为, PEP 必须包含一个关于向后兼容性的部分, 该部分详细描述了完成其余步骤的计划
2. 将 PEP 作为生产方向接受后, 实施另一种方法来完成先前由要删除或更改的功能提供的任务
   例如, 如果计划删除加法运算符, 则新版本的 Python 可以实现 "add()" 内置函数
3. 正式弃用 Python 文档中的过时构造
4. 向解析器添加可选的警告模式, 该模式将在使用不推荐的构造时通知用户
   换句话说, 将来表现不同的所有程序都必须在此模式下触发警告
   编译时警告优于运行时警告, 警告消息应该将人们从已弃用的构造引导到替代构造
5. 在过渡版本的 Python 发布和向后不兼容版本的发布之间必须至少有一年的过渡期
   用户至少需要一年时间来测试他们的程序, 并将他们从使用已弃用的构造转移到另一个构建

..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
