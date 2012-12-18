---

layout: post
title: "Client Mode Wireless"
date: 2012-11-13 19:06
comments: true

---
根据[这里](http://wiki.openwrt.org/doc/howto/clientmode)的方法可以用三台路由器覆盖全班8个宿舍，所有人都在同一网段，无缝打DotA。

主路由器的`/etc/config/wireless`里增加一行：

	option wds '1'

其它路由的`/etc/config/wireless`里新增一个interface，ssid和key要保持一致：

	config wifi-iface
	    option device 'radio0'
	    option ssid 'OpenWrt'
	    option mode 'sta'
	    option wds '1'
	    option network 'lan'
	    option encryption 'psk'
	    option key '********'

只保留一个DHCP服务器以免冲突。这种模式下有一些小bug，如果sta模式的那个无线断了，会导致另一个也不能连接，无解。

>真正的好男人并不是不玩游戏，不打DotA不打WOW的。而是在他玩游戏的时候，只要你一个短信，一个电话或一个QQ，他就会为你直接退出游戏。这种人俗称“猪一样的队友”，千万别和他组队。

噗，原来我是__猪一样的队友__。