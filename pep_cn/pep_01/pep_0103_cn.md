
PEP: 103
Title: Collecting information about git
Version: $Revision$
Last-Modified: $Date$
Author: Oleg Broytman <phd@phdru.name>
Status: Withdrawn
Type: Informational
Content-Type: text/x-rst
Created: 01-Jun-2015
Post-History: 12-Sep-2015

Withdrawal
==========

这个 PEP 被撤回了, 因为它过于通用, 并没有真正处理 Python 开发. 它不再更新.

内容被移至 `Python Wiki`_. 在 wiki 中进一步更新.

.. _`Python Wiki`: https://wiki.python.org/moin/Git

Abstract
========

此 Informational PEP 收集有关 git 的信息. 当然, git 有很多文档,
因此 PEP 专注于更复杂 (与 Python 开发更相关) 的问题, 场景和示例.

计划是在未来扩展 PEP, 收集有关 Mercurial 和 git 场景等效性的信息,
以帮助将 Python 开发从 Mercurial 迁移到 git.

PEP 的作者目前不打算编写关于从 Mercurial 到 git 的迁移 Python 开发的 Process PEP.


Documentation
=============

Git 附带了大量的在线和离线文档.


Documentation for starters
--------------------------

Git 教程: `第1部分
<https://www.kernel.org/pub/software/scm/git/docs/gittutorial.html>`_,
`第2部分
<https://www.kernel.org/pub/software/scm/git/docs/gittutorial-2.html>`_.

`Git 用户手册
<https://www.kernel.org/pub/software/scm/git/docs/user-manual.html>`_.
`每日 GIT 有20个命令
<https://www.kernel.org/pub/software/scm/git/docs/giteveryday.html>`_.
`Git 工作流程
<https://www.kernel.org/pub/software/scm/git/docs/gitworkflows.html>`_.


Advanced documentation
----------------------

`Git Magic
<http://www-cs-students.stanford.edu/~blynn/gitmagic/index.html>`_,
with a number of translations.

`Pro Git <https://git-scm.com/book>`_. 关于git的书.
在亚马逊购买或以 PDF, mobi 或 ePub 形式下载. 它有许多不同语言的翻译.
从 `GArik <https://github.com/GArik/progit/wiki>`_ 中下载俄语翻译.

`Git Wiki <https://git.wiki.kernel.org/index.php/Main_Page>`_.

`Git Buch <http://gitbu.ch/index.html>`_ (German).


Offline documentation
---------------------

Git 已经建立了帮助文档: run ``git help $TOPIC``.
例如, run ``git help git`` or ``git help help``.


Quick start
===========

Download and installation
-------------------------

Unix users: `使用包管理器下载并安装
<https://git-scm.com/download/linux>`_.

Microsoft Windows: 下载 `git-for-windows
<https://github.com/git-for-windows/git/releases>`_.

MacOS X: 安装 git 来自 `XCode <https://developer.apple.com/xcode/>`_
或从 `MacPorts <https://www.macports.org/ports.php?by=name&substr=git>`_ 下载
或 `git-osx-installer <http://sourceforge.net/projects/git-osx-installer/files/>`_
或 安装 git 来自 `Homebrew <http://brew.sh/>`_: ``brew install git``.

`git-cola <https://git-cola.github.io/index.html>`_ (`repository
<https://github.com/git-cola/git-cola>`__) 是一个用 Python 和 GPL 许可编写的 Git GUI.
Linux, Windows, MacOS X.

`TortoiseGit <https://tortoisegit.org/>`_ 是基于 TortoiseSVN 的 Git Windows Shell 界面; 开源.


Initial configuration
---------------------

这个简单的代码经常出现在文档中, 但它很重要, 所以在这里重复一遍.
Git 会在每次提交时存储作者和提交者姓名/电子邮件, 因此请配置您的真实姓名和首选电子邮件::

    $ git config --global user.name "User Name"
    $ git config --global user.email user.name@example.org


Examples in this PEP
====================

此 PEP 中的 git 命令示例使用以下方法. 假设您 (用户) 使用名为 ``python`` 的本地存储库,
该存储库具有名为 ``origin`` 的上游远程仓库. 你的本地仓库有两个分支 ``v1`` 和 ``master``.
对于大多数示例, 当前检出的分支是 "master". 也就是说, 假设你做过类似的事情::

    $ git clone https://git.python.org/python.git
    $ cd python
    $ git branch v1 origin/v1

