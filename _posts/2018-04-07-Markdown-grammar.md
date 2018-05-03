---
title: Markdown的常用语法
description: 关于Markdown的常用语法以及如何使用.
categories:
 - Markdown
 - Jekyll	
tags: [Markdown, Jekyll]
---

# 简介

Markdown 是一种轻量级标记语言，它用简洁的语法代替排版，使我们专心于码字。它的目标是实现易读易写，成为一种适用于网络的书写语言。同时，Markdown支持嵌入html标签。

**注意**：Markdown使用`#、+、*`等符号来标记， 符号后面必须跟上 至少1个 空格才有效！

# Markdown的常用语法

## 标题

markdown 标题支持两种形式：

### 用#标记

在 标题开头 加上1~6个#，依次代表一级标题、二级标题....六级标题

```yml
# 一级标题
## 二级标题
### 三级标题
#### 四级标题
##### 五级标题
###### 六级标题
```

### 用=和-标记

在 标题底下 加上任意个=代表一级标题，-代表二级标题  
一级标题  
`======`  
二级标题  
`----------`  

## 列表
Markdown 支持有序列表和无序列表。
### 无序列表使用`-、+`和`*`作为列表标记：

```yml
- Red
- Green
- Blue

* Red
* Green
* Blue

+ Red
+ Green
+ Blue
```

效果如下：
- Red
- Green
- Blue

### 有序列表则使用数字加英文句点.来表示：

```yml
1. Red
2. Green
3. Blue
```

效果如下：
1. Red
2. Green
3. Blue

## 引用
引用以>来表示，引用中支持多级引用、标题、列表、代码块、分割线等常规语法。  
**常见的引用写法**：
```yml
> 这是一段引用    //在`>`后面有 1 个空格
> 
> 这是引用的代码块形式    //在`>`后面有 5 个空格
>     
> 代码例子：
>   
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
```

```yml	
> 一级引用
> > 二级引用
> > > 三级引用
```	

```yml
> #### 这是一个四级标题
> 
> 1. 这是第一行列表项
> 2. 这是第二行列表项
```

效果如下：
> 这是一段引用    //在`>`后面有 1 个空格
> 
> 这是引用的代码块形式    //在`>`后面有 5 个空格
>     
> 代码例子：
>   
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

> 一级引用
> > 二级引用
> > > 三级引用

> #### 这是一个四级标题
> 
> 1. 这是第一行列表项
> 2. 这是第二行列表项

## 强调

两个`*`或`-`代表**加粗**，一个`*`或`-`代表**斜体**，`~~`代表**删除**。

```yml
**加粗文本** 或者 __加粗文本__

*斜体文本*  或者_斜体文本_

~~删除文本~~
```

效果如下：

**加粗文本** 或者 __加粗文本__

*斜体文本*  或者_斜体文本_

~~删除文本~~

## 图片与链接
图片与链接的语法很像，区别在一个` ! `号。二者格式：

```yml
图片：![]()    ![图片文本(可忽略)](图片地址)  
链接：[]()     [链接文本](链接地址)
链接又分为行内式、参考式和 自动链接：

这是行内式链接：[Pu1satilla's Blog](http://Pu1satilla.github.io)。

这是参考式链接：[Pu1satilla's Blog][url]，其中url为链接标记，可置于文中任意位置。

[url]: http://Pu1satilla.github.io/ "Pu1satilla's Blog"

链接标记格式为：[链接标记文本]:  链接地址  链接title(可忽略)

这是自动链接：直接使用`<>`括起来<http://Pu1satilla.github.io>

这是图片：![][avatar]

[avatar]: https://Pu1satilla.github.io/images/avatar.jpg
```
**图片**: ![图片文本(可忽略)](/assets/images/avatar.gif)

**链接**: []()     [链接文本](链接地址)

链接又分为`行内式`、`参考式`和 `自动链接`：

这是行内式链接：[Pu1satilla's Blog](http://Pu1satilla.github.io)。

这是参考式链接：[Pu1satilla's Blog][url]，其中url为链接标记，可置于文中任意位置。

[url]: http://Pu1satilla.github.io/ "Pu1satilla's Blog"

链接标记格式为：[链接标记文本]:  链接地址  链接title(可忽略)

这是自动链接：直接使用`<>`括起来<http://Pu1satilla.github.io>

这是图片：![][avatar]

[avatar]: /assets/images/avatar.gif


## 代码
代码分为行内代码和代码块。

行内代码使用 `代码` 标识，可嵌入文字中

代码块使用4个空格或```标识

```
这里是代码
```

代码语法高亮在 ```后面加上空格和语言名称即可

``` 语言
//注意语言前面有空格
这里是代码
```

例如：
```
这是行内代码`onCreate(Bundle savedInstanceState)`的例子。

>```
>protected void onCreate(Bundle savedInstanceState) {
>    super.onCreate(savedInstanceState);
>   setContentView(R.layout.activity_main);
>}
>```

这是代码块和语法高亮：

>``` java
>// 注意java前面有空格
>protected void onCreate(Bundle savedInstanceState) {
>    super.onCreate(savedInstanceState);
>   setContentView(R.layout.activity_main);
>}
>```
```

效果如下：  
这是行内代码onCreate(Bundle savedInstanceState)的例子:
```
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
}
```

这是代码块和语法高亮：
```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
}
```

```yml
***
---
___

* * *
```
效果均为一条分割线：  
* * *


## 表格
### 常用方式：

```
教程标题| 主要内容
-------|----------
关于Markdown | 简介Markdown，Markdown的优缺点
Markdown基础 | Markdown的**基本语法**，格式化文本、代码、列表、链接和图片、分割线、转义符等
Markdown表格和公式 | Markdown的**扩展语法**，表格、公式
```

效果如下：

教程标题| 主要内容
-------|----------
关于Markdown | 简介Markdown，Markdown的优缺点
Markdown基础 | Markdown的**基本语法**，格式化文本、代码、列表、链接和图片、分割线、转义符等
Markdown表格和公式 | Markdown的**扩展语法**，表格、公式

### 表格的对齐方式
```
| 姓名     | 年龄| 性别|
|:--------|---------:|:-------:|
| 张三| 18| 女      |
| 小明| 23| 男      |
```

效果如下：

| 姓名     | 年龄| 性别|
|:--------|---------:|:-------:|
| 张三| 18| 女      |
| 小明| 23| 男      |

## 换行
在行尾添加两个空格加回车表示换行：

```yml
这是一行后面加两个空格  换行
```
效果如下：

***
这是一行后面加两个空格  
换行  

***

## 常用弥补Markdown的Html标签
```yml
字体
<font face="微软雅黑" color="red" size="6">字体及字体颜色和大小</font>
<font color="#0000ff">字体颜色</font>
```
效果如下：

<font face="微软雅黑" color="red" size="6">字体及字体颜色和大小</font>
<font color="#0000ff">字体颜色</font>

```yml
换行
使用html标签<br/>换行
```
效果如下：

使用html标签<br/>
换行

```yml
文本对齐方式
<p align="left">居左文本</p>
<p align="center">居中文本</p>
<p align="right">居右文本</p>
```

效果如下：  
<p align="left">居左文本</p>
<p align="center">居中文本</p>
<p align="right">居右文本</p>

```yml
下划线
<u>下划线文本</u>
```
效果如下：  
<u>下划线文本</u>

`That's all, Enjoy it!`

