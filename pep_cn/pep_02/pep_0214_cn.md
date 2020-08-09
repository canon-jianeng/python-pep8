
PEP: 214
Title: Extended Print Statement
Version: $Revision$
Last-Modified: $Date$
Author: barry@python.org (Barry Warsaw)
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 24-Jul-2000
Python-Version: 2.0
Post-History: 16-Aug-2000


Introduction
============

此 PEP 描述了扩展标准 'print' 语句的语法, 以便它可以用于打印到任何类似文件的对象,
而不是默认的``sys.stdout``. 此PEP跟踪此功能的状态和所有权. 它包含功能的说明,
并概述了支持该功能所需的更改. 本 PEP 总结了在邮件列表论坛中进行的讨论,
并在适当情况下提供了 URL 以获取更多信息. 此文件的 CVS 修订历史记录包含确定的历史记录.


Proposal
========

该提议引入了 print 语句的语法扩展, 允许程序员可选地指定输出文件目标. 示例用法如下 ::

    print >> mylogfile, 'this message goes to my log file'

形式上, 扩展 print 语句的语法是 ::

    print_stmt: ... | '>>' test [ (',' test)+ [','] ] )

省略号表示原始 print_stmt 语法不变. 在扩展形式中,
紧跟在 >> 之后的表达式必须产生一个带有``write()``方法的对象
(即一个类似文件的对象). 因此这两个陈述是等价的 ::

    print 'hello world'
    print >> sys.stdout, 'hello world'

正如这两个陈述一样 ::

    print
    print >> sys.stdout

这两个语句是语法错误 ::

    print ,
    print >> sys.stdout,


Justification
=============

'print' 是一个 Python 关键字, 并引入了语言参考手册 [1]_ 的 6.6 节中描述的 print 语句.
print 语句具有许多功能:

- 它会将项目自动转换为字符串
- 它会自动在项目之间插入空格
- 除非语句以逗号结尾, 否则它会附加换行符

print 语句执行的格式是有限的; 为了更好地控制输出,
可以使用``sys.stdout.write()``和字符串插值的组合.

print 语句按定义输出到``sys.stdout``. 更具体地说, ``sys.stdout``必须是一个类似文件的对象
带有``write()``方法, 但它可以反弹以将输出重定向到特定标准输出以外的文件. 一个典型的习语是 ::

    save_stdout = sys.stdout
    try:
        sys.stdout = mylogfile
        print 'this message goes to my log file'
    finally:
        sys.stdout = save_stdout

这种方法的问题是绑定是全局的, 因此会影响 try: 子句中的每个语句.
例如, 如果我们添加了对实际想要打印到 stdout 的函数的调用, 那么此输出也会被重定向到日志文件.

这种方法对于将打印交错到各种输出流也非常不方便,
并且在面对合法的 try/except 或 try/finally 子句时使编码复杂化.


Reference Implementation
========================

SourceForge 的补丁管理器 [2]_ 上提供了一个参考实现, 它以 Python 2.0 源代码树的补丁形式出现.
这种方法增加了两个新的操作码, ``PRINT_ITEM_TO``和``PRINT_NEWLINE_TO``,
它只是将文件像对象一样从堆栈的顶部弹出并使用它代替``sys.stdout``作为输出.

(Python 2.0 中采用了此参考实现.)


Alternative Approaches
======================

已经提出了这种语法改变的替代方案 (最初由 Moshe Zadka 提出), 其不需要对 Python 进行语法改变.
可以提供一个``writeln()``函数 (可能作为内置函数), 它的作用就像扩展打印一样, 还有一些额外的功能 ::

    def writeln(*args, **kws):
        import sys
        file = sys.stdout
        sep = ' '
        end = '\n'
        if kws.has_key('file'):
            file = kws['file']
            del kws['file']
        if kws.has_key('nl'):
            if not kws['nl']:
                end = ' '
            del kws['nl']
        if kws.has_key('sep'):
            sep = kws['sep']
            del kws['sep']
        if kws:
            raise TypeError('unexpected keywords')
        file.write(sep.join(map(str, args)) + end)

