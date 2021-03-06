---
layout:     post
title:      JS使用Json传递可变参数
subtitle:   使用Json作为参数格式，更精确的传递参数吧
date:       2018-04-18
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Tool
---

当我们写一个需要传递多个参数的函数的时候，我们通常会这么写：

```js
helloFunction(paramA, paramB, paramC) {
  ...
  }
```
当有的参数常常为相同值，为了省力，我们会为个别参数预设默认值：

```js
helloFunction(paramA, paramB = b, paramC = c) {
  ...
  }
```
这样一来，我们在使用这个函数的时候就可以少写点东西了：

```js
helloFunction(e); // 效果同 helloFunction(e, b, c)
```
但是，如果我们想对paramA和paramC赋值，同时使用paramB的默认值呢：

```js
helloFunction(e,,f); // 效果同 helloFunction(e, b, f)
```
于是，我们的代码中出现了奇怪的两个逗号，上面的例子还好，糟糕的时候，也许会有类似

```js
helloFunction(e,,,d,,,,f);
```
这样可怕的代码出现。

为了避免这种情况，使传参和使用参数的人都更清楚到底哪些参数被传进来了，我们可以使用Json，把需要传递的参数包装起来：

```js
helloFunction({paramA = a, paramB = b, paramC = c}) {
  ...
  }
```
这样，当我们只需要对paramB的赋值进行改变的时候，我们可以这么写：

```js
// 效果等于 helloFunction({aramA : a, paramB : f, paramC : c})
helloFunction({paramB : f});
```
是不是清楚了很多？

这样一来唯一的缺点就是，使用API的人在调用的时候要明确写清楚，自己到底是在为哪个参数赋值。有利有弊，具体情况可以自己斟酌哪样更好。