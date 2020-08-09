
PEP: 200
Title: Python 2.0 Release Schedule
Version: $Revision$
Last-Modified: $Date$
Author: Jeremy Hylton <jeremy@alum.mit.edu>
Status: Final
Type: Informational
Content-Type: text/x-rst
Created:
Python-Version: 2.0
Post-History:



Introduction
============

此 PEP 描述了 Python 2.0 发布计划, 跟踪主要新功能的状态和所有权,
总结了在邮件列表论坛中进行的讨论, 并提供了有关更多信息,
补丁和其他未解决问题的 URL. 此文件的 CVS 修订历史记录包含确定的历史记录.

Release Schedule
================

[revised 5 Oct 2000]


* 26-Sep-2000: 2.0 beta 2
* 9-Oct-2000: 2.0 release candidate 1 (2.0c1)
* 16-Oct-2000: 2.0 final

Previous milestones
===================

* 14-Aug-2000: All 2.0 PEPs finished / feature freeze
* 5-Sep-2000: 2.0 beta 1

What is release candidate 1?
============================

我们相信候选版本1将修复我们打算为2.0最终版本修复的所有已知错误. 此版本应该比以前的测试版更稳定.
我们希望在最终发布之前看到更广泛的测试, 因此我们正在制作此候选版本.
最终版本将完全相同, 除非发布候选人的测试人员发现任何显示停止 (或棕色包) 的错误.

Guidelines for submitting patches and making changes
====================================================

提交更改时要有良好的判断力. 你应该知道我们的意思是什么,
或者我们不会给你提交权限 <0.5 wink>. 一些具有良好意义的具体例子包括:

* 做任何独裁者告诉你的事情.

* 首先讨论 python-dev 上任何有争议的变化. 如果您获得了很多+1票, 而且没有-1票,
  请进行更改. 如果你得到一些-1票, 请三思而后行; 考虑询问 Guido 的想法.

* 如果更改是您贡献的代码, 那么修复它可能是有意义的.

* 如果更改影响了其他人编写的代码, 那么首先询问他或她可能是有意义的.

* 您可以使用 SF Patch Manager 提交补丁并将其分配给某人进行审核.

必须在 PEP 中描述任何重要的新功能, 并在签入之前获得批准.

任何重要的代码添加 (例如, 新模块或大型补丁) 都必须包含回归测试和文档的测试用例.
在测试和文档准备好之前, 不应该签入补丁.

如果您修复了一个错误, 您应该编写一个可以捕获该错误的测试用例.

如果您从 SF Patch Manager 提交补丁或修复 Jitterbug 数据库中的错误,
请务必在 CVS 日志消息中引用补丁或错误编号.
还要确保更改补丁管理器或错误数据库中的状态 (如果您有权访问错误数据库).

任何已签入的代码都不能使回归测试失败.
如果签入失败, 必须在24小时内修复, 否则将退回.

所有提供的 C 代码必须是 ANSI C. 如果可能, 请使用两个不同的编译器进行检查,
例如: gcc 和 MSVC.

所有贡献的 Python 代码都必须遵循 Guido 的 Python 风格指南.
http://www.python.org/doc/essays/styleguide.html

据了解, 任何贡献的代码都将在开源许可下发布.
如果无法以这种方式发布, 请不要贡献代码.


Failing test cases need to get fixed
====================================

我们需要快速解决回归测试套件中的错误. 除非回归测试在应用更改的情况下干净地运行,
否则不应该将更改提交到 CVS 树. 如果失败, 代码中可能会出现漏洞.
(无论如何可能存在错误, 但这是另一回事.) 如果已知测试用例失败, 则它们没有用处.

::

    test case         platform    date reported
    ---------         --------    -------------
    test_mmap         Win ME      03-Sep-2000       Windows 2b1p2 prelease
        [04-Sep-2000 tim
         由 Audun S. Runde 报道: audun@mindspring.com mmap
         构造函数失败 w/
            WindowsError: [Errno 6] The handle is invalid
         由于没有其他版本的 Windows 失败的报告, 这看起来像是一个 ME 错误
        ]

Open items -- Need to be resolved before 2.0 final release
==========================================================

确定默认情况下是否应该启用 cycle-gc.

解决核心 xml 包和 XML-SIG XML 包之间的兼容性问题.

更新工具/编译器, 使其与列表推导, 导入和任何其他新语言功能兼容.

改善测试套件的代码覆盖率.

完成为使用 2.0b1 的功能编写 PEP (可悲, 但现实 -- 我们会通过练习变得更好).

将 bug 数据库缩小到一定程度的主要工作. 我 (tim) 之前看过这个:
如果你能把所有开放的 bug 都放在一个屏幕上, 那么人们通常都会这样.
但是让它在屏幕上流淌一个月, 它只是下地狱 (确实没有 "明显的进展"!).

