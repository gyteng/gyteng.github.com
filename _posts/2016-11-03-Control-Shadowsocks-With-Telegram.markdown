---

layout: post
title: "用Telegram控制Shadowsocks"
date: 2016-11-03 21:13
comments: true

---
最近花了好多时间重写[这个项目](https://github.com/shadowsocks/shadowsocks-manager)，带来插件机制，更方便扩展功能，同时也不仅限于webgui，因此不想用网页来管理的人还可以用telegram的bot来管理。

具体步骤如下：

安装`shadowsocks-manager`，这个项目需要`Node.js 6.*`的版本：

```
npm i -g shadowsocks-manager
```

申请telegram的bot，先跟[BotFather](https://telegram.me/BotFather)交谈，输入`/newbot`它就会创建一个，同时给了一个token，类似于：

```
Use this token to access the HTTP API:
172476948:AQItZe7PuRpq_rZqvlkEdx049oEJZV5KK9f
```

创建`~/.ssmgr/tg.yml`文件，把刚刚的token填进去，内容如下：

```
type: m
empty: false
shadowsocks:
  address: 127.0.0.1:6001
manager:
  address: 127.0.0.1:6002
  password: '123456'
plugins:
  telegram:
    use: true
    token: '172476948:AQItZe7PuRpq_rZqvlkEdx049oEJZV5KK9f'
  flowSaver:
    use: true
db: 'tg.sqlite'
```

运行`shadowsocks`，后面增加参数`--manager-address=127.0.0.1:6001`

运行`ssmgr`：

```
ssmgr -s 127.0.0.1:6001 -m 127.0.0.1:6002
```

运行另一个`ssmgr`：

```
ssmgr -c tg.yml
```

然后用自己的telegram跟之前创建的bot交谈，先输入`auth`成为管理员，然后就能通过它来控制shadowsocks的端口新增、删除、密码修改、流量统计等事情了：

* `add (端口号) (密码)` 添加一个端口并设置密码
* `del (端口号)` 删除某个端口
* `list` 查看已添加的端口
* `flow2hour` 查看2小时内产生的流量

更多功能就不一一列举了，可输入`help`查看具体的命令和参数，文档请[参见这里](https://github.com/shadowsocks/shadowsocks-manager/blob/master/plugins/telegram/README.md)。

![Telegram01](/media/pic/telegram01.png)

![Telegram02](/media/pic/telegram02.png)