第一个命令将远程存储库克隆到本地目录 `python``, 创建一个新的本地分支主服务器,
将 remotes/origin/master 设置为其上游远程跟踪分支并将其检入工作目录.

最后一个命令创建一个新的本地分支 v1, 并将 remotes/origin/v1 设置为其上游远程跟踪分支.

使用命令可以实现相同的结果::

    $ git clone -b v1 https://git.python.org/python.git
    $ cd python
    $ git checkout --track origin/master

最后一个命令创建一个新的本地分支主机,
将 remotes/origin/master 设置为其上游远程跟踪分支并将其检入工作目录.


Branches and branches
=====================

Git 术语可能有点误导. 以 "分支" 一词为例. 在 git 中它有两个含义.
分支是一个有针对性的一系列提交 (可能有合并). 分支是分配给一系列提交的标签或指针.
当你谈论提交时以及何时谈论他们的标签时, 重要的是要区分.
一系列提交本身没有命名, 通常只是延长和合并. 另一方面, 标签可以自由创建, 移动, 重命名和删除.


Remote repositories and remote branches
=======================================

远程跟踪分支是本地存储库中的分支 (指向提交的指针).
他们在那里为 git (并且为你) 记住了什么分支和提交已被拉出
并被推送到什么远程仓库 (你可以拉出并推送到许多远程机器).
远程跟踪分支位于 ``remotes/$REMOTE`` 命名空间下,
e.g. ``remotes/origin/master``.

要查看远程跟踪分支的运行状态::

    $ git branch -rv

查看指向提交的本地和远程跟踪分支 (和标记)::

    $ git log --decorate

您永远不会在远程跟踪分支上进行自己的开发. 您创建一个本地分支,
该分支具有远程分支作为上游, 并在该本地分支上进行开发.
在 push git 上将提交推送到远程仓库并更新远程跟踪分支,
在 pull git 上提取来自远程仓库的提交, 更新远程跟踪分支和快进, 合并或重新绑定本地分支.

当你像这样做一个初始克隆::

    $ git clone -b v1 https://git.python.org/python.git

git 克隆远程存储库 ``https://git.python.org/python.git`` 到目录 ``python``,
创建一个名为 ``origin`` 的远程, 创建远程跟踪分支, 创建一个本地分支 ``v1``,
配置它来跟踪上游的 remotes/origin/v1 分支并将 ``v1`` 检出到工作目录中.

一些命令, 如 ``git status --branch`` 和 ``git branch --verbose``, 报告了本地和远程分支之间的区别.
请记住, 它们只与本地存储库中的远程跟踪分支进行比较, 并且这些远程跟踪分支的状态可能已过时.
要更新远程跟踪分支, 您可以从远程存储库获取和合并 (或重新绑定) 提交或更新远程跟踪分支, 而无需更新本地分支.


Updating local and remote-tracking branches
-------------------------------------------

更新远程跟踪分支而不更新本地分支运行
``git remote update [$REMOTE...]``. 例如::

    $ git remote update
    $ git remote update origin


Fetch and pull
''''''''''''''

两者之间存在重大差异

::

    $ git fetch $REMOTE $BRANCH

和

::

    $ git fetch $REMOTE $BRANCH:$BRANCH

第一个命令从 $REMOTE 存储库中的命名 $BRANCH 中提取不在存储库中的提交,
更新远程跟踪分支并将头提交的 id (哈希) 保留在文件 .git/FETCH_HEAD 中.

第二个命令从 $REMOTE 存储库中的命名 $BRANCH 中提取不在存储库中的提交,
并更新本地分支 $BRANCH 及其上游远程跟踪分支. 但它拒绝在非快进的情况下更新分支.
它拒绝更新当前分支 (当前已检出的分支, HEAD 指向的分支).

第一个命令由 ``git pull`` 在内部使用.

::

    $ git pull $REMOTE $BRANCH

相当于

::

    $ git fetch $REMOTE $BRANCH
    $ git merge FETCH_HEAD

当然, 在这种情况下, $BRANCH 应该是您当前的分支.
如果要将不同的分支合并到当前分支中, 请先更新非当前分支然后合并::

    $ git fetch origin v1:v1  # Update v1
    $ git pull --rebase origin master  # Update the current branch master
                                       # using rebase instead of merge
    $ git merge v1

但是, 如果你还没有对 ``v1`` 进行提交, 那么场景必须变得有点复杂.
Git 拒绝更新非快速转发分支, 并且您不想强制执行,
因为这会删除您的非推送提交, 您需要恢复.
所以你想要修改 ``v1``, 但你不能重新定义非当前分支.
因此, 在 merging 之前 rebase  ``v1`` 并重新定义它::

    $ git checkout v1
    $ git pull --rebase origin v1
    $ git checkout master
    $ git pull --rebase origin master
    $ git merge v1

可以配置 git 使其一次获取或拉取一些分支或所有分支, 因此您可以简单地运行

::

    $ git pull origin

甚至

::

    $ git pull

用于获取或拉取的默认远程存储库是 ``origin``.
使用匹配算法计算 fetch 的默认引用集: git 获取两端具有相同名称的所有分支.


Push
''''

推送更简单一些. 只有一个命令 ``push``. 当你运行

::

    $ git push origin v1 master

git 将本地 v1 推送到远程 v1, 将本地主服务器推送到远程主服务器.
同样的::

    $ git push origin v1:v1 master:master

Git 将提交推送到远程仓库并更新远程跟踪分支. Git 拒绝推送不可快速转发的提交.
无论如何你都可以强行推送, 但请记住 -- 你可以强制推送到你自己的存储库,
但不要强行推送到公共或共享的存储库. 如果你发现 git 拒绝推送不可快速转发的提交,
那么从远程仓库获取更好的提取和合并提交 (或者在提取的提交之上重新提交你的提交), 然后推送.
如果你知道你做了什么以及为什么这样做, 那就强制推送. 请参阅下面的 "提交编辑和警告" 部分.

可以配置 git 以使其一次推送几个分支或所有分支, 因此您可以简单地运行

::

    $ git push origin

甚至

::

    $ git push

推送的默认远程存储库是 ``origin``. 使用匹配算法计算在2.0之前推送到 git 的默认引用集:
git 推送两端具有相同名称的所有分支. 使用简单算法计算推送到 git 2.0+ 的默认引用集:
git 将当前分支推回到 @{upstream}.

将2.0之前的 git 配置为新特性运行::

$ git config push.default simple

将 git 2.0+ 配置为旧的特性运行::

$ git config push.default matching

如果它是远程非裸存储库中的当前分支, Git 不允许推送分支: git 拒绝更新远程工作目录.
你真的应该只推送到裸存储库. 对于非裸存储库, git 更喜欢基于推送的工作流程.

当您想要在远程主机上部署代码并且只能使用推送
(因为您的工作站位于防火墙后面并且无法从中取出) 时,
您可以使用两个存储库分两步执行此操作: 从工作站推送到远程主机的裸仓库,
ssh 到远程主机, 从裸仓库拉取到非裸机部署仓库.

这在 git 2.3 中有所改变, 但请看 `博客文章
<https://github.com/blog/1957-git-2-3-has-been-released#push-to-deploy>`_
警告; 在 2.4 中, 推送部署功能是 `进一步改善
<https://github.com/blog/1994-git-2-4-atomic-pushes-push-to-deploy-and-more#push-to-deploy-improvements>`_.


