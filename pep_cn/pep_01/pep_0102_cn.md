
PEP: 102
Title: Doing Python Micro Releases
Version: $Revision$
Last-Modified: $Date$
Author: anthony@interlink.com.au (Anthony Baxter),
        barry@python.org (Barry Warsaw),
        guido@python.org (Guido van Rossum)
Status: Superseded
Type: Informational
Content-Type: text/x-rst
Created: 22-Aug-2001 (edited down on 9-Jan-2002 to become PEP 102)
Post-History:
Superseded-By: 101


Replacement Note
================

尽管本 PEP 中待办事项列表的大小远不如 PEP 101 中那么可怕, 但事实证明,
重复信息并不足以证明其中一个副本出现的危险. 因此, 不再维护该 PEP,
并且 PEP 101 完全涵盖少量发布.


Abstract
========

制作 Python 版本是一个艰巨的过程, 即使对于经验丰富的发布者, 也至少需要半天的工作.
直到最近, 大部分 -- 如果不是全部 -- 的负担由 Guido 自己承担.
但是最近的一些版本已经被其他人执行了, 所以这个 PEP 试图在一个地方收集进行 Python 修正发布所需的所有步骤.

主要的 Python 发布过程包含在 PEP 101 中 -- 这个 PEP 只是 PEP 101,
减少到仅包含与小版本, a.k.a. 补丁或错误修复版本相关的位.

它被组织为一个方法, 您可以实际打印出来并在完成它们时检查项目.


How to Make A Release
=====================

以下是制作 Python 版本所采取的步骤. 有些步骤比其他步骤更模糊, 因为几乎没有可自动化的步骤
(例如, 编写 NEWS 条目). 如果一个步骤通常由 An Expert 执行, 则给出该专家的名称.
否则, 假设该步骤由执行释放的指定人员发布管理器 (RM) 完成. 几乎在下面提到 RM 的每个地方,
这一步也可以由 BDFL 完成!

XXX: 我们应该包括一个依赖图来说明可以并行执行的步骤, 或者依赖于其他步骤的步骤.

我们在下面的示例中使用以下约定. 在给出版本号的情况下, 其形式为 X.Y.MaA,
例如, 2.1.2c1 用于 Python 2.1.2 发布候选1, 其中 "a" == alpha, "b" == beta,
"c" == 发布候选. 最终版本在 CVS 中标有 "releaseXYZ". 小版本是从主要版本的维护分支制作的,
例如, Python 2.1.2 由 release21-maint 分支构成.

1. 发送电子邮件至 python-dev@python.org, 表明即将发布的版本.

2. 将检查结果冻结到维护分支中. 此时, 除 RM 之外的任何人都不应该向分支机构
   (或其正式指定的代理商, 即 Guido 的 BDFL, Fred Drake 的文档,
   或 Thomas Heller for Windows) 做出任何提交. 如果 RM 搞砸了, 最后一分钟绝望的变化是必要的, 
   这可能意味着弗雷德和托马斯的额外工作. 所以尽量避免这种情况!

3. 在分支上, 将 Include/patchlevel.h 更改为两个位置, 来反映您刚刚创建的新版本号.
   您需要根据需要更改 PY_VERSION 宏以及 PY_VERSION 之上的一个或多个版本子部分宏.

4. 将 Misc/RPM/python-2.3.spec 的 "％define version" 行更改为与 ``PY_VERSION`` 更改为相同的字符串.
   E.g::

       %define version 2.3.1

   您可能还想将 ％define release 行重置为 '1pydotorg', 如果不是那样的话.

5. 如果您要更改 Python 的版本号 (例如, 从 Python 2.1.1 更改为 Python 2.1.2),
   您还需要更新 README 文件, 该文件顶部有一个大横幅, 用于声明其标识.
   如果您刚刚发布新的 alpha 或 beta 版本, 请不要这样做,
   但是如果您发布新的小型, 次要版本或主要版本, 请执行此操作.

