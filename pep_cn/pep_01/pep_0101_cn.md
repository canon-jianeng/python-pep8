
PEP: 101
Title: Doing Python Releases 101
Version: $Revision$
Last-Modified: $Date$
Author: Barry Warsaw <barry@python.org>, Guido van Rossum <guido@python.org>
Status: Active
Type: Informational
Content-Type: text/x-rst
Created: 22-Aug-2001
Post-History:


Abstract
========

制作 Python 版本是一个惊心动魄的过程.
你听说过 "herding cats" 这个词吗?
想象一下, 试图鞍上那些咕噜咕噜的小动物, 然后把它们带到城里,
他们的一些伙伴紧紧抓住你的裸露的背部, 用新锐的爪子固定住.
你提醒自己, 至少他们很可爱.

实际上, 没有一点夸张 <wink>.
Python 发布过程多年来稳步提升, 现在, 在我们了不起的社区的帮助下, 实际上并不太难.
此 PEP 尝试在一个位置收集进行 Python 发布所需的所有步骤.
它被组织为一个食谱, 您可以实际打印出来并在完成它们时检查项目.

Things You'll Need
==================

作为发布管理者, 您需要访问许多资源.
这是一个有希望完整的清单.

* A GPG key.

  Python 版本通过 GPG 进行数字签名;
  你需要一个密钥, 希望至少与其他一个发布管理者一起在 "web of trust" 上.

* A bunch of software:

  * "release.py", Python 发布管理者的朋友.
    它位于 GitHub 上的 python/release-tools repo 中.
    它没有 pip 安装或有任何类型的安装过程 -- 你必须自己把它放在你的路径上,
    或者只是用相对路径运行它, 或者其他什么.

  * "blurb", Misc/NEWS 管理工具. 发布过程目前使用三个 blurb 子命令:
    release, merge, and export. 可以通过 pip3 安装.

  * "virtualenv". 在构建文档时, 发布脚本会在 virtualenv 中安装 Sphinx
    (适用于 2.7 和 3.5+).

  * 相当完整的最新 TeX 发行版安装, 例如 texlive. 您需要它来构建 PDF 文档.

* 访问托管下载文件的服务器 ``dl-files.iad1.psf.io``,
  以及托管文档的服务器 ``docs.iad1.psf.io``. 您将直接在此处上传文件.

* 管理员访问 ``https://github.com/python/cpython``

* www.python.org 上的管理员帐户, 包括 "API key".

* 对 PEP 仓库的写访问权限.

  如果你正在读这篇文章, 你可能已经有了这个 -- 任何发布管理者的第一个任务就是起草发布时间表.
  但是如果你刚刚注册了......恭喜!

* 发布访问 http://blog.python.org, 一个 Blogger 托管的博客.
  此博客的 RSS 源用于 www.python.org 上的 "Python News" 部分.

* 订阅超级秘密发布管理者邮件列表, 可能会也可能不会被称为 ``python-cabal``.
  Bug Barry 对此有所了解.

How to Make A Release
=====================

以下是生成 Python 版本所采取的步:
有些步骤比其他步骤更模糊, 因为几乎没有可自动化的步骤 (例如, 编写 NEWS 条目).
如果一个步骤通常由 An Expert 执行, 则给出该专家的作用.
否则, 假设该步骤由执行释放的指定人员发布管理器 (RM) 完成.
角色及其现有专家是：

* RM = Release Manager

    - Ned Deily <nad@python.org> (US)
    - Larry Hastings <larry@hastings.org> (US)
    - Benjamin Peterson <benjamin@python.org> (US)
* WE = Windows - Steve Dower <steve.dower@microsoft.com>
* ME = Mac - Ned Deily <nad@python.org> (US)
* DE = Docs - Georg Brandl <georg@python.org> (Central Europe)
* IE = Idle Expert - Terry Reedy <tjreedy@udel.edu> (US)

    .. 注意:: 强烈建议 RM 在发布前一天联系专家.
       由于世界是圆的, 每个人都生活在不同的时区,
       因此, RM 必须确保在足够的时间内创建发布标记, 以便专家削减二进制版本.

       在所有专家更新他们的位之前, 您不应该公开发布 (通过更新网站和发送公告).
       在极少数情况下, Windows 或 Mac 的专家是 MIA, 您可以添加消息 "(平台)二进制文件将很快提供" 并继续.