Accepted and in progress
========================

* 目前没人离开. [4-Sep-2000 guido]

Open: proposed but not accepted or rejected
===========================================

* 再次有许多开放补丁. 我们需要尽快清除这些问题.

Previously failing test cases
=============================

如果您发现此部分与前一部分之间的测试反弹, 则它正在测试的代码出现问题！

::

    test case         platform    date reported
    ---------         --------    -------------
    test_fork1        Linux       26-Jul-2000
        [28-aug-2000 由 cgw 修复; 解决方案是在子进程中创建锁的副本]
        [19-Aug-2000 tim
         Charles Waldman 增强了一个补丁, 为子流程提供了新的
         "全局锁":
         http://sourceforge.net/patch/?func=detailpatch&patch_id=101226&group_id=5470
         虽然这似乎没有解决我们 *看到* 的症状, 但它 *确实* 到目前为止似乎正在修复失败的案例
        ]

    test_parser       all         22-Aug-2000
    test_posixpath    all         22-Aug-2000

    test_popen2       Win32       26-Jul-2000
        [31-Aug-2000 tim
         它再次死亡, 但出于完全不同的原因: 它使用 dict 将文件指针映射到进程句柄,
         并在 popen.close() 期间调用 dict 访问函数. 但是 .close 释放线程,
         这使得内部 popen 代码访问 dict 而没有有效的线程状态. dict 实现已更改, 因此不再接受.
         通过在 popen 的关闭例程的内部创建临时线程状态来修复, 并在此期间使用它获取全局锁定]
        [20-Aug-2000 tim
         在 os.name == "nt" 时更改了 popen2.py _test 函数来使用 "more" cmd.
         这使得 test_popen2 在 Win98SE 下传递.
         然而, Win98 "更多" 凭空创造了一条领先的新品, 我不确定其他 Windows 版本的 "更多" 也会这样做.
         所以, 其他人请尝试其他 Windows 风格!
        ]
        [still fails 15-Aug-2000 for me, on Win98 - tim
             test test_popen2 crashed -- exceptions.AssertionError:
         问题是测试使用 "cat", 但在 Windows 下没有这样的东西 (除非你安装它).
         所以这是在这里打破的测试, 而不是 (必然) 代码.
        ]

    test_winreg        Win32      26-Jul-2000
        [works 15-Aug-2000 for me, on Win98 - tim]

    test_mmap          Win32      26-Jul-2000
        [believe that was fixed by Mark H.]
        [works 15-Aug-2000 for me, on Win98 - tim]

    test_longexp      Win98+?     15-Aug-2000
        [发布版本失败,
         在详细模式下传递版本, 但看起来不应该通过,
         传递调试版本,
         在详细模式下传递调试版本, 看起来应该通过
        ]
        [18-Aug-2000, tim: 无法重现, 没有人看到它.
         我相信在使用 -v 时, *是* 在 regrtest.py 中的一个微妙的错误, 我会追寻它,
         但不能再引发 test_longexp 的任何错误; Fred 的变化也不能找出可疑的
         19-Aug-2000, tim: regrtest.py -v 中的 "细微错误"
         实际上是一个特性: -v 掩盖 *一些* 种类的失败, 因为它不会将测试输出与固定输出进行比较;
         即使在没有 -v 的情况下测试失败的某些情况下, 这就是说 "测试通过" 的原因
        ]

    test_winreg2      Win32       26-Jul-2000
        [20-Aug-2000 tim - 测试已从项目中删除]
        [19-Aug-2000 tim
         此测试永远不会在 Win98 上运行, 因为它正在寻找 W98 下不存在的注册表的一部分.
         出于其他原因, 模块 (winreg.py) 和此测试用例将在 2.0 之前删除.
        ]
        [still fails 15-Aug-2000 for me, on Win98 - tim
         测试 test_winreg2 失败 -- Writing: '测试失败：testHives',
         expected: 'HKEY_PERFORMANCE_DATA\012'
        ]

Open items -- completed/fixed
=============================

