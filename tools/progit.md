# ---Git入门---

# git三大区

![image-20240408013811407](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240408013811407.png)

- 工作区：存放文件的磁盘目录，此时git未追踪文件（可删）
- 暂存区：把代码**添加**到暂存区,从而让git追踪文件（可删）
- 本地库：把暂存区文件**提交**到本地库，此时会生成对应的历史版本（不可删）

# 常用命令

| 作用             | 命令                                 |
| ---------------- | ------------------------------------ |
| 设置用户名       | git config --global user.name 用户名 |
| 设置用户邮箱     | git config --global user.email 邮箱  |
| 初始化本地库     | git init                             |
| 查看本地库状态   | git status                           |
| 添加到暂存区     | git add 文件名                       |
| 提交到本地库     | git commit 文件名 -m "注释信息"      |
| 查看历史记录     | git reflog                           |
| 查看详细历史记录 | git log                              |
| 版本穿梭         | git reset --hard 版本号（哈希值）    |
| 删除暂存区文件   | git rm --cached 文件名               |

ps：用户配置信息在~/.gitconfig中查看

# 分支操作

| 作用                     | 命令                |
| ------------------------ | ------------------- |
| 创建分支                 | git branch 分支名   |
| 查看分支                 | git branch -v       |
| 切换分支                 | git checkout 分支名 |
| 把指定分支合并到当前分支 | git merge 分支名    |

ps:对于合并冲突需要程序员手动修改发生冲突的文件，然后add+commit。

# 远程仓库操作

| 作用                                           | 命令                         |
| ---------------------------------------------- | ---------------------------- |
| 查看当前所有远程地址别名                       | git remote -v                |
| 起别名                                         | git remote add 别名 远程地址 |
| 推送本地分支的内容到远程库                     | git push 别名 分支名         |
| 将远程库的内容克隆到本地                       | git clone 远程地址           |
| 将远程库当前分支的最新内容拉取到本地后直接合并 | git pull 别名 分支名         |

ps:clone会做的操作：拉取代码+初始化本地仓库+创建别名（默认origin）

# ---Progit笔记---

# 1 起步

## 1.1 起步 - 关于版本控制

## 1.2 起步 - Git 简史

## 1.3 起步 - Git 是什么？

git怎么对待/保存数据的？

git怎么去引用数据的？

git保存的文件的状态有哪些？

git项目的三个阶段？

## 1.4 起步 - 命令行

## 1.5 起步 - 安装 Git

## 1.6 起步 - 初次运行 Git 前的配置

## 1.7 起步 - 获取帮助

获取帮助文档的命令？

# 2 Git基础

## 2.1 Git 基础 - 获取 Git 仓库

#### 获取Git项目仓库的两种方式?

- 在已存在目录中初始化仓库

  ```
  git init
  ```

  该命令会创建一个.git子目录，这里包含了初始化Git仓库中所有的必须文件

- 克隆现有的仓库

  ```
  git clone <url>
  ```

  Git会克隆该Git仓库服务器上的所有数据，包括每一个文件的每一个版本都会被拉取下来。

  举例：

  ```
  git clone https://github.com/libgit2/libgit2
  ```

  该命令会在当前目录下创建名为libgit2的目录，并在这个目录下初始化一个.git文件夹，将远程仓库的所有数据拉取到.git文件夹中，然后从中读取最新版本的文件的拷贝。

  如果要自定义本地仓库名字，就在命令最后面添加一个名字就可以了。比如 `git clone https://github.com/libgit2/libgit2 mylibgit`就将目标目录名变成了mylibgit

## 2.2 Git 基础 - 记录每次更新到仓库

#### 工作目录下的文件的状态有哪些？

已跟踪文件：git已经知道的文件

未跟踪文件：除了已跟踪文件都是未跟踪文件

工作一段时间后，文件的状态变化如下：

![image-20240508224400990](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240508224400990.png)

#### 检查文件状态的命令？如果有未跟踪的文件会显示什么？暂存状态的文件呢？

```
git status
```

如果有未跟踪文件会提示 Untracked files:

如果有暂存区文件会提示 Changes to be committed:

#### 解释一下git add？git暂存的是那个版本的文件？

git add是精确地将内容添加到下一次提交中

git暂存的是运行 `git add` 命令时的版本

#### 用简短方式查看文件状态？

