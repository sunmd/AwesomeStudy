# git 学习指南

## 使用git之前的配置

- 配置 user.name 和 user.email

```git
git config --global user.name 'your_name'
git config --global user.email 'your_name@domain.com'
```

- config 的三个作用域

缺省等同于local

```git
git config --local    # local 只对某个仓库有效
git config --global   # global 对房前用户所有仓库有效
git config --system   # system对系统所有登录的用户有效
```

- 显示config 的配置, 加--list

```git
git config --list --local
git config --list --global
git config --list --system
```

## 建Git仓库

两种场景:

1. 把已有的项目代码纳入Git管理

```git
cd 项目代码所在的文件夹
git init
```

2. 新建的项目直接用Git管理

```git
cd 某个文件夹
git init ypur_project #会在当前路径下创建和项目名称同名的文件夹
cd your_project
```



## 给文件重命名的方法

```git
git reset --hard    # 暂存区 和 工作区 全部清除
git mv
```

## 通过git log 查看版本延边历史

```git
git log -n2 --oneline  #最近两次的一行历史
git branch -av    #查看分支的全部信息
git checkout -b    #创建分支
git log --all     #所有分支日志
git log --all --graph    #所有分支日志的图形化
```

## gitk 通过图形界面工具

```git
gitk    ##gitk来启动图形界面
```

## .git的目录

- HEAD  为当前头文件指向

```git
git checkout master   ## 切换分支
```

- config 为local为配置

```git
cat HEAD 查看HEAD文件的内容 
git cat-file 命令 显示版本库对象的内容、类型及大小信息。
git cat-file -t b44dd71d62a5a8ed3 显示版本库对象的类型
git cat-file -s b44dd71d62a5a8ed3 显示版本库对象的大小
git cat-file -p b44dd71d62a5a8ed3 显示版本库对象的内容

HEAD：指向当前的工作路径
config：存放本地仓库（local）相关的配置信息。
refs/heads:存放分支
refs/tags:存放tag，又叫里程牌 （当这次commit是具有里程碑意义的 比如项目1.0的时候 就可以打tag）
objects：存放对象 .git/objects/ 文件夹中的子文件夹都是以哈希值的前两位字符命名 每个object由40位字符组成，前两位字符用来当文件夹，后38位做文件。
```



## commit、tree和blob三个对象之间的关系

- 一个commit对应一个树(文件一个快照)
- tree 对应一个文件夹
- blob 对应一个文件
- git add 就会创建blob

```bash
find .git/objects -type -f
```

## 分离头指针情况下的注意事项

- 工作在没有分支的状态,头部(HEAD)没有指向分支
- commit没有和分支或者是tag绑定的话,是在gitk上不显示的,视为不重要可能会被清除的

### git checkout head 的^和~

1.  一个节点，可以包含多个子节点（checkout 出多个分支）
2.  一个节点可以有多个父节点（多个分支合并）
3.  ^是~都是父节点，区别是跟随数字时候，^2 是第二个父节点，而~2是父节点的父节点
4.  ^和~可以组合使用,例如 HEAD~2^2

### git 删除分支

git branch -d branch_name:使用-d 在删除前Git会判断在该分支上开发的功能是否被merge的其它分支。如果没有，不能删除。如果merge到其它分支，但之后又在其上做了开发，使用-d还是不能删除。

### git 修改最近一次提交

git commit --amend 命令应该是代替（或这说修改）上一次提交，不只是修改message。
比如上一次提交时有几个文件没有add以及commit，可以重新进行add之后再commit --amend提交。
但这次提交之后不会增加一次新的commit，而是相当于在上一次commit的基础上进行修改。

### git rebase -i <hash>

- 修改对应commit 的提示 ,要在hash中填写父类

- 同时要在对应hash上用r

- 父级要pick

git rebase工作的过程中，就是用了分离头指针。rebase意味着基于新base的commit来变更部分commits。它处理的时候，把HEAD指向base的commit，此时如果该commit没有对应branch，就处于分离头指针的状态，然后重新一个一个生成新的commit，当rebase创建完最后一个commit后，结束分离头状态，Git让变完基的分支名指向HEAD。

搜索工具可以根据 “how to rebase the oldest commit” 可以搜到 how do i git rebase the first commit，
https://stackoverflow.com/questions/22992543/how-do-i-git-rebase-the-first-commit

里面告诉我们执行 “git rebase -i --root”

### tag的使用

