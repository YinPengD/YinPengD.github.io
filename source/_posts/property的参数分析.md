---
title: ios中property的参数分析
date: 2017-11-06 22:19:21
tags: ios
categories:  ios
---


@Property是声明属性的语法，它可以快速方便的为实例变量创建存取器，并允许我们通过点语法使用存取器,@property表示由编译器来自动实现属性的getter/setter方法，不需要你自己再手动去实现,这篇文章对@property中四组对立的参数进行了基本的分析与总结，并对控件中什么时候使用强引用，什么时候使用弱引用进行了讨论。

<!--more-->
##### 第一组：内存管理特性 retain assign copy strong weak
##### 第二组：读 /写特性 readwrite readonly
##### 第三组：多线程特性 nonatomic atomic
##### 第四组：方法名特性 ： setter getter


**assign（默认参数）：使用场景：基本数据类型\(int\float\)\枚举\结构体**
setter方法直接赋值，不进行任何retain操作，不改变引用计数，id类型也要用assign，所以一般iOS中的代理delegate属性都会用assign来标示


**retain：使用场景：成员变量**
生成符合内存管理的set方法（release旧值，retain新值），适用于OC对象的成员变量。


**copy：使用场景： NSString\NSMutableString\block**
生成符合内存管理的set方法（release旧值，copy新值），不过该属性会被复制一个新的副本。很多时使用copy是为了方式Mutable（可变类型）在我们不知道的情况下修改了属性值，而用copy可以生成一个不可变的副本防止被修改。


**Strong：使用场景： strong :其他OC对象**
强引用，其存亡直接决定了所指向对象的存亡。使用该特性实例变量在赋值时，会释放旧值同时设置新值，对对象产生一个强引用，即引用计数+1。如果不存在指向一个对象的引用，并且此对象不再显示在列表中，则此对象会被从内存中释放。


**weak：weak :代理\UI控件**
弱引用，不决定对象的存亡。属性表明了一种”非拥有关系“，既不释放旧值，也不保留新值，即引用计数不变，当指向的对象被释放时，该属性自动被设置为nil。即使一个对象被持有无数个弱引用，只要没有强引用指向它，那么还是会被清除。


readwrite（默认参数）：同时生成set、get方法的声明与实现

readonly：只生成get方法的声明与实现（不生成set的方法的声明与实现）

atomic（默认参数）：原子性，性能低（一般开发OC中的APP不推荐使用，做金融等高安全的时候使用）

nonatomic：非原子性，性能高（强烈推荐使用，性能高）

setter：给成员变量的set方法重命名，set方法默认命名：- （void） set成员变量名（成员变量名首字母大写）：（成员变量数据类型）成员变量名

getter：给成员变量的set方法重命名，get方法默认命名：- （成员变量数据类型） 成员变量名


### 为什么 iOS 开发中，控件一般为 weak 而不是 strong？

从storyboard或者xib上创建控件，在控件放在view上的时候，已经形成了如下的引用关系,以UIButton为例：**UIViewController-&gt;UIView-&gt;subView-&gt;UIButton，UIButton已经被控件强引用了，所以拖出来的控件是弱引用**

```
@property(nonatomic,weak) IBOOutlet UIButton *btn;
```

而当自己自定义Button按钮的时候就需要对它进行强引用了：

```
@property(nonatomic,strong) UIButton *btn;
```

strong指针能够保持对象的生命，一个对象只要有strong指针指向它，那么它就不会被释放；相反的，如果一个没有一个strong指针指向它，那么它将会被自动释放。默认所有实例变量和局部变量都是Stong指针

weak型的指针变量仍然可以指向一个对象，但不属于对象的拥有者。即当对象被销毁的时候，这个weak指针也就自动指向nil（空指针）。
