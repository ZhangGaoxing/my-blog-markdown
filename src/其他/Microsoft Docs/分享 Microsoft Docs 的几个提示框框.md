自己仿的样式，前端就那样，没仿到灵魂，没考虑兼容，写 markdown 的博客时能用用。

字符图标：Font Awesome

## 效果

<link href="https://cdn.bootcss.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet">

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e0f2ff;color: #002b4d;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-info-circle" aria-hidden="true"></i>&nbsp;&nbsp;重要</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e2daf1;color: #38225d;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-info-circle" aria-hidden="true"></i>&nbsp;&nbsp;备注</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-lightbulb-o" aria-hidden="true"></i>&nbsp;&nbsp;提示</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #fff1cc;color: #664b00;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i>&nbsp;&nbsp;注意</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #ffdacc;color: #651b01;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i>&nbsp;&nbsp;警告</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

## 代码
### 不带 css 类的

```html
<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e0f2ff;color: #002b4d;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-info-circle" aria-hidden="true"></i>&nbsp;&nbsp;重要</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #e2daf1;color: #38225d;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-info-circle" aria-hidden="true"></i>&nbsp;&nbsp;备注</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #d2f9d2;color: #094409;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-lightbulb-o" aria-hidden="true"></i>&nbsp;&nbsp;提示</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #fff1cc;color: #664b00;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i>&nbsp;&nbsp;注意</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div style="display: block;position: relative;border-radius: 8px;padding: 1rem;background-color: #ffdacc;color: #651b01;margin: 10px">
    <p style="margin-top:0;font-weight: bold"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i>&nbsp;&nbsp;警告</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>
```

### 带 css 类的
```css
.alert-title {
    margin-top: 0;
    font-weight: bold
}

.alert-info {
    display: block;
    position: relative;
    border-radius: 8px;
    padding: 1rem;
    background-color: #e2daf1;
    color: #38225d;
    margin: 10px
}

.alert-success {
    display: block;
    position: relative;
    border-radius: 8px;
    padding: 1rem;
    background-color: #d2f9d2;
    color: #094409;
    margin: 10px
}

.alert-primary {
    display: block;
    position: relative;
    border-radius: 8px;
    padding: 1rem;
    background-color: #e0f2ff;
    color: #002b4d;
    margin: 10px
}

.alert-warning {
    display: block;
    position: relative;
    border-radius: 8px;
    padding: 1rem;
    background-color: #fff1cc;
    color: #664b00;
    margin: 10px
}

.alert-danger {
    display: block;
    position: relative;
    border-radius: 8px;
    padding: 1rem;
    background-color: #ffdacc;
    color: #651b01;
    margin: 10px
}
```
```html
<div class="alert-primary">
    <p class="alert-title"><i class="fa fa-info-circle" aria-hidden="true"></i>&nbsp;&nbsp;重要</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div class="alert-info">
    <p class="alert-title"><i class="fa fa-info-circle" aria-hidden="true"></i>&nbsp;&nbsp;备注</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div class="alert-success">
    <p class="alert-title"><i class="fa fa-lightbulb-o" aria-hidden="true"></i>&nbsp;&nbsp;提示</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div class="alert-warning">
    <p class="alert-title"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i>&nbsp;&nbsp;注意</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>

<div class="alert-danger">
    <p class="alert-title"><i class="fa fa-exclamation-triangle" aria-hidden="true"></i>&nbsp;&nbsp;警告</p>
    <p><span>不能在 macOS 上开发 Windows 应用。</span></p>
</div>
```
