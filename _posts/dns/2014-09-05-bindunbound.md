---
layout: post
title: "bind和unbound简单性能对比"
description: ""
category: dns
tags: ['dns', 'bind', 'unbound']
---
{% include JB/setup %}
## unbound 和 bind 性能测试

### 测试环境

unbound 机器配置：

* 内存 32GB
* CPU Intel(R) Xeon(R) CPU E5-2650 0 @ 2.00GHz 32核
* debian 7, 3.2.0内核, 32bit
* unbound 1.4.17

bind 机器配置：

* 内存 32GB
* CPU Intel(R) Xeon(R) CPU E5-2650 0 @ 2.00GHz 32核
* debian 7, 3.2.0内核, 32bit
* bind 9.8.4

resperf 机器配置：

* 内存 32GB
* CPU Intel(R) Xeon(R) CPU E5-2650 0 @ 2.00GHz 32核
* debian 7, 3.2.0内核, 32bit
* resperf 2.0.0.0
* 测试文件的内容是10，000，000行数据，每一行的内容为"www.example.com A"。测试前，先对缓存服务器查询'www.example.com'，让缓存服务器上有缓存记录。(我真正测试的时候，不是使用的这个域名，只是写在博客上，不方便写上真正的域名。)


### 测试结果

测试分成两种情况：

1. 关闭 querylog。

2. 开启 querylog。

为了让实验结果更有效，每种情况测试 5 次，取平均值，接下来的数据都采用平均值; CPU 使用率通过 top 命令监控获得。


####关闭 querylog

| 类型| cpu high(%)| cpu average(%) | QPS | thread | lost(%)|
|----|:-----:|:------:| :----:| :------:| ----:|
| unbound|  620| 364.69 | 33207.6 |  32| 0.68|
| bind| 669 | 350.21 |22111.2|32|0.708

####开启 querylog

| 类型| cpu high(%)| cpu average(%) | QPS | thread | lost(%)|
|----|:-----:| :------:| :----:| :------:| :----:|
| unbound| 644 | 390.34 |  30416.4|  32| 0|
| bind| 223 | 172.56 |5370.8|32|3.684


1. 可以看出，unbound 的性能确实比 bind 要高。
2. 在测试过程中，发现 unbound 在开启 querylog 的时候，会有些性能不稳定（就是相同的配置测试出来的结果有比较大的误差）。尽管如此，unbound 就算是在较差的情形下，性能还是比 bind 要高。
