---
layout:     post
title:      一次自定义域名的git-history定制之旅
subtitle:   
date:       2019-05-25
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Git
---
一个平凡的午后，刚提测需求等待bug上门的我，收到Mars先生推荐的一篇文章，里面介绍了一个相当酷的github项目:git-history。

## 什么是git-history？

看文件的diff也许是每个程序员每天必然会做的一件事情，昨天都干了啥，今天都干了啥，感谢git，让我们轻轻松松的查阅对项目中任何文件的所有修改历史。

然而，无论是github还是gitlab，在查看文件历史的时候，都不得不以commit为单位，导致你本来只想看一个文件的更改记录，却不得不在一大堆文件的更改里面找想要的那一个，而且没有办法用更直观的方式查看单个文件的变迁。

git-history就是为文件的改变提供最直观查看方式的工具。使用也足够简单：以一个github文件为例，你只需要把该文件的url域名做一些更改，就可以在网页上像翻书一样的查看文件的每一次变动，看下面的动画，多么愉快的体验～

<img src="/img/post_img/githistory-show.gif" alt="git-history"/>

很酷有没有！

看到这里，我赶紧兴致勃勃的用公司的gitlab域名gitlab.xxx.domain也试了试。果不其然，公司内部的gitlab不能用？怎么办？

所谓知己知彼，我们得先知道，这个git-history到底做了什么！

## git-history运行原理

为了进行测试，我分别找了一个github，一个gitlab文件，url如下：

https://**github.com**/marsishandsome/SparkSQL-Internal/blob/master/README.md

https://**gitlab.com**/blueyi/tensorflow_mirror/blob/master/README.md

然后，我按照git-history项目的要求，把他们的url分别改成这样:

https://**github.githistory.xyz**/marsishandsome/SparkSQL-Internal/blob/master/README.md

https://**gitlab.githistory.xyz**/blueyi/tensorflow_mirror/blob/master/README.md

两个页面都完美的展示了之前的效果。

那么，这个页面到底做了什么？

Chrome为我们这种问题少年提供了强有力的支持：在网页上右键—检查— 找到Network页面，再次刷新网页，让我们看看，页面实际发送了什么请求。

以github页面为例：

<img src="/img/post_img/github-net.png" alt="git-history network"/>

可以看到，最重要的两个请求分别是：

### 第一个：拉取commit信息

网页需要知道，在这个文件下，到底有过哪些commit，返回数据可以看出，是一个commit信息的数组，其中包含了commitid。

<img src="/img/post_img/github-commits.png" alt="github-commits.png"/>

第二个：拉取某个版本的file

针对某个commit，需要知道在这个commit中，该文件具体有什么修改

<img src="/img/post_img/github-commit.png" alt="github-commit.png"/>

gitlab发送的请求类似。

这时我们发现了，github和gitlab的请求还不一样？？？这有一个标准吗？我司的gitlab域名可以用gitlab的请求发送方式吗？

原来，不管是githuab还是gitlab，除了有网页的查看方式，都还提供了一套自己的API：

#### GITHUB:

https://developer.github.com/v3/

获取commit信息的url定义如下：

<img src="/img/post_img/github-commit-def.png" alt="github-commit-def.png"/>

#### GITLAB:

https://docs.gitlab.com/ee/api/

<img src="/img/post_img/gitlab-commit-def.png" alt="gitlab-commit-def.png"/>

到这里，答案已经很明显了：网页主要是使用github/gitlab API，首先查询文件的commit历史，然后根据每个commitid，一个个把commit详情查询出来，再以这些信息作出上面的效果。

所以，我需要做什么，才能在我司域名下也达到这样的效果呢？

## 我需要做的

#### 第0步：运行本地服务器，进行代码调试

首先，我很高兴地发现，git-history的开发人员，提供了一个本地命令行运行的方式，可以对本地文件进行类似的历史展示：

https://github.com/pomber/git-history/tree/master/cli

然而这个用法局限性很明显：我总不能让每个人都去下载一个git-history的项目，然后告诉大家，好了，你就每次在本地自己跑这个项目就行了！

谁要用啊！

必然是有一个服务器提供了这样的功能，你只需要更改一下域名就能达到效果，这样的工具才会有人愿意玩玩吧？所以，虽然本地运行很顺利，但这离我的目标还很远！

有意思的是，在查看代码目录以及运行方式的时候，我发现这实际上是一个react+nodejs的项目，而我们组在开发react native的时候恰好也是这样的用法。

<img src="/img/post_img/githistory-cli.png" alt="githistory-cli.png"/>

于是我猜，我应该也可以用它运行一个本地服务器吧？

