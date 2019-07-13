---
layout:     post
title:      An Introduction to Markdown
subtitle:   Markdwon使用指南
date:       2017-12-28
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Markdown
    - Github
    - An Introduction to
---

>本文针对 Github Flavored Markdown

## 标题
```
## 标题 1
### 标题 2
...
####### 标题 6
```

## 强调、突出、删除
```

*突出*

**强调**

_Emphasis_    //work just for English

__强调__

~~删除~~
```

## 链接

自动链接
```
https://github.com

<lchang199x@gmail.com>   //email is a kind of link
```

内联风格 (title 是可选的):
```
[GitHub](https://github.com "title")
```

引用风格 (title 是可选的):
```
[GitHub][ref]
······
[ref]: https://github.com/    //somewhere else
```

### 图片

内联风格 (title 是可选的):
```
![alt text](未标题-1.png "title")
```

引用风格 (title 是可选的):
```
![alt text][id]
······
[id]: 未标题-1.png
```

## 列表
```
有序列表:

1. 列表项 1
2. 列表项 2

无序列表:

* 列表项 1
* 列表项 2
- 列表项 3
- 列表项 4

任务列表
- [x] This is a complete item
- [ ] This is an incomplete item

列表项缩进两个空格就可以创建一个嵌套的列表：

1. 列表项 1
  1. 嵌套的列表可以是有序的
  2. 格式和正常的有序、无序列表没有差异
2. 列表项 2
  * 嵌套的列表可以是无序的
    * 这个嵌套的列表项有4个空格的缩进，因为它的父列表项自身就带有2个空格的缩进
    * 还允许更多层的嵌套
3. 列表项 3
```

## 引用
```
> 段落前面添加大于号和空格，就能够形成引用段落。 > > 这是嵌套的引用。
```

## 内联代码
```
`内联代码` 使用反引号包含 你也可以像 `` `这样` `` 转义反引号
```

## 代码块
```
每行缩进4个空格或者1个 tab：

这是一个正常的段落。

    这是代码块。
```

## 围栏式代码块
```
```c
#include <stdio.h>
if(true){
  printf("Hello World.");
}
'''
```

## 水平分割线

三个或更多的星号或横杠

## 强制换行

在行尾输入两个空格

## 表格

这是个简单的表格:
```
First Header | Second Header | Third Header
------------ | ------------- | ------------
Content Cell | Content Cell | Content Cell
Content Cell | Content Cell | Content Cell
```
出于美观的考虑, 可以把两端都包围起来:
```
| First Header | Second Header | Third Header |
| ------------ | ------------- | ------------ |
| Content Cell | Content Cell | Content Cell |
| Content Cell | Content Cell | Content Cell |
```
通过在标题分割行添加冒号 : ,可以定义表格单元的对其格式：向左靠齐，居中和向右靠齐：
```
First Header | Second Header | Third Header
:----------- | :-----------: | -----------:
Left | Center | Right
Left | Center | Right
```