Tags
''''

Git 会自动获取指向提取或拉取期间提取的标记. 要获取所有标记 (以及它们指向的提交),
运行 ``git fetch --tags origin``. 要获取某些特定标记, 请显式获取它们::

    $ git fetch origin tag $TAG1 tag $TAG2...

例如::

    $ git fetch origin tag 1.4.2
    $ git fetch origin v1:v1 tag 2.1.7

Git 不会自动推送标签. 这允许您拥有私有标签.
推送标签明确列出它们::

    $ git push origin tag 1.4.2
    $ git push origin v1 master tag 2.1.7

或者立即推送所有标签::

    $ git push --tags origin

不要使用 ``git tag -f`` 移动标签, 或者在发布标签后用 ``git tag -d`` 删除标签.


Private information
'''''''''''''''''''

克隆/获取/拉取/推送 git 时只复制数据库对象 (提交, 树, 文件和标记) 和符号引用 (分支和轻量级标记).
其他所有内容都是存储库专用的, 从不克隆, 更新或推送. 这是你的配置, 你的钩子, 你的私人排除文件.

如果要分配钩子, 将它们复制到工作树, 添加, 提交, 推送并指示团队手动更新和安装钩子.


Commit editing and caveats
==========================

不编辑已发布 (推送) 提交的警告也会出现在文档中, 但无论如何它都会重复, 因为它非常重要.

可以从强制推送中恢复, 但它是整个团队的 PITA. 请避免它.

要查看尚未发布的提交, 请将分支的头部与其上游远程跟踪分支进行比较::

    $ git log origin/master..  # from origin/master to HEAD (of master)
    $ git log origin/v1..v1  # from origin/v1 to the head of v1

对于具有上游远程跟踪分支的每个分支, git 都维护一个别名 @{upstream}
(短版本 @{u}), 因此上面的命令可以作为::

    $ git log @{u}..
    $ git log v1@{u}..v1

查看所有分支的状态::

    $ git branch -avv

比较本地分支与远程仓库的状态::

    $ git remote show origin

阅读 `如何从上游 rebase 中恢复
<https://git-scm.com/docs/git-rebase#_recovering_from_upstream_rebase>`_.
它在 ``git help rebase`` 中.

另一方面, 不要太害怕提交编辑. 您可以安全地编辑, 重新排序, 删除, 组合和拆分尚未推送的提交.
您甚至可以将提交推送到您自己的 (备份) 仓库, 稍后编辑它们并强制推送已编辑的提交来替换已经推送的提交.
在提交位于公共或共享存储库之前, 这不是问题.


Undo
====

无论你做什么, 都不要惊慌. 几乎 git 中的任何东西都可以撤消.


git checkout: restore file's content
------------------------------------

例如, ``git checkout`` 可用于将文件的内容恢复为提交的内容.
喜欢这个::

    git checkout HEAD~ README

这些命令将 README 文件的内容恢复到当前分支中的最后一个提交.
默认情况下, 提交 ID 只是 HEAD; 即 ``git checkout README`` 将 README 恢复到最新的提交.

(不要使用 ``git checkout`` 来查看提交中文件的内容, 使用 ``git cat-file -p``;
例如 ``git cat-file -p HEAD~:path/to/README``).


git reset: remove (non-pushed) commits
--------------------------------------

``git reset`` 移动当前分支的头部. 可以移动头部指向任何提交,
但它通常用于从分支的顶部移除提交或一些 (最好是非推送的)
-- 也就是说, 向后移动分支来撤消一些 (非推送) 提交.

``git reset`` 有三种操作模式 -- 软，硬和混合.
默认是混合的. ProGit `说明
<https://git-scm.com/book/en/Git-Tools-Reset-Demystified>`_
差别非常明显. 裸存储库没有索引或工作树, 因此在裸仓中只能进行软复位.


Unstaging
'''''''''

