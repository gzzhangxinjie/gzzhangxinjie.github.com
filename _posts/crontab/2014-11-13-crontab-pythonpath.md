---
layout: post
title: "crontab PYTHONPATH设置"
description: ""
category: crontab
tags: []
---
{% include JB/setup %}

### 问题描述
今天写了一个 python 脚本文件，在命令行执行的时候没有问题，可以正常操作，但是放在 crontab 跑的时候报了以下错误：

{% highlight text %}
    from django.core.management import setup_environ
    ImportError: No module named django.core.management
{% endhighlight %}

### 问题原因和解决办法

#### 问题原因：

crontab 中的运行环境和用户的 shell 环境不一定是相同的。

在这个问题中，是`PYTHONPATH` 路径问题；经过确认在命令行时使用的`PYTHONPATH`是 `/home/www-data/library`(可能不同情况下使用的 PYTHONPATH 是不一样的，自己要留意下)。

#### 解决办法：

在 crontab 中添加相应的路径到 `PYTHONPATH`，例如，

{% highlight bash linenos %}
PYTHONPATH=/home/www-data/library:$PYTHONPATH
{% endhighlight %}