```
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

新添加的未跟踪文件前面有 `??` 标记

新添加到暂存区中的文件前面有 `A` 标记

修改过的文件前面有 `M` 标记

输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态

#### 解释如下.gitignore文件？

```
*.a

!lib.a

/TODO

build/

doc/*.txt

doc/**/*.pdf
```

文件 `.gitignore` 的格式规范如下：

- 所有空行或者以 `#` 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
- 匹配模式可以以（`/`）开头防止递归。
- 匹配模式可以以（`/`）结尾指定目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（`!`）取反。

```
# 忽略所有的 .a 文件
*.a

# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a

# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO

# 忽略任何目录下名为 build 的文件夹
build/

# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt

# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

#### 查看文件具体位置的差异（命令）？

比较工作目录中当前文件和暂存区域快照之间的差异:

```
git diff
```

比对已暂存文件与最后一次提交的文件差异：

```
git diff --staged/--cached
```

#### 提交文件（命令）？提交时记录的是什么？

```
git commit
```

提交时记录的是放在暂存区域的快照。

#### 自动把所有已经跟踪过的文件暂存起来一并提交（命令）？

```
git commit -a
```

#### git中删除文件(命令)？该命令具体做了什么？

```
git rm <file>
```

从暂存区删除并且从工作目录中删除

#### 为什么不可以直接删除文件？

如果直接删除有文件，那么git status会显示Changes not staged for commit，git中任然会有该文件的快照。必须要在暂存区中删除该文件才可以。

#### 为什么git rm删除不了文件的情况？

如果要删除之前修改过或已经放到暂存区的文件，则必须使用强制删除选项 `-f`。这样子做主要可以防止误删没有添加到快照的数据。

#### 想要从暂存区删除，但是工作目录中保留（命令）？

```
git rm --cached <file>
```

#### git中改名的命名？该命令做了什么？

```
git mv <file_from> <file_to>
```

这个命令其实就等同于于如下三条命令：

```
mv <file_from> <file_to>
git rm <file_from>
git add <file_to>
```

## 2.3 Git 基础 - 查看提交历史

#### 查看提交历史(命令)？

```
git log
```

#### 想要显示每次提交的差异？

```
git log -p/--patch
```

#### 想要限制显示日志的条目数量（当前页）？

```
git log -p -2
```

#### 想要看到每次提交的简略统计信息？

```
git log --stat
```

#### 想要指定输出格式？

```
每个提交放在一行显示
git log --oneline(--pretty=online --abbrev-commit的简写)

用ascii码暂时分支、合并历史
git log --graph