6. 由于对版本号的多次引用, 还需要更改 LICENSE 文件.
   至于 README 文件, 更改这些是新的小型, 次要或主要版本所必需的.

   LICENSE 文件包含一个描述 Python 合法遗产的表格;
   你应该为你正在制作的 X.Y.Z 版本添加一个条目.
   您也应该在 CVS 主干上的 LICENSE 文件中更新此表.

7. 当年份发生变化时, 需要在许多地方更新版权说明, 包括 README 和 LICENSE 文件.

8. 对于 Windows 版本, 必须更新其他文件.

   PCbuild/BUILDno.txt 包含 Windows 内部版本号, 请参阅此文件中的说明如何更改它.
   保存项目文件 PCbuild/pythoncore.dsp 也会导致对 PCbuild/pythoncore.dsp 的更改.

   PCbuild/python20.wse 设置 Windows 安装程序版本资源
   (右键单击安装程序 .exe 并选择 "属性" 时显示), 还包含 Python 版本号.

   (在版本 2.3.2 之前, 需要手动编辑 PC/python_nt.rc, 此步骤现在由构建过程自动完成.)

9. 启动该过程后, 下一步要做的最重要的事情是更新 Misc/NEWS 文件.
   托马斯需要这个才能进行 Windows 发布, 他喜欢熬夜.
   这一步可能非常繁琐, 因此最好在制作分支后立即进行, 或者甚至在进行分支之前.
   越早越好 (但再次注意新的签到, 直到发布!)

   添加此版本的新级别项目. 例如, 如果我们发布 2.2a3,
   那么文件顶部必须有一个部分解释 "Python 2.2a3 中的新功能".
   接下来是标题为 "Python 2.2a2 中的新功能" 的部分.

   请注意, 您 /希望/ 当开发人员向主干添加新功能时, 他们已相应地更新了 NEWS 文件.
   你不能肯定, 所以仔细检查. 如果你是 Unix weenie, 它可以帮助验证 Thomas 对 Windows 的更改,
   以及 Jack Jansen 关于 Mac 上的更改.

   这个命令可以帮助你 (但替换正确的 -r 标签!)::

       % cvs log -rr22a1: | python Tools/scripts/logmerge.py > /tmp/news.txt

   IOW, 您将打印出之前版本中的所有 cvs 日志条目, 直到现在.
   然后, 您可以浏览 news.txt 文件, 查找要添加到 NEWS 的有趣内容.

10. 检查您的 NEWS 更改到维护分支. 忘记更新此文件中的发布日期很容易!

11. 检查 IDLE 的 NEWS.txt 的任何更改. 更新 Lib/idlelib/NEWS.txt 中的标头来反映其发布版本和日期.
    更新 Lib/idlelib/idlever.py 中的 IDLE 版本来匹配.

11. 一旦发布过程开始, 就需要根据 PEP 101 中的说明构建文档并将其发布到 python.org 上.

    请注意, Fred 负责将文档更改从主干合并到分支, 以及在清理阶段将任何分支更改从分支合并到主干.
    基本上, 如果它在 Doc/Fred 中会保管它.

12. Thomas 使用 MSVC 6.0 SP5 编译所有内容, 并将 python23.chm 文件移动到 src/chm 目录中.
    然后使用 Wise Installation System 生成安装程序可执行文件.

    安装程序在文件 MSVCRT.DLL 和 MSVCIRT.DLL 中包含 MSVC 6.0 运行时.
    如果这些文件是从构建安装程序的计算机的系统目录中获取的, 则会导致灾难,
    而必须绝对确保这些文件来自 MSVC SP5 CD 中包含的 VCREDIST.EXE 可再发行组件包.
    必须使用 winzip 解压缩 VCREDIST.EXE, 并且 Wise Installation System 会提示输入该目录.

    构建安装程序后, 应该使用 winzip 打开它, 并再次提取 MS dlls
    并检查与从 VCREDIST.EXE 解压缩的版本号相同的版本号.

    Thomas 将此文件上传到 starship. 然后, 他向 RM 发送一个通知,
    其中包含 Windows 可执行文件的位置和 MD5 校验和.

    请注意, Thomas 创建 Windows 可执行文件可能会在分支上生成更多提交.
    Thomas 将负责合并从主干到分支以及从分支到主干的 Windows 特定更改.

