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
        "https://dockerproxy.com",
        "https://hub-mirror.c.163.com",
        "https://mirror.baidubce.com",
        "https://ccr.ccs.tencentyun.com"
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

## wiringOP
```
git clone https://github.com/orangepi-xunlong/wiringOP.git
cd wiringOP
./build clean
./build
gcc -Wall -pthread -o wop wop.c -lwiringPi -lcrypt -lm -lrt
```

```c
#include <wiringPi.h>

int main (void) {
  int pin = 2;
  
  wiringPiSetup();
  pinMode(pin, OUTPUT);

  while(1) {
    digitalWrite (pin, HIGH);
    digitalWrite (pin, LOW);
  }

  return 0;
}
```

## Shadowsocks 安装

1. 安装
    ```bash
    sudo apt install -y shadowsocks-libev shadowsocks-v2ray-plugin
    ```
2. 配置
    ```bash
    sudo nano /etc/shadowsocks-libev/config.json
    ```
    ```json
    {
        "server":"0.0.0.0",
        "mode":"tcp_and_udp",
        "server_port":8388,
        "local_port":1080,
        "password":"Passw0rd",
        "timeout":86400,
        "method":"chacha20-ietf-poly1305",
        "plugin":"ss-v2ray-plugin",
        "plugin_opts":"server"
    }
    ```
3. 重启服务
    ```bash
    sudo service shadowsocks-libev restart
    ```

## Linux 配置 Shadowsocks 代理
```
{
    "server":"",
    "server_port":10800,
    "local_address":"0.0.0.0",
    "local_port":3000,
    "password":"",
    "timeout":60,
    "method":"chacha20-ietf-poly1305",
    "plugin":"obfs-local",
    "plugin_opts":"obfs=http;obfs-host=www.baidu.com"
}
```
```
sudo nohup ss-local -c /etc/shadowsocks-libev/config.json &
export ALL_PROXY=socks5://127.0.0.1:23474
```

## potainer.io 安装
```
docker volume create potainer_data
docker run -d --name portainer -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v potainer_data:/data portainer/portainer-ce
```

## potainer.io 添加远程节点（配置 Docker 远程访问）
```
sudo nano /usr/lib/systemd/system/docker.service
找到 ExecStart 字段，添加 -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## WSL 自启
```
sudo nano /etc/init-wsl
```
```
#!/bin/sh
echo initializing services
service docker start
```
```
wsl -d Ubuntu -u root /etc/init-wsl
```

## WSL2 Ubuntu 22.04 Docker
```
The reason this errors occurs is because Ubuntu 22.04 LTS uses iptables-nft by default. You need to switch to iptables-legacy so that Docker will work again:

Run sudo update-alternatives --config iptables
Enter 1 to select iptables-legacy
Now run sudo service docker start, and Docker will start as expected!
```

## SQL Server Docker
```
docker pull mcr.microsoft.com/mssql/server
docker volume create mssql_data
docker run -d --name mssql -p 14330:1433 --restart=always -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=@Passw0rd' -e -v mssql_data:/var/opt/mssql mcr.microsoft.com/mssql/server
```

## PostgreSQL Docker
```
docker pull postgres
docker volume create pgsql_data
docker run -d --name pgsql -p 54321:5432 --restart=always -e POSTGRES_PASSWORD='@Passw0rd' -e TZ='Asia/Shanghai' -e ALLOW_IP_RANGE=0.0.0.0/0 -v pgsql_data:/var/lib/postgresql/data postgres
```

## TimescaleDB Docker
```
docker pull timescale/timescaledb:latest-pg14
docker volume create tsdb_data
docker run -d --name timescaledb -p 54321:5432 --restart=always -e POSTGRES_PASSWORD='@Passw0rd' -e TZ='Asia/Shanghai' -e ALLOW_IP_RANGE=0.0.0.0/0 -v tsdb_data:/var/lib/postgresql/data timescale/timescaledb:latest-pg14
```

## MySQL Docker
```
docker pull mysql
docker volume create mysql_data
docker run -d --name mysql -p 33060:3306 -e MYSQL_ROOT_PASSWORD=@Passw0rd -e TZ=Asia/Shanghai --restart=always -v mysql_data:/var/lib/mysql mysql
```

## MongoDB Docker
```
docker pull mongo
docker volume create mongo_data
docker run -d --name mongodb -p 27017:27017 --restart=always -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=@Passw0rd -v mongo_data:/data/db mongo
```

## Redis Docker
```
docker pull redis
docker volume create redis_data
# 下载配置文件 http://download.redis.io/redis-stable/redis.conf
docker run -d --name redis -p 6379:6379 -v /home/$USER/redisconf:/usr/local/etc/redis -v redis_data:/data --restart=always redis redis-server /usr/local/etc/redis/redis.conf
```

## GRUB 32 位引导 64 位系统
1. 系统安装完成后进入 GRUB 临时引导
```
set root=(hd0,gpt2)
linux /boot/vmlinuz* root=/dev/mmcblk1p2 quiet
initrd /boot/initrd.img*
boot
```
2. 进入系统后
```
sudo apt purge grub-efi-amd64 grub-efi-amd64-bin grub-efi-amd64-signed
sudo apt install grub-efi-ia32-bin grub-efi-ia32 grub-common grub2-common
sudo grub-install --target=i386-efi /dev/mmcblk1p2 --efi-directory=/boot/efi/ --boot-directory=/boot/ --no-nvram --removable
sudo update-grub 或 sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## NVIDIA Docker
```
# 卸载驱动
sudo apt --purge remove nvidia*
# 查询驱动
sudo ubuntu-drivers devices
# 安装驱动
sudo apt install nvidia-driver-525-server
# 安装 nvidia-container-toolkit
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
      && curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
      && curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
            sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
            sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt install nvidia-container-toolkit
```

## WSL2 代理
1. 在用户目录 `%USERPROFILE%` 下面创建一个配置文件 `.wslconfig`
```
[wsl2]
networkingMode=mirrored
firewall=true
dnsTunneling=true
autoProxy=true

[experimental]
autoMemoryReclaim=dropcache
hostAddressLoopback=true
useWindowsDnsCache=true
```
2. Docker 配置文件 `/etc/docker/daemon.json` 新增
```
"iptables": false
```

## Docker curl 镜像
```
docker run --rm -it docker.io/appropriate/curl /bin/sh
```
