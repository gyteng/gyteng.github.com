---

layout: post
title: "将Ubuntu作为翻墙网关"
date: 2017-04-08 00:18
comments: true

---
最近公司购买了一台服务器放办公室测试用，因此额外装个虚拟机作为网关翻墙用。

首先开启IP转发，这样本机就可作为局域网内其他设备的网关了：

修改`/etc/sysctl.conf`，增加一行

```
net.ipv4.ip_forward = 1
```
运行命令使之生效
```
sysctl -p
```

然后安装 `shadowsocks-libev` 和 `dnsmasq`，后者是用于在本地做一个无污染的dns服务器。

用 `ss-tunnel` 建立一个隧道，这是用于dns查询：

```
ss-tunnel -s "xxx.xxx.xxx.xxx" -p "xxxx" -l "5353" -m "aes-256-cfb" -k "xxxxxx" -u -L "8.8.8.8:53"
```

用 `ss-redir` 建立一个透明代理：

```
ss-redir -s "xxx.xxx.xxx.xxx" -p "xxxx" -b "0.0.0.0" -l "1080" -m "aes-256-cfb" -k "xxxxxx" -u
```

根据 [dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list) 的例子来配置 `dnsmasq`

最后增加路由表：

```
iptables -F
iptables -X
iptables -Z

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

iptables -A INPUT -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --dport 1080 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j MASQUERADE
iptables -t nat -N SS
iptables -t nat -A SS -d 127.0.0.1 -j RETURN
iptables -t nat -A SS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SS -d 172.16.0.0/21 -j RETURN
iptables -t nat -A SS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SS -d ${serverip}/32 -j RETURN

# 在此处插入chnrouters的ip

iptables -t nat -A SS -p tcp -j REDIRECT --to-port 1080
iptables -t nat -A PREROUTING -p tcp -j SS
iptables -t nat -A OUTPUT -p tcp -j SS
```