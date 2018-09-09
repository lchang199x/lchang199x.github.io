---
layout:     post
title:      An Introduction to Json
subtitle:   Json必知必会
date:       2016-11-13
author:     Cliu
header-img: img/post-daily-bg.jpg
catalog: true
tags:
    - Json
    - An Introduction to
---

# 初识Json

Json全称Javascript Object Notation，即JavaScript对象表示法。

基于JavaScript对象字面量，仅关注属性的字面量而不包含函数字面量。

类似于xml，csv等，也是一种数据交换格式，用于在不同平台或系统间交换数据的文本。

Json的MIME类型是application/json。

注1：JavaScript的MIME类型是text/javascript，Java的MIME类型是application/java，这就说明了它们仨是不同的东西。

注2：MIME全程Multipurpose Internet Mail Extensions，即多用途互联网邮件扩展。MIME类型即互联网媒体类型，以“类型/子类型”的格式表示。

# 举例说明
```
//这是一个JavaScript对象字面量，双引号可换成单引号
{firstname:"Bill", lastname:"Gates", id:5566};

//这是一个JSON，除了字符串类型的值必须加双引号，所有名称也要加双引号
//双引号不可换成单引号
{"firstname":"Bill", "lastname":"Gates", "id":5566};
```

对于特殊字符，需要用反斜杠\进行转义：比如双引号\"content\"，反斜杠本身\\，以及任意unicode字符\u263A。

```
//JSON字符串，Javascript字符串用单引号和双引号皆可，JavaScript 变量均为对象
var jsonString = '{"animal":"cat"}';

//将JSON字符串转化为JavaScript对象（反序列化），JSON.parse()比eval()更加安全
//前者仅解析，后者解析之后还会执行，如果jsonString中存在JS脚本，evel()也会执行。
//另JSON.stringify()将对象序列化为Json文本类型
var obj = JSON.parse(jsonString);

//使用对象的属性
alert(obj.animal);
```

注1：序列化和反序列化（实现数据持久化和通信）：序列化 (Serialization)将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。

# org.json

org.json包是Java常用的Json解析工具，导入lib即可使用，Android也内置了该工具，主要提供JSONObject和JSONArray两个类，优势在轻量级。

```
//A JSONObject is an unordered collection of name/value pairs.
//JSONArray其实就是把多个Object放在[]中括号中
[{name:'a',value:'aa'},{name:'b',value:'bb'},{name:'c',value:'cc'},{name:'d',value:'dd'}]
```

注1：Map键值对用等号赋值，Json键值对用冒号分开，这是一个无关痛痒的联想。

注2：延伸阅读包括Gson、Fastjson、Jackson，用好一个就行。
