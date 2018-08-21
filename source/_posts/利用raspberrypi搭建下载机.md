---
title: 利用 raspberrypi 搭建下载机
date: 2018-08-19 21:44:36
categories: 
- 树莓派
tags:
---
此篇文章介绍利用 raspberrypi 搭建下载机。

<!--more-->

## aria2
[aria2](https://aria2.github.io/manual/en/html/aria2c.html?highlight=session#) is a utility for downloading files.

安装 aria2：
`sudo apt-get install aria2`

aria2 运行的时候需要两个文件，并且需要我们手动配置，一个是配置文件 **aria2.conf**，保存配置，另一个是 **aria2.session**，要不每次 aria2 关闭的时候，之前下载的进度都没了。

生成 token (外网连接加上验证)：
[PyJWT](https://pyjwt.readthedocs.io/en/latest/)
```bash
sudo pip install pyjwt
python
>>> import jwt
>>> jwt.encode({'some': 'payload'}, 'secret', algorithm='HS256', headers={'name' : 'zhanglei'})
```
会生成一个 token，将添加于 aria2.conf 中。

创建 aria2.conf 和 aria2.session：
```bash
cd 
mkdir .aria2
cd .aria2   
touch aria2.session  
vim aria2.conf
```

aria2.conf 中添加如下内容：
```bash
#文件保存目录 
dir=/home/pi/Downloads
#禁用IPv6, 默认:false
disable-ipv6=true  
#启用RPC, 默认:false
enable-rpc=true  
#允许所有来源, 默认:false
rpc-allow-origin-all=true  
#允许外部访问, 默认:false，false的话只监听本地端口
rpc-listen-all=true  
#设置的RPC授权令牌,上面生成的 token
rpc-secret= token
#RPC监听端口, 端口被占用时可以修改, 默认:6800
#rpc-listen-port=6800  
#允许断点续传
continue=true  
#从会话文件中读取下载任务
input-file=/home/pi/.aria2/aria2.session 
#在aria2退出时保存‘错误/未完成’的下载任务到会话文件
save-session=/home/pi/.aria2/aria2.session 
#最大同时下载任务数
max-concurrent-downloads=3
#整体下载速度限制, 运行时可修改, 默认:0,0意味着没限制
#max-overall-download-limit=0
#单个任务下载速度限制, 默认:0
#max-download-limit=0
#整体上传速度限制, 运行时可修改, 默认:0
#max-overall-upload-limit=0
#单个任务上传速度限制, 默认:0
#max-upload-limit=0
#同服务器连接数，默认:1
max-connection-per-server=5
#最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要，默认：20M
#min-split-size=20M
#单个文件最大线程数，默认：5
#split=5
```

启动：
`sudo aria2c --conf-path="/home/pi/.aria2/aria2.conf" -D`

给 aria2c 设置启动服务:
`sudo vim /etc/init.d/aria2c`

文件中写入下列内容：
```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          aria2
# Required-Start:    $remote_fs $network
# Required-Stop:     $remote_fs $network
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Aria2 Downloader
### END INIT INFO
 
case "$1" in
start)
 
echo -n "Starting aria2c"
sudo -u pi aria2c --conf-path=/home/pi/.aria2/aria2.conf -D
#把上面的两个pi换成你的用户名
;;
stop)
 
echo -n "Shutting down aria2c "
killall aria2c
;;
restart)
 
killall aria2c
sudo -u pi aria2c --conf-path=/home/pi/.aria2/aria2.conf -D
#把上面的两个pi换成你的用户名
;;
esac
exit
```

调整权限：
`sudo chmod 755 /etc/init.d/aria2c`

开机自启：
`sudo update-rc.d aria2c defaults`

## nginx

安装：
`sudo apt-get install nginx`

配置站点属性：
`sudo vim  /etc/nginx/sites-availiable/default`
修改：
```nginx
server {
 	listen 81;
	#listen [::]:80 default_server;
	root /var/www/html;
	server_name pi.com;
	...
```

启动：
`sudo /etc/init.d/nginx start`

## webui-aria2

通过web访问的方式控制树莓派的下载
```bash
cd /var/www
sudo git clone https://github.com/ziahamza/webui-aria2.git
sudo mv webui-aria2/* html/
```

内网中浏览器直接访问树莓派IP地址:端口号即可观看到效果，记得在webui界面的设置-连接设置-密码令牌中要输入配置文件aria2.conf 中的 token。

![](https://ws3.sinaimg.cn/large/006tKfTcly1fjlsle8k95j30zf0n9gnt.jpg)

## 花生壳

可穿透内网的动态域名解析软件，实现外网来访问 webui-aria2 界面进行下载的管理。

本地下载树莓派版本的花生壳：
[官网](http://hsk.oray.com/download/)

将下载后的软件包上传到树莓派上。

在树莓派上 cd 到安装包的目录，接下来进行安装：
`sudo dpkg -i phddns_rapi_3.0.1.armhf.deb`

安装成功后，将显示此树莓派唯一的 SN 码、默认密码以及远程管理地址。

配置花生壳：
浏览器进入[花生壳远程管理页面](http://b.oray.com/)，输入 SN 码和默认密码 admin，首次登陆需要进行初始化，重设密码
，填写手机。默认内置账户只有公网版服务，所以需要开通内网穿透功能。之后要添加两个映射，ip 对应着树莓派的局域网 ip (此ip 要进入路由器的管理界面，将树莓派设置为静态ip地址获取方式，设置时，mac地址为树莓派 wlan0 的 HWaddr),端口分别对应着 webui 的 81 端口( nginx 配置的端口)和 RPC 的 6800 端口(默认)。


外网中浏览器访问花生壳分配的对应内网 81 端口的外网 ip 地址和端口，记得在webui界面的的设置-连接设置-端口设置为花生壳分配的 RPC 端口，设置-连接设置-密码令牌中要输入配置文件 aria2.conf 中的 token。

其他：
终端输入 phddns 回车后，可以看到扩展的功能。

![](https://ws3.sinaimg.cn/large/006tKfTcly1fjlsmfw7zxj30zd0j577p.jpg)

## samba

文件共享服务，让局域网内可以访问。

安装 samba：
`sudo apt-get install samba samba-common-bin`

备份一份配置文件：
`sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak`

编辑配置文件：
`sudo vim /etc/samba/smb.conf`

下面的配置是让用户可以访问自己的 home 目录。
开启用户认证：找到####### Authentication #######,在后面添加一行 `security = user`，来使用户进行验证，禁止匿名登录。
配置用户：在[homes]节中，把 read only = yes 改为 read only = no 。

重启 samba 服务：
sudo /etc/init.d/samba restart

添加账户到共享文件夹，设置一个密码：
sudo smbpasswd -a pi

之后在 mac 上 Finder 中共享的会看到 raspberrypi，点击连接身份，以注册用户的身份登录。

![](https://ws4.sinaimg.cn/large/006tKfTcly1fjlsmxdgarj30mv0cq770.jpg)

## VNC

远程桌面。

安装 VNC：
`sudo apt-get install tightvncserver`

启动服务器：
`vncserver :1`
之后会提示创建密码，先是控制密码，然后是仅查看密码。

然后就可以从别的电脑的 VNC Viewer 访问了，连接时候会提示连接不安全，忽略就好。(自己本地通过192.168.1.102:5901)

给 VNC 建立启动服务：
`sudo vim /etc/init.d/tightvncserver`

文件中写入下列内容：
```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides: tightvncserver
# Required-Start: $syslog $remote_fs $network
# Required-Stop: $syslog $remote_fs $network
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Starts VNC Server on system start.
# Description: Starts tight VNC Server. Script written by James Swineson.
### END INIT INFO

# /etc/init.d/tightvncserver
VNCUSER='pi'
case "$1" in
 start)
 su $VNCUSER -c '/usr/bin/tightvncserver :1 -geometry 1920x1080'
 echo "Starting TightVNC Server for $VNCUSER"
 ;;
 stop)
 pkill Xtightvnc
 echo "TightVNC Server stopped"
 ;;
 *)
 echo "Usage: /etc/init.d/tightvncserver {start|stop}"
 exit 1
 ;;
esac
exit 0
```

调整权限：
`sudo chmod 755 /etc/init.d/tightvncserver`

开机自启：
`sudo update-rc.d tightvncserver defaults`

重启树莓派：
`sudo reboot`

![](https://ws4.sinaimg.cn/large/006tKfTcly1fjlsndftufj31330mkkan.jpg)

## 预留问题

1. 花生版免费版连接不是很稳定，并且每月只有1G的免费流量，尝试通过其他方式实现内网穿透；

2. 由于需求量不是很大，只是初步玩一玩，所以我没挂载大容量的移动硬盘，下载机的存储空间用的还是树莓派的32G启动盘；

3. aria2目前还没装扩展。 