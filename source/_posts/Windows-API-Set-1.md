---
title: Windows API Set 1
date: 2018-06-01 17:47:16
tags: 编程
---

# Windows Accessibility
### 综述
这个里面的函数似乎都用处不大，但是为了这个系列的完整性，姑且还是把他们记下来吧。~~从名字就可以看出来，这个里面的函数主要是Windows辅助功能之类的玩意儿，用来帮助残疾人使用Windows。。。~~//大雾

### 函数列表
能用的就3个
* RegisterPointerInputTarget (>= Windows 8)
* SoundSentryProc
* UnregisterPointerInputTarget (>= Windows 8)

#### RegisterPointerInputTarget && UnregisterPointerInputTarget
这两个函数好像主要是用来控制触屏和鼠标输入的函数，基本用不到系列。。。  
** 函数原型**

```cpp
BOOL WINAPI RegisterPointerInputTarget(
  _In_ HWND                hwnd,
  _In_ POINTER_INPUT_TYPE  pointerType
);

```

```cpp
BOOL WINAPI UnregisterPointerInputTarget(
  _In_ HWND                hwnd,
  _In_ POINTER_INPUT_TYPE  pointerType
);
```

这两个函数可以把**所有的**鼠标指针类型的输入定向到注册的程序。一次只能有一个窗口注册此函数。

#### SoundSentryProc
更加没有用的函数，可以在计算机内置扬声器发出声音是在屏幕上进行显示。/笑  
**函数原型**
```cpp
LRESULT CALLBACK SoundSentryProc(
   DWORD dwMillisec,
   DWORD fdwEffect
);

```
在这个函数下面涉及到一个叫做``SystemParametersInfo``的函数，这个函数的功能~~diaobao~~非常的强大，过几天再说。

### 相关知识整理
- 首先是hWnd这个神奇的玩意儿。这个名字是有h（handle）和Wnd（window）组合而成的。所以它指的是一个窗口句柄。在宏展开之后是
```cpp
struct nameHWND__ {int unused;}; 
typedef sturct nameHWND__ * HWND;
```
所以这家伙是一个指向结构体的指针，这个结构体包含一个成员变量unused。至于句柄这个东西，过天专门写一篇来整理一下。

- 其次，就没有了