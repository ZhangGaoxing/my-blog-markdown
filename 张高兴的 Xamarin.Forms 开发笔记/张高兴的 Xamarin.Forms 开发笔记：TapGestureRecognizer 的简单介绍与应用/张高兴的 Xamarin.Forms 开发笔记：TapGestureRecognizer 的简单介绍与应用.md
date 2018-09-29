---2017-11-24 23:48---

最近很少写应用了，一直在忙关于 ASP.NET 的东西（哈欠...）。抽点时间对 TapGestureRecognizer 做点总结。

## 一、简介

TapGestureRecognizer 就是对 Tap 手势进行识别。 Forms 里的大多数控件都继承自 View 类，而 View 类中有一个公共属性 GestureRecognizers，因此控件都可以添加各种手势识别。当然手势不止 Tap 这一种，更多的可以在 Xamarin 的指南中了解：https://developer.xamarin.com/guides/xamarin-forms/application-fundamentals/gestures/

```c#
public IList<IGestureRecognizer> GestureRecognizers { get; }
```

## 二、用法

在说用法之前，首先要搞清楚我们需要关注 Tap 的哪些属性。最重要的还是 Tap 的次数。

### 1. Xaml

```xml
<Label Text="0">
    <Label.GestureRecognizers>
        <TapGestureRecognizer NumberOfTapsRequired="1" Tapped="TapGestureRecognizer_Tapped" />
    </Label.GestureRecognizers>
</Label>
```

直接上代码，这里以 Label 举例，最最基本的用法都在这了，用 NumberOfTapsRequired 设置点击数， Tapped 绑定监听事件。对于 MVVM 涉及的绑定，可以去官方找找。

### 2. C# 代码

```c#
var tapGestureRecognizer = new TapGestureRecognizer();

tapGestureRecognizer.NumberOfTapsRequired = 1;
tapGestureRecognizer.Tapped += (s, e) =>
{
    // TODO
};

YourControl.GestureRecognizers.Add(tapGestureRecognizer);
```
　　
## 三、应用

写了一个小小的 Demo （GitHub：https://github.com/ZhangGaoxing/xamarin-forms-demo/tree/master/GestureRecognizersDemo）

### 1. 超链接

Forms 里是没有超链接的，有时候就很头疼。我顺便看了一下 Xaml Standard 的第一版草稿，里面还是没有超链接。可以用 TapGestureRecognizer 去仿制一个。这里就以 Label 跳转邮件链接和 Image 跳转网页链接举例。

Xaml 代码

```xml
<Label Text="Hyperlinks" FontSize="20" TextColor="Black" />
<StackLayout Orientation="Horizontal">
    <Label Text="Text : " VerticalTextAlignment="Center" />
    <Label x:Name="Email" Text="zhangyuexin121@live.cn" VerticalTextAlignment="Center" TextColor="DodgerBlue"/>
</StackLayout>
<StackLayout Orientation="Horizontal">
    <Label Text="Image : " VerticalTextAlignment="Center" />
    <Image x:Name="Weibo" Source="weibo.png" HeightRequest="35" WidthRequest="35" VerticalOptions="Center" />
</StackLayout>
```

C# 代码

```c#
var tapGestureRecognizer = new TapGestureRecognizer();
// 设置触发点击数
tapGestureRecognizer.NumberOfTapsRequired = 1;
tapGestureRecognizer.Tapped += (s, e) =>
{
    if (s is Label)
    {
        Device.OpenUri(new Uri("mailto:zhangyuexin121@live.cn"));
    }
    else if (s is Image)
    {
        Device.OpenUri(new Uri("http://www.weibo.com/279639933"));
    }
};

Email.GestureRecognizers.Add(tapGestureRecognizer);
Weibo.GestureRecognizers.Add(tapGestureRecognizer);
```

### 2. 为没有 Clicked 事件的控件添加假的 Clicked 事件

换句大白话来说，就是点击一个控件触发一个事件。这里以 Label 举例，点击 Label 以“0”和“1”变化。

Xaml 代码

```xml
<Label Text="Fake Click Event" FontSize="20" TextColor="Black" />
<Label Text="0" FontSize="18" FontAttributes="Bold" TextColor="Black">
    <Label.GestureRecognizers>
        <TapGestureRecognizer NumberOfTapsRequired="1" Tapped="TapGestureRecognizer_Tapped" />
    </Label.GestureRecognizers>
</Label>
```

C# 代码

```c#
private void TapGestureRecognizer_Tapped(object sender, EventArgs e)
{
    Label label = sender as Label;

    string val = label.Text;
    switch (val)
    {
        case "0":
            label.Text = "1";
            label.TextColor = Color.FromHex("#2196F3");
            break;
        case "1":
            label.Text = "0";
            label.TextColor = Color.Black;
            break;
        default:
            break;
    }
}
```

### 3. 效果图
![](http://blogres.zhangyue.xin/18-2-13/49075328.jpg)