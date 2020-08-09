
PEP: 217
Title: Display Hook for Interactive Use
Version: $Revision$
Last-Modified: $Date$
Author: moshez@zadka.site.co.il (Moshe Zadka)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 31-Jul-2000
Python-Version: 2.1
Post-History:


Abstract
========

Python 的交互模式是实现的强大优势之一 -- 能够在命令行上编写表达式并获得有意义的输出.
但是, 输出功能对所有人来说都不是万能的, 而且当前的输出功能往往达不到这个目标.
这个 PEP 描述了一种在 Python 中提供内置显示功能的替代方法, 因此用户可以控制交互式解释器的输出.


Interface
=========

当前的 Python 解决方案适用于许多用户, 这不应该破坏它.
因此, 在默认配置中, REPL 循环中不会发生任何变化.
要更改解释器打印交互式输入的表达式, 用户的方式必须将``sys.displayhook``重新绑定到可调用对象.
使用交互式输入表达式的结果调用此对象的结果应该是可打印的, 这将打印在``sys.stdout``上.


Solution
========

字节码``PRINT_EXPR``将调用``sys.displayhook(POP())``.
将``displayhook()``添加到 sys 内置模块中, 相当于::

    import __builtin__
    def displayhook(o):
        if o is None:
            return
        __builtin__._ = None
        print `o`
        __builtin__._ = o


Jython Issues
=============

方法``Py.printResult``将被类似地改变.


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