XXX: 我们应该包括一个依赖图来说明可以并行执行的步骤, 或者依赖于其他步骤的步骤.

发布步骤尽可能自动化并由发布脚本指导, 该发布脚本位于单独的仓库中:

    https://github.com/python/release-tools

我们在下面的示例中使用以下约定.
在给出发布版本号的情况下, 其形式为 X.Y.ZaN,
例如, 3.3.0a3 for Python 3.3.0 alpha 3,
其中 "a" == alpha, "b" == beta, "rc" == release candidate.

发布标记名为 "vX.Y.ZaN". 次要版本维护分支的分支名称为 "X.Y".

这有助于执行多个自动编辑步骤, 并指导您执行一些手动编辑步骤.

- 登录 irc.freenode.net 并加入 #python-dev 频道.

  您可能需要与世界各地的其他人协调.
  这个 IRC 频道是我们安排见面的地方.

- 检查是否有任何 showstopper 错误.

  转到 http://bugs.python.org 并查找可阻止此版本的任何开放错误.
  您正在查看正在发布的版本的开放错误的优先级; 以下是相关定义:

  release blocker
      在它们的跟踪, 阻止发布.
      您不得使用任何开放式版本阻止程序错误进行任何发布.

  deferred blocker
      不阻止此版本, 但它将阻止将来的版本.
      您可能无法使用任何打开的延迟阻止程序错误进行最终或候选发布.

  critical
      应该修复的重要错误, 但不会阻止发布.

  检查发布阻止程序, 然后解决它们, 将其降低到延迟, 或者停止发布并请求社区帮助.
  如果您正在进行最终或候选发布, 请对任何打开的延迟发布相同的内容.

- 检查稳定的 buildbots.

  转到 http://buildbot.python.org/all/waterfall

  查看您正在发布的版本的 buildbots. 忽略任何离线 (或通知社区, 以便可以重新启动).
  如果剩下的 (大部分) 是绿色的机器人, 你很高兴. 如果您有非脱机红色内置机器人,
  您可能需要暂停发布, 直到它们被修复. 检查问题并使用您的判断,
  考虑您是否正在制作 alpha, beta 或最终版本.

- 制作发布克隆.

  在 GitHub 上的 cpython 仓库的一个分支上, 在其中创建一个发布分支
  (从现在开始称为 "发布克隆"). 您可以使用用于 cpython 开发的相同 GitHub fork.
  使用 Python Developer's Guide 中推荐的标准设置, 您的 fork 将被称为 `origin`,
  标准 cpython repo 将被称为 `upstream`. 您将使用 fork 上的分支执行发布工程工作,
  包括标记发布, 您将使用它与其他专家共享来创建二进制文件.

- 通过发送电子邮件至 python-committers@python.org 通知所有提交者.

  由于我们现在正在使用分布式版本控制系统, 因此, 无需阻止每个人推送到主干 repo;
  你只需要在你自己的克隆中工作. 因此, 不会有任何登记冻结.

  但是, 所有提交者都应该知道发布克隆的时间点, 因为稍后的提交不会在没有额外努力的情况下进入发布.

- 确保发布克隆的当前分支是要从中发布的分支.  (``git status``)

- 运行 ``blurb release <version>`` 指定版本号 (例如, ``blurb release 3.4.7rc1``).
  这会将所有最近的新闻报道合并到一个标有此版本版本号的文件中.

- 检查文档是否存在标记错误.

  cd 到 Doc 目录并运行 ``make suspicious``. 如果发现任何标记错误, 请修复它们.

- 重新生成 Lib/pydoc-topics.py.

  当仍然在 Doc 目录中时, 运行 ``make pydoc-topics``.
  然后将 ``build/pydoc-topics/topics.py`` 复制到 ``../Lib/pydoc_data/topics.py``.

- 将您的更改提交到 pydoc_topics.py (以及您在文档中所做的任何修复).

- 考虑使用当前接受的标准版本运行 autoconf, 以防 ``configure`` 或其他 autoconf 生成的文件,
  最后使用较新版本或旧版本提交, 并且可能包含虚假或有害的差异.
  目前, autoconf 2.69 是我们事实上的标准. 如果存在差异, 请提交.

