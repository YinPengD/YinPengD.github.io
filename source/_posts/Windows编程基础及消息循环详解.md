---
title: Windows编程基础及消息循环详解
date: 2019-07-24 21:33:21
tags: [Windows编程,消息,UE4]
categories:  Windows编程
---

最近在学DirectX和编写UE4和Windows交互插件时都使用到了Windows的许多API,顺便翻了下Windows核心编程，所以我将相关的Windows编程内容总结与整理下来，本文简要介绍了Windows的一些编程基础，还详细的了解了其中的消息循环。

<!--more-->

通俗地说， Win32 API 就是 Windows 系统按照 C 语言语法提供给用户的一个由诸多底层函数和结构组成的集合， 借助该集合， 我们就可实现应用程序与 Windows 操作系统之间的交互。 例如， 要想 通知 Windows 显示某一特定窗口，我们可以调用 Win32API 函数 ShowWindow。

在Windows中，几个应用耜序可以并发运行。因此，硬件资源（如CPU时钟周期、内存、甚至显示器）必须为多个应用程序所共享，为了防止多个应用程序在无序状态下对某些资源访问或修改时所造成的冲突，Windows应用程序不具备直接访问硬件的能力。Windows系统的一个主要任务就是管理当前实例化的程片以及处理多个应用程序之间资源的分配。所以，如果我们想让某一应用程序对其他应用程序产生影响， 只能将该什务交由Windows系统来处理。例如，要想显示一个窗口，必须调用ShowWindow函数。我们无法对显存进行自接的写操作。

### 事件、消息队列、消息及消息循环介绍

通常，Windows程序启动后只是等待某些事件的发生。**事件**有多种产生途径，一些常见的例子包括键盘中某些键被按下，鼠标单击，或当窗口被创建、尺寸发生变化、发生移动、被关闭、最小化、最大化或变为可见状态时。

当某个事件发生时，Windows会为该驻件所针对的应用程序发送一条**消息**，表明该事件的发生，井在该应用程序的消息队列中增加一条消息，该**消息队列**只是一个保存了应用程序所接收到的消息的一个优先队列。应用程序在一个消息循环中不断地循环掉用GetMessage函数（或者是PeekMessage函数）来接受病检查消息队列，这个过程叫做**消息循环**。当接收到一条消息时，便将其分派给接收该消息的特定窗口的窗口过程。（不要忘记一个应用程序内部可能包含多个窗口）窗口过程是个与应用程序各窗口相关的特殊函数。（每个窗口都有一个窗口过程，但多个窗口可共亨同一个窗口过程。所以，我们就不必为每个窗口都编写一个窗口过程。）**窗口过程**负责该窗口的所有消息。

#### 消息传递的过程

用户或应用程序的某些行为会产生一些事件。操作系统找到事件所属的应用程序，然后向该应用程序发送条相应的消息。然后，该消息就被加入到该引用程序的消息队列中。之后，应用程序不断地检杳消息队列，每当接收到一条消息时，应用程序就将该消息分发给与该消息所属窗口相关的窗口过程。最后，窗口过程执行与当前消息对应的指令。

