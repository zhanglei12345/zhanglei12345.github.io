---
title: 在 Telegram 收发微信消息
date: 2018-08-17 21:44:36
categories: 
- VPS
tags:
- VPS
- Telegram
---
开发代号 [EH Forwarder Bot](https://github.com/blueset/ehForwarderBot)（简称 EFB）是一个可扩展的聊天平台隧道框架，基于 Python 3。目前已内置了 Telegram 主端 (Master Channel) 和微信从端 (Slave Channel)，用来在 Telegram 收发微信消息。

本文介绍了如何在 VPS(Ubuntu 16.04) 中安装并配置 EFB、Telegram 主端和微信从端，以及如何使用 Telegram 主端来收发微信消息。自己用了将近一个月了，体验很棒，尤其是丰富便捷的表情包，可以悄悄告诉你这种方式自带消息防撤回功能，不过需要注意的是这种登录方式是基于网页微信版本的，功能受限制，登录mac版本的微信后也会把其顶掉。

## 安装 EFB

远程登录 VPS，找到合适目录执行以下步骤：

```bash
git clone https://github.com/blueset/ehForwarderBot.git
cd ehForwarderBot
mkdir storage
chmod +rw ./storage
```

安装依赖:

```bash
sudo apt-get install python3-dev python3-setuptools
sudo apt-get install libwebp-dev
sudo apt-get install libmagic-dev ffmpeg
```

安装 python 依赖:

`pip3 install -r requirements.txt`

<!--more-->

## 创建 Telegram Bot

要创建一个新的 Bot，要先向 @BotFather 发起会话。发送指令 /newbot 以启动向导。期间，你需要指定这个 Bot 的名称与用户名（用户名必须以 bot 结尾）。完毕之后 @BotFather 会提供给你一个密钥（Token）。

接下来还要对刚刚启用的 Bot 进行进一步的配置：允许 Bot 读取非指令信息、允许将 Bot 添加进群组、以及提供指令列表。

* 发送 /setprivacy 到 @BotFather，选择刚刚创建好的 Bot 用户名，然后选择 “Disable”。
* 发送 /setjoingroups 到 @BotFather，选择刚刚创建好的 Bot 用户名，然后选择 “Enable”。
* 发送 /setcommands 到 @BotFather，选择刚刚创建好的 Bot 用户名，然后发送如下内容：

```
link - 将会话绑定到 Telegram 群组
chat - 生成会话头
recog - 回复语音消息以进行识别
extra - 获取更多功能
```

通过bot获取你自己的 Telegram ID：

`@get_id_bot 发送 /start`

## 配置 EFB

复制并编辑配置文件:

```bash
cp config.sample.py config.py
vi config.py
```

在配置文件中，token 后引号里面的内容替换为你之前获得的 Bot 密钥，admins 后方括号里面填入你自己的 Telegram ID。

```python
master_channel = 'plugins.eh_telegram_master', 'TelegramChannel'
slave_channels = [('plugins.eh_wechat_slave', 'WeChatChannel')]

eh_telegram_master = {
    "token": "***:***",
    "admins": [***]
}
```

## 启动 EFB

`python3 daemon.py start`

之后可看见二维码，用微信扫描登录即可。

## 使用 EFB Telegram 主端

现在，在 Telegram 里面搜索你之前指定的 Bot 用户名，点击 Start（开始）即可开始与微信互通消息了。

在最初，所有来自微信的消息都会通过 Bot 直接发送给你，要回复其中的任意一条消息，你需要在 Telegram 中选中那条消息，选择 Reply（回复），再输入消息内容。

如果需要向新联系人发送消息，只需发送 /chat 指令，选择一个会话。之后这条消息就会变成一个「会话头」，回复这条消息就可以向指定的联系人或群组发送消息。

当消息过多时，来自不同会话的消息会使 Telegram 上面的会话混乱不堪。EFB 支持将来自指定会话的消息分流到一个 Telegram 群组中。

* 在 Telegram 中新建一个空群组，并将你的 Bot 加入到这个群组中。
* 回到 Bot 会话，发送 /link，选择一个会话，并点击 “Link”。
* 在弹出的列表中选择刚刚创建的空群组即可。

利用这种方式可以实现等同于微信里面的一对一对话。