- 确保 ``Doc/tools/extensions/pyspecific.py`` 中的 SOURCE_URI 指向 git 仓库中的右分支
  (或者对于默认分支的不稳定版本的 ``default``).

- 通过发布脚本获取版本号.

  $ .../release-tools/release.py --bump X.Y.ZaN

  提醒: X，Y，Z 和 N 应为整数. a 应该是 "a", "b" 或 "rc" 之一 (例如, "3.4.3rc1").
  对于最终版本, 省略 aN ("3.4.3"). 对于新版本的第一个版本, Z 应为 0 ("3.6.0").

  这会自动更新各种版本号, 但您必须手动修改几个文件.
  如果您的 $EDITOR 环境变量设置正确, release.py 将弹出编辑器窗口, 其中包含您需要编辑的文件.

  更新 Misc/NEWS 文件很重要, 但近年来, 由于社区负责此文件的大部分内容, 因此这变得更加容易.
  您只需要查看文本的完整性, 并在今天的日期更新发布日期.

- 确保已提交所有更改. (``release.py --bump`` 不会为你检查它的变化.)

- 查看版权声明的年份. 如果最后一个版本是去年的某个时间,
  请将当前年份添加到几个地方的版权声明中：

  - README
  - LICENSE (确保更改主干和分支)
  - Python/getcopyright.c
  - Doc/copyright.rst
  - Doc/license.rst
  - PC/python_ver_rc.h 设置 Windows 的 DLL 版本资源
    (右键单击 DLL 并选择 "属性" 时显示).
    这不是 C 包含文件, 它是 Windows "资源文件" 包含文件.

- 检查 IE (如果有一个 <wink>) 来确保 Lib/idlelib/NEWS.txt 已经同样更新.

- 对于最终的主要版本, 编辑 Doc/whatsnew/X.Y.rst 的第一段来包含实际的发布日期;
  例如 "Python 2.5于2003年8月1日发布." 没有必要为 alpha 或 beta 版本编辑它.
  请注意, Andrew Kuchling 经常负责这一点.

- 在此目录中执行 "git status".

  你不应该看到任何文件. 即您最好不要在工作目录中进行任何未提交的更改.

- 标记 X.Y.ZaN 的版本.

  $ .../release-tools/release.py --tag X.Y.ZaN

  这将使用 `-s` 选项执行 `git tag` 命令, 以便使用 gpg 密钥对 repo 中的 release 标记
  进行签名. 出现提示时, 选择用于签署发布 tar 包等的私钥.

- 如果这是最终的主要版本, 请为 X.Y 分支树.

  此部分尚未针对基于 GitHub 的版本进行更新!

  在制作主要版本时 (例如, 对于 3.2), 您必须创建长期维护分支.

  - 记下树的当前版本ID.

    $ hg identify

  - 首先, 将原始主干设置为下一个版本.

    $ .../release-tools/release.py --bump 3.3a0

    - 在 README 中编辑所有版本引用

    - 将任何历史 "what's new" 条目从 Misc/NEWS 移至 Misc/HISTORY.

    - 编辑 Doc/tutorial/interpreter.rst
      (2个引用 '[Pp] ython3x', 一个引用 'Python 3.x', 也使标题中的日期保持一致).

    - 编辑 Doc/tutorial/stdlib.rst 和 Doc/tutorial/stdlib2.rst,
      每个都引用 '[Pp] ython3x'.

    - 添加一个新的 whatsnew/3.x.rst 文件 (顶部附近的注释和从前一个文件复制的顶层部分)
      并将其添加到 whatsnew/index.rst 中的 toctree.

    - 更新 configure.ac 中的版本号并重新运行 autoconf.

    - 更新 PC/ 和 PCbuild/ 中 Windows 版本的版本号, 它们引用了 python32.

      $ find PC/ PCbuild/ -type f | xargs sed -i 's/python32/python33/g'
      $ hg mv -f PC/os2emx/python32.def PC/os2emx/python33.def
      $ hg mv -f PC/python32stub.def PC/python33stub.def
      $ hg mv -f PC/python32gen.py PC/python33gen.py

    - 将这些更改提交到默认分支.

  - 现在, 回到之前提到的修订版并进行维护分支 *from there*.

    $ hg update deadbeef    # revision ID noted down before
    $ hg branch 3.2

  - 当您想要将新分支推回到主 CPython 仓库时, 必须将新分支名称添加到 "allow-branches"
    挂钩配置中, 以防止被推送的杂散命名分支. 登录到 hg.python.org 并编辑 (作为 "hg" 用户)
    ``/data/hg/repos/cpython/.hg/hgrc``.

