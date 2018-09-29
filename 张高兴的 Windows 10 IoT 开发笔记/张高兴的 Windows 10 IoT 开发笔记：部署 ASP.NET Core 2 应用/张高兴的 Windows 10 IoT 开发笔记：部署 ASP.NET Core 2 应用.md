---
2018/2/17 22:00:00
---

今天是大年初二，都去走亲戚了吧，享受一下这难得的能和亲友相聚的时光。而我就不一样了，今天一回到家就又开始瞎折腾了，哈哈哈。

## 问题背景
最近花了点时间用 ASP.NET Core 2 写了个个人博客，中间出了好多问题，过程弯弯曲曲的，但好歹最后还是完成部署在阿里云上了。这几天闲的没事看 .NET Core CLI，发现运行时标识符（Runtime IDentifier）居然有 win10-arm，这使我突然萌生了想把我的博客部署在 Raspberry Pi 上。（这就是纯属瞎折腾，部署在 Windows IoT 上确实没用，反正玩都玩了，干脆写篇博客吧...）

![1](http://blogres.zhangyue.xin/18-2-17/38402597.jpg)

## 发布（Publish）应用
在将应用部署在 Windows IoT 上之前，首先是要以某种合适的方法将应用发布到本机。（Windows 10 IoT 只包含运行时）

### 1. 更改项目输出类型
打开项目“属性”，将“应用程序”选项中的“输出类型”，改为“控制台应用程序”。

![2](http://blogres.zhangyue.xin/18-2-17/35525388.jpg)

或者你也可以直接编辑 .csproj 文件，将 <OutputType> 的值改为 Exe。

![3](http://blogres.zhangyue.xin/18-2-17/12715262.jpg)

### 2. 编辑 Program.cs
和在 Linux 上部署一样， 在 BuildWebHost 里加上这么一句话 .UseUrls("http://*:5000")。* 作为主机名，5000 为监听端口。

![4](http://blogres.zhangyue.xin/18-2-17/56674139.jpg)

### 3. 在控制台发布
在“工具”的“Nuget 包管理器”中，打开“程序包管理器控制台”。运行以下命令：
```
dotnet publish -c release -r win10-arm
```

因为是要部署在 Raspberry Pi 上， RID 用的 win10-arm。发布的路径是在 “你的项目\bin\Release\netcoreapp2.0\win10-arm\publish”。

## 部署应用
部署要遵顼以下步骤

### 1. 将发布文件复制到 Raspberry Pi
怎么去复制文件随便，这里我用的是 WinSCP ，因为我自己管理 Linux 的时候就用的这个，习惯了。但在复制之前，要先启用 Windows IoT 的 FTP 管理。需要在 PowerShell 或者 Device Portal 运行命令：
```
start C:\Windows\System32\ftpd.exe
```

接下来就是运行你的 FTP 管理工具，新建一个文件夹，然后把文件复制进去即可。

![5](http://blogres.zhangyue.xin/18-2-17/17706514.jpg)

### 2. 配置防火墙
使用 netsh 工具配置防火墙，运行命令：（一开始被这个问题困扰了半天，怎么都访问不到网站，做一个允悲的表情...）
```
netsh advfirewall firewall add rule name=”ASP.NET Core Web Server port” dir=in action=allow protocol=TCP localport=5000
```

### 3. 运行
切换到相应的目录，运行 .exe 即可。 

![6](http://blogres.zhangyue.xin/18-2-17/21874117.jpg)

## 问题
嗯，我的博客没有在 Raspberry Pi 上跑起来（但上面的东西都是对的），来看看异常 Unable to load DLL 'sni.dll'

![7](http://blogres.zhangyue.xin/18-2-17/76976226.jpg)

这个问题通常引用一下 Nuget 包 System.Data.SqlClient 就好了，但在 Raspberry Pi 上没好... 我开始了在 GitHub 上翻 issue 的旅程，揪心的事情还是发生了，人家压根就没支持 arm32 ... 也就是说，不算定制镜像的话，只有 Raspberry Pi 是不支持的，Dragonboard 410c 是 arm64， MinnowBoard 是 x64 （允悲）... 让我们期待 Raspberry Pi 4 吧！

![8](http://blogres.zhangyue.xin/18-2-17/44481006.jpg)

我又新建了一个默认的项目，部署在了 Raspberry Pi 上，这下没问题了... 

![9](http://blogres.zhangyue.xin/18-2-17/70473300.jpg)

但这一切并不能阻止我把博客部署在 Windows IoT 上，上虚拟机，终于成功了（摊手）...

![10](http://blogres.zhangyue.xin/18-2-17/57954747.jpg)
