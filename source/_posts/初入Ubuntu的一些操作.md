---
title: 初入 Ubuntu 的一些操作
date: 2019-01-16 18:15:36
categories: 
- VPS
tags:
- Ubuntu
---

### 查看系统版本

`cat /etc/os-release`

### 修改 root 密码

`passwd`

### 新建用户

新建用户：

`adduser username`

将新用户加入 sudo 组,这样就可以用 sudo 命令了：

`gpasswd sudo -a username`

> 若不执行此操作，执行 sudo 命令时，会提示 username is not in the sudoers file.  This incident will be reported.

### 更改 ssh 默认端口

`sudo vi /etc/ssh/sshd_config` ，修改 Port 自定义端口号，之后重启 ssh 服务：`sudo systemctl restart sshd.service` ，根据自己需要看是否禁止 root 用户登录，修改配置文件：`PermitRootLogin no`，之后重启 ssh 服务。

<!--more-->

### ssh 免密登陆

本机生成密钥：

```bash
mkdir -p ~/.ssh/vps
ssh-keygen -t rsa -C "***@gmail.com" -f ~/.ssh/vps/id_rsa
```

config 文件配置，在 ~/.ssh/ 下新建 config 文件， `vi config` :

```conf
# vps configuration
Host vps
	HostName ***.***.***.***
	Port ***
	IdentityFile ~/.ssh/vps/id_rsa
	User ***
```

本机登陆 vps, 需要输入密码：

`ssh vps`

创建 .ssh 目录：

`cd ~;mkdir .ssh`

本机将公钥拷贝至 vps 的用户的 .ssh 目录下, 需要输入密码：

`scp id_rsa.pub vps:~/.ssh/authorized_keys`

修改 vps 上 authorized_keys 文件权限：

`chmod 600 authorized_keys`

之后可实现本机免密登陆 vps。

### 安装 vim

```bash
sudo apt update
sudo apt install vim
```

### 安装 ftp 

[参考链接](http://wiki.ubuntu.org.cn/Vsftpd)  

安装：

```bash
sudo apt install ftp
sudo apt install vsftpd
# 查看是否打开21端口
sudo netstat -npltu | grep 21
# 登录
ftp localhost
```

修改配置文件 /etc/vsftpd.conf：

```conf
# 设置控制连接的监听端口号，默认为21
listen_port=<port>
# 是否开放本地用户的写权限
write_enable=YES
anon_mkdir_write_enable=YES
anon_upload_enable=YES
# 限制一切用户登录，只允许列表文件中的用户，用 userlist_file
userlist_enable=YES
userlist_deny=NO
userlist_file=/etc/vsftpd.user_list
```

创建 /etc/vsftpd.user_list，写入只允许登录的用户名。

启动服务：

`sudo service vsftpd start`

### 安装 git

```bash
sudo apt install git
```

### 安装 Docker

[Docker 官方安装教程](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

设置存储库：

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

安装 DOCKER CE： 

```bash
sudo apt update
# Install the latest version of Docker CE
sudo apt install docker-ce
# Verify that Docker CE is installed correctly by running the hello-world image.
sudo docker run hello-world
```

启动 Docker CE：

```bash
sudo systemctl enable docker 
sudo systemctl start docker
```

建立 docker 用户组：

默认情况下，docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。

建立 docker 组：

`sudo groupadd docker`

将当前用户加入 docker 组：

`sudo usermod -aG docker $USER`

### 搭建 shadowsocks

从 [Docker Hub](https://hub.docker.com/) 下载自己建立的镜像。

```bash
# 获取镜像
docker pull zhanglei12345/shadowsocks-libev
# 启动容器
docker run --restart=always -d -p hostPort:containerPort zhanglei12345/shadowsocks-libev:latest -s 0.0.0.0 -p containerPort -k mypassword -m aes-256-cfb
```

### 搭建 shadowsocksr

从 [Docker Hub](https://hub.docker.com/) 下载自己建立的镜像。

```bash
# 获取镜像
docker pull zhanglei12345/shadowsocksr
# 启动容器
docker run --restart=always -d -p hostPort:containerPort zhanglei12345/shadowsocksr:latest -p containerPort -k mypassword -m aes-256-cfb -O auth_sha1_v4 -o http_simple
```

### 安装 Docker Compose

[Releases · docker/compose · GitHub](https://github.com/docker/compose/releases)

```bash
# 下载对应版本
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 赋执行权限
sudo chmod +x /usr/local/bin/docker-compose
# 检查版本号
docker-compose --version
```