于是，npm install  + npm start，我的服务器运行起来了～

然后我果断试了试，用我的本地localhost地址，可以展示之前的效果吗？

于是我实验了这样的域名改写：

http://gitlab.localhost:3000/blueyi/tensorflow_mirror/blob/master/README.md

成功了!所以我的本地服务器和git-history提供的服务器是一样的效果～

然而，我的私有域名url依然跪了：

http://gitlab.xxx.domain.localhost:3000/hellomin/gitlabAPITest/blob/master/README.md

<img src="/img/post_img/git-fail.png" alt="github-commit.png"/>

好吧，还是一步一步看，我到底跪在哪了？

#### 第一步：访问网站时，能发出正确的请求去拉取commit和文件

通过之前讲过的，查看Chrome网络的方式，我的服务器发出的拉取commit历史的请求如下：

https://gitlab.com/api/v4/projects/hellomin%2FgitlabAPITest/repository/commits?path=README.md&ref_name=master

然而，按照gitlab的定义，正确的请求应该是这样：

http://gitlab.xxx.domain/api/v4/projects/hellomin%2FgitlabAPITest/repository/commits?path=README.md&ref_name=master

如何映射到正确的域名???

这下真是不得不看源码了！

首先，网页的入口index文件如下：

<img src="/img/post_img/githistoryindex.png" alt="githistory-cli.png"/>

从截图可以看出，index实际上引用的是app，于是我们又看app.js文件

<img src="/img/post_img/githistoryapp.png" alt="githistory-cli.png"/>

可以看出，这的getGitProvider函数，很有可能就是决定了某个域名用什么api的地方，于是我们继续找！

<img src="/img/post_img/githistoryprovider.png" alt="githistory-cli.png"/>

终于被我找到了定义域名的地方，显然，每种支持的域名都有一个xxx-provider文件，里面包含了域名的api定义等。为了支持我司的域名，我当然毫不犹豫的添加了一个域名的定义。

然而，这个xxx-provider文件该怎么写呢？打开gitlab的gitlab-provider文件，我们终于找到了，拉取commit历史的url定义的地方

<img src="/img/post_img/githistory-commit-code.png" alt="githistory-cli.png"/>

拉取commit详情的url定义也在这里

<img src="/img/post_img/githistory-commit-detail.png" alt="githistory-cli.png"/>

于是，我依葫芦画瓢，也写了一个我司的xxx域名的api定义在这里

<img src="/img/post_img/githistory-commit-my.png" alt="githistory-cli.png"/>

到这里基本大功告成，还有一些需要添加我司域名的地方，就不一一赘述了。说来也简单，只需要在项目里全局搜一下SOURCE.GITLAB，看它都在哪用过了，基本它怎么加你怎么加，就好了～

#### 第二步：正确的请求可以收到正确的回应

请求发出去了，没毛病，可我还是没办法看到想看的页面！好生气！

原来，是因为我没有做登陆，所以我司的gitlab没有给我查看文件的权限。在这里，我用了一个暂时的方案，即为该项目生成了一个我自己的private_key并带在url里，这样就完成了鉴权，但是实际生产中这种做法必然是不可行的。

可是，总得先看到效果对不对！

于是，目前为止，我们的网页可以发出正确的请求，也可以收到争取的回应了，效果如何？

这是Mars的github项目下的一个readme文件，运行完美！

<img src="/img/post_img/githistory-mars.gif" alt="githistory-cli.png"/>

再看看我在我司gitlab上专门为了测试建的一个文件：

<img src="/img/post_img/githistory-test.gif" alt="githistory-cli.png"/>

出来了！终于出来了啊啊啊啊啊啊啊啊啊～～～

## 还有什么值得改进的？

1. 权限的读取：登陆显然是必须的，都这么明文带private_key，还活不活了...

2. git-history的原版是有一个chrome插件的，也是github上的其他开发者创建的项目，其实就是在gitlab/github的网页上插入一个按钮，点击可以为你自动跳转到域名更改后的网页，相比于直接人肉改域名，这种按钮的方式还是优雅的多，可以开发一个针对我司gitlab的chrome插件，应该也可以实现类似效果。

以上，就是我想做，做了，还没做的事情。是不是其实都很简单？

相比于一开始的懵逼/崇拜，自己实践之后才发现，原来如此！有趣还是有趣，但是少了分神秘感，多了分有理有据的踏实，这种感觉真好～

感谢Mars先生，在我研究过程中帮我剪掉了不少错误的思路分支，虽然不在我司上班了，但远程结对编程的感觉，还是很好～

Schönes Wochenende!

我的2019周更计划已完成：21/52