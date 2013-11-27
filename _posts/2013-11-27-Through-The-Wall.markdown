---

layout: post
title: "穿墙"
date: 2013-11-27 22:49
comments: true

---
毕竟移动设备上配置代理不太方便，如果能在路由器这一层做就最好不过了。

首先在路由器上安装shadowsocks、redsocks和pdnsd，前两个都已经有人编译好了ar71xx版本，pdnsd源里边就有。

编辑`/etc/redsocks.conf`，注意type要选择autosocks5，这个模式可以自动在无法连接的时候调用socks代理。

	base {
		log_debug = on;
		log_info = on;
		log = stderr;
		daemon = off;
		redirector = iptables;
	}
			
	redsocks {
		local_ip = 0.0.0.0;
		local_port = 12345;
		ip = 127.0.0.1;
		port = 1080;
		type = autosocks5;
	}

接着在防火墙里添加两条，让80和443端口走redsocks。

	iptables -t nat -A PREROUTING -i br-lan -p tcp --dport 80 -j REDIRECT --to-port 12345
	iptables -t nat -A PREROUTING -i br-lan -p tcp --dport 443 -j REDIRECT --to-port 12345

这种方法并不能对付被DNS污染的域名，编辑`/etc/pdnsd.conf`以才用TCP方式查询域名：

	global {
		# debug = on;          
		perm_cache=1024;
		cache_dir="/var/pdnsd";
		run_as="nobody";
		server_port = 1053;    
		server_ip = any;
		status_ctl = on;
		query_method=tcp_only; 
		min_ttl=15m;
		max_ttl=1w;
		timeout=10;
	}

	server {
		label= "gyt";           
		ip = 8.8.8.8; 
		root_server = on;        
		uptest = none;          
	}

	source {
		owner=localhost;
		file="/etc/hosts";
	}

	rr {
		name=localhost;
		reverse=on;
		a=127.0.0.1;
		owner=localhost;
		soa=localhost,root.localhost,42,86400,900,86400,86400;
	}

在`/etc/dnsmasq.conf`里添加一行：

	conf-dir=/etc/dnsmasq.d

增加`/etc/dnsmasq.d/gfw`文件，里边加入需要以TCP方式解析的域名：

	server=/.google.com/127.0.0.1#1053
	server=/.twitter.com/127.0.0.1#1053
	server=/.facebook.com/127.0.0.1#1053
	server=/.youtube.com/127.0.0.1#1053

这下连接路由的设备无需配置就能穿墙了。