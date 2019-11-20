---
title: KVC&KVO的初步认识
date: 2017-07-15 11:46:34
tags: ios
categories: ios
---

KVO/KVC 是观察者模式在 Objective-C 中的实现，以非正式协议（Category）的形式被定义在 NSObject 中。从协议的角度看，是定义了一套让开发者遵守的规范和使用的方法。在 Cocoa 的 MVC 框架中，架起 ViewController 和 Model 沟通的桥梁。

<!--more-->
## KVC概述：即:Key-value coding,它是一种使用字符串标识符，间接访问对象属性的机制
### 使用：

##### 利用KVC进行简单赋值：

  ```
  void test(){
      YPPerson *person = [[YPPerson alloc] init];
      [person setValue:@"王五" forKey:@"name"];                   //KVC赋值
      [person setValue:@"19" forKeyPath:@"money"];                //自动类型转换
      NSLog(@"%@-------%.2f",person.name,person.money);
      }
  ```
##### 利用KVC修改私有成员变量

  ```
  void text2(){
      YPPerson *person = [[YPPerson alloc]init];
      [person setValue:@"88" forKeyPath:@"_age"];              //私有变量
      NSLog(@"%@",person);
  }
  ```
##### 利用KVC将模型转为字典

  ```
  普通的将模型转为字典：
  self.name =dict[@"name"];
  self.money = [dict[@"money"] floatValue];      //floatValue 装换为float类
  ```

  ```
  用kVCV将模型转为字典
  - (instancetype)initWithDict:(NSDictionary *)dict{
      if (self = [super init]) {
          [self setValuesForKeysWithDictionary:dict]；           ／／优点：一行代码就把所有的模型转为了字典
  }

    使用场景：简单的字典转模型

    开发中不建议使用setValuesForKeyWithDictionary:

           1.字典中的KEY必须在模型的属性中找到

            2.如果模型中带有模型就不好用
  ```
##### 取出所有模型的属性值

  ```
   YPPerson *person1 = [[YPPerson alloc ]init];
          person1.name = @"zhangsan";
          person1.money  = 12.99;
          person2.name = @"wangwu"
          person2.money = 13.45;
        
          NSArray allPersons = @[person1,person2];
          NSArray allPersonName = [allPersons valueForKey:@"naem"];
          NSLog(@"%@",allPersonName);
  ```

---
## KVO概述：即:Key-value Observing,当指定的对象的属性被修改了，允许对象接受到通知的机制。每次指定的被观察对象的属性被修改的时候，KVO都会自动的去通知相应的观察者


> 允许对象观察另一个对象的属性。该属性值改变时，会通知观察对象,

##### 如何监听：

```
-（void)viewDidLoad];
    [super viewDidLoad];
    
    YPPerson *person = [[YPPerson alloc] init];
    person.name = @"zs";
```

##### 作用：给对象绑定一个监听器（观察者）

Oberver : 观察者

KeyPath :要监听的属性

Options : 选项（方便在方法中拿到属性值）

```
[Person addObserver:self forKeyPath:@"name" options:NSKeyValueObserveringOptionNew | NSKeyValueObservingOptionOld context:nil];
person.naem = @"ls";
person.name = @"ww";
//移除监听
```

##### 当监听的属性值发改变是就调用这个方法

keyPath 要改变的属性

Object 要改变的属性的对象

chang 改变的内容

context 上下文

```
-（void)observeValueForKeyPath:(NSString *)KeyPath ofObject:(id)Object change:(NSDictionary<NSString *,id> *)chang context{
    NSLog(@"%@---------%@---------%@", keyPath, object, change);
}
```

会打印出：

new = ww;

old = ls;