- 在此目录中执行另一个 "git status".

  你不应该看到任何文件. 即您最好不要在工作目录中进行任何未提交的更改.

- 对于最终的主要版本, 必须在所有维护的分支中更新 Doc/tools/static/switchers.js,
  以便新的维护分支不再是 "dev" 并且有一个新的 "dev" 版本.
  此外, 在所有维护的分支中更新 Doc/tools/templates/indexsidebar.html.

- 是时候构建源代码压缩包了. 使用发行脚本创建源 gzip 和 xz tarball,
  文档 tar 和 zip 文件以及 gpg 签名文件.

  $ .../release-tools/release.py --export X.Y.ZaN

  最终版本可能需要一段时间, 它会将所有 tarball 和签名保留在名为 "X.Y.ZaN/src" 的子目录中,
  并将 "X.Y.ZaN/docs" 中的内置文档保留 (最终版本).

- 现在, 您希望执行检查刚刚创建的 tarball 这一非常重要的步骤,
  以确保完全干净的原始构建通过回归测试. 以下是采取的最佳步骤::

    $ cd /tmp
    $ tar xvf /path/to/your/release/clone/<version>//Python-3.2rc2.tgz
    $ cd Python-3.2rc2
    $ ls
    (Do things look reasonable?)
    $ ls Lib
    (Are there stray .pyc files?)
    $ ./configure
    (Loads of configure output)
    $ make test
    (Do all the expected tests pass?)

  如果您感觉很幸运且有时间 kill, 或者您正在制作候选版本或最终版本, 请运行完整的测试套件:

  $ make testall

  如果测试通过, 那么你可以感觉 tar 包很好. 如果某些测试失败,
  或者关于刚刚解压缩的目录的任何其他内容看起来很奇怪, 你最好现在停下来找出问题是什么.

- 将您的提交推送到 GitHub fork 中的远程发布分支. ::

    # Do a dry run first.
    $ git push --dry-run origin
    $ git push --dry-run --tags origin
    # Make sure you are pushing to your GitHub fork, *not* to the main
    # python/cpython repo!
    $ git push origin
    $ git push --tags origin

- 通知专家他们可以开始构建二进制文件.

- STOP STOP STOP STOP STOP STOP STOP STOP

  此时, 您必须接收其他专家的 "绿灯" 以创建发布. 在你等待的时候你可以做些事情,
  所以继续阅读直到你下一次停止.

- WE 使用 (在 Doc/ 中) 构建 Windows 帮助文件

  % make.bat htmlhelp   (on Windows)

  在 build/htmlhelp 中为 HTML Help Workshop 创建合适的输入.
  然后在创建的 python33.hhp 文件上启动 HTML Help Workshop, 最终生成 python33.chm 文件.

- 然后, WE 为每个 Windows 目标体系结构生成 Windows 安装程序文件
  (对于 Python 3.3, 这意味着 x86 和 AMD64).

  - 他为每个目标体系结构提供了一个检验树, 并为适当的体系结构构建了 pcbuild.sln 项目.

  - PC\icons.mak 必须已经与 nmake 一起运行.

  - 运行它的 cmd.exe 窗口必须在其路径中包含 Cygwin/bin (至少对于 x86).

  - cmd.exe 窗口必须在其路径中具有用于目标体系结构的 MS 编译器工具
    (VS 2010 for Python 3.3).

  - 然后, WE 编辑 Tools/msi/config.py (仅在本地存在的文件)
    以更新 full_current_version 并将 snapshot 设置为 false.
    目前发布的 config.py 看起来像 ::

      snapshot=0
      full_current_version="3.3.5rc2"
      certname="Python Software Foundation
      PCBUILD='PCbuild\\amd64'

    最后一行仅用于 amd64 检验.

  - 现在他用 ActivePython 运行 msi.py 或用 pywin32 运行 Python.

  WE 校验文件 (``*.msi``, ``*.chm``, ``*-pdb.zip``),
  将它们与 gpg 签名文件一起上传到 dl-files.iad1.psf.io,
  并通过电子邮件发送您的位置和 md5sums.