避免显示合并提交
git log --no-merges
```

![image-20240508231735999](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240508231735999.png)

![image-20240508231748075](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240508231748075.png)

## 2.4 Git 基础 - 撤消操作

#### 重新提交（命令）？

```
git commit --amend
```

#### 这个命令和我直接重新提交一下的区别？

git commit --amend不会修改快照，只是修改提交信息

如果直接git commit重新提交，那么会创建一个新的提交

#### 取消暂存的文件（命令）？

```
git reset HEAD <file>
```

#### 撤销对文件的修改(命令)？

```
git checkout -- <file>
```

git会用最近提交的版本覆盖掉它。

## 2.5 Git 基础 - 远程仓库的使用

#### clone一个仓库后，git会默认做什么？

git会默认取一个origin的简写，会默认设置本地master分支跟踪远程master分支。

#### 查看已经配置的远程仓库服务器（命令）？

```
git remote
想要把远程仓库的url一起显示:
git remote -v
```

#### 添加一个新的远程仓库（命令）？

```
git remote add <shortname> <url>
```

后续可以使用简写代替url

#### 拉取远程仓库中我没有的信息（命令）？

```
git fetch <remote>
```

#### 自动抓取后合并远程分支到当前分支（命令）？

```
git pull <remote>
```

#### 将当前分支推送到远程仓库（命令）？

```
git push <remote> <branch>
```

如果和其他人同一时间克隆，且他们先推送到远程仓库，那么我推送时就会被拒绝。必须先将他们的修改合并到本地后，才可以推送。

#### 查看远程仓库的更多信息（命令）？

```
git remote show <remote>
```

#### 修改远程仓库的简写名（命令）？该命令会做什么？

```
git remote rename <old_name> <new_name>
```

#### 删除一个远程仓库（命令）？

```
git remote remove/rm <remote>
```

## 2.6 Git 基础 - 打标签

#### 列出标签（命令）？

```
git tag
按照通配符模式查找标签:
git tag -l <通配符>
```

git支持的标签类型？创建附注标签的命令？查看标签信息和对应的提交信息的命令？创建轻量标签的命令？该命令会做什么？

为过去的提交打标签的命令？

将标签推送到远程仓库？为什么要这么做，有点类似什么？想要一次性推送很多标签呢？这个会推送两种标签吗？

删除本地仓库上的标签？还想要删除远程仓库的呢？

查看标签指向的文件版本的命令？该命令会导致什么结果？解释分离头指针？那假如此时我需要修复旧版本中的错误呢？

不是很懂这最后这部分。。。。。。

## 2.7 Git 基础 - Git 别名

通过git config为命令设置别名的方法？想要执行外部命令？

# 3 Git分支

## 3.1 Git 分支 - 分支简介

#### git保存的是什么？

git保存的是文件不同时刻的快照

#### git提交时会保存一个什么？这个东西包含哪些内容？

提交对象：作者姓名和邮箱、提交时出入的信息、指向父对象的指针

#### 分支创建（命令）？git怎么知道当前在那个分支上？

```
git branch <branch_name>
```

HEAD会指向当前分支

#### 查看各个分支当前所指的提交对象（命令）？

```
git log --decorate
```

#### 切换分支（命令）?

```
git checkout <branch_name>
```

#### 查看分叉历史（命令）？

```
git log --oneline --decorate --graph --all
```

#### git分支的实质是什么？

一个包含所指对象校验和的文件，所以创建的销毁它都很快

## 3.2 Git 分支 - 分支的新建与合并

#### 创建分支的同时切换分支（命令）？

```
git checkout -b <branch_name>
```

#### 合并分支（命令）？

```
git merge <branch_name>
```

#### 什么是快进？

想要合并的分支所指向的提交时当前所在提交的后继，那么git指针会直接向前移动，因为此时合并没有需要解决的分歧。

#### 删除分支（命令）？

```
git branch -d <branch_name>
```

#### 不是快进情况，git怎么合并？

三方合并：将两个分支末端指向的提交和两个分支的公共祖先合并。

合并提交：三方合并后会生成一个新的快照并且创建一个提交指向它。合并提交有多个父提交。

#### 如果两个分支都修改了同一处内容呢？

此时git只会做合并，但是不会自动穿件一个新的合并提交。git会停下来，等待我们去解决合并冲突。解决完后，使用git add来将其标记为冲突已解决。然后就可以git commit来完成合并提交了。

#### 查看合并是否完成？

任然是使用git status。

## 3.3 Git 分支 - 分支管理

#### 查看分支列表（命令）？

```
git branch

查看每一个分支的最后一次提交
git branch -v

过滤这个列表中已经合并或尚未合并到当前分支的分支
git branch --merged/--no-merged
```

#### 删除已经合并的分支和未合并的分支有啥影响？

删除已合并的分支直接使用git branch -d就可以了，因为已经将工作整合到其他分支了，所以删除没什么影响。

如果是未合并的分支，必须使用-D参数。因为删除这些分支就会丢失到那些工作。

## 3.4 Git 分支 - 分支开发工作流

#### 分支保存在哪里？

本地。当新建和合并分支时，所有操作都只发生在本地git版本库中，和服务器没有交互。

## 3.5 Git 分支 - 远程分支

#### 远程引用？

远程引用就是对远程仓库的引用，包括分支、标签等。

#### 获取远程引用的完整列表（命令）？

```
git ls-remote <remote>

获取远程分支更多信息
git remote show <remote>
```

#### 远程跟踪分支？

远程跟踪分支是远程分支状态的引用。以`<remote>/<branch>`命名

#### clone做的事情？

clone是会将远程服务器命名为origin，然后拉取数据，创建一个指向它的master指针，本地命名为origin/master，同时会有一个与origin的master分支指向同一个地方的本地master分支。

#### 与远程仓库同步数据（命令）？

```
git fetch <remote>
```

会抓取本地没有的数据，并且更新本地数据库，移动origin/master指针到更新后的位置。

#### 添加一个新的远程仓库引用到当前项目（命令）？

```
git remote add <shortname> <url>
```

#### 抓取origin服务器的一个子集服务器的数据，git会做什么？

git并不会抓取数据，而是设置远程跟踪分支。

#### 推送本地分支到远程分支（命令）？

```
git push <remote> <branch>

