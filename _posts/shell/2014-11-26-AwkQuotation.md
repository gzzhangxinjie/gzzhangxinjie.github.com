---
layout: post
title: "Awk 输出单引号和双引号"
description: ""
category: shell
tags: ['awk']
---
{% include JB/setup %}

今天刚好有个需求，需要将一个文件内容的某个字段提取出来，然后根据这个字段的内容生成 mysql 的 delete 语句。

文件内容的格式为, 文件名为file1：

{% highlight text%}
aa keyword1
bb keyword2
cc keyword3
dd keyword4
{% endhighlight %}

需要生成的格式为:

{% highlight bash %}
delete from table1 where column1='keyword1';
delete from table1 where column1='keyword2';
delete from table1 where column1='keyword3';
delete from table1 where column1='keyword4';
{% endhighlight %}

awk 可以轻松完成这个工作，但是就是要在单引号的使用上进行留意。比如，使用下面的语句就可以完成上面的工作。

{% highlight bash %}
cat file1 | awk '{print "delete from table1 where column1='\''"$2"'\'';"}'
{% endhighlight %}

### 输出单引号
实际上，输入单引号的语句为：

{% highlight bash %}
cat file1 | awk '{print "'\''"}'
cat file1 | awk 单引号{print 双引号 单引号\单引号 单引号 双引号} 单引号
{% endhighlight %}

如果在`\'`左右两边不加单引号，则会继续等待输入；所以需要变成`'\''`从而让这个单引号不生效。

http://www.cnblogs.com/rootq/articles/1417138.html 这篇文章讨论了为什么这样输出单引号。

输出单引号的第二种方法：
{% highlight bash %}
cat file1 | awk "{print \"'\"}"
{% endhighlight %}

输出单引号的第三种方法：
{% highlight bash %}
cat file1 | awk '{print "\x27"}'
{% endhighlight %}


### 输出双引号

相比之下，输出双引号就容易很多了，例如：

{% highlight bash %}
cat file1 | awk '{print "\""}'
cat file1 | awk 单引号{print 双引号\双引号 双引号} 单引号
{% endhighlight %}

