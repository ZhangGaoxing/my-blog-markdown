---2017-01-13 19:08---

 **2017年12月30日更新：在 Visual Studio 2017 中，默认安装 Xamarin 即可，不需 Android Studio。**
<br>
<br>

最近在自学 Xamarin 和 Android ，同时发现国内在做 Xamarin 的不多。我在自学中间遇到了很多问题，而且百度到的很多教程也有些过时，现在打算写点东西稍微总结下，顺便帮后人指指路了。由于手头没啥中文资料，我也是自己摸索出来的，而且我对 Android 也只是处于最开始的了解阶段（学习笔记嘛，别学边写嘛╮(╯▽╰)╭），难免会出现错误，有问题大家共同讨论（毕竟 .Net 就要靠我们腾达了→_→滑稽）。

**以 Visual Studio 2015 Community 为例**。

## 1. 安装 Xamarin

在 Visual Studio 的安装选项里，有“跨平台移动开发”这个选项，展开后选择“C#/.NET (Xamarin v4.2.1)”，选择完成后安装即可。（默认安装即可，不必科学上网，中途出现错误忽略即可，只要 VS 里能创建 Xamarin.Android 项目就行）

![](http://blogres.zhangyue.xin/18-2-13/86654975.jpg)

## 2. 安装 Android Studio

由于谷歌最近在中国开通了开发者网站 https://developers.google.cn ，下载一些开发工具就没必要科学上网了，这也是安装 Xamarin 不用科学上网的原因。在 https://developer.android.google.cn/studio/index.html 下载Android Studio，完成后一路下一步即可。要注意的是，请记住 Android SDK 的存放路径，在配置 Xamarin 环境的时候要用。

![](http://blogres.zhangyue.xin/18-2-13/70578480.jpg)

## 3. 下载 JDK

由于 Android 7.0 需要 JDK8 来开发，所以还需要下载个 JDK ，版本随便，windows x86 就行，网址 → http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html

## 4. 配置 Xamarin

在 Xamarin ，Android Studio ，JDK8 安装完成后打开 Visual Studio，选择**“工具”——“选项”**，之后便会打开“选项”窗口。左侧菜单列表中找到**“Xamarin”——“Android Settings”**，将 JDK 和 SDK 路径变更为前两个步骤的安装路径。

![](http://blogres.zhangyue.xin/18-2-13/62374651.jpg)

完成更改后选择**“工具”——“Android”——“Android SDK Manager”**，下载需要的 API 即可完成配置。（谷歌应该在国内有个源，Xamarin 的 SDK 管理器下载时要科学上网，而 Android Studio 的 SDK 管理器是可以满速的）

![](http://blogres.zhangyue.xin/18-2-13/5111697.jpg)