![mark](http://yp.guohaonan.cn/MPic/20190724/xGp32v2B5pDD.png?imageslim)

#### 消息队列

Windows操作系统的内核空间中有一个系统消息队列（system message queue），在内核空间中还为每个UI线程分配各自的线程消息队列(Thread message queue)。在发生输入事件之后，Windows操作系统的输入设备驱动程序将输入事件转换为一个“消息”投寄到系统消息队列；操作系统的一个专门线程从系统消息队列取出消息，分发到各个UI线程的输入消息队列中。

Windows的事件驱动模式，并不是操作系统把消息主动分发给应用程序；而是由应用程序的每个UI线程通过“消息循环”代码从UI线程消息队列获取消息。

#### Windows消息类别

- 键盘消息
- 字符消息
- 鼠标消息
- 定时器消息
- 控件消息
- 跨进程发送数据的消息

#### 消息传递时传递的消息参数

- message
  - 各种各样的消息值[Windows常用消息大全](https://blog.csdn.net/zhangguofu2/article/details/19236081)

| 消息范围              | 说 明                |
| --------------------- | -------------------- |
| 0 ～ **WM_USER** – 1  | 系统消息             |
| **WM_USER** ～ 0x7FFF | 自定义窗口类整数消息 |
| **WM_APP** ～ 0xBFFF  | 应用程序自定义消息   |
| 0xC000 ～ 0xFFFF      | 应用程序字符串消息   |
| > 0xFFFF              | 为以后系统应用保留   |

- wParam包含虚拟键码（virtual-key code），表示按下或释放的键
- lParam包含按键6个字段信息：
  - 重复按键次数（Repeat Count，0～15 位）：通常设为1。大于1说明按键速度大于程序处理能力。可以根据实际需要忽略或处理。
  - OEM扫描码(scan code，16～23位)：硬件产生的代码。
  - 扩充键标志（extended key，24位）：如果为扩充键（如右侧的Alt键或Ctrl键）按下时为1，否则为0。
  - 保留位（25～28位）：保留位是系统缺省保留的，一般不用。
  - 上下文代码（context code，29位）：如同时按下ALT，标志为1；否则为0。WM_SYSKEYUP或WM_SYSKEYDOWN常为1。WM_KEYUP或WM_KEYDOWN常为0。当所有程序都最小化时，没有窗口具有输入焦点，Windows仍将发送键盘消息给活动窗口；所有的按键都会产生WM_SYSKEYUP与WM_SYSKEYDOWN消息，此情况下如果没按下ALT，该字段为0，这样使最小化的活动窗口不处理这些按键。对于一些非英文键盘，有些字符是shift等组合键产生的，这时内容代码为1，但是其是非系统按键。
  - 键先前状态（previous key state，位30）：键此前是释放的，则为0，还则为1。很明显UP为1，DOWN可以为1或0，为1表示该键自动重复。
  - 转换状态（transition state，31位）：键被按下为0，键被松开时为1。如UP为1，DOWN为零。

#### 在Window程序中捕获消息

```
// 在程序窗口被初始化前绑定消息调用函数
BOOL InitApplication(HINSTANCE hinstance, int nCmdShow)
{
	HWND      hwnd;
	WNDCLASS  wndclass;
	wndclass.style       = CS_HREDRAW | CS_VREDRAW;
	wndclass.lpfnWndProc = WndProc; //绑定消息调用函数
	wndclass.cbClsExtra  = 0;
	wndclass.cbWndExtra  = 0;
	wndclass.hInstance   = 0;
	wndclass.hIcon       = LoadIcon(NULL, IDI_APPLICATION);
	wndclass.hCursor     = LoadCursor(NULL, IDC_ARROW);
	wndclass.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);
	wndclass.lpszMenuName  = NULL;
	wndclass.lpszClassName = szAppName;
}
// 循环调用，并处理获得的消息
LRESULT CALLBACK WndProc(HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam)
{
	switch (message)
	{
	case WM_CREATE:
		DragAcceptFiles(hwnd, TRUE);
		return 0;
	case WM_DESTROY:
		PostQuitMessage(0);
		return 0;
	}
	return DefWindowProc(hwnd, message, wParam, lParam);
}
```

#### 在UE4中捕获Windows消息

```
static FExampleHandler Handlers;
void UWindowsMessageHandlerBPLibrary::WindowsMessageHandlerSampleFunction()
{
    /**获取UE4程序的应用窗口*/
	FWindowsApplication* Application = (FWindowsApplication*)FSlateApplication::Get().GetPlatformApplication().Get();
	if (Application != nullptr)
	{
		Application->AddMessageHandler(Handlers); // 绑定消息调用函数
	}
}
class FExampleHandler
	: public IWindowsMessageHandler
{
public:
	FString PP;
	// 使用UE4处理windows消息
	virtual bool ProcessMessage(HWND Hwnd, uint32 Message, WPARAM WParam, LPARAM LParam, int32& OutResult) override
	{
		if (Message == WM_CAPTURECHANGED)
		{
			DragAcceptFiles(Hwnd, true);
		}
	return false;	
	}
};

```

#### 相关了解：

**句柄**是一种指向指针的指针。由于windows是一种以虚拟内存为基础的操作系统，其内存管理器经常会在内存中来回的移动对象，以此来满足各种应用程序对内存的需求。而对象的移动意味着对象内存地址的变化，正是因为如此，如果直接使用指针，在内存地址被改变后，系统将不知道到哪里去再调用这个对象。windows系统为了解决这个问题，系统专门为各种应用程序腾出了一定的内存地址（句柄）专门用来记录这些变化的地址（这些内存地址就是指向指针的指针），这些内存地址本身是一直不变化的。windows内存管理器在移动某些对象之后，他会将这些对象新的内存地址传给句柄，告诉他移动后对象去了哪里。

**HWND**是窗口句柄，其中包含窗口的属性。例如，窗口的大小、显示位置、父窗口。

HDC是图像的设备描述表，窗口显示上下文句柄，其中可以进行图形显示。

#### 相关文章：

- [Microsoft Windows的消息循环]([https://www.wikiwand.com/zh-sg/Microsoft_Windows%E7%9A%84%E8%A8%8A%E6%81%AF%E8%BF%B4%E5%9C%88#/Windows%E6%B6%88%E6%81%AF%E7%B1%BB%E5%88%AB](https://www.wikiwand.com/zh-sg/Microsoft_Windows的訊息迴圈#/Windows消息类别))
- [Windows 常用消息大全](https://blog.csdn.net/zhangguofu2/article/details/19236081)

