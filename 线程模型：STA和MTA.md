---
title: 线程模型：STA和MTA
description: 
published: true
date: 2025-06-27T01:30:16.672Z
tags: wpf, .net, winform
editor: markdown
dateCreated: 2025-06-26T09:33:33.902Z
---

# 线程模型：STA和MTA
## 1、NET支持两种线程模型：STA和MTA。
> **STA**（Single Thread Apartment，单线程套间）是一种线程模型， 用在程序的入口方法上，来指定当前线程的ApartmentState是STA，用在其他方法上不产生影响。 这个属性只在COM（Component Object Model，组件对象模型）Interop有用，如果全部是托管代码则无用。 
> 其它的还有MTA（Multi Thread Apartment，多线程套间）、NTA（Neutral Threaded Apartment，中立线程套间）。
> STA and MTA 之间最大的区别就是MTA 可以在同一个apartment 中使用所有的共享资源并发执行多个线程。
> 而多个STA虽然可以共享数据，但是不能并发执行线程,存在性能问题。
{.is-info}

## 2、STA模型和UI框架的关系
线程模型（Apartment State） 主要是针对 COM 互操作（Drag-Drop、剪贴板、Ole/ActiveX 等）的线程约束机制：

 ### 2.1 STA（Single-Threaded Apartment）：
>  单线程公寓：一个 STA 公寓里只包含它的创建线程，所有对 COM 对象的调用都要经由该线程的消息循环（Dispatcher／WindowProc）来调度。
>  必须有消息循环（Win32 的 GetMessage／DispatchMessage 或 WPF 的 Dispatcher.Run）来驱动 COM 消息、UI 渲染、鼠标键盘事件。
>  WPF/WinForms 要求	WPF 和 WinForms 的所有 UI 线程都必须是 STA，否则创建窗口时会抛出：The calling thread must be STA…。	
用途：**任何要承载 WPF/WinForms 窗口、使用剪贴板、OLE、拖放、ActiveX 控件的线程；典型是应用的主线程，或你专门创建的 Secondary UI Thread。**
{.is-info}

 
 ### 2.2 MTA（Multi-Threaded Apartment）
> 多线程公寓：所有加入 MTA 的线程共享同一个上下文，调用不再需要经过单一线程的消息泵。
> 不要求特定线程有消息循环；线程间可并发调用，不通过单一队列排队。
> MTA 线程上不能直接创建或操作 WPF/WinForms 的 UI 元素。
**用于执行纯后台计算、不需要消息循环的任务。例如并行运算、I/O、网络请求，或者需要访问线程安全的 COM 对象（那些支持 MTA）的场景。**
{.is-info}

## 3、STA线程中创建窗体的方法
### 3.1 Winform使用Application.Run(form)运行；
> ``` var thread = new Thread(() =>
> {  
>     // 2．创建窗体
>     var form = new SecondaryForm();
>     
>     // 3．启动消息循环并显示窗体
>     //    这句会阻塞当前线程，直到窗体关闭
>     Application.Run(form);
> });
> thread.SetApartmentState(ApartmentState.STA);
> thread.IsBackground = true;
> thread.Start(); 
{.is-success}

### 3.2 winform 使用Form.ShowDialog()运行：


>
>ShowDialog() 会在内部启动一个临时的消息泵，让窗体可交互、可重绘。
直到对话框关闭，ShowDialog() 才返回 。
> '''c#// 验证
> var thread = new Thread(() =>
> {  
>     // 2．创建窗体
>     var form = new SecondaryForm();>     
>     // 3．启动消息循环并显示窗体
>     //    这句会阻塞当前线程，直到窗体关闭
>     form.ShowDialog();   
> });
> thread.SetApartmentState(ApartmentState.STA);
> thread.IsBackground = true;
> thread.Start(); '''
{.is-success}

### 3.3 Wpf使用Dispatch.Run()启动消息循环
> 
> ``` // 在主线程中启动一个新的 STA 线程
> var thread = new Thread(() =>
> {
> 
>     // 在该线程上创建并显示 WPF 窗口
>     var window = new SecondaryWindow();
>     window.Show();
>     // 启动该线程的 Dispatcher 消息循环
>     System.Windows.Threading.Dispatcher.Run();
> });
> thread.SetApartmentState(ApartmentState.STA);
> thread.IsBackground = true;
> thread.Start();
{.is-success}

### 3.4 Wpf使用ShowDialog（）启动消息循环
> 
>ShowDialog() 在内部创建并运行一个 Dispatcher 循环，直到窗口关闭才返回。
这种方式更简便，但窗口会以模态形式存在，阻塞当前线程直到关闭。
>
> ``` // 在主线程中启动一个新的 STA 线程
> var thread = new Thread(() =>
> 	{
> 
>     // 在该线程上创建并显示 WPF 窗口
>     var window = new SecondaryWindow();
>      bool? result = dlg.ShowDialog();
> });
> thread.SetApartmentState(ApartmentState.STA);
> thread.IsBackground = true;
> thread.Start();
{.is-success}

**非模态窗口：** 必须在 STA 线程里手动调用 Dispatcher.Run() 来启动消息泵。

**模态窗口：** 直接调用 ShowDialog()，WPF 会自动在内部启动消息循环。



