---2017-07-15 21:08---

需求：在 A 应用内启动 B 应用，如果 B 应用未安装则跳转应用商店搜索。

启动方式使用 Uri 启动，本文使用尽可能简单，并且能拿来直接用的代码。不涉及启动后的应用数据交互，如需深入了解，请戳 MSDN：https://docs.microsoft.com/zh-cn/windows/uwp/launch-resume/launch-app-with-uri

## 1. 获取 B 应用 Uri 以及 B 应用激活事件

如果 B 应用已注册 Uri 的话，那很好，记住它备用，可以跳过看第2点了。如果没有，接着看下面。

那么如何为 B 应用注册 Uri 呢？

打开 B 应用程序清单 Package.appxmanifest ，在“声明”选卡项中添加一个新的“协议”声明（如果你做过后台任务的话那一定很熟悉）。在“名称”(name)那一栏中填写你需要注册的 Uri （随便编）。填写完成后保存，这样就完成了 Uri 的注册。

![](http://blogres.zhangyue.xin/18-2-13/65456885.jpg)

Uri 启动应用是以激活的形式启动的应用，和磁贴与Toast通知的激活启动一样，需要在 App.xaml.cs 文件里重写 OnActivated() 事件。Uri 激活时会赋一个 ID，在 OnActivated() 事件中可以进行一些处理，比如跳转其他不同页面，下面的代码是像 OnLaunched() 事件一样直接跳转到 MainPage.xaml。

```c#
protected override void OnActivated(IActivatedEventArgs args)
{
    Frame rootFrame = Window.Current.Content as Frame;

    if (rootFrame == null)
    {
        rootFrame = new Frame();

        rootFrame.NavigationFailed += OnNavigationFailed;

        Window.Current.Content = rootFrame;
    }

    rootFrame.Navigate(typeof(MainPage));

    Window.Current.Activate();
}
```
 
## 2. A 应用启动 B 应用

知道了 B 应用的 Uri 后，下面就要在 A 应用中启动 B 应用了。和 MSDN 的“推荐设置”方法不同，这里采用的是先判断 B 应用在设备上存不存在，如果存在直接启动，不存在启动商店搜索。下面直接给出代码，注意把 Uri 换成相应的 Uri 即可。 Uri 内的 ProductID 是一定要写的，不然会报错。

```c#
var result = await Windows.System.Launcher.QueryUriSupportAsync(new Uri("link.qtz:?ProductId=1"), Windows.System.LaunchQuerySupportType.Uri);

if (result == LaunchQuerySupportStatus.Available)
{
    await Windows.System.Launcher.LaunchUriAsync(new Uri("link.qtz:?ProductId=1"));
}
else
{
    await Windows.System.Launcher.LaunchUriAsync(new Uri("ms-windows-store://pdp/?productid=9nblggh4t3t0"));
}
```