``writeln（）``接受三个可选的关键字参数. 在该提议的上下文中,
相关的参数是'file', 可以使用``write()``方法将其设置为类文件对象. 从而::

    print >> mylogfile, 'this goes to my log file'

将写成 ::

    writeln('this goes to my log file', file=mylogfile)

``writeln()`` 具有附加功能, 关键字参数 'nl' 是指定是否附加换行符的标志,
以及参数'sep', 它指定在每个项目之间输出的分隔符.


More Justification by the BDFL
==============================

该提案受到新闻组的质疑. 一系列挑战不喜欢 '>>', 而是宁愿看到其他一些符号.

* 挑战: 为什么不选择其中之一?

  ::

    print in stderr items,....
    print + stderr items,.......
    print[stderr] items,.....
    print to stderr items,.....

  回答: 如果我们想使用特殊符号 (``print <symbol>``expression),
  Python 解析器要求它不是一个可以启动表达式的符号 -- 否则它无法决定哪种形式的打印语句被使用.
  (Python 解析器是一个简单的 LL(1) 或递归下降解析器.)

  这意味着我们不能使用 "import as" 中使用的 "仅在上下文中处理关键字",
  因为标识符可以启动表达式. 这排除了 +stderr, \[sterr\] 和 stderr.
  它给我们留下了二进制运算符符号和其他当前非法的冗杂符号, 例如 'import'.

  如果我必须在 "打印文件" 和 "打印" 文件之间做出选择, 我肯定会选择 "">>".
  部分是因为 'in' 将是一个新发明 (我知道没有其他语言使用它, 而 '>>' 用于 sh, awk, Perl 和 C++),
  部分是因为 '>>', 非字母表, 更突出, 更有可能引起读者的注意.

* 挑战: 为什么文件和其他文件之间必须有逗号?

  回答: 将文件与以下表达式分开的逗号是必要的! 当然, 您希望文件是一个任意表达式,
  而不仅仅是一个单词. (你肯定希望能够编写``print >> sys.stderr``.)
  如果没有表达式, 解析器将无法区分表达式的结束位置和下一个表达式的开始位置, e.g.

  ::

      print >>i +1, 2
      print >>a [1], 2
      print >>f (1), 2

* 挑战: 为什么需要语法扩展? 为什么不用 writeln(file, item, ...)?

  回答: 首先, 这缺少 print 语句的一个特性: 想要打印的尾随逗号, 它会抑制最终的换行符.
  请注意, 'print a' 仍然不等于 'sys.stdout.write(a)' -- print 在项之间插入一个空格,
  并将任意对象作为参数; ``write()``不插入空格并需要单个字符串.

  当您考虑 print 语句的扩展时, 添加一个在一个维度 (输出所在的位置) 添加新特征
  但在另一个维度 (项目之间的空格以及尾随的选择中删除的功能或方法是不正确的. 换行与否). 我们可以添加一大堆方法或函数来处理各种情况, 但这似乎增加了比必要更多的混乱,
  并且只有在我们完全弃用 print 语句时才有意义.

  我觉得这场辩论的确是关于 print 是否应该是一种功能或方法, 而不是一种陈述.
  如果您在函数阵营中, 当然在现有的 print 语句中添加特殊语法并不是您喜欢的.
  我怀疑对新语法的反对主要来自那些已经认为 print 语句不好的人. 我对吗?

  大约10年前, 我与自己辩论是否从输出功能或声明中做出最基本的功能;
  基本上我试图在 "print(item, ...)" 和 "print item, ..." 之间做出决定.
  我选择将其作为声明, 因为 print 需要尽早开始教学, 并且在初学者编写的程序中非常重要.
  此外, 因为为这么多事情引领潮流的 ABC 也是一个声明. 在 ABC 和 Python 之间交互的典型举动中,
  我将名称从 WRITE 更改为 print, 并颠倒了添加换行符的约定, 要求使用额外的语法添加换行符
  (ABC 使用尾部斜杠来表示换行符)来要求额外的语法 (尾随逗号) 来压制换行符.
  我保留了输出上的空格分隔项目的功能.

  Full example: in ABC,

  ::

      WRITE 1
      WRITE 2/

  具有相同的效果 ::

      print 1,
      print 2

  在 Python 中, 输出有效 "1 2\n".

  我不是百分之百确定声明的选择是正确的
  (ABC有令人信服的理由, 它使用语句语法来处理任何有副作用的东西, 但 Python 没有这个约定),
  但我也不相信这是不对的. 我当然喜欢 print 声明的经济性. (我是狂热的 Lisp-hater -- 语法方面,
  而不是语义方面! -- 语法中过多的括号让我烦恼. 不要写``return(i) 或 if(x == y):``在你的 Python 代码中! :-)

  无论如何, 我还没准备好弃用 print 语句, 多年来我们已经有很多请求指定文件的选项.

