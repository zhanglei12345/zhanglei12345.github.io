---
title: VPS 上搭建 Typecho
date: 2017-08-11 21:44:36
categories: 
- VPS
tags:
- VPS
- Typecho
---
> 念念不忘，必有回响

[Typecho](http://typecho.org/) 是一个强大的个人博客系统，是基于 PHP 开发的非常轻量级的博客框架，原生支持 Markdown 排版语法，易读更易写。Typecho 的特点：轻量高效、先进稳定、简洁友好。

#### 准备工作 

* VPS
* 域名
* nginx
* php7.0
* sqlite3

#### VPS

[bandwagon](https://bandwagonhost.com/aff.php?aff=18070&a=add&pid=56)
得到服务器及 IP,自己重新安装了 Ubuntu 16.04 x86_64 系统。(512 MB  的内存，10G的硬盘容量，每月500G流量)

#### 域名注册

[godaddy](https://sg.godaddy.com/)
购买了一年的有效期 ~~freeblog.me~~(已弃用)

#### 域名解析

[DNSPod](https://www.dnspod.cn/)
进入DNSPod中添加自己的域名及对应的服务器IP，同时要去godaddy中将你的Nameservers修改成DNSPod自家的，等几分钟就好了，`ping  freeblog.me` 可查看到你的服务器IP，域名解析成功。

<!--more-->

#### 远程登录到服务器

自己登录的 root 用户。
根据 [typecho](http://typecho.org/) 对服务器的要求，需要安装以下：

```bash
apt install nginx
apt install sqlite3
apt install php7.0
apt install php7.0-sqlite3
apt install php7.0-curl
```

官网下载 typecho 1.0正式稳定版的压缩包并上传到服务器的 /var/www/，`tar -xzf 1.0.14.10.10.-release.tar.gz`,解压完会出现 build 目录，对该目录赋权限`chmod  -Rf 755 *`。并新增 /var/www/log/freeblog/access_log 和 /var/www/log/freeblog/error_log 用来存放 nginx 的日志。

启动nginx配置：

`service nginx start`

浏览器访问服务器ip可看到 nginx 的欢迎界面。

nginx 修改配置：

```bash
cd /etc/nginx/sites-available
vi freeblog
```

freeblog文件中添加如下内容：
```nginx
server {
	listen          80;
	server_name     freeblog.me;
	root            /var/www/build;
	access_log		/var/www/log/freeblog/access_log;
	error_log		/var/www/log/freeblog/error_log;
	index           index.html index.htm index.php;

	if (!-e $request_filename) {
		rewrite ^(.*)$ /index.php$1 last;
	}

	location ~ .*\.php(\/.*)*$ {
		include fastcgi.conf;
		fastcgi_pass  unix:/run/php/php7.0-fpm.sock;  # 注意：typecho 官网给的配置为 127.0.0.1:9000,使用此方式浏览器会报 502 错，在/etc/php/7.0/fpm/pool.d/www.conf 文件中可看到 listen = /run/php/php7.0-fpm.sock。所以要对 fastcgi_pass 进行修改。
	}

}
```

编辑完成后要:

```bash
cd /etc/nginx/sites-enabled
rm -f default
ln -s /etc/nginx/sites-available/freeblog .
```

启动服务：

```bash
service php7.0-fpm restart
service nginx reload  或  service nginx restart
```

创建sqlite3数据库：

```bash
cd /var/www/build/usr
sqlite3 freeblog.db  # 可创建sqlite数据库，.database 查看数据库， .exit 退出数据库
chmod 666 freeblog.db
chmod 777 /var/www/build/usr  # 注意：不加此权限安装时浏览器会报500数据库错误
```

浏览器中输入 freeblog.me 进行typecho的安装

进入首页，点击第一篇文档的链接，浏览器会报500的错误，查看nginx记录的错误日志:

```
Fatal error: Uncaught TypeError: Argument 1 passed to Typecho_Common::exceptionHandle() must be an instance of Exception, instance of Error given in /var/www/html/blog/var/Typecho/Common.php:235 Stack trace: #0 [internal function]: Typecho_Common::exceptionHandle(Object(Error)) #1 {main} thrown in /var/www/html/blog/var/Typecho/Common.php on line 235
```

此时要打开 /var/www/html/blog/var/Typecho/Common.php ，找到 exceptionHandle() 函数，把该函数的参数改为`exceptionHandle(Throwable $exception)`。

再次刷新第一篇文章的页面，页面提示 `Call to undefined function utf8_decode() `，此时 `apt install php7.0-xml` ，重启nginx即可。

#### Certbot 配置 Let’s Encrypt SSL 安全证书

获取certbot客户端:

```bash
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto --help
```

配置 nginx 、验证域名所有权,在/etc/nginx/sites-available/freeblog中添加：

```nginx
location ^~ /.well-known/acme-challenge/ {
        default_type "text/plain";
        root /var/www/build;
}

location = /.well-known/acme-challenge/ {
        return 404;
}
```

重载nginx:

`service nginx reload`

生成证书(要在安装客户端的目录下执行):

`./certbot-auto certonly --webroot -w /var/www/build -d freeblog.me`

证书生成成功后会有提示：

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/freeblog.me/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/freeblog.me/privkey.pem
   Your cert will expire on 2017-11-24. To obtain a new or tweaked version of this certificate in the future, simply run certbot-auto again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
 - Your account credentials have been saved in your Certbot configuration directory at /etc/letsencrypt. You should make a secure backup of this folder now. This configuration directory will also contain certificates and private keys obtained by Certbot so making regular backups of this folder is ideal.
```

继续配置nginx,最后/etc/nginx/sites-available/freeblog的内容如下：

```nginx
server {
        listen 80;
        server_name freeblog.me;
        return 301 https://$server_name$request_uri;
}
server {
        listen          443 ssl http2;
        server_name     freeblog.me;
        root            /var/www/build;
        access_log      /var/www/log/freeblog/access_log;
        error_log       /var/www/log/freeblog/error_log;
        index           index.html index.htm index.php;

        ssl_certificate      /etc/letsencrypt/live/freeblog.me/fullchain.pem;
        ssl_certificate_key     /etc/letsencrypt/live/freeblog.me/privkey.pem;

        location ^~ /.well-known/acme-challenge/ {
                default_type "text/plain";
		root	/var/www/build;
        }

        location = /.well-known/acme-challenge/ {
                return 404;
        }

        if (!-e $request_filename) {
                rewrite ^(.*)$ /index.php$1 last;
        }

        location ~ .*\.php(\/.*)*$ {
                include fastcgi.conf;
                fastcgi_pass  unix:/run/php/php7.0-fpm.sock;
        }

}
```

重载nginx，`service nginx reload`，之后浏览器重新查看 freeblog.me，可观察到已经https了。

可从证书生成的提示中看到证书有效期只有90天，所以每三个月需要更新一次安全证书。用crontab定时任务，每两个月的周六1:30更新证书和1:35重载nginx:

```bash
30 1 * */2 6 /root/download/certbot-auto renew --quiet --no-self-upgrade > /dev/null 2>&1
35 1 * */2 6 /etc/init.d/nginx reload
```

#### 启用 HSTS

HSTS 是“HTTP Strict Transport Security”（HTTP严格安全传输）的缩写。

访问网站时，用户很少直接在地址栏输入 https，总是通过点击链接，或者3xx重定向，从 HTTP 页面进入 HTTPS 页面。攻击者完全可以在用户发出HTTP请求时，劫持并篡改该请求。

HSTS 的作用就是强制浏览器只能发出 HTTPS 请求，并阻止用户接受不安全的证书。

在nginx配置文件中加入：

`Strict-Transport-Security: max-age=31536000; includeSubDomains; preload;`

申请 HSTS Preloading List：

[HSTS Preloading List](https://hstspreload.org/) 是一个网站列表，它被硬编码到chrome浏览器中，仅仅通过https访问。我从申请开始到通过大概花了3天多时间。

#### 主题

简约主题:[maupassant](https://github.com/pagecho/maupassant)。

目前所用主题:[pinghsu](https://github.com/chakhsu/pinghsu)。

#### 实现首页不显示全文

第一种方式是编辑文章时利用 `<!–more–>` 标签显示文章的摘要。(推荐)

第二种方式是网页登录到博客后台，修改主题的代码(需要给主题目录 /var/www/build/usr/themes 写权限`chmod -Rf 777 themes`,顺便我也给了plugins和uploads权限，`chmod -Rf 777 plugins`，`chmod -Rf 777 uploads`)。在index.php文件找到代码 `<?php $this->content('阅读剩余部分...'); ?>`, 将其替换为`<?php $this->excerpt(300, '...'); ?>`, 即显示300个字节，同时在archive.php中进行相同的修改操作。这种方式在首页不会显示内容的markdown格式。

#### 代码高亮

在maupassant主题上增加代码高亮，我用的[Prism](http://prismjs.com/download.html), 勾选需要支持的语言, 下载prism.css和prism.js, 上传至服务器 /var/www/build/usr/themes/maupassant 目录下, 即上传到自己的主题下面, 同时要在浏览器中修改自己主题下面的header.php,在 `</head>` 之前添加  

```html
<!--代码高亮-->
    <link rel="stylesheet" href="<?php $this->options->themeUrl('prism.css'); ?>">
    <script type="text/javascript" src="<?php $this->options->themeUrl('prism.js'); ?>"></script>
```