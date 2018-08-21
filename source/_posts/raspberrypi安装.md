---
title: raspberrypi 安装
date: 2018-08-19 21:44:36
categories: 
- 树莓派
tags:
---
此篇文章主要介绍树莓派3代B型从刚入手后进行 Raspbian 系统的安装。

## 配件

1. 电源线
2. 7寸液晶屏
3. 散热片
4. 蓝牙键鼠
5. 连接显示器 HDMI-HDMI 线
6. 外壳+风扇
7. 32G存储的SD卡，用做硬盘(外需读卡器进行刚开始的格式化)

<!--more-->

## 安装系统

[官方安装文档](https://www.raspberrypi.org/documentation/installation/noobs.md)

下载 [NOOBS](https://www.raspberrypi.org/downloads/noobs/);

格式化 SD 卡为 FAT 格式,利用工具[SD Card Formatter](https://www.sdcard.org/downloads/formatter_4/);

解压 NOOBS.zip 到 SD 卡根目录,利用软件 [Keka](http://www.kekaosx.com/zh-cn/) 来进行解压;

将 SD 卡插入到树莓派对应卡槽中，接通电源，启动系统；

在安装界面选择安装 Raspbian。

## SSH 登录

系统安装好后，查看它的局域网IP地址：
`$ sudo ifconfig`

更改系统设置，SSH 登录系统默认是禁止的：
Preferences -> Raspberry Pi Configuration -> Interfaces -> SSH Enabled

树莓派默认用户：**pi**，pi 默认密码是 **raspberry**

ssh 登录：`ssh pi@ip地址`

修改密码：`$ passwd`

## 连接 WIFI

如果外接了显示器，则直接在屏幕右上角选择即可；

若使用 ssh 登录，则需要使用以下命令编辑 wifi 配置文件：
`sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`
在文件末添加：
```
network={
   ssid="wifiname"
   psk="password"
}
```

重启：
`sudo reboot`

## 软件安装

更改软件源：
`sudo nano /etc/apt/sources.list`
将默认的注释掉，添加：
`deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ jessie main contrib non-free rpi`

> 或者使用以下地址代替上面的地址：
> 清华大学 Raspbian http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/
> 中国科学技术大学 Raspbian http://mirrors.ustc.edu.cn/raspbian/raspbian/

更改后，Ctrl+O回车保存，Ctrl+X退出nano编辑器。

更新源：
`sudo apt-get update`

更新已安装的包(先不执行)：
`sudo apt-get upgrade`

安装 vim：
`sudo apt-get install vim`

![](https://ws1.sinaimg.cn/large/006tKfTcly1fjlrdmbafsj31kw1nv4qp.jpg)