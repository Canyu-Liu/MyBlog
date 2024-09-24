---
title: WinUI3配置文件选择器
date: 2023-08-15 15:29:17
tags: WinUI3
categories:
- [.NET,WinUI3]
---
WinUI 3 Gallery中关于文件选择器的示例代码直接复制过来用仍需要一些额外的操作。
<!--more-->
{% note default %}
2024-09-23更新：把原先的两篇文章精简成了一篇。写太长了自己隔一年回头看都觉得有点麻烦...
{% endnote %}

## 初始配置
示例代码中用到了WindowHelper和NativeHelper这两个类，需要手动从WinUI3的仓库复制到自己的项目中，不然代码会报错。
![这是图片](/uploads/124758.jpg "Magic Gardens")
![这是图片](/uploads/193604.jpg "Magic Gardens")
WindowHelper和NativeHelper类在Helper文件夹下。
![这是图片](/uploads/192624.jpg "Magic Gardens")

## 在MainWindow中使用
使用文件选择器需要获取窗口句柄，在MainWindow中使用不需要GetWindowForElement，先将其注释掉。然后将GetWindowHandle括号里的window改成this。
{% codeblock lang:csharp %}
//var window = WindowHelper.GetWindowForElement(this);
//获取当前的WinUI 3窗口的窗口句柄
var hWnd = WinRT.Interop.WindowNative.GetWindowHandle(this);
{% endcodeblock %}

## 在MainPage中使用
在MainPage中就不能靠GetWindowHandle(this)来获取窗口句柄了，我们需要先使用GetWindowForElement获取到窗口元素。
还原一下之前的代码
{% codeblock lang:csharp %}
var window = WindowHelper.GetWindowForElement(this);
var hWnd = WinRT.Interop.WindowNative.GetWindowHandle(window);
{% endcodeblock %}
然后在MainWindow中使用WindowHelper的TrackWindow方法。
{% codeblock MainWindow.xaml.cs lang:csharp %}
public MainWindow()
{
    InitializeComponent();
    AppWindow.SetIcon(Path.Combine(AppContext.BaseDirectory, "Assets/WindowIcon.ico"));
    Content = null;
    Title = "AppDisplayName".GetLocalized();

    dispatcherQueue = Microsoft.UI.Dispatching.DispatcherQueue.GetForCurrentThread();
    settings = new UISettings();
    settings.ColorValuesChanged += Settings_ColorValuesChanged;
    //在这里跟踪MainWindow
    WindowHelper.TrackWindow(this);
}
{% endcodeblock %}
这行代码是必须要加的，不加的话GetWindowForElement会得到一个null。

参考文章：
[stackoverflow](https://stackoverflow.com/questions/76856384/winui3-exception-thrown-while-trying-to-create-a-filepicker)
[stackoverflow](https://stackoverflow.com/questions/76435347/cant-use-winui-3-file-picker)