```console
$ git tag  #观察tag 的列表
v1.0
v2.0

$ git tag -a v1.4 -m "my version 1.4"
$ git tag
v0.1
v1.3
v1.4


$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
    
$ git tag -a v1.2 9fceb02 #给以前的commit打标签
# 默认情况下，git push 命令并不会传送标签到远程仓库服务器上。 在创建完标签后你必须显式地推送标签到共享服务器上。 这个过程就像共享远程分支一样——你可以运行 git push origin <tagname>。

$ git push origin v1.5
Counting objects: 14, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 2.05 KiB | 0 bytes/s, done.
Total 14 (delta 3), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.5 -> v1.5
 
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 160 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.4 -> v1.4
 * [new tag]         v1.4-lw -> v1.4-lw
 
$ git tag -d v1.4-lw
Deleted tag 'v1.4-lw' (was e7d5add)

$ git push origin --delete <tagname>
```

### git比较差异

```git
git diff [<options>] [<commit>] [--] [<path>…​]
git diff [<options>] --cached [<commit>] [--] [<path>…​]
git diff [<options>] <commit> [<commit>…​] <commit> [--] [<path>…​]
git diff [<options>] <commit>…​<commit> [--] [<path>…​]
git diff [<options>] <blob> <blob>
git diff [<options>] --no-index [--] <path> <path>
```



git diff --cached: 暂存区和head之间差异 -号是head +号暂存区

git diff: 暂存区和工作区之间差异 -号是暂存区,+号是工作区

git diff HEAD -- readme.md     # 工作区 <===> HEAD

暂存区与HEAD比较：git diff --cached

暂存区与工作区比较: git diff

暂存区恢复成HEAD : git reset HEAD

暂存区覆盖工作区修改：git checkout 



## git暂存区恢复成HEAD

git reset HEAD:git暂存区恢复成HEAD

git reset 有三个参数
--soft 这个只是把 HEAD 指向的 commit 恢复到你指定的 commit，暂存区 工作区不变
--hard 这个是 把 HEAD， 暂存区， 工作区 都修改为 你指定的 commit 的时候的文件状态
--mixed 这个是不加时候的默认参数，把 HEAD，暂存区 修改为 你指定的 commit 的时候的文件状态，工作区保持不变

```git
先 git reflog 查看命令历史, 记录每次的命令
然后进行你想找回的commit
git reset --hard commit_id
```

### git 正确删除文件的方法

```git
git rm <filename>
```

### git开发中临时存储工作区数据

```git
git stash       # 存储对应的工作区
git stash list  # 查看对应的堆栈数据
git stash apply # 之前存放的内容拿出来,并且不删除堆栈数据
git stash pop   # 之前存放的内容拿出来,并且删除堆栈数据
用法：git stash pop stash@{2}
            git stash pop = git stash pop stash@{0}
```

### .gitignore git不管理的文件

### git仓库配分到本地文件

```git
git clone --bare /user/local/wer/.git  ya.git
git clone --bare file:///user/local/wer/.git  zhineng.git
git remote -av
git remote add zhineng  file:///user/local/zhineng.git
```

## git同步远程已删除的分支和删除本地多余的分支

```git
git branch -a ## 查看本地分支和远程分支情况 
git remote show origin ## 查看本地分支和追踪情况
git remote prune origin ## 同步删除这些分支
```



## git fetch

- **使用git fetch更新代码，本地的库中master的commitID不变，还是等于1。但是与git上面关联的那个orign/master的commit ID变成了2**。这时候我们本地相当于存储了两个代码的版本号，我们还要通过merge去合并这两个不同的代码版本，如果这两个版本都修改了同一处的代码，这时候merge就会出现冲突，然后我们解决冲突之后就生成了一个新的代码版本。
- **这时候本地的代码版本可能就变成了commit ID=3，即生成了一个新的代码版本。**

**使用git pull的会将本地的代码更新至远程仓库里面最新的代码版本**

#### **不要用git pull，用git fetch和git merge代替它**。

git pull的问题是它把过程的细节都隐藏了起来，以至于你不用去了解git中各种类型分支的区别和使用方法。当然，多数时候这是没问题的，但一旦代码有问题，你很难找到出错的地方。看起来git pull的用法会使你吃惊，简单看一下git的使用文档应该就能说服你。

将下载（fetch）和合并（merge）放到一个命令里的另外一个弊端是，你的本地工作目录在未经确认的情况下就会被远程分支更新。当然，除非你关闭所有的安全选项，否则git pull在你本地工作目录还不至于造成不可挽回的损失，但很多时候我们宁愿做的慢一些，也不愿意返工重来。

