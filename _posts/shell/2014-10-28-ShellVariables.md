---
layout: post
title: "shell 变量作用域"
description: ""
category: shell
tags: [shell]
---
{% include JB/setup %}

今天在用 bash 的时候，碰到一个变量作用域相关的问题，简单记录下。

###代码段1

{% highlight bash linenos %}
#!/bin/bash
declare -a myarray
myarray=(0)

cat testfile.txt | while read line
do
    myarray=(${myarray[@]} $line)
done

echo ${myarray[@]}
{% endhighlight %}

代码的本意是将`testfile.txt`文件中的内容读取并放到`myarray`数组中，然后显示`myarray`的内容。其中，`testfile.txt`的内容为：

~~~
1
2
3
4
5
~~~

但是，最后运行结果是：

~~~
0
~~~

也就是说，`testfile.txt` 文件里面的内容并没有按照预想的那样，将所有的内容放到 `myarray` 数组中。

###代码段2

{% highlight bash linenos %}
#!/bin/bash
declare -a myarray
myarray=(0)

while read line
do
    myarray=(${myarray[@]} $line)
done < testfile.txt

echo ${myarray[@]}
{% endhighlight %}

`testfile.txt`的内容也是

~~~
1
2
3
4
5
~~~

最后代码的执行结果为：

~~~
0 1 2 3 4 5
~~~

这段代码中，`testfile.txt` 文件的内容都被放到 `myarray` 数组中, 与预想中的一样。

### 原因

代码段1 和 代码段2的区别在于`管道`和`重定向`。

在第一段代码中，bash 遇到管道之后会创建一个新的进程，于是在 `while` 循环里面的 `myarray` 是 subshell 的局域变量，subshell 对变量的修改并不会改变原来 shell 中变量的值。

在第二段代码中，bash 遇到重定向并不会创建一个新的进程，因此可以顺利改变 `while` 循环中变量的值。

正是这个原因，使得代码1和代码2的运行结果不一样。
