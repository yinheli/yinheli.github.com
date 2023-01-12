---
draft: false
date: 2015-02-11
title: iptables 笔记
tags: 
  - iptables
---

看看服务器都已经应用了哪些规则

```
iptables -L -n
iptables -Z
```

清理掉全部的规则

```
iptables -F
iptables -X
```

定义自己的规则

> 特别小心, 如果没有开放 ssh 端口就把 INPUT 的全部 DROP, shell 就掉了.
> 确保有一个 ssh 22 端口是开放的.
> 另一个保险是确保 iptables 不是启动运行, 这样还可以通过管理后台执行服务器重启来挽救. 否则.... 你的懂的

```
iptables -I INPUT 1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# 各个链的规则
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# 允许 ping
iptables -A INPUT -p icmp -j ACCEPT
# 允许回环地址
iptables -A INPUT -i lo -j ACCEPT

# ssh 端口记录日志
iptables -A INPUT 1 -p tcp --dport 22 -m limit --limit 3/minute --limit-burst 8 -j LOG --log-prefix ssh-burst:

# 10s 内, 超过5次请求, 抛弃
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 10 --hitcount 5 -j DROP
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 10 --hitcount 5 -j LOG --log-prefix ssh-conn-max-time:

iptables -A INPUT -p tcp --dport 25000 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 25000 -m state --state NEW -m recent --update --seconds 10 --hitcount 10 -j DROP
iptables -A INPUT -p tcp --dport 25000 -m state --state NEW -m recent --update --seconds 10 --hitcount 10 -j LOG --log-prefix app-conn-max-time:

# 限制某些端口的最大连接数
iptables -A INPUT -p tcp --syn --dport 22 -m connlimit --connlimit-above 5  -j DROP
iptables -A INPUT -p tcp --syn --dport 25000 -m connlimit --connlimit-above 10 -j DROP

# 开放的端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 25000 -j ACCEPT
iptables -A INPUT -p tcp --dport 20000 -j ACCEPT
iptables -A INPUT -p tcp --dport 9100 -j ACCEPT

# 这条规则用在INPUT链默没有DROP的情况, 作用与-P DROP相同, 当前面所有的规则都没匹配时, 自然落到这个 REJECT 上.
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
iptables -A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

删除某条记录

方法 1, 把配置的那个 `A` 换成 `D`

`iptables -L -n --line-numbers` 使用 `--line-numbers` 打印行, 然后删除行
例如: `iptables -D INPUT 2`

保存所有改动

```
/etc/init.d/iptables save
```

<!--more-->

```
# 完整应用脚本, 用于快速初始化服务器 iptables

iptables -L -n

iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

iptables -F
iptables -X
iptables -Z

iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

iptables -A INPUT 1 -p tcp --dport 22 -m limit --limit 3/minute --limit-burst 8 -j LOG --log-prefix ssh-burst:

iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 10 --hitcount 5 -j DROP
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 10 --hitcount 5 -j LOG --log-prefix ssh-conn-max-times:

iptables -A INPUT -p tcp --syn --dport 22 -m connlimit --connlimit-above 30  -j DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT


iptables -A INPUT -p tcp --dport 25000 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 25000 -m state --state NEW -m recent --update --seconds 10 --hitcount 10 -j DROP
iptables -A INPUT -p tcp --dport 25000 -m state --state NEW -m recent --update --seconds 10 --hitcount 10 -j LOG --log-prefix app-conn-max-time:

iptables -A INPUT -p tcp --syn --dport 25000 -m connlimit --connlimit-above 10 -j DROP
iptables -A INPUT -p tcp --dport 25000 -j ACCEPT

iptables -A INPUT -p tcp --syn --dport 80 -m connlimit --connlimit-above 60 -j DROP
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

iptables -A INPUT -p tcp --syn --dport 443 -m connlimit --connlimit-above 60 -j DROP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

iptables -A INPUT -p tcp --dport 20000 -j ACCEPT
iptables -A INPUT -p tcp --dport 13307 -j ACCEPT

iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
iptables -A FORWARD -j REJECT --reject-with icmp-host-prohibited

iptables -L -n

/etc/init.d/iptables save
```