* 挑战: 为什么不 > 而不是 >>?

  回答: 对于 DOS 和 Unix 用户, >> 建议 "追加", 而 > 建议 "覆盖";
  语义最接近追加. 此外, 对于 C++ 程序员, >> 和 << 是 I/O 运算符.

* 挑战: 但是在 C++ 中, >> 输入, << 输出!

  回答: 没关系; C++ 显然是从 Unix 中取出并反转了箭头.
  重要的是, 对于输出, 箭头指向文件.

* 挑战: 当然你可以设计一个``println()``函数可以做所有``print >> file``可以做的事情; 为什么不够呢?

  回答: 我从一个简单的编程练习中想到了这一点. 假设一个初级程序员被要求编写一个打印乘法表的函数.
  合理的解决方案是 ::

      def tables(n):
          for j in range(1, n+1):
              for i in range(1, n+1):
                  print i, 'x', j, '=', i*j
              print

  现在假设第二个练习是将打印添加到另一个文件. 使用新语法,
  程序员只需要学习一个新东西:``print >> file``, 答案可以是这样的 ::

      def tables(n, file=sys.stdout):
          for j in range(1, n+1):
              for i in range(1, n+1):
                  print >> file, i, 'x', j, '=', i*j
              print >> file

  只有一个 print 语句和一个``println()``函数, 程序员首先要学习``println()``,
  将原始程序转换为``println()`` ::

      def tables(n):
          for j in range(1, n+1):
              for i in range(1, n+1):
                  println(i, 'x', j, '=', i*j)
              println()

  和 **then** 关于文件关键字参数 ::

      def tables(n, file=sys.stdout):
          for j in range(1, n+1):
              for i in range(1, n+1):
                  println(i, 'x', j, '=', i*j, file=sys.stdout)
              println(file=sys.stdout)

  因此, 转换路径更长 ::

      (1) print
      (2) print >> file

  vs.

  ::

      (1) print
      (2) println()
      (3) println(file=...)

  注意: 在编译时将文件参数默认为``sys.stdout``是错误的,
  因为当调用者分配给``sys.stdout``, 然后使用``tables()``时, 它不能正常工作指定文件.
  这是一个常见的问题 (也会出现``println()``函数. 迄今为止的标准解决方案一直是 ::

      def tables(n, file=None):
          if file is None:
              file = sys.stdout
          for j in range(1, n+1):
              for i in range(1, n+1):
                  print >> file, i, 'x', j, '=', i*j
              print >> file

  我在实现中添加了一个功能 (我也建议``println()``),
  如果文件参数是``None``, 则自动使用``sys.stdout``. 从而,

  ::

      print >> None, foo bar

  (当然, ``print >> x``, 其中 x 是一个值为 None 的变量) 表示与...相同

  ::

      print foo, bar

  并且``tables()``函数可以写成如下 ::

      def tables(n, file=None):
          for j in range(1, n+1):
              for i in range(1, n+1):
                  print >> file, i, 'x', j, '=', i*j
              print >> file

.. XXX 这需要更多的理由, 以及它自己的一部分


References
==========

.. [1] http://docs.python.org/reference/simple_stmts.html#print
.. [2] http://sourceforge.net/patch/download.php?id=100970


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
