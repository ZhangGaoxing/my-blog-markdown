## .NET Core 使用二进制文件安装

1. 下载
    ```
    wget https://download.visualstudio.microsoft.com/download/pr/11d6ec80-4d7f-4100-8a54-809ed30b203e/1c0267225b22437aca9fdfe04160d1d5/dotnet-sdk-3.0.100-preview7-012821-linux-arm.tar.gz
    ```
2. 解压
    ```
    mkdir ~/dotnet3p7 && tar -xvf dotnet-sdk-3.0.100-preview7-012821-linux-arm.tar.gz -C ~/dotnet3p7
    ```
3. 创建链接
    ```
    sudo ln -s ~/dotnet3p7/dotnet /usr/bin/dotnet
    ```

## 安装 PostgreSQL

1. 下载
    ```
    sudo apt-get update
    sudo apt-get install postgresql-11
    ```
2. 修改 postgres 密码
    ```
    sudo -u postgres psql
    postgres=# ALTER USER postgres WITH PASSWORD '123456'; 
    postgres=# \q
    ```
3. 修改系统用户 postgres 密码（要与数据库用户密码一致）
    ```
    sudo passwd -d postgres
    sudo -u postgres passwd
    ```
4. 修改数据库配置实现远程访问
    ```
    sudo nano /etc/postgresql/11/main/postgresql.conf
    ```
    1. `#listen_addresses = 'localhost'` 改为 `listen_addresses = '*'`
    2. `#password_encryption = md5` 改为 `password_encryption = md5`
5. 配置数据库访问认证
    ```
    sudo nano /etc/postgresql/11/main/pg_hba.conf
    ```
    添加一行 `host all all 0.0.0.0/0 md5`
6. 重启服务
    ```
    sudo /etc/init.d/postgresql restart
    ```

## 安装 Docker

1. 使用脚本安装
    ```
    curl -sSL https://get.docker.com | sh
    ```
2. 建立 Docker 用户组
    ```
    sudo groupadd docker
    sudo usermod -aG docker $USER
    ```
3. 更换源
    ```
    sudo nano /etc/docker/daemon.json
    ```
    添加
    ```
    {
        "registry-mirrors": [
            "https://dockerhub.azk8s.cn",
            "https://reg-mirror.qiniu.com"
        ]
    }
    ```
4. 重启服务
    ```
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

## 启用 FTP 服务

1. 安装 `vsftp`
    ```
    sudo apt-get install vsftpd
    ```
2. 编辑配置文件
    ```
    sudo nano /etc/vsftpd.conf
    ```
    `#write_enable=YES` 改为 `write_enable=YES`
3. 重启服务
    ```
    sudo service vsftpd restart
    ```

## 更换 Raspbian 源

1. 编辑 `sources.list`
    ```
    sudo nano /etc/apt/sources.list
    ```
    删除原文件所有内容后添加
    ```
    deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
    deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
    ```
2. 编辑 `raspi.list`
    ```
    sudo nano /etc/apt/sources.list.d/raspi.list
    ```
    删除原文件所有内容后添加
    ```
    deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
    ```

## 启用 USB Camera

1. 安装 Video for Linux
    ```
    sudo apt-get install v4l-utils
    ```
2. 安装 `fswebcam`或`xawtv`
    ```
    sudo apt-get install fswebcam
    ```
    ```
    sudo apt-get install xawtv
    sudo apt-get install streamer
    ```
3. 保存图像
    ```
    sudo fswebcam --save /home/$USER/image.jpg -d /dev/video0 -r 640x480
    ```
    ```
    sudo streamer -c /dev/video0 -b 32 -o image.jpg
    ```

## OPi Zero WiFi 在 Debian 中无法使用

在 `/etc/NetworkManager/NetworkManager.conf` 中添加
```
[device]
wifi.scan-rand-mac-address=no
```

## WSL2 中配置 Docker 自启

1. 配置 `sudo` 权限
    ```
    sudo visudo
    ```
    替换 `username` 后添加
    ```
    username ALL=(ALL:ALL) NOPASSWD: /usr/sbin/service
    ```
2. 编辑 `bash.rc`
    ```
    sudo nano ~/.bashrc
    ```
    添加
    ```
    service docker status > /dev/null || sudo service docker start
    ```
3. 配置 Windows 自启
    在启动文件夹（shell:startup）下添加 `vbs` 脚本
    ```
    Set ws = CreateObject("Wscript.Shell")
    ws.run "ubuntu", vbhide
    ```
    
## PostgreSQL 设置 search_path
    ```
    SHOW search_path;
    ALTER DATABASE <database_name> SET search_path TO schema1,schema2;
    ```
    
## Git 设置代理
    ```
    git config --global http.proxy socks5://127.0.0.1:1080
    git config --global https.proxy socks5://127.0.0.1:1080
    
    git config --global --unset http.proxy
    git config --global --unset https.proxy
    ```