13. Sean 肖恩执行他的红帽魔术, 生成一组 RPM. 他将这些文件上传到 python.org.
    然后他向 RM 发送一个通知, 其中包括 RPM 的位置和 MD5 校验和.

14. 这是构建时间!

    现在, 您已准备好构建源代码压缩包. 首先 cd 到分支的工作目录.  E.g.
    % cd .../python-22a3

15. 在此目录中执行 "cvs update". 不要包含 -A 标志!

    您不应该看到任何 "M" 文件, 但您可能会看到个别 "P" 或 "U" 文件.
    即你最好不要在你的工作目录中有任何未提交的更改,
    但你可能会选择一些 Fred's 或 Thomas 的最后一刻更改.

16. 现在使用像 "rXYMaZ" 这样的符号名称标记分支,
    e.g. r212

    ::

        % cvs tag r212

    一定要只标记 Python CVS 树的 python/dist/src 子目录!

17. 更改为中间目录, 即您可以在其中执行分支的新的, 原始的 cvs 导出.
    您将在此位置创建一个名为 "Python-X.Y.M" 的新目录. 执行标记分支的 CVS 导出.

    ::

        % cd ~
        % cvs -d cvs.sf.net:/cvsroot/python export -rr212 \
                              -d Python-2.1.2 python/dist/src

18. 生成 tarball. 请注意, 我们没有在 tar 命令中使用 'z' 选项,
    因为 1) 据我们所知, 这只有 GNU tar 支持, 2) 我们将最大化压缩级别,
    这不是支持的选项. 我们生成 tar.gz tar.bz2 两种格式, 因为后者大约小1/6.

    ::

        % tar -cf - Python-2.1.2 | gzip -9 > Python-2.1.2.tgz
        % tar -cf - Python-2.1.2 | bzip2 -9 > Python-2.1.2.tar.bz2

19. 计算刚刚创建的 tgz 和 tar.bz2 文件的 MD5 校验和

    ::

        % md5sum Python-2.1.2.tgz

    请注意, 如果您没有 md5sum 程序, 则在 Tools/scripts/md5sum.py 文件中有一个 Python 替换.

20. 为每个文件创建 GPG 密钥.

    ::

        % gpg -ba Python-2.1.2.tgz
        % gpg -ba Python-2.1.2.tar.bz2
        % gpg -ba Python-2.1.2.exe

21. 现在, 您希望执行检查刚刚创建的 tarball 这一非常重要的步骤,
    来确保完全干净的原始构建通过回归测试. 以下是最佳步骤::

        % cd /tmp
        % tar zxvf ~/Python-2.1.2.tgz
        % cd Python-2.1.2
        % ls
        (Do things look reasonable?)
        % ./configure
        (Loads of configure output)
        % make test
        (Do all the expected tests pass?)

    如果测试通过, 那么你可以感觉 tar 包很好.
    如果某些测试失败, 或者关于刚刚解压缩的目录的任何其他内容看起来很奇怪,
    你最好现在停下来找出问题是什么.

22. 您需要将 tgz 和 exe 文件上传到 creosote.python.org.
    此步骤可能需要很长时间, 具体取决于您的网络带宽.
    scp 从你自己的机器到 creosote 的两个文件.