- ME 构建 Mac 安装程序包并将它们与 gpg 签名文件一起上传到 dl-files.iad1.psf.io.

- scp 或 rsync 将 ``release.py --export` `构建的所有文件
  放到 dl-files.iad1.psf.io 上的主目录中.

  在等待文件完成上载时, 您可以继续执行剩余的任务.
  您也可以请 #spthon-dev 或 python-committers 上的人员在完成上传后下载文件,
  以便他们也可以在他们的平台上测试它们.

- 现在你需要转到 dl-files.iad1.psf.io 并将所有文件移到那里.
  我们的策略是每个 Python 版本都有自己的目录, 但每个目录都包含该版本的所有版本.

  - 在 dl-files.iad1.psf.io 上, cd/srv/www.python.org/ftp/python/X.Y.Z 必要时创建它.
    确保它由组 'downloads' 和 group-writable 拥有.

  - 将发行版 .tgz 和 .tar.xz 文件移动到位, 以及 .asc GPG 签名文件.
    Win/Mac 二进制文件通常由专家自己放在那里.

    确保它们具有全球可读性. 它们也应该是可写的, 并且由下载组拥有.

  - 使用 ``gpg --verify`` 确保它们完整上传.

  - 如果这是 final 或 rc 版本:
    将 doc zips 和 tarball 移动到 /srv/www.python.org/ftp/python/doc/XYZ,
    必要时创建目录, 并调整 "当前" 符号链接 .../doc 指向该目录.
    请注意, 如果您要发布旧版本的维护版本, 请不要更改当前链接.

  - 如果这是 final 或 rc 版本 (甚至是维护版本), 还要将 docs 文件
    解压缩到 docs.iad1.psf.io 上的 /srv/docs.python.org/release/X.Y.Z[rcA].
    确保文件位于 "docs" 组中并且可以进行组写. 如果它是仅发布安全修复版本的版本,
    请告诉 DE 使用 "版本切换器" 构建版本并将其放在那里.

  - 让 DE 检查文档是否构建并且可以正常工作.

  - 如果这是最终的主要版本: 告诉 DE 在 docs.python.org 的 nginx 配置中
    调整 docs.python.org/XY 的重定向, 更新脚本 Doc/tools/dailybuild.py 来指向
    正确的稳定或开发分支, 并安装它并进行初始检查. Doc 的 switchers.js 脚本也需要更新.
    一般情况下, 请不要触摸顶层的 /srv/docs.python.org/ 目录中的内容,
    除非您知道自己在做什么.

  - 请注意, 文档和下载都是缓存 CDN 的背后. 如果您在通过网站下载后更改存档,
    则需要清除 CDN 中的陈旧数据, 如下所示:

    $ curl -X PURGE https://www.python.org/ftp/python/2.7.5/Python-2.7.5.tar.xz

    您应该始终清除目录列表的缓存, 因为人们使用它来浏览发布文件:

    $ curl -X PURGE https://www.python.org/ftp/python/2.7.5/

- 对于额外的偏执狂, 请对发布进行彻底清洁测试.
  这包括从 www.python.org 下载 tarball.

  确保 md5 校验和匹配. 然后解压缩 tarball, 并做一个干净的 make 测试. ::

    $ make distclean
    $ ./configure
    $ make test

  确保回归测试套件通过. 如果没有, 你搞砸了!

现在是时候改变网站了.

想要执行这些步骤, 您必须拥有编辑网站的权限.
如果您没有, 请向 pydotorg@python.org 上的某人询问是否有适当的权限.
(或者问 Ewa, 他与 RevSys 协调新网站的工作.)

XXX 对于基于 Django 的 python.org, 这已经完全过时了.

这个页面可能会派上用场:

http://docutils.sourceforge.net/docs/user/rst/quickref.html

release.py 不会自动更新任何网站更新.

- 登录到 http://www.python.org/admin .

- 如果这是此版本的第一个版本 (即使是新的补丁版本), 您还需要为该版本创建 "页面".
  找到 "页面" 部分并单击 "添加", 然后填写表单.

  请注意, 最简单的方法可能是从现有的 Python 版本 "页面" 中复制字段, 然后进行编辑.

  发布应该只有一个 "页面" (例如, 3.5.0, 3.5.1).
  为所有预发布版本重用相同的页面, 随时更改版本号和文档.