cherry-pick，以及rebase与master差别的课程

### commit 中文乱码解决

git config --global i18n.commitencoding utf-8

git config --global i18n.logoutputencoding utf-8

export LESSCHARSET=utf-8



## 新建一个远程分支进行测试

```git
git checkout -b <名字> <远程分支>
```

## 注意事项

1. 不用 git push -f
2. 不要rebase 公共分支

## 规范建设

### 规范梳理

初期我们在互联网上搜索了大量有关git commit规范的资料，但只有Angular规范是目前使用最广的写法，比较合理和系统化，并且有配套的工具（IDEA就有插件支持这种写法）。最后综合阿里巴巴高德地图相关部门已有的规范总结出了一套git commit规范。

**commit message格式**

- 

```
<type>(<scope>): <subject>
```



**type(必须)**



用于说明git commit的类别，只允许使用下面的标识。

feat：新功能（feature）。

fix/to：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。

- fix：产生diff并自动修复此问题。适合于一次提交直接修复问题

- to：只产生diff不自动修复此问题。适合于多次提交。最终修复问题提交时使用fix

docs：文档（documentation）。

style：格式（不影响代码运行的变动）。

refactor：重构（即不是新增功能，也不是修改bug的代码变动）。

perf：优化相关，比如提升性能、体验。

test：增加测试。

chore：构建过程或辅助工具的变动。

revert：回滚到上一个版本。

merge：代码合并。

sync：同步主线或分支的Bug。



**scope(可选)**

scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

例如在Angular，可以是location，browser，compile，compile，rootScope， ngHref，ngClick，ngView等。如果你的修改影响了不止一个scope，你可以使用*代替。



**subject(必须)**

subject是commit目的的简短描述，不超过50个字符。

- 建议使用中文（感觉中国人用中文描述问题能更清楚一些）。

- 结尾不加句号或其他标点符号。

根据以上规范git commit message将是如下的格式：

```
fix(DAO):用户查询缺少username属性 feat(Controller):用户查询接口开发
```

以上就是我们梳理的git commit规范，那么我们这样规范git commit到底有哪些好处呢？

- 便于程序员对提交历史进行追溯，了解发生了什么情况。

- 一旦约束了commit message，意味着我们将慎重的进行每一次提交，不能再一股脑的把各种各样的改动都放在一个git commit里面，这样一来整个代码改动的历史也将更加清晰。

- 格式化的commit message才可以用于自动化输出Change log。



监控服务

通常提出一个规范之后，为了大家更好的执行规范，就需要进行一系列的拉通，比如分享给大家这种规范的优点、能带来什么收益等，在大家都认同的情况下最好有一些强制性的措施。当然git commit规范也一样，前期我们分享完规范之后考虑从源头进行强制拦截，只要大家提交代码的commit message不符合规范，直接不能提交。但由于代码仓库操作权限的问题，我们最终选择了使用webhook通过发送警告的形式进行监控，督促大家按照规范执行代码提交。除了监控git commit message的规范外，我们还加入了大代码量提交监控和删除文件监控，减少研发的代码误操作。

**整体流程**



![img](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLUCVicia5R1fhl9ukZ68TRoAdDyaRbvqFebeMI8WT0RBTsp4SfTOO8DcSAYlh76r7woibKDZJ8tMDibA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 服务注册：服务注册主要完成代码库相关信息的添加。

- 重复校验：防止merge request再走一遍验证流程。

- 消息告警：对不符合规范以及大代码量提交、删除文件等操作发送告警消息。

- DB：存项目信息和git commit信息便于后续统计commit message规范率。



webhook是作用于代码库上的，用户提交git commit，push到仓库的时候就会触发webhook，webhook从用户的commit信息里面获取到commit message，校验其是否满足git commit规范，如果不满足就发送告警消息；如果满足规范，调用gitlab API获取提交的diff信息，验证提交代码量，验证是否有重命名文件和删除文件操作，如果存在以上操作还会发送告警消息，最后把所有记录都入库保存。

IDEA中使用Terminal连接git bash输入命令，在git log 时日志信息出现中文乱码，，，，

尝试过很多方法都失败，最终找到这个方法：

在Terminal窗口运行以下三个命令后问题得到解决。



## github

### 搜索技巧

光标放在搜索框里面，然后回车，界面上就会出现advanced search

in:readme

stars:>3000
