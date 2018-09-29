---2018/3/1 18:00:00---

前几天刚换了电脑，在配置 Xamarin 开发环境的时候出了点问题。以前很少升级 Visual Studio ，重装之后发现 Xamarin.Android 的 SDK 管理器变成了 Xamarin SDK Manager。在其中下载了最新的 Android 8.1 的镜像，用以前的 AVD 管理器中新建虚拟机失败，提示我找不到镜像，试了好多办法都不顶用，最后翻文档才找到了解决方法。

![](http://blogres.zhangyue.xin/18-3-1/83481643.jpg)

文档地址：https://developer.xamarin.com/guides/android/getting_started/installation/android-emulator/xamarin-device-manager/

对，你没看错，还是预览版……（2018年3月1日）

## 为什么要使用其替换 Google Device Manager
在 Android SDK Tools - 26.0.1（也就是 Android 8.0） ， Google 新的 CLI 工具移除了对老的基于界面的 AVD 和 SDK 管理器的支持。Xamarin 因此也将 Android SDK Manager 替换成了 Xamarin SDK Manager ，但 AVD 并没有替换，因为 Xamarin Android Device Manager 现在仍然是预览版。若要使用 Android 8.0 及以上虚拟机，则必须使用 Xamarin Android Device Manager 。

![](http://blogres.zhangyue.xin/18-3-1/37510487.jpg)

## 使用的先决条件
* Visual Studio 2017 版本 15.5 及以上
* Xamarin for Visual Studio 版本 4.8 及以上
* Android SDK Tools 版本 26.0.2 及以上，Android SDK Platform-Tools 版本 26.0.0 及以上，Android SDK Build-Tools 版本 26.0.0 及以上

## 步骤
### 1. 下载安装
[Xamarin Device Manager installer for Windows](https://go.microsoft.com/fwlink/?linkid=865528)

### 2. 以管理员身份运行
当安装完成后 Visual Studio 中的 AVD 管理器将会被替换成 Xamarin Android Device Manager ，可以直接在 Visual Studio 的菜单中打开，或者在 Windows 菜单中找到管理器，以管理员身份运行。

![](http://blogres.zhangyue.xin/18-3-1/31996050.jpg)

![](http://blogres.zhangyue.xin/18-3-1/84682478.jpg)

当然，如果要是安装完成直接打开的话可能会出现如下错误。

![](http://blogres.zhangyue.xin/18-3-1/56536694.jpg)

这是因为先决条件的第三点，相关的 Tools 未安装或者版本过低，需要进入 SDK 管理器下载相关 Tools。

![](http://blogres.zhangyue.xin/18-3-1/87067453.jpg)

### 3. 创建虚拟机
进入主界面后点击 New 进入新建页面。

![](http://blogres.zhangyue.xin/18-3-1/3871958.jpg)

选择相应的 Android 版本和设备，配置相应的属性即可。可以在 SDK 管理器中下载镜像，也可以在创建虚拟机时下载镜像。

![](http://blogres.zhangyue.xin/18-3-1/3727091.jpg)

等待创建完毕启动即可。

![](http://blogres.zhangyue.xin/18-3-1/74513233.jpg)