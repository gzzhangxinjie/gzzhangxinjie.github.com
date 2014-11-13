---
layout: post
title: "crontab PYTHONPATH设置"
description: ""
category: crontab
tags: []
---
{% include JB/setup %}

今天写了一个 python 脚本文件，在命令行执行的时候没有问题，可以正常操作，但是放在 crontab 跑的时候报了一下错误：

{% highlight text %}
    from django.core.management import setup_environ
    ImportError: No module named django.core.management
{% endhighlihgt %}

### 问题原因和解决办法

#### 问题原因：

crontab 中的运行环境和用户的 shell 环境不一定是相同的。

在这个问题中，是`PYTHONPATH` 路径问题；经过确认在命令行时的`PYTHONPATH`是 `/home/www-data/library`。

#### 解决办法：

在 crontab 中添加 `PYTHONPATH`，例如，

{% highlight bash linenos %}
PYTHONPATH=/home/www-data/library
{% endhighlight %}

在实际解决问题的过程中，一定确认 `PYTHONPATH` 的路径。
