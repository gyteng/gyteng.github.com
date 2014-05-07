---

layout: post
title: "Chrome Webstore"
date: 2012-05-15 10:06
comments: true

---
Chrome应用商店里有若干个倒计时工具，都不能满足我的要求，有一个稍微贴近我需求的扩展，可是图标太难看，一直让它呆在地址栏右侧感觉十分不舒服。需要一个每隔若干分钟弹出提醒的功能，还要在当前标签页中间给个提醒，强制我去休息一下，而且不带开关，一启动Chrome就开始计时。看了一下官方的[开发文档](https://developers.google.com/chrome/web-store/docs/get_started_simple?hl=zh-CN)觉得不太复杂，还有各种例子的代码，即使不会javascript也能搞定，便打算自己做了一个。

搞不懂为什么上传到扩展中心还需要一次性支付[开发人员注册费](http://support.google.com/chrome_webstore/bin/answer.py?hl=zh-Hans&answer=187591&topic=1212368&ctx=topic)，虽然只要$5可我哪来Google电子钱包呢？

另外，Chrome__导入开发中的扩展__之后，在运行当中修改了扩展的文件再重新载入会带来一系列奇怪的问题，特别是调用了诸多browserAction时。
