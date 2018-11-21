---
title: 在 Ubuntu 16.04上搭建 Shadowsocks
date: 2018-08-10 21:44:36
categories: 
- VPS
tags:
- Ubuntu
- Shadowsocks
---
本文介绍在 VPS(bandwagon,系统为 Ubuntu 16.04)上搭建 shadowsocks 和 shadowsocksR 服务。本机上使用 [ShadowsocksX-NG](https://github.com/shadowsocks/ShadowsocksX-NG) 客户端,并实现命令行终端走代理模式。

## shadowsocks

登录[Client Area](https://bandwagonhost.com/clientarea.php),在 My Servers 里选择对应的 VPS 进入控制面板，由于刚买的 bandwagon 自带 centos，自己把系统重装为 Ubuntu 16.04 x86_64 (个人偏爱)，重装完成后会告诉你 root 密码和 ssh 端口号。

远程 ssh 登录修改 root 密码：
```bash
ssh root@ip -p port
passwd
```

<!--more-->

### 搭建 ss 

1. 准备：
```bash
apt update
apt upgrade
apt install python3-pip
apt install shadowsocks
```

2. 修改 /etc/shadowsocks/config.json (该文件已存在)的以下内容：
```
server为vps的ip
修改server_port
修改password
```

3. 测试 ss 是否可用：
`ssserver -c /etc/shadowsocks/config.json`

4. 配置 Systemd 管理 Shadowsocks:
` vi /etc/systemd/system/shadowsocks-server.service` 

    ```bash
    [Unit]
    Description=Shadowsocks Server
    After=network.target
    
    [Service]
    ExecStart=/usr/local/bin/ssserver -c /etc/shadowsocks/config.json
    Restart=on-abort
    
    [Install]
    WantedBy=multi-user.target		
    ``` 

5. 启动 ss：
`systemctl start shadowsocks-server`

6. 设置开机启动 ss：
`systemctl enable shadowsocks-server`

### 开启 BBR

linux 内核版本要高于 4.9，才支持 BBR。

1. 准备：
```bash
uname -a  # 查看目前的内核版本
dpkg --get-selections |grep linux-image  # 查看安装的内核
apt update  # 更新源
apt-cache showpkg linux-image  # 查看可用的 linux 内核版本
apt install linux-image-4.11.0-14-generic  # 安装内核
```

2. 重启：
```bash
reboot
uname -a
apt autoremove  #自动删除旧版本
```

3. 开启 BBR：
```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
lsmod | grep bbr  #如果看到 tcp_bbaptr 则表示开启成功
```

### 利用 Docker 搭建 ss

[参考链接](https://github.com/shadowsocks/shadowsocks-libev#debian--ubuntu)

1. 准备：安装好 Docker
[Docker 官方安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

2. 编写 Dockerfile:  
    ```bash
    FROM ubuntu:18.04
    
    RUN apt update && \
        apt install -y shadowsocks-libev && \
        apt autoremove
    
    ENTRYPOINT ["/usr/bin/ss-server"]
    ```
    
3. 在 Dockerfile 文件路径下，docker build:
`sudo docker build -t ssdocker:v1 .`

4. 查看镜像是否构建成功：
`sudo docker image ls -a`

5. docker run：
`sudo docker run -d -p 12137:12137 ssdocker:v1 -s 0.0.0.0 -p 12137 -k mypassword -m aes-256-cfb`

6. 查看 ss 容器是否启动成功：
`sudo docker container ls -a`

## shadowsocksR

### 搭建 ssr

1. 获取源代码：
```bash
git clone -b manyuser https://github.com/shadowsocksr-backup/shadowsocksr.git 
cd shadowsocksr
bash initcfg.sh
```

2. 修改 shadowsocksr 目录下的 user-config.json 配置文件:
```
"server_port":prot,         //端口
"password":"password",     //密码
"protocol":"auth_sha1_v4",       //协议插件
"obfs":"http_simple",      //混淆插件
"method":"aes-256-cfb",    //加密方式
```

3. 配置 Systemd 管理 shadowsocksR:
`vi /etc/systemd/system/shadowsocksr-server.service`

    注意修改shadowsocksr的目录
    ```bash
    [Unit]
    Description=ShadowsocksR server
    After=network.target
    Wants=network.target
    
    [Service]
    Type=forking
    PIDFile=/var/run/shadowsocksr.pid
    ExecStart=/usr/bin/python /root/git-clone-repository/shadowsocksr/shadowsocks/server.py --pid-file /var/run/shadowsocksr.pid -c /root/git-clone-repository/shadowsocksr/user-config.json -d start
    ExecStop=/usr/bin/python /root/git-clone-repository/shadowsocksr/shadowsocks/server.py --pid-file /var/run/shadowsocksr.pid -c /root/git-clone-repository/shadowsocksr/user-config.json -d stop
    ExecReload=/bin/kill -HUP $MAINPID
    KillMode=process
    Restart=always
    
    [Install]
    WantedBy=multi-user.target
    ```

4. 启动 ssr：
`systemctl start shadowsocksr-server`

5. 设置开机启动 ssr：
`systemctl enable shadowsocksr-server`

### 利用 Docker 搭建 ssr

[参考链接：ShadowsocksR 服务端安装教程](https://github.com/shadowsocksr-backup/shadowsocks-rss/wiki/Server-Setup)

1. 准备：安装好 Docker
[Docker 官方安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

2. 获取 ssr 源代码：
`git clone -b manyuser https://github.com/shadowsocksr-backup/shadowsocksr.git`

3. 初始化配置：
    ```bash
    cd shadowsocksr
    bash initcfg.sh
    ```
    
4. 编写 Dockerfile:  
    ```bash
    FROM ubuntu:18.04

    RUN apt update && \
        apt install -y python && \
        apt autoremove
            
    WORKDIR /ssdir

    COPY . .

    ENTRYPOINT ["python","/ssdir/shadowsocksr/shadowsocks/server.py"]
    ```
    
5. 在 Dockerfile 文件路径下，docker build:
`sudo docker build -t ssrdocker:v1 .`

6. 查看镜像是否构建成功：
`sudo docker image ls -a`

7. docker run：
`sudo docker run -d -p 12138:12138 ssrdocker:v1 -p 12138 -k mypassword -m aes-256-cfb -O auth_sha1_v4 -o http_simple`

8. 查看 ss 容器是否启动成功：
`sudo docker container ls -a`

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
