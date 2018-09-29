---2016-12-30 19:36---

ListView 默认的排列方向是纵向 ( Orientation="Vertical" ) ，但如果我们需要横向显示的 ListView 怎么办？

Blend for Visual Studio 现在就派上用场了。不只是 ListView ，其他的控件也可以用 Blend 定制你自己的 UI 样式。

下面新建一个项目 "HorizontalListViewDemo" ，用于演示横向 ListView ，解决方案结构如下：( GitHub: https://github.com/ZhangGaoxing/uwp-demo/tree/master/HorizontalListViewDemo )

![](http://blogres.zhangyue.xin/18-2-13/27779762.jpg)

由于只是一个演示项目，ListView 的绑定数据素材引自 Bob Tabor 的 UWP 入门开发视频 https://mva.microsoft.com/zh-cn/training-courses/-windows-10--14541 。

## 项目分析

### 1. MainPage 的结构

MainPage 由两部分组成，一个标题，一个 ListView 。

```xml
<Grid Background="Black">
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto" />
        <RowDefinition Height="*" />
    </Grid.RowDefinitions>
    <!--标题-->
    <StackPanel Margin="15,15,15,0">
        <TextBlock Text="Horizontal ListView Demo" 
                   FontSize="30" FontWeight="Bold"
                   Foreground="White" />
        <TextBlock Text="横向 ListView" Foreground="White" />
    </StackPanel>

    <ListView Name="MyListView" Grid.Row="1"  />
</Grid>
```
 
### 2. 用 Blend 定制样式

首先右击项目，点击“在 Blend 中设计”。

![](http://blogres.zhangyue.xin/18-2-13/79073314.jpg)

在“对象和时间线”中找到 "MyListView" ，右击。

![](http://blogres.zhangyue.xin/18-2-13/67381186.jpg)

在“编辑其他模板”中有 ItemTemplate，ItemContainerStyle，ItemsPanel 三个选项。ItemTemplate 用于数据绑定，数据绑定的模板一般是手写完成，用 Blend 也是可以创建数据绑定模板的。ItemContainerStyle 是容器的样式，说白了就是 ListView 中的 Item 的显示样式，像 Width，Background 等都可以在其中定制。ItemsPanel 是横向 ListView 的关键，ListView 的显示方向就在其中。下面是横向 ListView 的 ItemsPanel xaml代码。

```xml
<!--横向布局-->
<ItemsPanelTemplate x:Key="HorizontalItemsPanelTemplate">
    <VirtualizingStackPanel Orientation="Horizontal"
        VerticalAlignment="Top"
        ScrollViewer.HorizontalScrollMode="Enabled"
        ScrollViewer.VerticalScrollMode="Disabled"/>
</ItemsPanelTemplate>
```
 
### 3. 所有代码

#### MainPage.xaml

```xml
<Page
    x:Class="HorizontalListViewDemo.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:HorizontalListViewDemo"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:data="using:HorizontalListViewDemo.Model"
    mc:Ignorable="d">

    <Page.Resources>
        <!--数据绑定模板-->
        <DataTemplate x:Key="DataTemplate" x:DataType="data:Book">
            <StackPanel Margin="8">
                <Image Source="{x:Bind CoverImage}" HorizontalAlignment="Center" Margin="0,0,0,5" Width="90" />
                <TextBlock Text="{x:Bind Title}" Foreground="White" HorizontalAlignment="Center" FontSize="16" />
                <TextBlock Text="{x:Bind Author}" Foreground="White" HorizontalAlignment="Center" FontSize="10" />
            </StackPanel>
        </DataTemplate>
        <!--容器模板-->
        <Style x:Key="HorizontalItemContainerStyle" TargetType="ListViewItem">
            <Setter Property="MinWidth" Value="{StaticResource SplitViewCompactPaneThemeLength}"/>
            <Setter Property="Padding" Value="0"/>
            <Setter Property="UseSystemFocusVisuals" Value="True" />
            <Setter Property="Template">
                <Setter.Value>
                    <ControlTemplate TargetType="ListViewItem">
                        <ListViewItemPresenter ContentTransitions="{TemplateBinding ContentTransitions}"
                        Control.IsTemplateFocusTarget="True"
                        SelectionCheckMarkVisualEnabled="False"
                        PointerOverBackground="{ThemeResource SystemControlHighlightListLowBrush}"
                        PointerOverForeground="{ThemeResource ListViewItemForegroundPointerOver}"
                        SelectedBackground="Transparent"
                        SelectedForeground="{ThemeResource SystemControlForegroundAccentBrush}"
                        SelectedPointerOverBackground="{ThemeResource SystemControlHighlightListLowBrush}"
                        PressedBackground="{ThemeResource SystemControlHighlightListMediumBrush}"
                        SelectedPressedBackground="{ThemeResource SystemControlHighlightListMediumBrush}"
                        DisabledOpacity="{ThemeResource ListViewItemDisabledThemeOpacity}"
                        HorizontalContentAlignment="Stretch"
                        VerticalContentAlignment="{TemplateBinding VerticalContentAlignment}"
                        ContentMargin="{TemplateBinding Padding}"/>
                    </ControlTemplate>
                </Setter.Value>
            </Setter>
        </Style>
        <!--横向布局-->
        <ItemsPanelTemplate x:Key="HorizontalItemsPanelTemplate">
            <VirtualizingStackPanel Orientation="Horizontal"
                VerticalAlignment="Top"
                ScrollViewer.HorizontalScrollMode="Enabled"
                ScrollViewer.VerticalScrollMode="Disabled"/>
        </ItemsPanelTemplate>

    </Page.Resources>
    
    
    <Grid Background="Black">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto" />
            <RowDefinition Height="*" />
        </Grid.RowDefinitions>
        <!--标题-->
        <StackPanel Margin="15,15,15,0">
            <TextBlock Text="Horizontal ListView Demo" 
                       FontSize="30" FontWeight="Bold"
                       Foreground="White" />
            <TextBlock Text="横向 ListView" Foreground="White" />
        </StackPanel>

        <ListView Name="MyListView" Grid.Row="1" 
                  SelectionMode="None" IsItemClickEnabled="True"
                  HorizontalAlignment="Center"
                  Margin="20" BorderThickness="1" BorderBrush="White"
                  ScrollViewer.HorizontalScrollMode="Enabled" ScrollViewer.HorizontalScrollBarVisibility="Auto"
                  ScrollViewer.VerticalScrollMode="Disabled"
                  ItemsSource="{x:Bind Books}"
                  ItemTemplate="{StaticResource DataTemplate}"
                  ItemContainerStyle="{StaticResource HorizontalItemContainerStyle}"
                  ItemsPanel="{StaticResource HorizontalItemsPanelTemplate}" />

    </Grid>
</Page>
```

#### MainPage.xaml.cs

```c#
using HorizontalListViewDemo.Model;
using System.Collections.Generic;
using Windows.UI.Xaml.Controls;

namespace HorizontalListViewDemo
{
    public sealed partial class MainPage : Page
    {
        private List<Book> Books;

        public MainPage()
        {
            this.InitializeComponent();
            Books = BookManager.GetBooks();
        }
    }
}
```

#### Book.cs

```c#
using System.Collections.Generic;

namespace HorizontalListViewDemo.Model
{
    public class Book
    {
        public int BookId { get; set; }
        public string Title { get; set; }
        public string Author { get; set; }
        public string CoverImage { get; set; }
    }

    public class BookManager
    {
        public static List<Book> GetBooks()
        {
            var books = new List<Book>
            {
                new Book { BookId = 1, Title = "Vulpate", Author = "Futurum", CoverImage = "Assets/1.png" },
                new Book { BookId = 2, Title = "Mazim", Author = "Sequiter Que", CoverImage = "Assets/2.png" },
                new Book { BookId = 3, Title = "Elit", Author = "Tempor", CoverImage = "Assets/3.png" },
                new Book { BookId = 4, Title = "Etiam", Author = "Option", CoverImage = "Assets/4.png" },
                new Book { BookId = 5, Title = "Feugait Eros Libex", Author = "Accumsan", CoverImage = "Assets/5.png" },
                new Book { BookId = 6, Title = "Nonummy Erat", Author = "Legunt Xaepius", CoverImage = "Assets/6.png" },
                new Book { BookId = 7, Title = "Nostrud", Author = "Eleifend", CoverImage = "Assets/7.png" },
                new Book { BookId = 8, Title = "Per Modo", Author = "Vero Tation", CoverImage = "Assets/8.png" },
                new Book { BookId = 9, Title = "Suscipit Ad", Author = "Jack Tibbles", CoverImage = "Assets/9.png" },
                new Book { BookId = 10, Title = "Decima", Author = "Tuffy Tibbles", CoverImage = "Assets/10.png" },
                new Book { BookId = 11, Title = "Erat", Author = "Volupat", CoverImage = "Assets/11.png" },
                new Book { BookId = 12, Title = "Consequat", Author = "Est Possim", CoverImage = "Assets/12.png" },
                new Book { BookId = 13, Title = "Aliquip", Author = "Magna", CoverImage = "Assets/13.png" }
            };

            return books;
        }
    }
}
```

<br>
效果图

![](http://blogres.zhangyue.xin/18-2-13/87023369.jpg)
