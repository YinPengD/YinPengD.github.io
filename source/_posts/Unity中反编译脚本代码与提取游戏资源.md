---
title: Unity中反编译脚本代码与提取游戏资源
date: 2017-12-23 18:55:34
tags: [Unity3D,反编译,脚本,资源]
categories: Unity3D
---

在Unity的脚本代码的学习之中，我们除了通过看教程中的脚本演示，Unity博客，论坛，知识板块等网站的学习,还可以通过看已上线的游戏的源码，通过这种看已上线游戏源码方式的优点在于相比上述的方式要更加的**规范**，**深入**，**全面**（毕竟是已完成的游戏）,当我们缺少优质素材用于练习时我们也可以通过提取游戏资源的方式，提取你想要的风格的游戏资源，这将会大大方便我们在学习游戏开发中的进程。
<!--more-->

## 1.Unity中反编译脚本代码

#### 1.1 、反编译工具

**dnSpy** 是一款针对 .NET 程序的逆向工程工具,基于 ILSpy 发展而来的 .net 程序集的编辑，反编译，调试神器。。该项目包含了反编译器，调试器和汇编编辑器等功能组件，而且可以通过自己编写扩展插件的形式轻松实现扩展。该项目使用 dnlib 读取和写入程序集，以便处理有混淆代码的程序（比如恶意程序）而不会崩溃。

#### 1.2、Unity源码文件位置

unity的源码都存放在dll中，那么反编译的工作就是把从dll 中提取出源码，基本上我们的代码都在**Assembly-CSharp.dll**这个文件中

#### 1.3、破解Unity源码文件

> 将Assembly-CSharp.dll文件拖动到dnspy反编译工具中就实现了破解
>

我用一个的独立游戏做的示例：

![反编译源码](http://yp.guohaonan.cn/unity/notes%E5%8F%8D%E7%BC%96%E8%AF%91%E6%BA%90%E7%A0%81.png)

*上图可以看出反编译出了编码者完整的代码逻辑*

## 2.提取游戏中的资源

> 游戏中的美术资源没法完全加密，即便使用特别复杂的加密方式，也有办法将其中的资源提出来,这里只借助现成的工具，做些浅显的资源提取。

#### 2.1、资源提取工具

> 相比于Disunity与UnityAssetsExplorer工具，UnityStudio 拥有可视化界面，可以批量导出贴图，模型，字体，音频等，可以预览，最新版支持Unity5.x，所以在这使用**UnityStudio**作为资源提取工具。

#### 2.2、下载地址

> https://github.com/Perfare/UnityStudio

#### 2.3、Unity中资源位置

>点击菜单 File 中的“Load folder...”，载入 unity 游戏的 Assets -> bin -> data 文件夹。也可以选择“Load file...”，载入 .unity3d 或者 .boundle ，.assets文件。

#### 2.4、预览资源

> 选择 Assets List，可以看到里面有很多资源文件。点击即可在右侧窗口进行预览，可以预览贴图，Shader，模型的资源,还可以直接播放音频。

#### 2.5、保存资源

> 选中需要的资源，点击菜单工具栏里的 Export -> Selected assets，即可将选中的资源保存到本地。

用一个的独立游戏做的示例：
![提取资源](http://yp.guohaonan.cn/unity/notes%E6%8F%90%E5%8F%96%E8%B5%84%E6%BA%90.png)

*上图中在seneHierarghy界面中可以查看资源的结构目录，资源中的音乐文件都可以直接播放测试，在右边的视图中可以使用wsad将对模型进行旋转查看*

### 其他相关博客：

如何避免代码被反编译 ： http://www.xuanyusong.com/archives/2664
Unity3d 反编译破解游戏 简单示例: http://blog.csdn.net/huutu/article/details/46573327)