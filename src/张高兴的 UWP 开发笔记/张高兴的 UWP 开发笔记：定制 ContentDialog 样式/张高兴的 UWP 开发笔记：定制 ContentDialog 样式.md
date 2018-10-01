---2017-03-25 17:42---

我需要一个背景透明的 ContentDialog，像下图一样。如何定制？写了一个简单的示例（https://github.com/ZhangGaoxing/uwp-demo/tree/master/ContentDialogDemo）

![](http://blogres.zhangyue.xin/18-2-13/30463496.jpg)

首先在项目里新建一个资源字典，并在 App.xaml 添加以下代码将此资源字典合并

![](http://blogres.zhangyue.xin/18-2-13/1075978.jpg)

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <ResourceDictionary Source="Style.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

这时新添加的资源字典还是空的，我们需要找到 ContentDialog 的默认样式。这些默认样式在已安装的 Windows 10 SDK 中被提供，比如 SDK 默认安装在 C 盘的时候，控件样式字典 **generic.xaml** 可以在 **C:\Program Files (x86)\Windows Kits\10\DesignTime\CommonConfiguration\Neutral\UAP\10.0.14393.0\Generic** 这里找到。找到后用 Visual Studio 打开，如下图。

![](http://blogres.zhangyue.xin/18-2-13/44832789.jpg)

接下来按 Ctrl+F 搜索 ContentDialog 找到默认样式复制到刚才新建的资源字典中，然后根据需要定制样式即可。

![](http://blogres.zhangyue.xin/18-2-13/82383036.jpg)

像我需要的透明 ContentDialog 只需要更改 Property 为 Background 的 Value 值为 Transparent 即可。**注意不要忘记给一个 x:Key 值**，也就是起个名称，这里为 x:Key="TransparentDialog" 。


样式定制完成，并且资源字典也合并完成，下面就是要在代码中去调用了。资源字典的调用也是靠键值对，输入对应的键来返回对应的值。

在项目合适的位置新建一个 Style 类型的字段，用来获取样式。

```c#
Style transparent = (Style)Application.Current.Resources["TransparentDialog"];
```

样式获取完成后设置 ContentDialog 的 Style 属性即可

```c#
var contentDialog = new ContentDialog()
{
    Content = new Content("This is a transparent ContentDialog."),
    PrimaryButtonText = "确定",
    FullSizeDesired = false
};

contentDialog.Style = transparent;

contentDialog.PrimaryButtonClick += (_s, _e) =>
{
    contentDialog.Hide();
};
await contentDialog.ShowAsync();
```

这样，一个定制样式的 ContentDialog 就完成了。