---
layout: post
title:  "Elixir版本的shadowsocks在客户端连接时总遇到refuse的问题"
date:   2014-01-24 11:08:54
categories: shadowsocks
---

记录一次犯二的debug

之前在调试elixir版本的SS的时候，发现client在连接server会有很多的refuse。调试了很久，问了几个朋友，一直无果。
今天看ruby的tcp书的时候看到这个东西，`Socket::SOMAXCONN`。如果这个值过小，会导致connect的请求被refuse。这不是我正好要找的东西么？看了一下erlang的文档，结果发现:

    {backlog, B}
    B is an integer >= 0. The backlog value defaults to 5.
    The backlog value defines the maximum length that the queue of
    pending connections may grow to.

才5啊，怪不得会refuse，改成128后果然就正常了。

基础太薄弱，记录下来，作为鞭策。
