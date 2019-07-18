---
layout:     post
title:      使用Git进行协同开发第二步:dev&merge request
subtitle:   
date:       2018-07-09
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Git
    - Tool
---
ok，现在我本地有一个master分支，follow远端的我的master分支。可是由于项目开发时间还没到，我fork的源项目的master分支暂时还是上一个版本的开发分支，管理人员在源项目下新建了一个dev-9.7.5分支，用于下一个版本的代码上传，因此，我需要follow这个dev-9.7.5分支，在此基础上编写代码并提交一个merge request！

首先要做的是更新一下源项目的仓库情况

```console
$ git fetch Main
```
然后，看看我的项目都有哪些分支

```console
$ git branch -a
  master
  remotes/Main/dev
  remotes/Main/dev-9.7.5
  remotes/Main/master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
```

可以看到，源项目已经有一个remotes/Main/dev-9.7.5分支了

为了最后能提交merge request到这个分支，我们以这个分支为基础，在本地创建一个名为dev-9.7.5-my-feature的分支：

```console
$ git checkout -b dev-9.7.5-my-feature   remotes/Main/dev-9.7.5
```

然后，把这个分支上的代码和源项目的975分支进行一下同步：

```console
$ git rebase QYMP/dev-9.7.5
Current branch dev-9.7.5-usercenter is up to date.
```

然后就可以开始写代码啦！

假设我写好了代码并且已经git commit，那么我的push命令就应该把这个commit提交到我的远端项目：

```console
$ git push origin dev-9.7.5-usercenter
```

再登录gitlab网页，应该就可以看到远端的commit了，然后在网页的ui上提交一个merge request到源项目的dev-9.7.5分支，就可以请人来code review啦！