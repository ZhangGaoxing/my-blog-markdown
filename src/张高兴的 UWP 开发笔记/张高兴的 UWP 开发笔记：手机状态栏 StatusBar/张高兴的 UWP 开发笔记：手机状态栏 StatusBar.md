---2017-01-17 22:01---

UWP 有关应用标题栏 TitleBar 的文章比较多，但介绍 StatusBar 的却没几篇，在这里随便写写。状态栏 StatusBar 用法比较简单，花点心思稍微设计一下，对应用会是个很好的点缀。

说明一下，当应用运行在 PC 上时我们叫 TitleBar ，运行在 Mobile 上时我们叫 StatusBar ，这是两个不同的玩意儿。

在使用 StatusBar 之前，你需要在项目的引用里添加 **Windows Mobile Extensions for the UWP** ，并且引用 **Windows.UI.ViewManagement** 命名空间。

![](http://blogres.zhangyue.xin/18-2-13/13689361.jpg)

StatusBar 类中一共有**三个方法**。分别为一个静态方法 GetForCurrentView() ，用于取得当前 StatusBar 实例。两个异步方法 HideAsync() 和 ShowAsync() ，分别用来显示与隐藏 StatusBar 。

**五个属性**。两个可空的 Color 类型 BackgroundColor 与 ForegroundColor ，分别用来设置背景色与前景色。 double 类型的 BackgroundOpacity ，取值范围为 0-1 ，用来设置 StatusBar 透明度。两个只读属性，返回 Rect 矩形的 OccludedRect 和 StatusBarProgressIndicator 类型的 ProgressIndicator ，ProgressIndicator 属性不太了解。

**两个事件**。Hiding 和 Showing 。
 
下面给出一个简单的示例（GitHub : https://github.com/ZhangGaoxing/uwp-demo/tree/master/StatusBarDemo）

![](http://blogres.zhangyue.xin/18-2-13/95584596.jpg)

MainPage.xaml

```xml
<Page
    x:Class="StatusBarDemo.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:StatusBarDemo"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>

        <TextBlock Text="Status Bar Demo" FontSize="35" Margin="15" />

        <StackPanel Grid.Row="1" Margin="15">
            <TextBlock Text="Show or hide status bar" />
            <RadioButton Name="Show" GroupName="ShowOrHide" IsChecked="True" Checked="RadioButton_Checked">Show</RadioButton>
            <RadioButton Name="Hide" GroupName="ShowOrHide" Checked="RadioButton_Checked">Hide</RadioButton>
        </StackPanel>

        <StackPanel Grid.Row="2" Margin="15">
            <TextBlock Text="Change status bar background color" />
            <RadioButton Name="Black" GroupName="Color" IsChecked="True" Checked="Color_Checked">Black</RadioButton>
            <RadioButton Name="White" GroupName="Color" Checked="Color_Checked">White</RadioButton>
            <RadioButton Name="Accent" GroupName="Color" Checked="Color_Checked">System Accent Color</RadioButton>
        </StackPanel>

        <StackPanel Grid.Row="3" Margin="15">
            <TextBlock Text="Change status bar background opacity" />
            <Slider Name="Opacity" Minimum="0" Maximum="10" Value="10" ValueChanged="Opacity_ValueChanged" />
        </StackPanel>

    </Grid>
</Page>
```

MainPage.xaml.cs

```c#
using System;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.ViewManagement;
using Windows.Foundation.Metadata;
using Windows.UI;
using Windows.UI.Xaml.Media;


namespace StatusBarDemo
{
    public sealed partial class MainPage : Page
    {
        StatusBar statusBar;
        // 获取系统当前颜色
        SolidColorBrush accentColor = (SolidColorBrush)Application.Current.Resources["SystemControlBackgroundAccentBrush"];

        public MainPage()
        {
            // 判断是否存在 StatusBar
            if (ApiInformation.IsTypePresent("Windows.UI.ViewManagement.StatusBar"))
            {
                statusBar = StatusBar.GetForCurrentView();
            }
            else
            {
                Application.Current.Exit();
            }

            this.InitializeComponent();
        }

        // 显示，隐藏
        private async void RadioButton_Checked(object sender, RoutedEventArgs e)
        {
            RadioButton r = sender as RadioButton;

            if (r.Name == "Show")
            {
                await statusBar.ShowAsync(); 
            }
            else
            {
                await statusBar.HideAsync();
            }
        }

        // 颜色
        private void Color_Checked(object sender, RoutedEventArgs e)
        {
            RadioButton r = sender as RadioButton;

            if (r.Name == "Black")
            {
                statusBar.BackgroundColor = Colors.Black;
                statusBar.ForegroundColor = Colors.White;  
            }
            else if(r.Name == "White")
            {
                statusBar.BackgroundColor = Colors.White;
                statusBar.ForegroundColor = Colors.Black;
                statusBar.BackgroundOpacity = 1;
            }
            else
            {
                statusBar.BackgroundColor = accentColor.Color;
                statusBar.ForegroundColor = Colors.Black;
                statusBar.BackgroundOpacity = 1;
            }
        }

        // 透明度
        private void Opacity_ValueChanged(object sender, Windows.UI.Xaml.Controls.Primitives.RangeBaseValueChangedEventArgs e)
        {
            statusBar.BackgroundOpacity = Opacity.Value / 10;
        }
    }
}
```