- 如果这不是版本的第一个发布, 请打开现有的 "页面" 进行编辑, 并将其更新到新版本. 不要保存!

- 现在为该版本创建一个新的 "release" 版本. 目前 "Releases" 按顺序 "下载".

  同样, 最简单的事情可能是从现有的 Python 版本 "页面" 复制字段, 随时进行编辑.

  表单上神秘的 "发布页面" 字段需要此版本的 "页面" 的 ID 号.
  您可以通过检查此版本的 "更改页面" 表单的 URL 来实现.
  例如, 用于编辑 Python 3.5 的 "页面" 的 URL 是:

  https://www.python.org/admin/pages/page/1232/

  页面的 ID 号是最后一个字段; 这是1232.

- 请注意, 按照惯例, 页面上的 "内容" 和发行版上的 "内容" 是相同的,
  *除了* "页面" 有一个关于软件下载位置的部分.

- "保存" 发布.

- 使用可下载文件填充发行版.

  你的朋友和我的朋友 Georg Brandl 制作了一个名为 "add-to-pydotorg.py" 的可爱工具.
  您可以在 "发布" 树 ("release.py" 旁边) 中找到它. 您在 dl-files.iad1.psf.io 上运行该工具,
  如下所示:

  % AUTH_INFO=<username>:<python.org-api-key> python add-to-pydotorg.py <version>

  这将遍历 <version> 的正确下载目录, 查找标有 <version> 的文件, 并使用这些文件在网站上
  填写 "Release Files" 来获取正确的 "release". 请注意, 每次运行时都会清除相关版本的
  "Release Files". 您可以从任何您喜欢的目录运行它, 如果文件发生变化, 您可以根据需要多次
  运行它. 在 dl-files 的主目录中保留一份副本并保持新鲜.

  如果在发行版中添加了新类型的文件 (例如, 添加到 Python 3.5 的基于 Web 的安装程序或
  可再发行的 zip 文件), 则需要更新 add-to-pydotorg.py 以便识别这些新文件.
  (当删除文件类型时, 最好更新 add-to-pydotorg.py.)

- 如果这是最终版本:

  - 将新版本添加到 `https://www.python.org/doc/versions/` 并从任何 '开发中' 部分
    删除当前版本.

  - 对于 X.Y.Z, 编辑所有以前的 X.Y 版本的页面来指向新版本.
    这包括发布的 "Downloads -> Releases" 条目的内容字段::

      注意: Python x.y.m 已被取代
      `Python x.y.n </downloads/release/python-xyn/>`_.

    并且, 对于那些具有单独发布页面条目的版本 (将这些条目逐步排除?), 也要更新这些页面,
    例如 `download/releases/x.y.z` ::

      注意: Python x.y.m 已被取代
      `Python x.y.n </download/releases/x.y.n/>`_.

  - 其他步骤 (新网站的另一个更新)??

现在是时候写邮件列表的公告了. 这是模糊的, 因为没有太多可以自动化.
您可以使用较早的公告作为模板, 但可以针对内容进行编辑!

- STOP STOP STOP STOP STOP STOP STOP STOP

  - 你从 WE 获得了绿灯吗?

  - 你从 DE 获得了绿灯吗?


- 公告准备好后, 将其发送到以下地址:

  python-list@python.org
  python-announce@python.org
  python-dev@python.org

- 同时将公告发布到 `Python Insider 博客 <http://blog.python.org>`_.
  要添加新条目，请转到 "你的 Blogger 主页, 在这里. <https://www.blogger.com/home>`_

现在是时候做一些清理了. 这些步骤非常重要!

- 如果分支尚未处于 "仅安全修复模式", 则暂时禁用其他人推送到主 cpython 仓库中的分支
  并绕过正常状态检查. 转到 设置 -> 分支页面:

  https://github.com/python/cpython/settings/branches/

  并选择您正在处理的分支的 "编辑" 按钮. 在 "分支保护" 页面上,
  选中 "限制谁可以推送到此分支" 框并添加自己. 此外, 取消选中 "包含管理员" 框并保存更改.

  (处于 "仅限安全修复程序" 模式的分支已经受到限制, 您应该是允许推送到分支机构的人员列表的成员.
  因此, 对于 "仅限安全修复程序" 分支, 此步骤是无操作.)

