---
layout: post
title:  "EM::Connection在pause后将发生什么"
date:   2014-01-19 16:18:54
categories: shadowsocks
---

昨天有网友问题，EM::Connection在puase之后将发生什么，我不是太熟悉tcp协议，这里针对应用层进行一下探讨。

pause的机制其实就是停止自动receive，在普通的ruby tcp socket编程中，是没有这个api的，可以说这是EM::Connection的api。

不借助其他工具的ruby tcp编程需要手动的进行`socket.recv(4096)`，这里为一次只获取4096字节的长度。

根据这里<http://en.wikipedia.org/wiki/TCP_tuning>的描述，可以看到系统是有一个receive buffer的，这个可以由用户进行设置，当receive buffer满的时候，将会自动的停止往这个buffer写东西，这是另外一个层的东西了，得空再去深究。

所以程序这里需要知道的是，pause不会带来什么问题，也是一个好的解决办法。

之后考虑抛弃EM，实现一个纯ruby版本的Shadowsocks，想想感觉相当有趣。
