---
title: 入手折腾 Samsung Chromebook Pro
date: 2018-08-19 21:44:36
categories:
- 生活杂记
tags:
- Chromebook
---
前段时间被 Chromebook 吸引，入手了一款 Samsung Chromebook Pro,具体配置可自行 google，下面对折腾记录一下。

登录
--

Chromebook 开机登录需要 google 帐号，自己通过在树莓派上搭建的可自由上网的路由器搞定，登录入系统后可通过 play store 下载 shadowsocks，挂代理后就不需要一直连接树莓派了(由于目前只搭建了全局，还没做优化)，连接上任一 wifi 即可。

进入开发者模式
-------

按住 Esc 键和刷新(F3)按键，然后按住 Power 按钮，调用 Recovery 模式。
出现 Recovery 屏幕后，按 Ctrl + D。该操作没有提示，因此只需完成即可。此后，系统会提示确认并重新启动以进入开发者模式。

**注：进入开发者模式会清空用户数据，注意备份**。要跳过 OS 加载屏幕，请等待30秒，或者按 Ctrl + D，Chromebook 将继续启动。

进入开发者模式后，就可以自己下载 APK 文件进行应用安装了。

利用 crouton 安装 Ubuntu
--------------------

进入开发者模式后，还可以基于 crouton 安装 Ubuntu，操作如下：

 1. 从 https://goo.gl/fd3zc 下载最新的 crouton 版本，放到 ~/Download 目录下
 2. 打开 Chrome，键入 Ctrl + Alt + T，进入 shell 界面，输入 shell 并回车
 3. 执行： sudo sh ~/Downloads/crouton -t xfce ，也可以安装其他桌面(该过程需要连接树莓派的网络，否则会出现某个源下载不了导致失败)
 4. 安装完成之后即可进入桌面：sudo enter-chroot 进入终端，sudo startxfce4 进入图形界面
 5. 在 Chrome os 和 Ubuntu 之间进行切换：从 C 到 U，shift + ctrl + alt + 前进键，从 U 到
    C，shift + ctrl + alt + 后退键
 6. 之后就可以畅快的玩耍 Chrome os 和 Ubuntu了。

<!--more-->

xiwi 窗口化
--------

在两个系统之间切换很麻烦，所以可以使用 xiwi 来实现窗口化：

 1. 给 Chrome 安装 crouton 扩展：https://goo.gl/OVQOEt
 2. 安装 xiwi：sudo sh ~/Downloads/crouton -u -t xiwi

安装 vscode
---------

在 Chrome os 中下载 vscode 的 ubuntu 版本，默认放在 Downloads 目录下，在 Ubuntu 终端界面进行安装：

 1. sudo dpkg -i code*.deb  //下载的 vscode 的对应版本
 2. sudo apt-get install -f   //安装依赖

安装成功之后，可以通过 xiwi -T code -f . 使 vscode 窗口化。
