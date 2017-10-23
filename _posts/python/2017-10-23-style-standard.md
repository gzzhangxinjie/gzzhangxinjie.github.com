---
layout: post
title: "python 风格规范"
description: ""
category: python
tags: ['python', 'style']
---
{% include JB/setup %}

### 1. 编程风格

遵循 [PEP8](https://www.python.org/dev/peps/pep-0008/) 规则

### 2. 命名

* 变量、函数、方法、包、模块

    小写，并使用下划线分隔单词 (lower_case_with_underscores)
    
* 类、异常

    大驼峰（CapWords）

* 受保护的方法和内部函数

    单下划线开头，小写，并使用下划线分隔单词 （_single_leading_underscore）
    
* 私有函数(method)

    双下划线开头，小写，并使用下划线分隔单词 （__double_leading_underscore）
    
* 常量

    字母全部大写，并使用下划线分隔单词（ALL_CAPS_WITH_UNDERSCORES）
    
* 避免使用让人困惑的缩写

    例如，
    
    ```
    # 正确
    check_lists = ["aa", "bb", "cc"]
    
    # 错误
    c_l = ["aa", "bb", "cc"]
    ```



* 尽量不要使用只有一个字母的变量名（例如，a, b, c, I, l 等）

    *例外：在很简短的代码块中，如果变量名的意思可以从上下文中明显地看出来。如*
    

    ```
for e in elements:
    e.mutate()
    ```
    
### 3. 代码布局

#### 缩进

使用 4 个空格符

    
#### 每行的长度

不要过分在意，但不应该过长。80 ~ 100 个字符是可以接受的，尽量减少 100 个字符以上。

#### 长语句换行

编写长语句时，可以使用换行符（\）换行。在这种情况下，下一行应该与上一行的最后一个 “.” 句点或 “=” 对齐，或者是缩进 4 个空格符：

```
this_is_a_very_long(function_call, 'with many parameters') \
    .that_returns_an_object_with_an_attribute

MyModel.query.filter(MyModel.scalar > 120) \
             .order_by(MyModel.name.desc()) \
             .limit(10)

```

如果你使用括号 “()” 或花括号 “{}” 为长语句换行，那么下一行应与括号或花括号对齐：

```
this_is_a_very_long(function_call, 'with many parameters',
                    23, 42, 'and even more')
                    
```

对于元素众多的列表或元组，在第一个 “[” 或 “(” 之后马上换行：

```
items = [
    'this is the first', 'set of items', 'with more items',
    'to come in this line', 'like this'
]
```

#### 引用

把代码引用部分放在文件的顶部，按下面的顺序分成三个部分，每个部分之间空一行。 

1. 系统引用 
2. 第三方引用 
3. 项目内引用

这样就可以明确显示每个模块的引用来源。


###4. 文档

#### 函数
遵循[PEP 257](https://www.python.org/dev/peps/pep-0257/)提出的文档字符串准则。
    
* 对于功能简单的函数，可以撰写一行文档字符串。

    ```
    def add(x, y):
        """
        该函数的作用是，对传入的参数进行数值相加。
        """
        return x + y
    ```

* 对于较为复杂的函数，必须撰写多行文档字符串

    多行文档字符串应包括：
    
    * 一行摘要
    * 如果合适的话，请描述使用场景
    * 参数介绍
    * 返回的数据类型和语义
    
    例如：
    
    ```
def canonical_prefix_name(name, zone_name):
    """
    得到标准的域名前缀 (named zonefile 中的表示格式)
    如 (www.163.com 163.com) > www , (163.com, 163.com) -> @
    :param name: 域名
    :param zone_name: 域
    :return: string
    """
    ```
    
#### 模块头部

模块文件的头部包含有 utf-8 编码声明（如果模块中使用了非 ASCII 编码的字符，建议进行声明），以及标准的文档字符串。

```
# -*- coding: utf-8 -*-
"""
    简单介绍该模块的作用。
"""
```

### 引用

1. [Flask开发团队Pocoo的内部编码风格指南](http://codingpy.com/article/pocoo-internal-style-guide-cn/)

2. [Python开发指南：最佳实践精选](http://codingpy.com/article/bobp-guide-for-python-development/)

