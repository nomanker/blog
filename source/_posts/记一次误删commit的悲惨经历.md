---
title: 记一次误删commit的悲惨经历
categories:
  - 学习之路
date: 2021-05-16 15:34:26
tags:
- Git
---

就在昨天，写完了一部分功能后，像以往一样点击commit，然后发现有两个文件忘记add了，本人有中等程度的强迫症，并不想再commit一次（其实完完全全可以先再次commit，然后直接rebase就行了，还是🥬），于是直接把刚刚提交的那个commit给drop了，然后就杯具了，之前写的所有的代码全都没了（本来以为只是仅仅取消了这次commit，结果真就连代码一块都回到上个node了，哭辽），后来查了好久怎么恢复误删的commit，今天来总结一下。

***

### 1.什么是Git？

Git是一个分布式代码管理工具，在此之前大多使用的是SVN（中央式代码管理仓库）

* 分布式：可以在本地提交代码，不需要依赖网络，并且会将每次提交自动备份到本地。每个开发者都可以将远程仓库clone到本地，并且会把其修改记录一并拿过来。
* 中央式：所有的代码保存在中央服务器，提交必须依赖网络，每次提交都会到中央仓库，如果是协同开发的话可能会频繁触发代码合并，进而增加提交的成本。

***

### 2.Git中的文件状态

在Git中文件大概分为三种状态：已修改（modified）、已暂存（staged）、已提交（committed）

* 修改：Git可以感知到工作目录中哪些文件被修改了，然后把修改的文件加入到modified区域
* 暂存：通过add命令将工作目录中修改的文件提交的暂存区中，等候被commit
* 提交：将暂存区中的文件commit到Git目录中永久保存

***

### 3.使用Git前需要知道的基础

#### 3.1 commit节点

在Git中每次提交都会生成一个节点，而每个节点都有一个hash值作为唯一标识，多次提交会形成一个线性节点链（暂不考虑分支合并的情况）

#### 3.2 HEAD

可以称HEAD为指针或者引用，它可以指向任意一个节点，并且指向的节点始终为当前工作目录，即当前工作目录就是HEAD指向的节点。

HEAD也可以移动或者是指向不同的分支。

#### 3.3 远程仓库

使用远程仓库能够极大地提高协同开发的效率，并且能够避免某些情况下的代码丢失，常见的Git远程仓库主要有Github、Gitlab等。

注意：切忌把未经测试的代码提交至远程仓库（会被leader or 学长骂死）

***

### 4.常用的Git命令

设置commit的用户和邮箱

~~~bash
git config user.name "xx"               #设置 commit 的用户
git config user.email "xx@xx.com"       #设置 commit 的邮箱
git commit --amend --author "xxx <xxx@gmail.com>"    #修改上次提交的用户信息
git config format.pretty oneline        #显示历史记录时，每个提交的信息只显示一行
~~~

查看历史

~~~bash
git log --pretty=oneline filename #一行显示
git log -p -2      #显示最近2次提交内容的差异
git show cb926e7   #查看某次修改
~~~

管理修改

~~~bash
git status              #查看工作区、暂存区的状态
git checkout -- <file>  #丢弃工作区上某个文件的修改
git reset HEAD <file>   #丢弃暂存区上某个文件的修改，重新放回工作区
~~~

查看差异

~~~bash
git diff              #查看未暂存的文件更新 
git diff --cached     #查看已暂存文件的更新 
git diff HEAD -- readme.txt  #查看工作区和版本库里面最新版本的区别
git diff <source_branch> <target_branch>  #在合并改动之前，预览两个分支的差异
~~~

像add、commit、push这些基础命令这里就不再列举

而这次救了我的命的，是下面这个

==版本回退==

~~~bash
git reset --hard HEAD^    #回退到上一个版本
git reset --hard cb926e7  #回退到具体某个版
git reflog                #查看命令历史,常用于帮助找回丢失掉的commit
~~~

***

### 5.merge和rebase的区别

曾经因为年幼无知，只会无脑commit，而且每次commit的comment还都是一样的（主要是怕代码丢了，写一点就commit一下），结果push上去后被学长骂死，后痛下决心决定学习如何将多次commit合并，便有了标题的内容。

首先看下图：

![image-20210516214418625](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210516214418625.png)

可以看到，merge操作会根据这两个分支的共同祖先以及feature/awesomestuff这个分支的两个commit进行合并，然后将合并中修改的内容再生成一个新的commit，如下图

![image-20210516214742272](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210516214742272.png)

而rebase操作则是从两个分支的共同祖先开始提取当前所在分支（master分支）上的修改，然后将master分支指向目标分支处，将刚刚提取的修改依次应用到这个最新的提交后面。操作会舍弃master分支上提取的commit同时不会像merge操作那样生成一个额外的commit用来提交修改的合并内容，相当于把master分支上的修改在目标分支上原样复制了一遍。

#### 5.1 一些区别

* merge是一个合并操作，会将两个分支的修改合并在一起，默认操作的情况下会提交合并中修改的内容；而rebase并没有进行合并操作，只是提取了当前分支的修改，将其复制在了目标分支的最新提交后面
* merge的提交历史忠实地记录了实际发生过什么，关注点在真实的提交历史上面；而rebase的提交历史反映了项目过程中发生了什么，关注点在开发过程中

#### 5.2 rebase合并多个commit

格式：

~~~bash
git rebase -i [startpoint][endpoint]
~~~

其中-i的意思是–interactive，即弹出交互式的界面让用户编辑完成合并操作，[startpoint] [endpoint]则指定了一个编辑区间，如果不指定[endpoint]，则该区间的终点默认是当前分支HEAD所指向的commit(注：该区间指定的是一个前开后闭的区间)。

通过git log命令查看point的hash值，然后使用如下命令

~~~bash
// 合并从当前head到15f745b(commit id)
git rebase -i 15f745b
或:
// 合并最近的两次提交
git rebase -i HEAD~2
~~~

合并选项

* pick：保留该commit（缩写:p）
* reword：保留该commit，但需要修改该commit的注释（缩写:r）
* edit：保留该commit, 但要停下来修改该提交(不仅仅修改注释)（缩写:e）
* squash：将该commit和前一个commit合并（缩写:s）
* fixup：将该commit和前一个commit合并，但不要保留该提交的注释信息（缩写:f）
* exec：执行shell命令（缩写:x）
* drop：丢弃该commit（缩写:d）

在Jetbrains系的IDE中合并commit更加方便,如图

![image-20210516220334620](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210516220334620.png)