::

    [4-Sep-2000 guido: Fredrik finished this on 1-Sep]
    * PyErr_Format - Fredrik Lundh
      使此功能免受缓冲区溢出的影响.

    [4-Sep-2000 guido: Fred 于9月28日加入了 popen2, popen3]
    为 Linux 添加 popen2 支持 -- Fred Drake

    [4-Sep-2000 guido: done on 1-Sep]
    处理 SocketServer 的缓冲问题

    [04-Sep-2000 tim:  完成; 安装程序运行; w9xpopen 不是问题]
    [01-Sep-2000 tim:  提供预发布]
    Windows ME: 对此一无所知. 安装程序会运行吗? 它需要 w9xpopen hack 吗?

    [04-Sep-2000 tim:  完成; 现在测试了几种 Windows 风格]
    [01-Sep-2000 tim:  完成但未经测试, 除了 Win98SE]
    Windows 安装程序: 如果 HKLM 不可写, 请返回 HKCU
    (因此 Python 可以安装在没有管理员权限的 NT 和 2000 上).

    [01-Sep-200 tim - 正如 Guido 所说, posixmodule.c 中的运行时
    代码不会在 NT/2000 上调用它, 因此无需避免在任何地方安装它.
    但是，为安装程序添加了代码 *来* 安装它.]
    Windows 安装程序: 仅在 Win95/98 下安装 w9xpopen.exe.

    [23-Aug-2000 jeremy - tim 报告 "最近完成"]
    Windows: 在 HKLM 之前在 HKCU 寻找注册表信息 - Mark Hammond.

    [20-Aug-2000 tim - done]
    删除 winreg.py 和 test_winreg2.py. Paul Prescod (作者)
    现在想要使注册表 API 更像 MS .NET API. 不清楚是否可以及时完成 2.0,
    但是, 无论如何, 如果我们让 winreg.py 赶出, 我们将永远坚持下去, 甚至保罗也不再想要它了.

    [24-Aug-2000 tim+guido - done]
    Win98 Guido: popen 挂在 Guido 上, 甚至冻结整个机器.
    是由 Windows 9x 上的 Norton Antivirus 2000 (6.10.20) 引起的.
    解决方案: 禁用病毒防护.

Accepted and completed
======================

* 更改 \x 转义的含义 - PEP 223 - Fredrik Lundh

* 在 u"" 字符串中添加 \U1234678 转义 - Fredrik Lundh

* 支持操作码参数 > ``2**16`` - Charles Waldman SF Patch
  100893

* "import as" - Thomas Wouters 扩展 'import' 和 'from ... import' 机制,
  来便将符号作为另一个名称导入. (不添加新关键字.)

* 列表理解 - Skip Montanaro Tim Peters 仍然需要做 PEP.

* 还原旧的 os.path.commonprefix 行为我们是否有适用于所有平台的测试用例?

* Tim O'Malley 的 cookie 模块具有良好的许可证

* 锁步迭代 ("zip" 函数) - Barry Warsaw

* SRE - Fredrik Lundh [至少我 *认为* 它完成了, 因为
  15-Aug-2000 - tim]

* 修复 xrange 打印特性 - Fred Drake 删除 xrange 类型的 tp_print 处理程序;
  它产生了一个列表显示而不是 'xrange(...)'. 当 N!=1 时, 新代码产生对 xrange() 的最小调用,
  包含在 (``...*N``) 中. 这使得 repr() 更易于读取, 同时使得 reprs() 更广泛地被宣传为做什么.
  在交互式解释器中工作时, 它还使 xrange 对象变得明显.

* 扩展打印声明 - Barry Warsaw PEP 214
  http://www.python.org/dev/peps/pep-0214/ SF Patch #100970
  http://sourceforge.net/patch/?func=detailpatch&patch_id=100970&group_id=5470

* 轮询系统调用的接口 - Andrew Kuchling SF Patch 100852

* 增强分配 - Thomas Wouters 添加 += 和 family, 以及 Python 和 C 钩子以及 API 函数.

* gettext.py 模块 - Barry Warsaw


Postponed
=========

* 列表上的扩展切片 - Michael Hudson 使列表 (和其他内置类型) 处理扩展切片.

* 压缩 Unicode 数据库 - Fredrik Lundh SF Patch 100899 At
  least for 2.0b1.  可以作为错误修复包含在 2.0 中.

* Range literals - Thomas Wouters SF Patch 100902 我们最终对该提案抱有很多疑问.

* Eliminated SET_LINENO opcode - Vladimir Marangozov 通过使用代码对象的 lnotab
  而不是 SET_LINENO 指令实现小优化.
  使用代码重写技术 (Guido 皱眉头) 来支持使用 SET_LINENO 的调试器.

  http://starship.python.net/~vlad/lineno/ for (working at the time)
  patches

  讨论 python-dev:

  - http://www.python.org/pipermail/python-dev/2000-April/subject.html
    Subject: "为什么我们需要追溯对象?"

  - http://www.python.org/pipermail/python-dev/1999-August/002252.html

* C 代码的测试工具 - Trent Mick


Rejected
========

* 'indexing-for' - Thomas Wouters Special syntax to give Python code
  access to the loop-counter in 'for' loops. (Without adding a new
  keyword.)


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