使用一个或多个路径的混合模式重置可用于取消更改 -- 即,
从用于提交的 ``git add`` 添加的索引更改中删除. 见 `书
<https://git-scm.com/book/en/Git-Basics-Undoing-Things>`_
有关取消暂停和其他撤消技巧的详细信息.


git reflog: reference log
-------------------------

使用 ``git reset`` 删除提交或移动分支的头部听起来很危险.
但有一种方法可以撤消: 另一种重置回原始提交. Git 不会立即删除提交;
未引用的提交 (在 git 术语中称为 "dangling commits")
在数据库中停留一段时间 (默认为两周), 因此您可以重置回它或创建指向原始提交的新分支.

对于分支头部的每一个动作 - 使用 ``git commit``, ``git checkout``,
``git fetch``, ``git pull``, ``git rebase``, ``git reset``
依此类推 - git 存储一个引用日志 (简称 reflog). 对于每一个动作, git 存储头部所在的位置.
命令 ``git reflog`` 可用于查看 (和操作) 日志.

除了每个分支头部的移动之外, git 还存储了 HEAD 的移动 - 一个 (通常) 命名当前分支的符号引用.
HEAD 随 ``git checkout $BRANCH`` 改变.

默认情况下, ``git reflog`` 显示 HEAD 的移动, 即命令相当于 ``git reflog HEAD``.
要显示分支头部的移动, 请使用命令 ``git reflog $BRANCH``.

所以要撤消 ``git reset`` 查找 ``git reflog`` 中的原始提交,
用 ``git show`` 或 ``git log`` 验证它并运行 ``git reset $COMMIT_ID``.
Git 将分支头部的移动存储在 reflog 中, 因此您可以撤消后重新撤消.

在更复杂的情况下, 您需要移动一些提交以及重置分支的头部. Cherry-pick 把他们带到新的分支.
例如, 如果要将分支 ``master`` 重置回原始提交但保留在当前分支中创建的两个提交, 请执行类似的操作::

    $ git branch save-master  # create a new branch saving master
    $ git reflog  # find the original place of master
    $ git reset $COMMIT_ID
    $ git cherry-pick save-master~ save-master
    $ git branch -D save-master  # remove temporary branch


git revert: revert a commit
---------------------------

``git revert`` 恢复提交或提交, 也就是说, 它创建一个新的提交或提交来恢复给定提交的效果.
这是撤消已发布的提交的唯一方法 (``git commit --amend``, ``git rebase`` 和 ``git reset``
用非快进方式更改分支, 因此它们只应用于非推送提交.)

恢复合并提交存在问题. ``git revert`` 可以撤消合并提交创建的代码,
但不能撤消合并的事实. 请参阅讨论 `如何还原错误的合并
<https://www.kernel.org/pub/software/scm/git/docs/howto/revert-a-faulty-merge.html>`_.


One thing that cannot be undone
-------------------------------

无论你撤消什么, 都有一件事无法撤消 - 覆盖未提交的变更.
未提交的更改不属于 git, 因此 git 无法帮助保留它们.

当你要执行一个覆盖未提交更改的命令时, 大多数时候 git 会警告你.
Git 不允许你用 ``git checkout`` 切换分支.
当你要使用非清理的工作树进行改造时, 它会阻止你.
它拒绝在非提交文件上推送新的提交.

但是有些命令可以完成 - 覆盖工作树中的文件. 像 ``git checkout $PATHs``
或 ``git reset --hard`` 这样的命令会静默覆盖包括你未提交更改的文件.

考虑到这一点, 你可以理解 "提前提交, 经常提交" 的立场. 尽可能经常地提交.
在编辑器或 IDE 中为每次保存提交. 您可以在推送之前编辑提交 - 编辑提交消息,
更改提交, 重新排序, 合并, 拆分, 删除. 但是保存你在 git 数据库中的更改,
要么提交更改, 要么至少用 ``git stash`` 存储它们.


Merge or rebase?
================

