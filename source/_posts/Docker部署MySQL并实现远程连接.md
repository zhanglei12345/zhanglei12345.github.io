---
title: Docker 部署 MySQL 并实现远程连接
date: 2019-01-17 16:20:05
categories: 
- VPS
tags:
- Docker
---

### Docker 部署 MySQL

下载官方镜像：
`docker pull mysql`

列出镜像：
`docker image ls`

创建主机存储目录：
`cd && mkdir mysql-data mysql-conf && cd mysql-conf`

建立 mysql 配置文件，暂时将配置设置为跟容器构建时的配置一致，`vi config-file.cnf`：
```conf
[mysqld]
skip-host-cache
skip-name-resolve
```

启动容器：
`docker run --name mysql --restart=always -v /home/user/mysql-conf:/etc/mysql/conf.d -v /home/user/mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d -P mysql:latest`

进入 mysql 容器：
`docker exec -it mysql bash`

连接 mysql 数据库：
`mysql -uroot -p`

新建 mysql 用户：
`create user 'mysql-user'@'%' identified by 'mysql-pwd';`

新建数据库：
`create database testbase;`

赋用户权限：
`grant all privileges on testbase.* to 'mysql-user'@'%';`

立即生效：
`flush privileges;`

<!--more-->

### 远程客户端连接

Sequel Pro 客户端连接时报如下错误：
```
MySQL said: Authentication plugin ‘caching_sha2_password’ cannot be loaded: dlopen(/usr/local/mysql/lib/plugin/caching_sha2_password.so, 2): image not found
```

原因是由于密码加密方式 caching_sha2_password 客户端不支持。

修改 mysql 用户的密码加密方式：
`alter user 'mysql-user'@'%' identified with mysql_native_password by 'mysql-pwd';`

之后客户端可成功连接。

参考：
[Docker Hub mysql](https://hub.docker.com/_/mysql?tab=description)