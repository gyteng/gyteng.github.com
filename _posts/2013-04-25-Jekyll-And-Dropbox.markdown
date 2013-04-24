---

layout: post
title: "Dropbox和Jekyll"
date: 2013-04-25 00:15
comments: true

---
Jekyll写完每次都要打开github来push是件麻烦的事情，正好今天申请了亚马逊免费的EC2，考虑可以将push的事情交给它。

安装命令行版的Dropbox，把整个Jekyll的文件夹同步到VPS里，具体可以参考[Using Dropbox CLI](http://www.dropboxwiki.com/Using_Dropbox_CLI)。

安装incron来监控文件的变化，这个跟cron类似，执行`incrontab -e`，这里要监控的是./jekyll/_posts文件夹，类似地写上：

	/path/to/Dropbox/jekyll/_posts IN_MODIFY,IN_DELETE,IN_CLOSE_WRITE,IN_MOVE /path/to/command/

现在每当_posts文件夹被修改后就能够自动push了。