23. 在等待的同时, 您可以开始调整网页来包含公告.

    1. 在 python.org 网站 CVS 树的顶部, 为 X.Y.Z 版本创建一个子目录.
       您实际上可以复制早期补丁版本的子目录, 但一定要删除 X.Y.Z/CVS 目录和 "cvs add X.Y.Z",
       例如::

           % cd .../pydotorg
           % cp -r 2.2.2 2.2.3
           % rm -rf 2.2.3/CVS
           % cvs add 2.2.3
           % cd 2.2.3

    2. 编辑内容文件: 通常可以使用 X.YaZ 全局替换 X.Ya(Z-1).
       但是, 你需要考虑 "What's New?" 部分.

    3. 将 Misc/NEWS 文件复制到 python.org 的 X.Y.Z 目录中的 NEWS.txt;
       这包含自从此版本的 Python 发布以来 Python 的 "full scoop".

    4. 复制您之前在此创建的 .asc GPG 签名.

    5. 此外, 更新 MD5 校验和.

    6. 通过执行 "make" 或 "make install" 预览网页 (只要您为此版本创建了新目录!)

    7. 同样, 编辑 ../index.ht 文件, 即 python.org 主页.
       在 Big Blue Announcement Block 中, 将新版本的段落移到顶部,
       并将 "Python X.YaZ out out" 一词加粗. 编辑内容, 并在本地预览,
       但不要进行 "make install"!

24. 现在我们正在等待 scp 到 creosote  完成.
    Da de da, da de dum, hmm, hmm, dum de dum.

25. 完成后, 您需要访问 creosote.python.org 并将所有文件移到那里.
    我们的策略是每个 Python 版本都有自己的目录, 但每个目录可能包含多个版本.
    我们保留所有旧版本, 在我们有新版本时将它们移动到 "prev" 子目录中.

    所以, 有一个名为 "2.2" 的目录, 其中包含 Python-2.2a2.exe 和 Python-2.2a2.tgz,
    以及包含 Python-2.2a1.exe 和 Python-2.2a1.tgz 的 "prev" 子目录.

    So...

    1. 在 creosote 上, cd 到 ~ftp/pub/python/X.Y 必要时创建它.

    2. 将以前的版本文件移动到名为 "prev" 的目录中, 必要时创建目录 (确保目录中有 g+ws 位).
       如果这是新 Python 版本的第一个 alpha 版本, 请跳过此步骤.

    3. 将 .tgz 文件和 .exe 文件移动到此目录. 确保它们具有全球可读性.
       它们也应该是可写的, 并且由网站管理员组拥有.

    4. md5sum 文件, 并确保它们完整上传.


26. 必要时, X.Y/bugs.ht 文件. 最好为此步骤获取 BDFL 输入.

27. 转到父目录 (即网页层次结构的根目录) 并在那里执行 "make install".
    你的发布现在已经上线了!

28. 现在是时候写邮件列表的公告了. 这是模糊的, 因为没有太多可以自动化.
    您可以使用 Guido 早期公告之一作为模板, 但请编辑内容!

    公告准备好后, 将其发送到以下地址::

        python-list@python.org
        python-announce@python.org
        python-dev@python.org

29. 发送有关该版本的 SourceForge 信息项目. 从项目的 "菜单栏" 中, 选择 "消息" 链接;
    一旦进入消息，请选择 "提交" 链接. 在主题框中键入一个合适的主题 (例如 "Python 2.2c1 发布" :-),
    在 "详细信息" 框中添加一些文本 (至少包括 www.python.org 上的发布 URL 以及随着发布您感到满意的事实)
    并单击提交按钮.

    随意删除任何旧的信息.

现在是时候做一些清理了. 这些步骤非常重要!

1. 编辑文件 Include/patchlevel.h, 以便 PY_VERSION 字符串显示类似 "X.YaZ+" 的内容.
   请注意尾部的 "+" 表示主干将随着开发而向前发展. E.g. 这条线应该是这样的::

       #define PY_VERSION              "2.1.2+"

   确保其他 ``PY_`` 版本宏包含正确的值. 提交这一改变.

2. 对于额外的偏执狂, 请对版本进行彻底清洁理测试. 这包括从 www.python.org 下载 tarball.

3. 确保 md5 校验和匹配. 然后解压缩 tarball, 并进行干净的 make 测试.

   ::

       % make distclean
       % ./configure
       % make test

   确保回归测试套件通过. 如果没有, 你搞砸了!

Step 5 ...