互联网上充斥着关于这个主题的激烈讨论: "merge 或 rebase?" 他们中的大多数都没有意义.
当 DVCS 在一个拥有大量复杂项目且拥有许多分支的大型团队中使用时, 根本无法避免合并.
所以这个问题减少了 "是否使用 rebase, 如果是的话 - 何时使用 rebase?"
考虑到非常建议不要重新发布已提交的提交, 问题会进一步减少: "是否在非推送提交中使用 rebase?"

这个小问题是由团队决定的. 为了保持线性历史的美感, 建议在拉取时使用 rebase,
即做 ``git pull --rebase`` 甚至为每个新分支配置 rebase 的自动设置::

    $ git config branch.autosetuprebase always

并为现有分支配置 rebase::

    $ git config branch.$NAME.rebase true

例如::

    $ git config branch.v1.rebase true
    $ git config branch.master.rebase true

之后 ``git pull origin master`` 变得相当于 ``git pull --rebase origin master``.

建议在使用 rebase 更新主线分支时, 在单独的功能或主题分支中创建新提交.
当主题分支准备就绪时, 将其合并到主线中. 为了避免一次性解决大量冲突的繁琐任务,
您可以不时地将主题分支合并到主线, 并切换回主题分支来继续处理它. 整个工作流程都是这样的::

    $ git checkout -b issue-42  # create a new issue branch and switch to it
        ...edit/test/commit...
    $ git checkout master
    $ git pull --rebase origin master  # update master from the upstream
    $ git merge issue-42
    $ git branch -d issue-42  # delete the topic branch
    $ git push origin master

删除主题分支时, 只删除标签, 提交保留在数据库中, 它们现在合并到主数据库中::

    o--o--o--o--o--M--< master - the mainline branch
        \         /
         --*--*--*             - the topic branch, now unnamed

删除主题分支以避免使用小主题分支混乱分支命名空间.
有关已解决的问题或实施的功能的信息应该在提交消息中.

但是, 即使是长期合并的分支机构, 即使少量的重新定位也可能太大.
想象一下, 你在 ``v1`` 和 ``master`` 分支都做了工作,
定期将 ``v1`` 合并到 ``master`` 中. 一段时间后
你会在 ``master`` 中进行大量的合并和非合并提交.
然后, 您希望将已完成的工作推送到共享存储库，
并发现有人已经将一些提交推送到 ``v1``. 现在你可以选择两个同样糟糕的选择:
要么你获取并重新定义 ``v1`` 然后必须重新创建你在 ``master`` 中工作的所有东西
(将 ``master`` 重置为原点, 合并 ``v1 `` 并且挑选来自旧主人的所有非合并提交);
或者合并新的 ``v1`` 并且适度放宽线性历史的美丽.


Null-merges
===========

Git 有一个内置的合并策略, 用于 Python 核心开发人员称之为 "null-merge" 的内容::

    $ git merge -s ours v1  # null-merge v1 into master


Branching models
================

Git 没有假设任何关于分支和合并的特定开发模型.
有些项目更倾向于从最旧的分支到最新的分支, 有些更喜欢向后挑选提交,
有些则使用压缩 (将多个提交合并为一个). 一切皆有可能.

有几个例子可以开始. `git 帮助工作流程
<https://www.kernel.org/pub/software/scm/git/docs/gitworkflows.html>`_
描述了 git 作者如何开发 git.

ProGit 书中有几章专门讨论不同项目中的分支机构管理: `Git 分支 - 分支工作流程
<https://git-scm.com/book/en/Git-Branching-Branching-Workflows>`_
和 `分布式 Git - 为项目做贡献
<https://git-scm.com/book/en/Distributed-Git-Contributing-to-a-Project>`_.

还有一篇着名的文章 `一个成功的 Git 分支模型
<http://nvie.com/posts/a-successful-git-branching-model/>`_
by Vincent Driessen.
它建议了一套关于创建和管理主线, 主题和错误修复分支的非常详细的规则.
为了支持该模型, 作者实现了 `git 流
<https://github.com/nvie/gitflow>`_
扩展.


Advanced configuration
======================

Line endings
------------

Git 内置机制来处理具有不同行尾样式的平台之间的行结尾.
为了允许 git 进行 CRLF 转换, 使用 `.gitattributes 将 ``text`` 属性赋值给文件
<https://www.kernel.org/pub/software/scm/git/docs/gitattributes.html>`_.
对于必须具有特定行结尾的文件, 请指定 ``eol`` 属性. 对于二进制文件, 该属性自然是 "二进制".

例如::

    $ cat .gitattributes
    *.py text
    *.txt text
    *.png binary
    /readme.txt eol=CRLF

要检查 git 用于文件的属性, 请使用 ``git check-attr`` 命令. 例如::

$ git check-attr -a -- \*.py


Useful assets
-------------

`GitAlias <http://gitalias.com/>`_
(`仓库 <https://github.com/GitAlias/gitalias>`_) 是一个很大的别名集合.
仔细选择常用命令的别名可以为您节省大量的击键次数!

`GitIgnore <https://www.gitignore.io/>`_
和 https://github.com/github/gitignore 是各种 IDE 和编程语言的 ``.gitignore`` 文件的集合.
包括 Python!

