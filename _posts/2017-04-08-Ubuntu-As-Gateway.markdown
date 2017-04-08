---

layout: post
title: "将Ubuntu作为翻墙网关"
date: 2017-04-08 00:18
comments: true

---
最近公司购买了一台服务器放办公室测试用，因此额外装个虚拟机作为网关翻墙用。

* 开启IP转发，这样本机就可作为局域网内其他设备的网关了：

  修改`/etc/sysctl.conf`，增加一行

  ```
  net.ipv4.ip_forward = 1
  ```
  运行命令使之生效
  ```
  sysctl -p
  ```

* 安装 `shadowsocks-libev` 和 `dnsmasq`，后者是用于在本地做一个无污染的dns服务器。

* 用 `ss-tunnel` 建立一个隧道，用于dns查询：

  ```
  ss-tunnel -s "xxx.xxx.xxx.xxx" -p "xxxx" -l "5353" -m "aes-256-cfb" -k "xxxxxx" -u -L "8.8.8.8:53"
  ```

* 根据 [dnsmasq-china-list](https://github.com/felixonmars/dnsmasq-china-list) 的例子来配置 `dnsmasq`

* 增加路由表：