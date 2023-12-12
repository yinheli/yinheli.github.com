---
title: 远程初始化树莓派
description: 通过远程 WiFi 方式初始化树莓派
date: 2017-09-06
tags:
  - Raspberry
draft: false
---

没有键盘，显示器，只有 wifi 网络的情况下，启用 树莓派, 官方下载树莓派镜像，使用 Etcher 刷入 sd 卡

在 sd 卡根目录创建空白 ssh 文件，标记启用 ssh

找个 linux 服务器，挂载 sd 卡的 ext 分区，修改：`vi /etc/network/interfaces`

```
// ...

// 添加
allow-hotplug wlan0
iface wlan0 inet dhcp
    wpa-ssid "joy"
    wpa-psk "xxxx" // 密码（明文）
```

`vi /etc/wpa_supplicant/wpa_supplicant.conf`

```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="joy"
    psk="xxxx"
}
```

配置 wifi 重连

```
cp /etc/ifplugd/action.d/ifupdown /etc/ifplugd/action.d/ifupdown.bak
cp /etc/wpa_supplicant/ifupdown.sh /etc/ifplugd/action.d/ifupdown
```