想要推送到命名不同的远程分支:
git push origin <local_branch>:<remote_branch>
```

#### 将本地分支跟踪远程分支（命令）？

```
git checkout -b <local_branch> <remote>/<remote_branch>
git checkout --track <remote>/<remote_branch>
git checkout <remote_branch>
```

#### 设置已有分支跟踪一个刚拉取下来的远程分支（命令）？

```
git branch -u <remote>/<remote_branch>
```

#### 查看所有的跟踪分支（命令）？

```
git branch -vv
```

这个命令也会显示更多的本地分支的信息

#### 拉取本地没有的数据（命令）？

```
git fetch + git merge
如下会自动合并，最好不要用：
git pull
```

#### 删除远程分支的命令（命令）？

```
git push <remote> --delete <remote_branch>
```

git服务器一般会保留数据一段时间直到垃圾回收运行，所以还是有机会恢复的。

## 3.6 Git 分支 - 变基

#### 变基？

实质：是丢弃一些现有的提交，然后相应地新建一些内容一样但实际上不同的提交。

原理：首先找到这两个分支（即当前分支 `experiment`、变基操作的目标基底分支 `master`） 的最近共同祖先 `C2`，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件， 然后将当前分支指向目标基底 `C3`, 最后以此将之前另存为临时文件的修改依序应用。

此图为将c4的修改变基到g3上：

![image-20240512005930925](C:\Users\Mr.Helen\AppData\Roaming\Typora\typora-user-images\image-20240512005930925.png)

变基后，还需要回到master，然后merge。合并后，就可以删除原来的分支了。完整步骤如下：

```
git checkout experiment
git rebase master

git checkout master
git merge experiment

git branch -d experiment
```

#### 变基和合并的区别？

变基是将一系列提交按照原有次序依次应用到另一分支上，而合并是把最终结果合在一起。

#### 将那个当前分支变基到其他分支下（命令）？

```
git rebase --onto master server client
```

选取在client分支但是不在server分支的修改，将其重放在master分支。

#### 不用切换到主题分支的变基（命令）？

```
git rebase <basebranch> <topicbranch>
```

将主题分支变基到目标分支，这样子可以不用切换到主题分支的步骤。

#### 什么时候不要使用变基？

如果别人可能基于这些提交进行开发，那么不要执行变基

# Git Commit Message怎么写

1. [用空行分开主题和正文](https://www.jianshu.com/p/0117334c75fc#1)
2. [限制主题在50个字母](https://www.jianshu.com/p/0117334c75fc#2)
3. [主题行首字母要大写](https://www.jianshu.com/p/0117334c75fc#3)
4. [不要用句号结束主题行](https://www.jianshu.com/p/0117334c75fc#4)
5. [主题行用祈使语气](https://www.jianshu.com/p/0117334c75fc#5)
6. [每行72个字](https://www.jianshu.com/p/0117334c75fc#6)
7. [在正文部分解释什么，为什么，以及怎么做的](https://www.jianshu.com/p/0117334c75fc#7)

```
用不超过50个字简述一下有哪些改变  

如果必要的话，写更多的细节。每行不要超过72个字。第一行被视为 commit 信息以及余下正文的主题。空行分开概要和正文是非常必要的（除非你不写正文）；如果你把 log, shortlog, rebase这样的工具混着用会让人迷惑。

解释一下这个commit解决了什么问题。专注于为什么这么做而不是怎么做的（代码已经解释了）。这个改变是否有副作用或其他不直观的后果。这里就是解释这些事情的地方。

空行之后还有段落。

- 要点符号也是可以的

- 通常用连字符，星号来表示要点符号。用一个空格起头，用空行隔开，当然，惯例没有这么详细

如果你用了 issue tracker， 把这些引用放在底部，就像这样：

解决了：#123
参考： #456, #789
```

核心其实就是如下：

```
（主题）最好一行解决

简要说明怎么解决的。

为什么有这种问题。

具体怎么解决这个问题的。

```

```
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200

   Simplify serialize.h's exception handling

   Remove the 'state' and 'exceptmask' from serialize.h's stream implementations, as well as related methods.（概述）

   As exceptmask always included 'failbit', and setstate was always called with bits = failbit, all it did was immediately raise an exception. Get rid of those variables, and replace the setstate with direct exception throwing (which also removes some dead code).（为什么）

   As a result, good() is never reached after a failure (there are only 2 calls, one of which is in tests), and can just be replaced by !eof().（为什么）

   fail(), clear(n) and exceptions() are just never called. Delete them.（具体做法）
```