校验! 这可以与步骤4交错. 假装您是用户: 从 python.org 下载文件, 并从中创建 Python.
这个步骤太容易被忽视了, 有几次我们发布了无用的发布文件. 一旦一般服务器问题导致所有文件的神秘腐化;
一旦源 tarball 错误地构建; SF 截断文件上的文件上传过程不止一次; 等等.


What Next?
==========

Rejoice.  Drink.  Be Merry.
写一个像这样的 PEP. 或者像 Guido 一样去度假.

你刚刚发布了一个 Python 版本!

实际上, 还有一步. 您应该将分支机构的所有权转交给 Jack Jansen.
所有这些意味着现在他将负责向分支机构提交承诺. 他将使用它来构建 MacOS 版本.
他可能会向您发送有关 Mac 版本的信息, 这些信息应该合并到 www.python.org 上的信息页面中.
当他完成后, 他会将分支标记为 "rX.YaZ-mac". 他还将负责将任何与 Mac 相关的更改合并到主干中.


Final Release Notes
===================

任何主要版本的最终版本, 例如 Python 2.2, 有特殊要求,
特别是因为它将是最长久的版本之一
(即 beta 版不会持续超过几周, 但最终版本可以持续多年!).

出于这个原因, 我们希望在三个主要版本之间实现更高的协调: Windows, Mac 和源代码.
Windows 和源代码版本受益于相应的发行版机器人的近距离.
但 Mac-bot, 杰克詹森, 距离我们只有6个小时的路程.
因此, 我们将此额外步骤添加到最终的发布过程中
发布:

1. 坚持最后的发布, 直到杰克批准, 或直到我们失去耐心 <wink>.

在发布新的错误修复版本时, python.org 站点还需要进行一些调整.

2. 文档应该安装在 doc/<version>/.

3. 添加 doc/<previous-minor-release>/index.ht 中的链接到新版本的文档.

4. 应该更新所有较旧的 doc/<old-release>/index.ht 文件以指向新版本的文档.

5. /robots.txt 应该进行修改, 以防止旧版本的文档被搜索引擎抓取.


Windows Notes
=============

Windows 有一个 GUI 安装程序, 各种版本的 Windows 都有 "特殊限制",
Windows 安装程序还包含预编译的 "外部" 二进制文件 (Tcl/Tk, expat 等).
所以 Windows 测试很烦人但非常必要.

在上传安装程序的同时, Thomas 安装了两次 Python:
一次进入安装程序建议的默认目录, 然后进入名称中包含嵌入空格的目录.
对于每个安装, 他从 DOS 框运行完整的回归套件, 有和没有 -0.

他还尝试在 Start -> Menu -> the Python group 下创建的 **每个** 快捷方式.
用这种方式尝试 IDLE 时, 您需要验证 Help -> Python Documentation 是否有效.
当用这种方式尝试 pydoc 时 ("模块文档" 开始菜单条目), 确保 "开始浏览器" 按钮工作,
并确保您可以搜索随机模块 (托马斯使用 "random" <wink>), 然后 "转到选中" 按钮有效.

令人惊讶的是, 这里可能会出现多大问题 -- 甚至更令人惊讶的是, 最后一秒签到会破坏其中一件事.
如果你是 "Windows 极客", 请记住, 你可能是唯一经常在 Windows 上测试的人, 而 Windows 只是一团糟.

在至少一种 Win9x 和 NT/2000/XP 之一上重复上述所有内容.
在 NT/2000/XP 上, 尝试管理员和普通用户 (非高级用户) 帐户.

WRT 上面的步骤5 (验证发布媒体), 因为到时候发布文件已经准备好下载了 Thomas 通常在
他上传的安装程序上运行很多 Windows 测试, 除了完整的字节比较之外, 他通常不对第5步做任何事情
("fc /b", 如果使用 Windows shell) 下载的文件对他上传的文件.


Copyright
=========

This document has been placed in the public domain.


..  
   Local Variables:  
   mode: indented-text  
   indent-tabs-mode: nil  
   End:  
