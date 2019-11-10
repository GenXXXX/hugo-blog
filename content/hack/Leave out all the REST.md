---
title: Leave out all the REST
date: 2015-11-10
showDate: true
mathjax: true
tags: ["技术"]
---

毕设导师在看了我过去的项目经历之后问：

“你听说过，RESTful，吗？”

“不知道啊，老师。”

“我看你过去做过网站，然后你又对移动应用开发感兴趣。但是单纯做App没能解决什么技术性的问题。我觉得你可以考虑做一个构建在REST api之上的通用应用模板，帮助提高移动创业开发的效率。”

对于一个无从下手的新技术概念，还是需要通过问自己一些本体论问题来入手：

它是谁？它来自哪里？它为什么存在？

于是有了这篇科普文。

<!--more-->

## <strong>什么是RESTful？</strong>
#### REpresentational State Transfer,表现层状态转移。

所谓REST是定义服务器与客户端之间进行交互的API的一种架构风格。
REST API遵循这些接口约束：

1. 资源的识别
将分布式超媒体系统中所使用到的全部数据，统一视为资源（Resource），对各种类型的资源，进行基本操作。

2. 通过表述对资源执行的操作
四种基本操作：POST、GET、PUT、DELETE

3. 自描述的消息
URL及API具有自描述特性，能直接看出文件层级或类型信息

4. 作为应用状态引擎的超媒体

### <strong><em>最佳实践：</em></strong>

- URL root：
API版本格式
example.org/api/v1
或者 api.example.com/v1

- URI使用名词复数形式，例如：

>GET /products : will return the list of all products

>POST /products : will add a product to the collection

>GET /products/4 : will retrieve product #4

>PATCH/PUT /products/4 : will update product #4

- 保证head和get方法的安全性，不改变资源状态

- 资源地址推荐嵌套结构，例如：

>GET /friends/29109231/profile （获取id为29109231的友人的说明信息）


- 对返回结果大小进行限制

- 传递正确的HTTP Satus Code

- 返回结果明白易懂

## <strong>为什么要使用RESTful？</strong>
三个架构属性（来自Roy的博士论文原文）：

### 1. 可见性

监视系统不必为了确定一个请求的全部性质而去查看该请求之外的多个请求

### 2. 可伸缩性

不必在多个请求之间保存状态，从而允许服务器组件迅速释放资源，并进一步简化其实现。因为服务器不必跨越多个请求管理资源的利用

### 3. 可靠性

减轻从局部故障中恢复的任务量

### <em><strong>基于操作的架构风格为什么不好：</strong></em>
基于操作的API设计由于操作之间的关联性，会使得系统复杂化
URL设计缺乏一致性会使其不具备自描述能力
基于操作设计的副作用不适合用于设计缓存机制

### <em><strong>所以基于资源的设计风格好好好：</strong></em>

1. 资源之间相互独立，耦合性低（无状态）

2. 对资源操作类型有限，设计URL一致性高

3. 对资源进行读操作很方便设计缓存

## <strong>RESTful API如何实现？</strong>

优秀的设计范例有：
GitHub、FB、豆瓣

## <strong>RESTful API如何使用？</strong>

之前我理解的是，既然有很多网站的api都是基于REST架构的，那么就可以有一个统一的方法来获取各个网站的资源，从而在移动端利用一个设计好的“万金油模板”快速生成一个开放了REST api的网站应用。
虽然大致明白了REST是什么，解决了什么问题。但是我在仔细看了几个符合RESTful API的网站之后，我对原先的理解产生了动摇。即使它们的api都是REST风格的但是——
豆瓣获取的资源是图书、电影、FM、相册信息，脸书获取的资源是好友关系、新鲜事、照片，感觉在做应用的时候，完全不能放进同一个模板啊。完全に驴唇不对马嘴啊。而且就算强行把豆瓣的广播、脸书的新鲜事放到移动应用的同一个View里面，向两个网站发送的请求URL和解析返回值的时候也是完全不一样的。
难道会出现这样的说明：
“该通用应用模板，支持豆瓣、微博、GitHub的REST架构api。更多的网站支持将在下一个版本迭代中添加。”
导师，叫我毕业设计做这个，你是不是想让我多陪你一年。

所以现在的新想法是，自己构架一个网站（或者重构一个不符合REST风格的老网站，工作量偏大？），并且后端实现和api符合REST风格。然后，在移动端可以利用api快速制作一个移动客户端。

毕业设计的开题报告马上就要来了，决定做什么之后硬着头皮也得上了。可是为什么冥冥间，我仿佛听到某公园乐队的劝阻：

## <blockquote>Leave out all the REST!</blockquote>
