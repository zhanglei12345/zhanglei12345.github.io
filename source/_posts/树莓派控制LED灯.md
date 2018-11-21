---
title: 树莓派控制 LED 灯
date: 2018-08-15 21:44:36
categories: 
- 树莓派
tags: 
- 树莓派 
- LED
---
通过树莓派实现 5个 led 灯的循环亮灯，目前设置的亮灯间隔为 0.5s，亮灯循环 10次。

## 配件

200Ω 的电阻 5个、发光二极管 LED 5个、面包板、杜邦线

## 原理

![](https://ws3.sinaimg.cn/large/006tKfTcly1fjlt2uriifj30dx06rmxi.jpg)

当我们使用 GPIO 引脚作为输出时，Raspberry Pi 将替换上图中的开关和电池。每个引脚可以打开或关闭，或者在计算方面变为 HIGH 或 LOW。当引脚为高电平时，它输出 3.3 伏（3v3）; 当引脚为低电平时，它是关闭的。

<!--more-->

## 接线方式

树莓派Ground引脚接面包板的 -。

LED正极(长脚)接树莓派的 GPIO 引脚，负极接电阻再接 -。

![](https://ws2.sinaimg.cn/large/006tKfTcly1fjlt39xn7jj31kw1q24or.jpg)

## 控制程序(Python)

```python
import RPi.GPIO as GPIO
import time

channels = [13,21,16,20,26]

def setup_GPIO():
    GPIO.setmode(GPIO.BCM)
    GPIO.setwarnings(False)
    GPIO.setup(channels, GPIO.OUT)

def on_led(i):
    GPIO.output(i, GPIO.HIGH)

def off_led(i):
    GPIO.output(i, GPIO.LOW)

def control_led():
    while True:
        try:
            for i in channels:
                on_led(i)
                time.sleep(0.5)
                off_led(i)
        except (KeyboardInterrupt,Exception):
            break

if __name__ == '__main__':
    setup_GPIO()
    control_led()
    # Ctrl + C 终止一次之后，发现下次程序再运行的时候，上次终止时的那个LED灯也是亮的，所以在终止之后加了全部设置为LOW
    for i in channels:
        off_led(i)
    GPIO.cleanup()
```

### RPi.GPIO Installation

[官网](https://pypi.python.org/pypi/RPi.GPIO)

The RPi.GPIO module is installed by default in Raspbian. To make sure that it is at the latest version:

```bash
$ sudo apt-get update
$ sudo apt-get install python-rpi.gpio python3-rpi.gpio
```

It is recommended that you install RPi.GPIO using the pip utility as superuser (root):

`sudo pip install RPi.GPIO`

### 使用

导入:

`import RPi.GPIO as GPIO`

编写 IO 引脚的方式

设置:

`GPIO.setmode(GPIO.BOARD)` 或 `GPIO.setmode(GPIO.BCM)`
> BCM 方式是按照 GPIO 引脚编号，BOARD 方式是按照物理编号

获取:

`mode = GPIO.getmode()`
> The mode will be GPIO.BOARD, GPIO.BCM or None

警告:

GPIO 可能有多个脚本/电路，如果 RPi.GPIO 检测到引脚已配置为默认值（输入）以外的其他值，则在尝试配置脚本时会发出警告。要禁用这些警告：

`GPIO.setwarnings(False)`

设置一个 channel：

```python
# input
GPIO.setup(channel, GPIO.IN)
# output
GPIO.setup(channel, GPIO.OUT)
# initial
GPIO.setup(channel, GPIO.OUT, initial=GPIO.HIGH)
#release 0.5.8 onwards
chan_list = [11,12] 	#   chan_list = (11,12)
GPIO.setup(chan_list, GPIO.OUT)
```

读取引脚的值：

`GPIO.input(channel)`
> This will return either 0 / GPIO.LOW / False or 1 / GPIO.HIGH / True.

设置引脚输出的的状态：

```python
GPIO.output(channel, state)
#release 0.5.8 onwards
chan_list = [11,12] 
GPIO.output(chan_list, GPIO.LOW)
GPIO.output(chan_list, (GPIO.HIGH, GPIO.LOW))
```
> State can be 0 / GPIO.LOW / False or 1 / GPIO.HIGH / True.

在脚本结束的时候做清理：

`GPIO.cleanup()`