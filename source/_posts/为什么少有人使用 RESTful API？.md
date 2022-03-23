---
title: 为什么少有人使用 RESTful API？
toc: true
excerpt: 这是一段文章介绍
date: 2022-03-23 10:57:26
updated: 2022-03-23 10:57:26
categories: 后端
tags: [ 'HTTP', 'RESTful']
---
## 前言

在上年写了一篇[《后端API接口的错误信息返回规范》](https://juejin.cn/post/6876377628371058701 "https://juejin.cn/post/6876377628371058701")，这是符合 RESTful 规范（严谨一点是类 RESTful）的错误信息。掘友们却对这种规范存在不同意见，都倾向于*只要是后端可以正确收到请求，那都是200。异常都交给业务处理。*

而且不仅是错误信息返回规范，API 设计都很少遵循 RESTful 风格，大多数都是类 JSON RPC。这是为什么呢？

## RESTful API

想要回答这个问题，首先得知道 RESTful 风格的 API 是什么样的。

其实关于 RESTful 的基本概念，我在上篇文章[《方法论：如何学习HTTP》](https://juejin.cn/post/7046195769681903630 "https://juejin.cn/post/7046195769681903630")也介绍过了，在这就不再赘述了。

接下来看看一个完整的 RESTful API 是怎么形成的。

### Richardson Maturity Model

Leonard Richardson提出了一种以他命名的模型 **Richardson Maturity Model** ，循序渐进的介绍了迈向RESTful的不同等级。
![](https://raw.githubusercontent.com/leoRogerCopyThat/blog-images/main/202203231101350.webp)

但需要强调的是，RMM 虽然是思考 RSET 的一个好方法，但它不是REST的定义。RMM 有用的地方只是在于它提供了一个很好的 step by step 的方法来理解 REST 思想背后的基本理念。

#### **Level 0**

这是最基本的等级，在这等级中只是简单的使用HTTP作为远程交互的传输隧道，而不是当成传输协议来使用，不涉及任何 web 的机制。举个例子：

我今天约了小伙伴打羽毛球，于是发起一个请求向体育馆询问哪个时间段可以预定羽毛球场。

```http
POST /stadiumReception HTTP/1.1
[various other headers]

{
    "date": "2022-01-05",
    "sport": "badminton"
}
```

前台返回今天羽毛球场可以预定的时间段

```http
HTTP/1.1 200 OK
[various headers]

{
    "times": [
        {
            "start": "1400",
            "end": "1600"
        },
        {
            "start": "1700",
            "end": "1830"
        }
    ]
}
```

得到时间段信息，再约一下时间，决定预定17点到18点半，打完吃饭。

```http
POST /stadiumReception HTTP/1.1
[various other headers]

{
    "date": "2022-01-05",
    "sport": "badminton",
    "start": "1700",
    "end": "1830"
}
```

预定成功!

```http
HTTP/1.1 200 OK
[various headers]

{
    "orderId": "123456789"
}
```

可以看到这是非常典型的 JSON RPC 格式，当然在实际开发中 API 设计要比这好很多。

#### **Level1：面向资源**

在 RMM 模型（Richardson Maturity Model）中迈向 REST 顶点的第一步就是面向资源，所以把独立资源替代单一的接口节点。

因此，可能会将不同运动，比如羽毛球等抽象为独立的资源。所以请求修改为如下：

```http
POST /sports/badminton HTTP/1.1
[various other headers]

{
    "date": "2022-01-05"
}
```

询问某个日期，羽毛球场的可预定时间。

```http
HTTP/1.1 200 OK
[various headers]

{
    "times": [
        {
            "start": "1400",
            "end": "1600",
            "id": "123456789"
        },
        {
            "start": "1700",
            "end": "1830",
            "id": "987654321"
        }
    ]
}
```

返回比原有基础上增加了单独的 id，可以表示羽毛球场的特定空闲时间段。

这时就能选定某个时间段，将其 id 向预约接口预定。

```http
POST /reservations/987654321 HTTP/1.1
[various other headers]
```

预定成功!

```http
HTTP/1.1 200 OK
[various headers]

{
    "orderId": "abcd987654321"
}
```

可以看到与 level0 最大的不同就是对单一的节点做操作，变成了对不同的资源做操作。

#### **Level2：HTTP动词**

Level2，真正将 HTTP 作为了一种传输协议，最直观的一点就是 Level2 使用了 **HTTP动词** ，GET/PUT/POST/DELETE/PATCH等，这些都是HTTP的规范，规范的作用自然是重大的，用户看到一个 POST 请求，就知道它不是**幂等**的，使用时要小心。而看到 GET，就知道它是幂等的，调用多几次都不会造成问题，当然，这些的前提都是 API 的设计者和开发者也遵循这一套规范。

所以对于询问某个日期，羽毛球场的可预定时间，可改为

```http
GET /sports/badminton/20220105?status=idle HTTP/1.1
[various other headers]
```

HTTP 将 GET 定义为安全操作，这意味着它不会对任何状态进行更改。所以我们可以任意多次调用 GET，并每次都能得到相同的结果。而且 web 还会缓存 GET 请求与结果，这是 Web 架构工作良好的关键因素。

返回结果一样的，接着我们预定一个时间段。

```http
POST /reservations/987654321 HTTP/1.1
[various other headers]
```

请求不变，但返回却改变了。如果一切顺利，服务将返回一个 201 响应码，表示有一个新资源产生了。

```htttp
HTTP/1.1 201 Created
Location: orders/abcd987654321
[various headers]

{
    "orderId": "abcd987654321"
}
```

201 响应还包括了 Location 属性，指明客户端可以使用该属性的URL来获取该订单资源的最新状态。

如果出现错误，例如其他人预订了该时段，则：

```http
HTTP/1.1 409 Conflict
[various headers]

{
    "error": "该时间段已被预定"
}
```

此响应的重要部分是使用正确的 HTTP 响应码来表示出错的地方。在该场景中，409 是一个很好的选择，表明其他人已经以互斥的方式更新了资源。 在 Level 2，我们明确使用某种类型的错误响应，而不是使用返回码 200 但包含错误响应。

#### **Level3：HATEOAS**

HATEOAS（Hypertext As The Engine Of Application State），中文翻译为“将超媒体当作应用状态引擎”，这个描述的核心是超媒体概念，换句话说：是链接的思想。

从 Level2 的请求开始：

```http
GET /sports/badminton/20220105?status=idle HTTP/1.1
[various other headers]
```

返回的信息是与Level2不同的

```http
HTTP/1.1 200 OK
[various headers]

{
    "times": [
        {
            "start": "1400",
            "end": "1600",
            "id": "123456789",
            "link": "/reservations/123456789"
        },
        {
            "start": "1700",
            "end": "1830",
            "id": "987654321",
            "link": "/reservations/987654321"
        }
    ]
}
```

每个时间段现在都有一个 link 元素，其中包含一个 URI，告诉我们如何预约。超媒体控制的要点是，告诉我们下一步可以做什么，以及我们需要操作的资源 URI。我们不必知道预约的请求发到哪里去，响应中的超媒体控制会告诉我们怎么去做。

预约请求仍用回 level2 的

```http
POST /reservations/987654321 HTTP/1.1
[various other headers]
```

回复中包含了许多超媒体控制，用于下一步要做的不同事情。

```htttp
HTTP/1.1 201 Created
Location: orders/abcd987654321
[various headers]

{
    "orderId": "abcd987654321",
    "link": {
        "rel": "delete",
        "uri": "/orders/abcd987654321"
    }
}
```

返回提供了如何取消该订单。

Level3 的 Restful API，给使用者带来了很大的便利，使用者只需要知道如何获取资源的入口，之后的每个 URI 都可以通过请求获得，无法获得就说明无法执行那个请求。

### RESTful 的缺点

需要强调的一点是， **RESTful 不是一种规范，而是一种风格** ，它是不具有强制性的。所以风格这种东西说不准，公说公有理，婆说婆有理。就算是相同水平的程序员设计的 “RESTful” 也很难一样。而在开发团队中，最忌讳的就是不统一！

1、RESTful 是面向资源的，所以接口都是一些名词，尤指复数名词。简单的 CRUD 还是很合适的，但很多业务逻辑都很难将其抽象为资源。比如说登录/登出，怎么看也不是一个资源，如果硬是抽象为 创建一个 session / 删除一个session。这不仅反直觉，还违背了 RESTful 的思想。

2、RESTful 只提供了基本的增删改查，对于复杂的逻辑是一点办法没有，比如批量下载、批量删除等。对于复杂的查询，更是无从下手。而且开发时会面临诸多选择，修改资源用 PUT 还是 PETCH？采用查询参数还是用 body？

3、关于错误码的问题更是复杂的一批，RESTful 建议使用 status code 作为错误码，以便统一。在实际开发中，业务逻辑的含义数不胜数，很难统一。比如 400 状态码到底是表示传参有问题，还是该资源已被占用了。404 是表示接口不存在，还是资源不存在。

发展到最后，开发人员的时间精力都不是用于怎么实现这个逻辑，而是变成了纠结这逻辑到底是个什么资源？把时间都给浪费掉，最后还给出了一个不伦不类的 “RESTful” API。

 **最重要一点是 RESTful 与 RPC 没有高低之分** ，所有的代码规范、接口设计以及各种规定，其实都是为了在团队内部形成共识、防止个人习惯差异引起的混乱。相比之下形成 JSON-RPC 规范相对容易以及更方便使用。

就算团队里面规定必须用动作表示 URL，所有请求都要使用 POST，这也是没问题的。

所以无论是 RESTful 还是 RPC，归根到底还是为了开发人员和产品应用服务的，如果它能带来便利、减少混乱，就值得用；反之，如果带来的麻烦比要解决的还多，那就 duck 不必了。

## 结尾

前两年由于经验不足，给了一些自以为正确的意见，现在想想还真是太年轻了🤭，有句话说的对：学的越多感觉自己越无知。

原文：[https://juejin.cn/post/7051801217705443341](https://juejin.cn/post/7051801217705443341)