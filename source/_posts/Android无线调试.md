---
title: Android无线调试
date: 2018-01-13 20:41:48
tags:
- Android
- 无线调试
layout:
updated:
comments: true
categories:
- Android调试技巧
permalink:
---
{% asset_img ANDROID_WIFI_DEBUG.jpg %}

> 版权声明：本文为 *冬夏* 原创文章，可以随意转载，但请注明出处。

正常来说，我们在调试 Android 应用的时候都需要用 USB 数据线将 Android 设备和电脑连接。但是有的时候我们的应用需要通过 USB 来连接外设，或者担心 USB 接口松动，这个时候就可以使用使用 Android 无线调试功能。

<!--more-->

# 前提条件 #

* 手机打开 USB 调试功能
  1. 找到 手机 -> 设置 -> 全部参数（关于手机） -> 连续点击版本号5次即可进入开发者模式（不同设备位置有可能有细微差别）
  2. 找到 手机 -> 设置 -> 开发者选项 -> 打开 USB 调试功能

  {% asset_img USB_DEBUG.png %}

* 添加 adb 到环境变量
  - windows

    1. 我的电脑右键 -> 属性 -> 高级 -> 环境变量

  - linux

    1. 在命令行终端输入 gedit ~/.bashrc
    2. 在文本最后添加： PATH=[Android Sdk Root]/platform-tools:$PATH，并保存退出，其中，[Android Sdk Root]是你Android Sdk 的目录
    3. 在命令行终端输入 source ~/.bashrc, 使环境变量生效（设置环境变量方法有多种，这里只是其中一种）

* 检查 adb 版本大于 v1.0.25
  - 在命令行终端 输入 adb --version 即可查看版本号，或者直接输入 adb，第一行即为版本号

* 查看设备IP
- 将 Android 设备连接上 WIFI，并且和电脑处于同一路由下
- 点击 设置 -> 关于手机 -> 状态信息 -> 找到设备IP

# 具体步骤 #

1. 将 Android 手机用数据线连接到电脑（必须）

2. 在命令行终端输入 adb tcpip <port>，其中 <port> 为端口号，可以任意指定，如：adb tcpip 5555

3. 在命令行终端输入 adb connect <host>:<port>即可连接上 Android 手机，其中 <host>为 Android 手机ip，<port> 为端口号，adb connect 192.168.1.101:5555

4. 当要断开连接时，在命令行终端输入adb disconnect <host>:<port>，如 adb disconnect 192.168.1.101:5555

{% asset_img ANDROID_DEBUG.png %}

# 插件 #

如果觉得命令行的方式太麻烦，可以使用插件版本

1. 将 Android 手机用数据线连接到电脑（必须）

2. AndroidStudio -> Settings -> Plugin -> Brouse repositories -> ADB WIFI 安装插件（重启生效）

3. AndroidStudio -> Tools -> Android -> ADB WIFI -> ADB USB to WIFI

{% asset_img ADB_WIFI.png %}

参考资料：

[1] <http://blog.csdn.net/daditao/article/details/19077281>

[2] <http://www.linuxidc.com/Linux/2015-08/121192.htm>

[3] <http://blog.csdn.net/zxw2844/article/details/8560052>

[4]<http://blog.csdn.net/github_2011/article/details/70738203>
