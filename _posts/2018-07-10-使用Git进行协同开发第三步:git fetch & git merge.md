---
layout:     post
title:      使用Git进行协同开发第三步:git fetch & git merge
subtitle:   
date:       2018-07-10
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Git
    - Tool
---
### git fetch
#### 使用场景：

在进行开发之前，通常会对现有的代码进行一次更新，以确保在最新的代码基础上动手。fetch就是一个很好的方式。与之后要谈到的pull命令相比，fetch的更新不会对你本地的开发代码造成任何影响。

#### 常用方式：

更新一个远端仓库Main的所有最新信息（commit，tag， branch等等）

```console
$ git fetch Main
```

更新一个远端仓库Main下面的dev分支

```console
$ git fetch Main dev
```

更多fetch参数：https://git-scm.com/docs/git-fetch

### git merge
#### 使用场景：

如果在从master分出来的dev分支上完成开发任务的你，准备把dev分支的所有commit合并回master分支了，有一种方式就是使用merge。

举个栗子，假设你有两个本地分支如下：

**master分支：A->B->C->D**

**dev分支：B->E->F**

可以看出，dev分支一开始是从master的B节点开始的，EF就是你的开发成果了。要把EF的修改搬回master分支，可以这么做：

```console
$ git merge master dev
```

由此得到的本地master分支如下：

**master分支：A->B->C->D->G**

注意这个G。

**G=E+F+conflict fix**

也就是说，merge的操作，实际上是**将dev分支上，从dev和master发生分叉开始，到dev的最新commit之间的所有修改，也就是E+F，合并为一个commit G**。这个commit包含了dev上的所有修改，即是E+F，以及为了修复E+F和master目前代码所产生的conflict所产生的修改。

网站上的原文如下：

> Then "git merge topic" will replay the changes made on the topic branch since it diverged from master (i.e., E) until its current commit (C) on top of master, and record the result in a new commit along with the names of the two parent commits and a log message from the user describing the changes.

如果这时候你在master分支git log查看一下当前状态，可以看到的结构是：

**master分支：A->B->C->D->E->F->G**

但是如果你干掉最近的一个commit：

```console
$ git reset --hard HEAD~1
```

再次git log，你会看到：

**master分支：A->B->C->D**

就好像没有执行过merge一样～

更多merge参数：https://git-scm.com/docs/git-merge
