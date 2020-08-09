
PEP: 219
Title: Stackless Python
Version: $Revision$
Last-Modified: $Date$
Author: gmcm@hypernet.com (Gordon McMillan)
Status: Deferred
Type: Standards Track
Content-Type: text/x-rst
Created: 14-Aug-2000
Python-Version: 2.1
Post-History:


Introduction
============

本 PEP 讨论了核心 Python 所需的更改, 以便有效地支持生成器, 微线程和协同程序.
它与 PEP 220 有关, 它描述了如何扩展 Python 以支持这些功能.
本 PEP 的重点是严格按照允许这些扩展工作所需的更改.

虽然这些 PEP 基于 Christian Tismer 的 Stackless [1]_ 实现, 但他们并不认为 Stackless 是参考实现.
Stackless (带有扩展模块) 实现了延续, 并且从 continuation 中可以实现协程, 微线程
(如 Will Ware [2]_ 所做的那样) 和生成器. 但是在一年多的时间里, 没有人发现任何其他有效的继续使用,
所以似乎没有人要求他们的支持.

但是, Stackless 对 continuation 的支持是实现的一个相对较小的部分,
因此可以将其视为 "a" 参考实现 (而不是 "the" 参考实现).


Background
==========

生成器和协程已经以多种方式以多种语言实现. 实际上, Tim Peters 使用线程完成了
生成器 [3]_ 和 coroutines [4]_ 的纯 Python 实现 (并且 Java 存在基于线程的协程实现).
然而, 基于线程的实现的可怕开销严重限制了这种方法的有用性.

Microthreads (a.k.a "绿色" 或 "用户" 线程) 和协同程序涉及在基于单个堆栈的语言实现中难以适应的控制转移.
(生成器可以在单个堆栈上完成, 但它们也可以被视为一个非常简单的协同程序.)

真实线程为每个控制线程分配一个完整大小的堆栈, 这是开销的主要来源.
但是, 协程和微线程可以用几乎没有开销的方式在 Python 中实现.
因此, 这个 PEP 提供了一种方法, 使 Python 能够真实地管理数千个独立的活动 "线程"
(相比于今天可能有几十个单独的活动线程的限制).

这个 PEP 的另一个理由 (在 PEP 220 中探讨) 是协程和生成器通常允许比现在的 Python 更直接地表达算法.


Discussion
==========

首先要注意的是 Python, 虽然它将解释器数据 (正常的 C栈使用)
与堆栈上的 Python 数据 (解释程序的状态) 混合在一起,
但两者在逻辑上是分开的. 他们碰巧使用相同的堆栈.

一个真正的线程会接近一个进程大小的堆栈, 因为实现无法知道线程需要多少堆栈空间.
单个帧所需的堆栈空间可能是合理的, 但堆栈切换是一个神秘且不可移植的过程, C 不支持.

然而, 一旦 Python 停止将 Python 数据放在 C 堆栈上, 堆栈切换就变得容易了.

PEP 的基本方法是基于这两个想法. 首先, 将 C 的堆栈使用与 Python 的堆栈使用分开.
其次, 将每个帧与足够的堆栈空间相关联以处理该帧的执行.

在正常使用中, Stackless Python 具有正常的堆栈结构, 除了它被分成块.
但是在存在协程或微线程扩展的情况下, 这种相同的机制支持具有树结构的堆栈.
也就是说, 扩展可以支持正常 "调用/返回" 路径之外的帧之间的控制转移.


Problems
========

这种方法的主要困难是 C 调用 Python. 问题是 C 堆栈现在拥有字节码解释器的嵌套执行.
在这种情况下, 不允许协程或微线程扩展在字节码解释器的不同调用中将控制转移到帧. 
如果要完成一个帧并从错误的解释器退回到 C, 则可以删除 C 堆栈.

理想的解决方案是创建一种机制, 从而不需要嵌套执行字节代码解释器.
协程或微线程扩展的简单解决方案是识别情况并拒绝允许在当前调用之外进行传输.

我们可以将涉及 C 调用 Python 的代码分类为两个阵营: Python 的实现和 C扩展.
希望我们可以提供折衷方案: Python 的内部用法 (以及想要付出努力的 C扩展编写者) 将不再使用解释器的嵌套调用.
没有付出努力的扩展仍然是安全的, 但是与协程或微线程不能很好地兼容.

通常, 当递归调用转换为循环时, 需要一些额外的簿记. 循环将需要保持自己的参数和结果的 "堆栈",
因为真正的堆栈现在只能保持最新的. 代码将更加冗长, 因为当我们完成时它并不那么明显.
虽然 Stackless 没有以这种方式实现, 但它必须处理相同的问题.

在普通的 Python 中, ``PyEval_EvalCode``用于构建一个帧并执行它.
Stackless Python 引入了``FrameDispatcher``的概念. 就像``PyEval_EvalCode``一样, 它执行一帧.
但是解释器可能会向``FrameDispatcher``发出信号, 表示已经交换了一个新帧, 并且应该执行新帧.
当一个帧完成时, ``FrameDispatcher``跟在后面指针上以恢复 "调用" 帧.

所以 Stackless 将递归转换为循环, 但它不是管理帧的``FrameDispatcher``.
这是由解释器 (或知道它正在做什么的扩展) 完成的.

一般的想法是, C 代码需要执行 Python 代码, 它为 Python 代码创建一个框架, 将其指针设置为当前框架.
然后它在框架中交换, 发出 "FrameDispatcher" 信号, 然后离开.
C 堆栈现在是干净的 -- Python 代码可以将控制转移到任何其他帧 (如果扩展为它提供了这样做的方法).

在 vanilla 案例中, 这种魔法可以从程序员身上隐藏
(甚至在大多数情况下, 来自 Python-internals 程序员).
然而, 许多情况存在另一个难度.

map 内置功能涉及这种方法的两个障碍. 它不能简单地构造一个框架并且不受影响,
不仅因为涉及到一个循环, 而且每个循环都需要一些 "后期" 处理.
为了与他人合作, Stackless 为 map 本身构建了一个框架对象.

解释器的大多数递归并不复杂, 但是相当频繁地, 需要一些 "post" 操作.
Stackless 不能解决这些问题, 因为需要更改代码量. 相反, Stackless 禁止从嵌套解释器转移.
虽然不理想 (有时令人费解), 但这种限制几乎不会造成严重损害.


Advantages
==========

对于普通的 Python, 这种方法的优点是 C 堆栈使用变得更小, 更可预测.
Python 代码中的无限递归变成了内存错误, 而不是堆栈错误 (因此, 在非 Cupertino 操作系统中, 可以从中恢复).
当然, 代价是通过将字节码解释器循环的递归转换为更高阶循环 (以及随之而来的簿记) 而增加的复杂性.

最大的好处来自于认识到 Python 堆栈实际上是一棵树,
并且帧调度程序可以在树的叶节点之间自由地传递控制,
因此允许微线程和协同程序之类的东西.


References
==========

.. [1] http://www.stackless.com
.. [2] http://web.archive.org/web/20000815070602/http://world.std.com/~wware/uthread.html
.. [3] Demo/threads/Generator.py 在源代码 (发行版) 中
.. [4] http://www.stackless.com/coroutines.tim.peters.html



..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
