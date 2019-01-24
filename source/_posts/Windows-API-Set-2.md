---
title: Windows API Set 2
date: 2018-06-01 17:48:19
tags: 编程
---
# DWM(Desktop Window Manager)
本来打算两天更一次的，不过以后估计都要周更或者半月更了。。。高一也不容易啊。。。还要忙oi  

### 首先来说一说什么是DWM
>The desktop composition feature, introduced in Windows Vista, fundamentally changed the way applications display pixels on the screen. When desktop composition is enabled, individual windows no longer draw directly to the screen or primary display device as they did in previous versions of Windows. Instead, their drawing is redirected to off-screen surfaces in video memory, which are then rendered into a desktop image and presented on the display.  
Windows Vista中引入的桌面组合功能从根本上改变了应用程序在屏幕上显示像素的方式。启用桌面组合后，各个窗口不再像以前版本的Windows中那样直接绘制到屏幕或主显示设备上。相反，他们的绘图被重定向到视频内存中的离屏表面，然后渲染成桌面图像并显示在显示器上。

>Desktop composition is performed by the Desktop Window Manager (DWM). Through desktop composition, DWM enables visual effects on the desktop as well as various features such as glass window frames, 3-D window transition animations, Windows Flip and Windows Flip3D, and high resolution support.  
桌面组合由桌面窗口管理器（DWM）执行。通过桌面组合，DWM支持桌面上的视觉效果以及各种功能，如玻璃窗框，3D窗口过渡动画，Windows Flip和Windows Flip3D以及高分辨率支持。

>The Desktop Window Manager runs as a Windows service. It can be enabled and disabled through the Administrative Tools Control Panel item, under Services, as Desktop Window Manager Session Manager.  
桌面窗口管理器作为Windows服务运行。可以通过“服务”下的“管理工具”控制面板中的“桌面窗口管理器会话管理器”来启用和禁用它。

以上先引用msdn里的一段描述。不管你看得懂还是看不懂，总而言之就是在原来的直接桌面窗口绘制上面加了一层，让窗口的外观和切换变得更加的“炫酷”（虽然我一点都不这么觉得）。比如说毛玻璃效果和win7的3D窗口切换就是通过这个东西来实现的。不过在Win10里面，这个东西的存在感似乎变低了。估计是M$爸爸又在打什么主意了吧。

### 然后我们来说一些正事
待续