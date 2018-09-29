---
2017/12/31 22:31
---

## Shortcut 简介

Shortcut 是 Android 7.1 (API Level 25) 的新特性，类似于苹果的 3D Touch ，但并不是压力感应，只是一种长按菜单。Shortcut 是受启动器限制的，也就是说国内大厂的定制系统大多数是不支持的，那些所谓的可以 pin 在桌面上的应用功能的快捷启动图标本质上就是 Shortcut 。

![](http://blogres.zhangyue.xin/18-2-13/74358344.jpg)

## Shortcut 在 Xamarin.Forms 中的实现分析

*本文讨论的是动态 Shortcut 实现。*

实现方式无非两种思路，一种 **Native to Forms** ，另一种 **Forms to Native** 。博主最开始考虑的是 Forms to Native ，但没成功。在设置 ShortcutInfo 时需要一个 Intent ，其中一个构造函数为
```c#
public Intent(Context packageContext, Type type);
```
看着很容易，只要传入一个 Content 以及 把对应的页面 typeof 一下即可，但会抛出异常。原因是传入的 Forms Page 类并不是 Java 的原生类型。查阅 Xamarin.Android 的相关文档发现，这个 Type 是必须继承 **Activity** 类的。那么，所有的 Forms 页面均不可传入，Forms to Native 这条路也就不能走了。

Native to Forms 呢？

既然是需要依赖 Activity 的，那就通过新建一个 Android Activity 去调用 Forms 页面。

## 代码实现

下面新建一个空的 Cross-Platform 项目 ShortcutDemo ，使用 Shared Project 共享代码。（GitHub：https://github.com/ZhangGaoxing/xamarin-forms-demo/tree/master/ShortcutDemo）

### 修改 Shared Project

添加两个 ContentPage 用作测试。

![](http://blogres.zhangyue.xin/18-2-13/42911774.jpg)

### 修改 Xamarin.Android

添加两个活动，ShortcutContainerActivity.cs 与 FormsActivity.cs 。

#### ShortcutContainerActivity.cs

ShortcutContainerActivity.cs 用来作为展示 Forms 页面的跳板，因此将其继承的 Activity 改成 **global::Xamarin.Forms.Platform.Android.FormsAppCompatActivity** 。同时把 OnCreate 的代码改成如下所示
```c#
protected override void OnCreate(Bundle savedInstanceState)
{
    TabLayoutResource = Resource.Layout.Tabbar;
    ToolbarResource = Resource.Layout.Toolbar;

    base.OnCreate(savedInstanceState);

    global::Xamarin.Forms.Forms.Init(this, savedInstanceState);
   
    Intent intent = Intent;
    // 获取传进来的页面名称
    string pageName = intent.GetStringExtra("PageName");

    var app = new App();
    // 设置显示的页面
    switch (pageName)
    {
        case "Page1":
            app.MainPage = new ShortcutDemo.Views.Page1();
            break;
        case "Page2":
            app.MainPage = new ShortcutDemo.Views.Page2();
            break;
        default:
            break;
    }

    LoadApplication(app);
}

```
要注意的是，顶部的 Activity 特性标签要改动，**除了 MainLauncher 要改为 false 以外，其他的全部要和 MainActivity.cs 里的一样**，不然会抛出异常，可能是主题不统一的原因。
```c#
[Activity(Label = "ShortcutDemo", Icon = "@drawable/icon", Theme = "@style/MainTheme", MainLauncher = false, ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation)]
```

#### FormsActivity.cs

FormsActivity.cs 作为正常启动应用的活动，只是将其从 MainActivity.cs 中剥离开来。代码如下：
```c#
[Activity(Label = "ShortcutDemo", Icon = "@drawable/icon", Theme = "@style/MainTheme", MainLauncher = false, ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation)]
public class FormsActivity : global::Xamarin.Forms.Platform.Android.FormsAppCompatActivity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        TabLayoutResource = Resource.Layout.Tabbar;
        ToolbarResource = Resource.Layout.Toolbar;

        base.OnCreate(savedInstanceState);

        global::Xamarin.Forms.Forms.Init(this, savedInstanceState);
        LoadApplication(new App());
    }
}
```

#### MainActivity.cs

MainActivity.cs 作为应用程序的入口，由于 Forms 的初始化以及加载已被剥离至 FormsActivity.cs 中，可将 MainActivity.cs 的继承改为 Activity 类。

1. 在其中添加一个 SetShortcut() 方法用于设置 Shortcut 。首先添加一个 List 用于存放 ShortcutInfo，以备最后动态设置 Shortcut 作为参数传入。
```c#
List<ShortcutInfo> shortcutInfoList = new List<ShortcutInfo>();
```
2. 接下来实例化一个 Intent 。其中 SetClass 将跳板活动 ShortcutContainerActivity 传入；**SetAction 是必须设置的**，要不然报错都不知道怎么回事；PutExtra 用于向下一个活动传递参数，我们这里传入的名称用于在跳板活动里设置 MainPage 。
```c#
Intent page1 = new Intent();
page1.SetClass(this, typeof(ShortcutContainerActivity));
page1.SetAction(Intent.ActionMain);
page1.PutExtra("PageName", "Page1");
```

3. 下面实例化 ShortcutInfo 。SetRank 为设置排序序号，最多显示5个 Shortcut ，也就是 0-4 ；SetIcon 为设置图标；SetShortLabel 与 SetLongLabel 则是设置长名称与段名称；SetIntent 则把上一步实例化的 Intent 传入；最后将其加入 List 。
```c#
ShortcutInfo page1Info = new ShortcutInfo.Builder(this, "Page1")
    .SetRank(0)
    .SetIcon(Icon.CreateWithResource(this, Resource.Drawable.Page1))
    .SetShortLabel("Page1")
    .SetLongLabel("Page1")
    .SetIntent(page1)
    .Build();
shortcutInfoList.Add(page1Info);
```

4. 最后获取 ShortcutManager 进行动态设置 Shortcut
```c#
ShortcutManager shortcutManager = (ShortcutManager)GetSystemService(Context.ShortcutService);
shortcutManager.SetDynamicShortcuts(shortcutInfoList);
```

因此全部的 MainActivity.cs 的代码如下：
```c#
[Activity(Label = "ShortcutDemo", Icon = "@drawable/icon", Theme = "@style/MainTheme", MainLauncher = true, ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation)]
public class MainActivity : Activity
{
    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);

        SetShortcut();

        StartActivity(typeof(FormsActivity));
    }

    private void SetShortcut()
    {
        List<ShortcutInfo> shortcutInfoList = new List<ShortcutInfo>();

        Intent page1 = new Intent();
        page1.SetClass(this, typeof(ShortcutContainerActivity));
        page1.SetAction(Intent.ActionMain);
        page1.PutExtra("PageName", "Page1");
        ShortcutInfo page1Info = new ShortcutInfo.Builder(this, "Page1")
            .SetRank(0)
            .SetIcon(Icon.CreateWithResource(this, Resource.Drawable.Page1))
            .SetShortLabel("Page1")
            .SetLongLabel("Page1")
            .SetIntent(page1)
            .Build();
        shortcutInfoList.Add(page1Info);

        Intent page2 = new Intent();
        page2.SetClass(this, typeof(ShortcutContainerActivity));
        page2.SetAction(Intent.ActionMain);
        page2.PutExtra("PageName", "Page2");
        ShortcutInfo page2 = new ShortcutInfo.Builder(this, "Page2")
            .SetRank(1)
            .SetIcon(Icon.CreateWithResource(this, Resource.Drawable.Page2))
            .SetShortLabel("Page2")
            .SetLongLabel("Page2")
            .SetIntent(page2)
            .Build();
        shortcutInfoList.Add(page2);

        ShortcutManager shortcutManager = (ShortcutManager)GetSystemService(Context.ShortcutService);
        shortcutManager.SetDynamicShortcuts(shortcutInfoList);
    }
}
```

## 效果图

![](http://blogres.zhangyue.xin/18-2-13/69532143.jpg)