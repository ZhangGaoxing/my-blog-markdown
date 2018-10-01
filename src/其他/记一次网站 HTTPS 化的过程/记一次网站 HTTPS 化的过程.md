--- 2018/4/5 11:09:00 ---

## 为什么要 HTTPS 化
习总书记说过：“没有网络安全，就没有国家安全”。HTTPS 作为一种安全的、加密的传输协议，普及已经成为一种趋势。使用 HTTP 协议传输，用户在互联网上的任意一个节点都有可能被钓鱼、被监听，因为 HTTP 传输的数据为明文，也就是正儿八经的大白话。而传输的数据为密文的 HTTPS ，就并不存在这个问题，即使这些加密数据被监听，也很难破解这些数据的真正意义。最直接一点的说，网络安全专业毕业的隔壁老王并不知道你在我的网站上看了些什么。

当然除了上一段第一句话，其他的都是扯淡。最近在做微信小程序，数据接口需要 HTTPS ，想了些办法，没辙了，不得不把网站 HTTPS 化了。

## 申请免费的 SSL 证书
阿里云和腾讯云都有，服务器和域名都在阿里云上，就以**阿里云**为例。你可以直接点击[此网址](https://common-buy.aliyun.com/?spm=5176.8064714.317898.pricedetail1111.3bc4FE6cFE6cv6&commodityCode=cas#/buy)，或在阿里云网站上找到“CA证书服务”。

1. 证书品牌选择“赛门铁克”（Symantec），保护类型为“1个域名”，证书类型为“免费型DV SSL”，购买即可。

    ![](http://blogres.zhangyue.xin/18-4-5/48793332.jpg)

2. 之后进入证书控制台，补全相应的信息，等待审核通过即可，由于申请的是免费证书，没人打电话审核，审核通过大约5分钟的样子。由于证书是免费证书，并且不支持通配符，**二级域名是算为单独的一个域名的**。如果你的域名解析在阿里云上，勾选“**证书绑定的域名在【阿里云的云解析】产品中，授权系统自动添加一条记录以完成域名授权验证。**”将自动配置域名解析，还是很方便的。

    ![](http://blogres.zhangyue.xin/18-4-5/35097734.jpg)

    ![](http://blogres.zhangyue.xin/18-4-5/574510.jpg)

3. 单击“操作”中的“下载”，在弹出的页面中下载对应的服务器的证书。

    ![](http://blogres.zhangyue.xin/18-4-5/52528186.jpg)

## 绑定证书
将下载的证书复制到服务器中，服务器为 Windows Server 2016，运行 IIS 。

1. 使用 Cortana ，输入 **MMC** ，运行管理控制台。
2. 单击 “文件” -> “添加/删除管理单元” -> 从“可用的管理单元”列表中选择“证书” -> “添加” -> 选择“计算机帐户”。

    ![](http://blogres.zhangyue.xin/18-4-5/29994664.jpg)

3. 右键单击 “个人” -> “所有任务” -> “导入”，根据提示，默认导入即可。

    ![](http://blogres.zhangyue.xin/18-4-5/89102640.jpg)

4. 接着打开 IIS，找到要绑定的网站，添加网站绑定即可

    ![](http://blogres.zhangyue.xin/18-4-5/53432167.jpg)

现在你可以尝试着访问你的网站了，但是你会发现只有 IE 和 Edge 可以使用 HTTPS 打开，但基于 Chrome 的浏览器却找不到网页。

## 打开 TLS 1.2
TLS 1.2 默认是关闭的，需要手动打开它。本来我找到了一个 Power Shell 脚本，直接复制运行的，现在发现网站历史记录被我给清空了……没办法，百度也没百度到，但搜到了一个小软件，叫 IIS Crypto。一键设置，非常好用，比脚本好用多了…… [下载地址](https://www.nartac.com/Products/IISCrypto)

下载完成后打开，有个“Best practices”，软件会自动帮你选择协议，当然你也可以自己勾选 TLS ，但 SSL 就不要勾选了，选完点“Apply”应用一下，然后重启服务器即可。

![](http://blogres.zhangyue.xin/18-4-5/53356887.jpg)

之后在 Chrome 上就能够通过 HTTPS 访问了。

## HTTP 跳转 HTTPS
在浏览器直接输入域名的话还是会使用 HTTP 协议，要想默认 HTTPS 还需要强制跳转。由于使用的是 IIS ，那么可以直接使用 **IIS URL Rewrite** 模块（Tip：Azure 国际版的 Web 应用可以使用这个进行反向代理，看你明不明白了嘻嘻），[下载地址](https://www.iis.net/downloads/microsoft/url-rewrite)。下载安装完成后打开 IIS。

1. 选择对应的站点，找到“URL 重写”

    ![](http://blogres.zhangyue.xin/18-4-5/1783891.jpg)

2. “添加规则” -> “空白规则”

    ![](http://blogres.zhangyue.xin/18-4-5/10071714.jpg)

3. 匹配 URL -- 模式：(.*)

    ![](http://blogres.zhangyue.xin/18-4-5/76456242.jpg)

4. 条件 -- 添加 -- 条件输入：{HTTPS} -- 类型：与模式匹配 -- 模式：off

    ![](http://blogres.zhangyue.xin/18-4-5/56001319.jpg)

5. 操作 -- 操作类型：重定向 -- 重定向 URL ：https://{HTTP_HOST}/{R:1} -- 重定向类型：永久(301)

    ![](http://blogres.zhangyue.xin/18-4-5/87310728.jpg)

保存即可。

<br />

nice！兄dei！

![](http://blogres.zhangyue.xin/18-4-5/94082207.jpg)