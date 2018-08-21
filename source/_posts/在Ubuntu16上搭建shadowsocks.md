---
title: 在 Ubuntu 16.04上搭建 Shadowsocks
date: 2018-08-19 21:44:36
categories: 
- VPS
tags:
- Ubuntu
- Shadowsocks
---
本文介绍在 VPS(bandwagon,系统为 Ubuntu 16.04)上搭建 shadowsocks 服务,并开启 BBR。本机上使用 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG) 客户端,并实现命令行终端走代理模式。

## shadowsocks

登录[Client Area](https://bandwagonhost.com/clientarea.php),在 My Servers 里选择对应的 VPS 进入控制面板，由于刚买的 bandwagon 自带 centos，自己把系统重装为 Ubuntu 16.04 x86_64 (个人偏爱)，重装完成后会告诉你 root 密码和 ssh 端口号。

<!--more-->

远程 ssh 登录修改 root 密码：
```bash
ssh root@ip -p port
passwd
```

搭建 ss：
```bash
apt update
apt upgrade
apt install python3-pip
apt install shadowsocks
```
修改 /etc/shadowsocks/config.json (该文件已存在)的以下内容：
```
server为vps的ip
修改server_port
修改password
```

测试 ss 是否可用：
`ssserver -c /etc/shadowsocks/config.json`

配置 Systemd 管理 Shadowsocks:
` vi /etc/systemd/system/shadowsocks-server.service`
```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target		
```

启动 ss：
`systemctl start shadowsocks-server`

设置开机启动 ss：
`systemctl enable shadowsocks-server`

## 开启 BBR

linux 内核版本要高于 4.9，才支持 BBR。

准备：
```bash
uname -a  # 查看目前的内核版本
dpkg --get-selections |grep linux-image  # 查看安装的内核
apt update  # 更新源
apt-cache showpkg linux-image  # 查看可用的 linux 内核版本
apt install linux-image-4.11.0-14-generic  # 安装内核
```

重启：
```bash
reboot
uname -a
apt autoremove  #自动删除旧版本
```

开启 BBR：
```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr  #如果看到 tcp_bbr 则表示开启成功
```

## ShadowsocksX-NG

在[ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG)的github上进行下载。

## proxy shell

实现命令行终端走代理模式。

在 .zshrc 中添加如下内容：
```bash
# proxy on command-line
proxyon(){
	export http_proxy=http://127.0.0.1:1087
	export https_proxy=http://127.0.0.1:1087
	echo " command-line proxy on "
}

proxyoff(){
	unset http_proxy
	unset https_proxy
	echo " command-line proxy off "
}
```

开启：`proxyon`   
停止：`proxyoff`

通过 `curl http://ipinfo.io/` 验证命令行是否在代理模式下。
