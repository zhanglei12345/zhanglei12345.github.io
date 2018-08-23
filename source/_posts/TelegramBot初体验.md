---
title: Telegram Bot 初体验
date: 2018-08-16 21:44:36
categories: 
- VPS
tags:
- Telegram
---
> Telegram 是一个专注于速度和安全性的 IM 应用，它超级快速，简单和免费。 
> 您可以同时在您所有的设备上使用 Telegram - 您的消息可以在任意数量的手机，平板电脑或计算机之间无缝同步。

这篇文章主要介绍如何开发一个简单 telegram 机器人，我的机器人@happy_mybot。

<!--more-->

## 创建 Telegram Bot

[bots](https://core.telegram.org/bots) 这是官网对 bot 的介绍。

要创建一个新的 Bot，要先向 @BotFather 发起会话。发送指令 /newbot 以启动向导。期间，你需要指定这个 Bot 的名称与用户名（用户名必须以 bot 结尾）。完毕之后 @BotFather 会提供给你一个密钥（Token）。

接下来还要对刚刚启用的 Bot 进行进一步的配置：允许 Bot 读取非指令信息、允许将 Bot 添加进群组、以及提供指令列表。

发送 /setprivacy 到 @BotFather，选择刚刚创建好的 Bot 用户名，然后选择 “Disable”。
发送 /setjoingroups 到 @BotFather，选择刚刚创建好的 Bot 用户名，然后选择 “Enable”。
发送 /setcommands 到 @BotFather，选择刚刚创建好的 Bot 用户名，然后发送如下内容：
```
start - say hello
songci	- Get songci
tangshi - Get tangshi
```

## 使用 python-telegram-bot 库

github 上 [python-telegram-bot](https://github.com/python-telegram-bot/python-telegram-bot) 库是对官方 bot api 的一个封装，使开发更加地快速。跟着该库的介绍进行安装。

telegram支持两种获取消息的方式，Polling 和 [Webhook](https://github.com/python-telegram-bot/python-telegram-bot/wiki/Webhooks)。我使用的简单点的 Polling。
>You should have a good reason to switch from polling to a webhook. Don't do it simply because it sounds cool.

具体代码可看我托管在github上的[happybot](https://github.com/zhanglei12345/happybot)。

