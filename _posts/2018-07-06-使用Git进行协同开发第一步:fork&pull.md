---
layout:     post
title:      使用Git进行协同开发第一步:fork&pull
subtitle:   
date:       2018-07-06
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Git
    - Tool
---
## 找到你想要参与的项目，fork这个项目，从而得到一个线上的你自己的项目。

想参与的项目：ssh://git@gitlab.xxx.domain/xxx/Main.git

我fork得到的项目：ssh://git@gitlab.xxx.domain/hellomin/Main.git

## 将这个线上的你自己的项目，clone到本地：

```console
$ git clone ssh://git@gitlab.xxx.domain/hellomin/Main.git
```

## 进入本地项目目录，查看远端仓库信息：

```console
$ git remote -v
```

应该可以看到自己的远端项目的信息：

```console
$ git remote -v

origin	ssh://git@gitlab.xxx.domain/hellomin/Main.git (fetch)

origin	ssh://git@gitlab.xxx.domain/hellomin/Main.git (push)
```

为了和我们想要参与的项目保持一致，需要把该项目也添加到我们的远端仓库中：

```console
$ git remote add Main ssh://git@gitlab.xxx.domain/xxx/Main.git
```

再次查看远端仓库信息，可以发现添加了一个新的远端仓库

```console
$ git remote -v

Main ssh://git@gitlab.xxx.domain/xxx/Main.git (fetch)

Main	ssh://git@gitlab.xxx.domain/xxx/Main.git (push)

origin	ssh://git@gitlab.xxx.domain/hellomin/Main.git (fetch)

origin	ssh://git@gitlab.xxx.domain/hellomin/Main.git(push)

```

然后同步该项目的代码：

```console
$ git fetch Main master & git pull Main master
```

也可以直接rebase

```console
$ git fetch Main/master
```

酱紫就可以在本地开始基于远端仓库的最新代码开发了！

下一篇讲写好了代码怎么提交一个merge request！
