---

layout: post
title: "Shadowsocks流量限制"
date: 2014-11-08 22:35
comments: true

---
最近要把Shadowsocks共享给几个人使用，必须作一些流量限制。已经有人做好了[带流量限制功能的Shadowsocks](https://github.com/mengskysama/shadowsocks/tree/manyuser)，但这是通过改变服务端来实现的，不够简洁还要安装MySQL。查了下相关的东西发觉这需求可以用iptables来实现：

	PORT=12345

	PID_FILE=/root/user/log/$PORT.pid
	LOG_FILE=/root/user/log/$PORT.log
	BYTES_FILE=/root/user/log/$PORT.bytes
	CONF_FILE=/root/user/ss$PORT.conf

	MAX=100000000

	startSS()
	{
	    kill -9 `cat $PID_FILE`
	    iptables -D OUTPUT -s xxx.xxx.xxx.xxx -p tcp --sport $PORT
	    iptables -I OUTPUT -s xxx.xxx.xxx.xxx -p tcp --sport $PORT
	    ssserver -c $CONF_FILE > $LOG_FILE 2>&1 &
	    echo "$!" > $PID_FILE
	    echo 'Shadowsocks start.'
	    cat $CONF_FILE
	}

	startSS
	while true
	do
	    value=`iptables -n -L -v -x | grep "spt:$PORT"| awk '{print $2}'`
	    echo $value/$MAX
	    echo $value > $BYTES_FILE
	    if [ $value -gt $MAX ]; then
	        kill -9 `cat $PID_FILE`
	        echo '0' > $BYTES_FILE
	        rm $PID_FILE
	        iptables -D OUTPUT -s xxx.xxx.xxx.xxx -p tcp --sport $PORT
	    elif [ $(date +%M) -eq 1 -a  $(date +%H) -eq 3 ]; then
	        echo 'Restart.'
	        startSS
	    fi
	    sleep 50
	done

这是个简单粗暴的解决方案，先启动Shadowsocks，并记录pid，一旦检测到超流量就kill，每天凌晨三点重置流量计数器。