`pre-commit <http://pre-commit.com/>`_
(`仓库 <https://github.com/pre-commit>`_) 是一个用于管理和维护多语言预提交钩子的框架.
该框架是用 Python 编写的, 并且有许多用于许多编程语言的插件.


Advanced topics
===============

Staging area
------------

暂存区域也称索引 aka 缓存是 git 的一个显着特征. 暂存区域是 git 在提交之前收集补丁的地方.
收集补丁和提交阶段之间的分离提供了一个非常有用的 git 功能: 您可以在提交之前查看收集的补丁,
甚至可以编辑它们 - 删除一些 hunks, 添加新的 hunks 并再次审核.

要将文件添加到索引, 请使用 ``git add``. 在提交之前收集补丁意味着您需要为每次更改执行此操作,
而不仅仅是添加新的 (未跟踪的) 文件. 为了简化提交, 以防你想要提交所有内容
而不查看运行 ``git commit --all`` (或只是 ``-a``) - 该命令将每个更改的跟踪文件添加到索引然后提交.
提交一个或多个文件而不管索引中收集的补丁运行 ``git commit [--only|-o] -- $FILE...``.

要在索引中添加大量补丁, 请使用 ``git add --patch`` (或只是 ``-p``).
要从索引中删除收集的文件, 请使用 ``git reset HEAD -- $FILE...``
添加/检查/删除收集的 hunks 使用 ``git add --interactive`` (``-i``).

要查看索引和最后一次提交之间的差异 (即收集的补丁), 请使用 ``git diff --cached``.
要查看工作树和索引之间的差异 (即未收集的补丁), 只需使用 ``git diff``.
要查看工作树和最后一次提交之间的差异 (即收集和未收集的补丁), 请运行 ``git diff HEAD``.

请看 `WhatIsTheIndex
<https://git.wiki.kernel.org/index.php/WhatIsTheIndex>`_ and
`IndexCommandQuickref
<https://git.wiki.kernel.org/index.php/IndexCommandQuickref>`_ in Git
Wiki.


Root
----

在运行任何命令之前, Git 切换到根目录 (项目的顶级目录, 其中 ``.git`` 子目录存在).
Git 记得切换之前当前的目录. 某些程序会考虑当前目录.
例如, ``git status`` 显示相对于当前目录的已更改和未知文件的文件路径;
``git grep`` 在当前目录下搜索; ``git apply`` 仅适用于触摸当前目录下方文件的补丁.

但是大多数命令从根运行并忽略当前目录. 想象一下, 例如, 你有两个工作树,
一个用于分支 ``v1`` 而另一个用于 ``master``. 如果要从第二个工作树内的子目录合并 ``v1``,
则必须编写命令, 就像在顶层目录中一样. 让我们拿两个工作树, 例如 ``project-v1`` 和 ``project``::

    $ cd project/subdirectory
    $ git fetch ../project-v1 v1:v1
    $ git merge v1

请注意 ``git fetch ../project-v1 v1:v1`` 中的路径是 ``../project-v1``
而不是 ``../../project-v1``, 尽管事实是我们从子目录运行命令, 而不是从根运行.


ReReRe
------

Rerere 是一种有助于解决重复合并冲突的机制. 最常见的重复合并冲突源是合并到主线的主题分支,
然后删除合并提交; 这通常是为了测试主题分支和训练 rerere; 删除合并提交来获得干净的线性历史记录
并仅使用最后一次合并提交完成主题分支.

Rerere 通过在成功提交之前和之后记住树的状态来工作.
这种方式 rerere 可以自动解决冲突, 如果它们出现在相同的文件中.

可以使用 ``git rerere`` 命令手动使用 Rerere, 但大多数情况下它会自动使用.
在工作树中启用这些命令的 rerere::

    $ git config rerere.enabled true
    $ git config rerere.autoupdate true

您不需要在全局范围内转换 rerere - 您不希望在裸存储库或单分支存储库中重新存在;
您只需要在 repos 中重新执行 repos, 您经常执行合并并解决合并冲突.

See `Rerere <https://git-scm.com/book/en/Git-Tools-Rerere>`_ in The
Book.


Database maintenance
--------------------

Git 对象数据库和 ``.git`` 下的其他文件/目录需要定期维护和清理.
例如, 提交编辑左未引用的对象 (悬挂对象, 在 git 术语中), 应该修剪这些对象来避免在 DB 中收集残骸.
命令 ``git gc`` 用于维护. Git 自动运行 ``git gc --auto`` 作为一些命令的一部分来进行快速维护.
建议用户偶尔运行 ``git gc --aggressive``; ``git help gc`` 建议每隔几百个变更集运行一次;
对于更密集的项目, 它应该像每周一次, 而不是频繁 (每两周或每月) 用于较不活跃的项目.