- 将您的发布克隆合并到主开发代码库中::

    # Pristine copy of the upstream repo branch.
    $ git clone --branch X.Y git@github.com:python/cpython.git merge
    $ cd merge
    # Fetch the newly created and signed tag from your repo
    $ git fetch --tags git@github.com:your-github-id/cpython.git vX.Y.ZaN
    # Null merge the temporary release branch discarding any changes
    $ git merge --edit --no-squash -s ours vX.Y.ZaN
    # Examine and update Misc/NEWS etc as needed.  For example, you may
    # need to move NEWS entries for cherry-picked items.
    # Commit any changes.
    $ git commit -m '...'

- 使用发布脚本执行指导后发布步骤.

  $ .../release-tools/release.py --done X.Y.ZaN

  查看并提交这些更改.

- 提交并推送到主要 repo.::

    # Do a dry run first.
    $ git push --dry-run
    $ git push --dry-run --tags
    # If it looks OK, take the plunge.  There's no going back from
    # this step!
    $ git push
    $ git push --tags

- 如果您对 GitHub 上的分支的权限进行了临时更改, 请立即撤消这些临时更改.
  https://github.com/python/cpython/settings/branches/

  取消选中 "限制谁可以推送到此分支". 选中 "包含管理员" 框并保存更改.

  (同样, 这不适用于 "仅限安全修复" 分支. 这一步也是这些分支的无操作.)

- 您可以删除远程发布克隆, 或者只是将其重新用于下一个版本.

- 发送电子邮件给 python-committers 通知他们该版本已发布.

- 使用发布日期更新任何版本 PEP (例如, 361).

- 更新跟踪器 https://bugs.python.org:

  - 将所有延迟阻止程序问题回退到发布阻止程序以用于下一个版本.

  - 在版本 X.Y 进入 alpha 时, 添加版本 X.Y+1.

  - 当版本 X.Y 进入 beta 版时, 将非 doc RFE 更改为版本 X.Y+1.

  - 从您的发行版不支持的版本更新 "行为" 问题到下一个受支持的版本.

  - 回顾未解决的问题, 因为这可能会发现潜伏的 showstopper 错误,
    除了提醒人们修复他们忘记的简单问题.


What Next?
==========

* 校验! 假装你是用户: 从 python.org 下载文件, 然后从中创建 Python.
  这个步骤太容易被忽视了, 有几次我们发布了无用的发布文件.
  一旦一般服务器问题导致所有文件的神秘腐败; 一旦源 tarball 错误地构建;
  SF 截断文件上的文件上传过程不止一次; 等等.

* Rejoice.  Drink.  Be Merry. 写一个像这样的 PEP. 或者像 Guido 一样去度假.

你刚刚发布了 Python 版本!


Windows Notes
=============

Windows 有一个 MSI 安装程序, 各种版本的 Windows 都有 "特殊限制",
Windows 安装程序还包含预编译的 "外部" 二进制文件 (Tcl/Tk, expat 等).
所以 Windows 测试很烦人但非常必要.

在上传安装程序的同时, WE 会从中安装两次 Python: 一次进入安装程序建议的默认目录,
然后进入名称中包含嵌入空格的目录. 对于每个安装, 他从 DOS 框运行完整的回归套件,
有和没有 -0. 对于维护版本, 他还测试升级安装是否成功.

他还尝试 *every* 在 Start -> Menu -> Python 组下创建的快捷方式.
以这种方式尝试 IDLE 时, 您需要验证 Help -> Python 文档是否有效.
当以这种方式尝试 pydoc 时 ("模块文档" 开始菜单条目), 确保 "开始浏览器" 按钮有效,
并确保您可以搜索随机模块 (如, "随机" <wink>), 然后 "转到选中" 按钮工作.

令人惊讶的是, 这里可能会出现多大问题 -- 甚至更令人惊讶的是, 最后一秒签到会破坏其中一件事.
如果你是 "Windows 极客", 请记住, 你可能是唯一经常在 Windows 上测试的人, 而 Windows 只是一团糟.

对每个目标体系结构重复测试. 同时尝试管理员和普通用户 (不是高级用户) 帐户.


Copyright
=========

本文档已置于公共域名.


..  
  Local Variables:  
  mode: indented-text  
  indent-tabs-mode: nil  
  End:  
