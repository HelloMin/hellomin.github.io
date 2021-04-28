---
layout:     post
title:      简述Java中的ThreadLocal使用方法
subtitle:   
date:       2018-12-10
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Java
---
本文为译文，原文链接：https://www.baeldung.com/java-threadlocal。原文简单易懂，推荐阅读。

## 1. 引言

在这篇文章中，我们主要关注的是java.lang包中的[ThreadLocal](https://docs.oracle.com/javase/7/docs/api/java/lang/ThreadLocal.html) 类，它可以用来为当前的线程独立的保存数据，而我们只需要用一个ThreadLocal对象把数据封装起来就好了。

## 2. ThreadLocal API
TheadLocal的构造使得我们可以存储只有特定线程才能访问的数据。
比如，如果我们想要存储一个和特定线程相绑定的整数，可以这么做：

```
ThreadLocal<Integer> threadLocalValue = new ThreadLocal<>();
```
之后，当我们想要使用这个线程的数据，只需要调用get()或者set()方法。其机制可以简单理解为，ThreadLocal将数据保存进了一张以线程作为查找键key的Map表格中。
基于以上的描述，当我们对变量threadLocalValue调用get()函数，就可以获得该线程下的一个整数。
```
threadLocalValue.set(1);
Integer result = threadLocalValue.get();
```
我们还可以通过ThreadLocal的静态方法withInitial() 构造一个ThreadLocal对象，同时传给它一个初值：

```
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 1);
```

除了获取，如果需要删除一个值，我们可以使用remove函数：

```
threadLocal.remove();
```

为了了解如何恰当的使用ThreadLocal，首先，我们需要看一个没有使用ThreadLocal的例子，然后重写这个例子，从而更好的理解ThreadLocal的结构。
## 3. 用Map保存用户数据
假设我们需要一个程序，用于保存与特定用户相关的上下文数据，并且使用用户的id来访问这些数据

```
public class Context {
    private String userName;

    public Context(String userName) {
        this.userName = userName;
    }
}
```

我们希望每个用户id对应着一个线程。为此，我们创建了SharedMapWithUserContext类，这是一个实现了Runnable接口的类，其中run()函数的实现调用了UserRepository类中的数据库，从而返回一个特定用户id的上下文对象。
之后，我们将用户的上下文保存在ConcurentHashMap中，并且以用户id作为查找的键。

```
public class SharedMapWithUserContext implements Runnable {

    public static Map<Integer, Context> userContextPerUserId
      = new ConcurrentHashMap<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContextPerUserId.put(userId, new Context(userName));
    }

    // standard constructor
}
```

我们可以简单测试一下我们的代码：只需创建并且运行两个线程，分别赋予它们不同的用户id，然后判断我们已经在userContextPerUserId表格中创建了独立的两项：

```
SharedMapWithUserContext firstUser = new SharedMapWithUserContext(1);
SharedMapWithUserContext secondUser = new SharedMapWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();

assertEquals(SharedMapWithUserContext.userContextPerUserId.size(), 2);
```

##4. 使用ThreadLocal保存用户数据
我们可以使用ThreadLocal来保存用户上下文对象，重写上面的例子。每个线程都有自己独立的ThreadLocal对象。
使用ThreadLocal需要我们加倍细心，因为每个ThreadLocal对象都是与一个特定的线程相关联的。在我们的例子里，我们给每个特定用户id创建了一个专有的线程，我们创建了这些线程，也因此对它们有着完整的权限。
我们在run( )函数中获取用户上下文，并使用ThreadLocal的set()函数把它保存起来：

```
public class ThreadLocalWithUserContext implements Runnable {

    private static ThreadLocal<Context> userContext
      = new ThreadLocal<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContext.set(new Context(userName));
        System.out.println("thread context for given userId: "
          + userId + " is: " + userContext.get());
    }

    // standard constructor
```

为了测试，我们可以创建两个线程，它们都会对自己的用户id执行上文的操作：

```
ThreadLocalWithUserContext firstUser
  = new ThreadLocalWithUserContext(1);
ThreadLocalWithUserContext secondUser
  = new ThreadLocalWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();
```

运行以上代码之后，我们会在标准输出中看到，ThreadLocal已经为每个线程保存了数据：

```
thread context for given userId: 1 is: Context{userNameSecret='18a78f8e-24d2-4abf-91d6-79eaa198123f'}
thread context for given userId: 2 is: Context{userNameSecret='e19f6a0a-253e-423e-8b2b-bca1f471ae5c'}
```

可以看出，每个用户都有其特定的上下文。

## 5. 不要在ExecutorService中使用ThreadLocal

如果我们想使用一个ExecutorService，并提交一个Runnable任务给它，同时使用ThreadLocal可能会导致无法预料的结果。因为我们无法确定，是否特定userId对应的所有Runnable任务，每次被执行的时候，都是同一个线程在执行。

也因为每个线程都对应着不同的用户id，我们的ThreadLocal会被不同的用户id共享。因此，我们不应同时使用TheadLocal和ExecutorService。只有在我们确定哪个线程会执行哪个runnable任务的时候，才可以使用ThreadLocal。

##6. 小结
在本文中，我们学习了ThreadLocal的构造，并实现了一个使用共享的ConcurrentHashMap来为多个线程保存特定用户id上下文的逻辑。
之后，我们重写了这段代码，使用ThreadLocal来保存数据，从而保证数据只为特定的线程，特定的用户id使用。

以上所有的示例代码都可以在 [GitHub project](https://github.com/eugenp/tutorials/tree/master/core-java-concurrency)中找到– 这是一个Maven项目，所以可以简单的被引用和运行。