``git gc --aggressive`` 不仅可以删除悬空对象, 还可以将对象数据库重新打包为索引和更优化的包 (s);
它还包含符号引用 (分支和标记). 另一种方法是运行 ``git repack``.

There is a well-known `message
<https://gcc.gnu.org/ml/gcc/2007-12/msg00165.html>`_
来自 Linus Torvalds 关于 ``git gc --aggressive`` 的 "愚蠢".
现在可以安全地忽略该消息. 它已经陈旧过时了, ``git gc --aggressive`` 从那时起变得更好了.

对于那些仍然喜欢 ``git repack`` 而不是 ``git gc --aggressive`` 的人来说,
推荐的参数是 ``git repack -a -d -f --depth=20 --window=250``.
请看 `这个详细的实验
<http://vcscompare.blogspot.ru/2008/06/git-repack-parameters.html>`_
用于解释这些参数的影响.

偶尔运行 ``git fsck [--strict]`` 来验证数据库的完整性.
``git fsck`` 可能会产生一个悬空物体清单; 这不是错误, 只是提醒您进行定期维护.


Tips and tricks
===============

Command-line options and arguments
----------------------------------

`git help cli
<https://www.kernel.org/pub/software/scm/git/docs/gitcli.html>`_
建议不要结合短选项/旗帜. 大多数时候结合工作: ``git commit -av`` 完美地工作,
但有些情况则不然. 例如, ``git log -p -5`` 不能组合为 ``git log -p5``.

有些选项有参数, 有些甚至有默认参数. 在这种情况下, 这种选项的参数必须用粘性方式拼写: ``-Oarg``,
永远不要 ``-O arg`` 因为对于有默认参数的选项, 后者意味着 "使用选项的默认值 ``-O``
并将 ``arg`` 传递给选项解析器". 例如, ``git grep`` 有一个选项 ``-O``,
它将找到的文件的名称列表传递给程序; ``-O`` 的默认程序是一个呼叫器 (通常是 ``less``),
但你可以使用你的编辑器::

    $ git grep -Ovim  # but not -O vim

顺便说一句, 如果 git 被指示使用 ``less`` 作为呼叫器 (即, 如果没有在 git 中配置呼叫器,
它默认使用 ``less``, 或者如果它从 GIT_PAGER 得到 ``less`` 或者 PAGER 环境变量,
或者如果配置了 ``git config [--global] core.pager less``, 或 ``less``,
则命令 ``git grep -Oless``) ``git grep `` 将 ``+/$pattern`` 选项传递给 ``less`` 这很方便.
不幸的是, 如果呼叫器不完全是 ``less``, 那么 ``git grep`` 不会传递模式, 即使它是带参数的 ``less``
(类似于 ``git config [--global] core.pager less -FRSXgimq``);
幸运的是, ``git grep -Oless`` 总是传递模式.


bash/zsh completion
-------------------

即使对于那些乐于使用命令行的人来说, 手动输入 ``git rebase --interactive --preserve-merges HEAD~5``
也有点困难, 这就是 shell 完成有很大帮助的地方. Bash/zsh 带有可编程完成功能,
通常是自动安装和启用的, 所以如果你安装了 bash/zsh 和 git, 很可能你已经完成了 - 只需在命令行中使用它.

如果您没有安装必要的位, 请安装并启用 bash_completion 包.
如果你想将你的 git 完成升级到 `git contrib 的最新和最好的下载必需文件
<https://git.kernel.org/cgit/git/git.git/tree/contrib/completion>`_.

Git-for-windows 附带 git-bash, 安装并启用了 bash 完成功能.


bash/zsh prompt
---------------

对于命令行爱好者来说, shell 提示可以携带很多有用的信息. 要在提示使用中包含 git 信息
`git-prompt.sh
<https://git.kernel.org/cgit/git/git.git/tree/contrib/completion/git-prompt.sh>`_.
阅读文件中的详细说明.

在网上搜索 "git prompt" 来查找其他提示变种.


SSH connection sharing
----------------------

SSH 连接共享是 OpenSSH 的一个特性, 也许是 PuTTY 等衍生产品.
SSH 连接共享是一种通过建立一个连接并将其重新用于连接到同一服务器的所有
后续客户端来减少 ssh 客户端启动时间的方法.
SSH 连接共享可用于加速很多短 ssh 会话, 如 scp, sftp, rsync,
当然还有 git over ssh. 如果您定期通过 ssh 获取/拉取/推送远程存储库, 则建议使用 ssh 连接共享.

要打开 ssh 连接共享, 请将此类内容添加到您的
~/.ssh/config::

    Host *
    ControlMaster auto
    ControlPath ~/.ssh/mux-%r@%h:%p
    ControlPersist 600

请看 `OpenSSH 维基
<https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Multiplexing>`_
和 `搜索 <https://www.google.com/search?q=ssh+connection+sharing>`_ 了解更多信息.

SSH 连接共享可以在 GitHub, GitLab 和 SourceForge 存储库中使用,
但请注意 BitBucket 不允许它并在短暂的不活动时间后强行关闭主连接,
因此您将看到 ssh 中的错误: "连接到 bitbucket. 组织由远程主机关闭."


