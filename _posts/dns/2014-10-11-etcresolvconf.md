---
layout: post
title: "/etc/resolv.conf配置"
description: ""
category: dns
tags: [dns, /etc/resolv.conf]
---
{% include JB/setup %}

`/etc/resolv.conf` 用于配置 DNS 信息，用户可以在这个文件中配置 nameserver 信息以及一些 options。

`/etc/resolv.conf` 文件中最多可以指定 3 个 nameserver，默认情况下，系统首先会向第一个 nameserver 查询，查询超时之后向第二个 nameserver 查询，如果还是超时，则继续向下一个 nameserver 查询。当查询次数超过限制时，返回错误。


`/etc/resolv.conf` 可以设置 options：

1. options timeout:n
    
    设置查询超时时间为 n 秒，默认是5秒，最大值是30秒。

2. options rotate

    轮询 `/etc/resolv.conf` 中定义的 nameserver，而不是每次都从第一个 nameserver 开始查询。经过测试(测试程序和测试结果就不在这里说了)，如果一段程序只查询一个域名，多次执行执行这一段程序，它选中的 nameserver 是一样的，并不会轮询；而如果再程序中连续查询多次，它就会轮询 `/etc/resolv.conf`中的 nameserver。
    
3. options attempts:n

    尝试次数，当向每一个 nameserver 查询的次数到达 n 次并且没有得到结果时，返回错误。默认值是 2，最大值是 5。
    
### 影响范围

`/etc/resolv.h` 配置的 DNS 信息会影响到所有使用 resolv.h 库的程序。  
