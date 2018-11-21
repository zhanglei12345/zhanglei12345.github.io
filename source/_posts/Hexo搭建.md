---
title: 入门 Hexo 博客搭建 
date: 2018-8-23 21:03:36
categories: 
- BLOG
tags:
- Hexo
---

由于个人博客之前一直是搭在 VPS 上的，可看我的另一篇博客 [VPS 上搭建 Typecho](https://haoleio.com/2017/08/11/VPS%E4%B8%8A%E6%90%AD%E5%BB%BATypecho/)，同时自己的 .me 域名到期续费太贵就没继续用了，索性将博客全部迁移至 Github Pages，绑定了新的域名。1 年时间也没水几篇博客，逃！下面记录下自己搭建的流程吧，总的来说挺简单的。

### 安装

参照 [Hexo 官网](https://hexo.io/zh-cn/docs/)进行安装:

    `npm install -g hexo-cli`

### 建站

```bash
hexo init username.github.io
cd username.github.io
npm install
```

主题自己使用的 [apollo](https://github.com/pinggod/hexo-theme-apollo.git)

```bash
npm install --save hexo-renderer-jade hexo-generator-feed hexo-generator-sitemap hexo-browsersync hexo-generator-archive
git clone https://github.com/pinggod/hexo-theme-apollo.git themes/apollo
```

Hexo 3.0 把服务器独立成了个别模块，您必须先安装 hexo-server 才能使用，后续 git 方式部署需要用到。

`npm install hexo-deployer-git --save`

<!--more-->

### 指令

详细配置及指令请参考[官网](https://hexo.io/zh-cn/docs/commands)。

常用指令：
1. 新建一篇文章

    `hexo new <title>`
    > 等同于在 **source/_posts/** 下新建一个.md文件
2. 清除缓存文件(db.json)和已生成的静态文件(public)

    `hexo clean`
3. 生成静态文件

    `hexo g`
4. 启动服务器,可先在本地预览。默认情况下，访问网址为：http://localhost:4000/

    `hexo s`
5. 部署网站

    `hexo d`

### 添加 disqus 评论

按照 [disqus](https://disqus.com/) 上给的流程，给自己的站点添加评论。apollo 主题下只需修改主题的 _config.yml 配置文件即可，注意要写 Your website shortname

### 部署

部署之前必须在 github 中创建一个 username.github.io 的仓库。然后修改 _config.yml 配置文件，正确配置 deploy 参数，比如我的：

```YAML
deploy:
  type: git
  repo: https://github.com/zhanglei12345/zhanglei12345.github.io.git
  branch: master
```
    
接下来可以根据主题需要，修改主题自身的配置。

### 绑定域名

我在 [godaddy](https://sg.godaddy.com/) 上购买了为期三年的域名，域名解析用的 [DNSPod](https://www.dnspod.cn/)，进入 DNSPod 中添加自己的域名及对应的服务器，同时要去 godaddy 中将你的 Nameservers 修改成 DNSPod 自家的，等几分钟就好,我的设置：

![](https://ws1.sinaimg.cn/large/006tNbRwly1fujzpa9r9tj319a0fogo8.jpg)

设置完成后,ping 一下绑定的域名，ip 跟 zhanglei12345.github.io 的 ip 相同，绑定成功。
在 zhanglei12345.github.io 目录下，新建一文件 CNAME，写上自己的域名，比如我的就是 haoleio.com，重新构建部署博客。
查看 github 上该仓库设置中的 GitHub Pages 参数，可以把域名删掉保存然后再重新填写一遍保存，要不然开启 HTTPS 可能会报错。

![](https://ws2.sinaimg.cn/large/006tNbRwly1fuk063xkikj314u0bygnl.jpg)

### 博客源文件管理

由于 hexo-deployer-git 插件在执行部署操作的时候，首先会自动初始化 git 仓库(位置在 .deploy_git 中)，并关联到指定 repo 与 branch，后续 public 文件夹中自动生成的页面代码将会拷贝至此目录中进行代码管理。GitHub Page 会根据 master 分支的内容来生成页面，并且 master 分支的内容也只包含 public 文件夹里自动生成的文件。可以新建一个分支来管理写博客的源文件。

新建 hexo 分支：

    `git branch hexo`

可结合 hexo 的部署查看一下 .gitignore，之后 git 提交自己的博客源文件，由于自己提交的通常基本都是 hexo 分支，可进入 github 将该仓库的 hexo 分支设置为默认分支。
(注意主题都是 git clone 过来的，可以通过删除主题下的 .git 目录来提交自己的修改)

### 在另外一台设备上管理博客