git on server
=============

发布存储库或一组存储库的最简单方法是 ``git daemon``. 守护程序提供匿名访问,
默认情况下它是只读的. 可以通过 git 协议 (git://URL) 访问存储库.
可以启用写访问, 但协议缺少任何身份验证方法, 因此应该仅在受信任的 LAN 中启用它.
有关详细信息, 请参阅 ``git help daemon``.

Git over ssh 提供身份验证和 repo 级别授权, 因为存储库可以是用户或组可写的
(请参阅 ``git help config`` 中的参数 ``core.sharedRepository``).
如果这对某些项目的需求来说过于宽松或过于严格, 那么可以配置一个包装器
`gitolite <http://gitolite.com/gitolite/index.html>`_,
它可以配置为允许用极大的粒度进行访问; gitolite 是用 Perl 编写的, 有很多文档.

可以使用创建用于浏览存储库的 Web 界面 `gitweb
<https://git.kernel.org/cgit/git/git.git/tree/gitweb>`_
或 `cgit <http://git.zx2c4.com/cgit/about/>`_. 
两者都是 CGI 脚本 (用 Perl 和 C 编写). 除了 web 界面,
它们都为 git (http(s):// URLs) 提供只读的 http 访问. `Klaus
<https://pypi.python.org/pypi/klaus>`_ 是一个小而简单的 WSGI Web 服务器,
它实现了 Web 界面和 git 智能 HTTP 传输; 支持 Python 2 和 Python 3, 执行语法突出显示.

还有更先进的基于 Web 的开发环境, 包括管理用户, 组和项目的能力; 私人, 集团可访问和公共存储库;
它们通常包括问题跟踪器, 维基页面, 拉取请求以及其他用于开发和通信的工具.
在这些环境中有 `Kallithea
<https://kallithea-scm.org/>`_ 和 `pagure <https://pagure.io/>`_,
两者都是用 Python 编写的; pagure 由 Fedora 开发人员编写,
用于开发一些 Fedora 项目. `GitPrep
<http://gitprep.yukikimoto.com/>`_ 是另一个 GitHub 克隆, 用 Perl 编写的.
`Gogs <https://gogs.io/>`_ 是用 Go 编写的.
`GitBucket <https://gitbucket.github.io/gitbucket-news/about/>`_ 是用 Scala 编写的.

最后但并非最不重要的是, `GitLab <https://about.gitlab.com/>`_.
它可能是 git 最先进的基于 Web 的开发环境. 用 Ruby 编写,
社区版是免费和开源的 (MIT license).


From Mercurial to git
=====================

有很多工具可以将 Mercurial 存储库转换为 git.
最着名的可能是 `hg-git <https://hg-git.github.io/>`_
和 `fast-export <http://repo.or.cz/w/fast-export.git >`_
(很多年前它的名字叫 ``hg2git``).

但是, 一个更好的工具, 也许是最好的工具 `git-remote-hg
<https://github.com/felipec/git-remote-hg>`_.
它从 git 提供对 Mercurial 存储库的透明双向 (拉取和推送) 访问.
它的作者写了一篇 `替代方案的比较
<https://github.com/felipec/git/wiki/Comparison-of-git-remote-hg-alternatives>`_
这似乎是最客观的.

要使用 git-remote-hg, 安装或克隆它, 添加到 PATH
(或将脚本 ``git-remote-hg`` 复制到已经在 PATH 中的目录中)
并将 ``hg::`` 添加到 Mercurial URL. 例如::

    $ git clone https://github.com/felipec/git-remote-hg.git
    $ PATH=$PATH:"`pwd`"/git-remote-hg
    $ git clone hg::https://hg.python.org/peps/ PEPs

要使用存储库, 只需使用常规的 git 命令, 包括 ``git fetch/pull/push``.

要开始将 Mercurial 习惯转换为 git, 请参阅该页面
`Mercurial for Git users
<https://www.mercurial-scm.org/wiki/GitConcepts>`_ at Mercurial wiki.
在页面的后半部分, 有一个表列出了相应的 Mercurial 和 git 命令.
应该在两个方向上完美地工作.

Python 开发人员指南还有一章 `Mercurial for git developers
<https://docs.python.org/devguide/gitdevs.html>`_,
它记录了 git 和 hg 之间的一些区别.


Git and GitHub
==============

`gitsome <https://github.com/donnemartin/gitsome>`_ - Git/GitHub 命令行界面 (CLI).
用 Python 编写, 适用于 MacOS, Unix, Windows. 具有自动完成功能的 Git/GitHub CLI,
包括许多可与所有 shell 一起使用的 GitHub 集成命令, 内置 xonsh 和 Python REPL,
可以运行 Python 命令以及 shell 命令, 命令历史记录, 可自定义突出显示, 完整记录.


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
   vim: set fenc=us-ascii tw=70 :  
