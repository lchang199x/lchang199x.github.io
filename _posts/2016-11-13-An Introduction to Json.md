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

#### 什么是Json

Json全称Javascript Object Notation，即JavaScript对象表示法。

基于JavaScript对象字面量，仅关注属性的字面量而不包含函数字面量。

类似于xml，csv等，也是一种数据交换格式，用于在不同平台或系统间交换数据的文本。

Json的MIME类型是application/json。

注1：JavaScript的MIME类型是text/javascript，Java的MIME类型是application/java，这就说明了它们仨是不同的东西。

注2：MIME全称Multipurpose Internet Mail Extensions，即多用途互联网邮件扩展。MIME类型即互联网媒体类型，以“类型/子类型”的格式表示。

#### 举例说明
```js
//这是一个JavaScript对象字面量，双引号可换成单引号
{firstname:"Bill", lastname:"Gates", id:5566};

//这是一个JSON，除了字符串类型的值必须加双引号，所有名称也要加双引号
//双引号不可换成单引号
{"firstname":"Bill", "lastname":"Gates", "id":5566};
```

对于特殊字符，需要用反斜杠\进行转义：比如双引号\"content\"，反斜杠本身\\，以及任意unicode字符\u263A。

```js
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

# Json解析

##### org.json

[org.json](http://json.org/) 包是Java常用的Json解析工具，主要提供`JSONObject`和`JSONArray`两个类，优势在轻量级，在Java中，导入lib即可使用。

```java
//A JSONObject is an unordered collection of name/value pairs
String jsonString = " {"
                + "\"name1\" : \"value1\","
                + "\"name2\" : \"value2\","
                + "\"name3\" : \"value3\""
                + "}";
JSONObject object = new JSONObject(jsonString);

//An JSONArray is an ordered collection of values
//A value can be a string, number, object, array, true, false or null
String arrayString = "["
                + "\"string\","
                + 100
                + ","
                + false
                + ","
                + null
                + ","
                + "{\"foo\" : \"bar\"},"
                + "[1,2,3]"
                + "]";
JSONArray array = new JSONArray(arrayString);
```

JSONObject内部使用`HashMap<String, Object>`来存储键值对，并提供has, get, put, remove等方法进行增删查改操作。
JSONArray类内部使用`ArrayList<Object>`来存储字符串、数值、布尔类型值、`JSONObject.NULL`、JSONObject或JSONArray，
并提供size, add, get, set, remove, contains等方法来进行增删查改操作。

**Android内置了该工具**。

注1：Map键值对用等号赋值，Json键值对用冒号分隔。  
注2：当使用gradle管理Java项目时，可用`compile 'org.json:json:20160810'`添加依赖。  
注3：Json中的null和Java/Js中的null还是有点区别的，在org.json中，Java/Js中的null被当成undefined，Json中的null用JSONObject.NULL来表示。  
注4：JSONObject和JSONArray都有声明为`public String toString(int indentFactor) throws JSONException`的toString()方法重载，可以打印可读/美观的json字符串形式。

#### Gson
[Gson](https://github.com/google/gson) 是Google提供的用于Java对象序列化和反序列化的库。
主要提供了Gson、GsonBuilder、JsonElement、JsonObject、JsonArray (注意Json仅首字母大写)、TypeToken等类，
其中JsonObject和JsonArray两个类内部接口和org.json中的十分相似，只不过没有以字符串为参数的构造函数重载。
序列化/反序列化方面，Gson提供了两个十分简洁的方法Gson.toJson()和Gson.fromJson()。

在Android中使用Gson时，应使用`implementation 'com.google.code.gson:gson:2.8.5'`添加依赖。

```java
Gson gson1 = new Gson();
//使用GsonBuilder创建Gson对象，可以自定义很多配置
Gson gson = new GsonBuilder()
                .disableHtmlEscaping()
                .serializeNulls()
                .setPrettyPrinting()
                .create();
//基本类型及引用类型的序列化/反序列化
String s1 = gson.toJson("abcd");
String[] arr = gson.fromJson("[\"abc\"]", String[].class);

//泛型序列化/反序列化
Type collectionType = new TypeToken<Collection<Integer>>(){}.getType();
Collection<Integer> collection 
		  = gson.fromJson("[1,2,3,4]", collectionType);
```

对于更加复杂的情形，Gson支持序列化/反序列化过程的自定义，具体参见[Gson用户指南](https://github.com/google/gson/blob/master/UserGuide.md)。

# 高级话题

#### Protocol Buffer
`Protocol Buffer (Protobuf)`是由Google开源的一种序列化结构化数据的机制，通过.proto文件来定义，然后通过相应编译器编译成对应语言的数据访问类。Protobuf可用于数据传输及数据存储，和xml, json相比具有更好的性能。扩展阅读：[官方网站](https://developers.google.cn/protocol-buffers/) & [在Android中的应用](https://github.com/google/protobuf-gradle-plugin)。

