---2017-01-13 22:17---

完成环境配置后开始第一个简单项目。打开 Visual Studio 新建一个 Xamarin.Android 项目 “HelloAndroid”。（GitHub：https://github.com/ZhangGaoxing/xamarin-android-demo/tree/master/HelloAndroid）

![](http://blogres.zhangyue.xin/18-2-13/59190707.jpg)

解决方案结构如下

![](http://blogres.zhangyue.xin/18-2-13/6202536.jpg)

## 1. 项目结构分析

* **Properties** 存放着应用的一些配置信息。直接双击 “Properties” 可以设置应用的一些属性。AndroidManifest.xml 则是 Android 应用的配置文件，像活动，权限等都要在其中注册，但不需要手动注册，编译时 Xamarin 会自动完成。AssemblyInfo.cs 存放应用的编译信息，像名称，描述，版本等。
* **引用** 与一般的 .Net 项目一样。
* **Components** 暂时不了解怎么用……
* **Assets** 下存放的是原生的资源文件，像文本之类的，不会经过编译，直接打包。目录下有一个简单的帮助文件。
* **Resources** 下存放的都是要经过编译的资源文件。和 Android 项目下的 res 目录是一样的，drawable 下存放的是图片文件，layout 下是应用布局文件，value 下则是字符串。和 Assets 目录一样，也有一个简单的帮助文件。Resource.Designer.cs 则是一些自动生成的代码。
* **MainActivity.cs** 则是默认创建的主活动。

## 2. 代码说明

由于空项目自动创建了一个活动和一个布局，则使用默认的模板。

### Main.axml

双击 Main.axml 打开布局编辑器，你可以和正常的 .Net 项目一样从工具箱中拖拽控件，也可以使用类似Xaml的方式来编写布局。这里我们需要一个 Button 用来点击，和一个 TextView 用来显示 “Hello, Android”。每创建一个控件，相应的 id 会自动添加到 Resource.Id 中（找不到 id 的话请重新生成一下项目）。效果示意图如下

![](http://blogres.zhangyue.xin/18-2-13/14957941.jpg)

界面 xml 代码如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/showHello" />
    <Button
        android:text="Click Me"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/sayHello" />
</LinearLayout>
```

### MainActivity.cs

Android 项目中任何活动都要重写 onCreate() 方法，同样的 Xamarin 也已经自动创建了一个符合 C# 命名规则的 OnCreate() 方法。和 Android 项目一样，活动创建完成后需要加载布局，SetContentView () 方法没变只不过符合了 C# 的命名规则，将 Resource.Layout 下的布局传入即可。这里传入的是 Resource.Layout.Main 。

```c#
public class MainActivity : Activity
    {protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
            // 加载布局
            SetContentView (Resource.Layout.Main);
        }
    }
```

创建事件首先要获取布局中的控件，在 Xamarin 中可以使用泛型方法 FindViewById<T>() 来获取控件，需要传入一个 id 值。

```c#
// 获取布局中的控件
Button say = FindViewById<Button>(Resource.Id.sayHello);
TextView show = FindViewById<TextView>(Resource.Id.showHello);
　　接下来创建 Button 的 Click 方法。和一般的 .Net 项目一样，直接上代码。

// 绑定 Click 事件
say.Click += (sender, e) =>
{

};
```

这个简单的项目实现的是点击计数，并使用 Toast 通知显示，下面给出完整代码

```c#
using Android.App;
using Android.Widget;
using Android.OS;

namespace HelloAndroid
{
    [Activity(Label = "HelloAndroid", MainLauncher = true, Icon = "@drawable/icon")]
    public class MainActivity : Activity
    {
        int count = 0;
        protected override void OnCreate(Bundle bundle)
        {
            base.OnCreate(bundle);
            // 加载布局
            SetContentView (Resource.Layout.Main);
            // 获取布局中的控件
            Button say = FindViewById<Button>(Resource.Id.sayHello);
            TextView show = FindViewById<TextView>(Resource.Id.showHello);
            // 绑定 Click 事件
            say.Click += (sender, e) =>
            {
                count++;
                show.Text = "Hello, Android";
                say.Text = $"You Clicked {count}";
                // Toast 通知
                Toast.MakeText(this, $"You Clicked {count}", ToastLength.Short).Show();
            };
        }
    }
}
```

<br>
效果图（需要注意的是，使用模拟器调试时应用会直接闪退，应该是应用支持文件没传进模拟器吧，我猜的。真机调试时第一次安装了三个应用，一个运行时应用，一个API支持应用，还有一个自己的应用。）
![](http://blogres.zhangyue.xin/18-2-13/12768911.jpg)

