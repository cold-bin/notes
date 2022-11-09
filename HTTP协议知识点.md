---
title: "HTTP协议知识点"
date: 2022-10-23T21:23:13+08:00
tags: []
categories: [HTTP协议]
draft: false
---

#### 简介

超文本传输协议（英文：**H**yper**T**ext **T**ransfer **P**rotocol，缩写：HTTP）是一种用于分布式、协作式和超媒体信息系统的应用层协议。HTTP是万维网的数据通信的基础。它是基于TCP协议的应用层传输协议，简单来说就是客户端和服务端进行数据传输的一种规则。

**TCP**：大家感兴趣的话就去学习《计算机网络》，反正以后都会学，这里简单描述一下，是一种面向连接的、可靠的、基于字节流的传输层通信协议。

#### URL

URL(Uniform Resource Locator)是“统一资源定位符”的英文缩写，用于描述一个网络上的资源, 基本格式如下

```plain
scheme://host[:port#]/path/.../[?query-string][#anchor]
例如
http://localhost:8080/login?username=admin&password=123456
```

scheme: 指定使用的协议（如：http，https，ftp(File Transfer Protocol，文件传输协议)等）

host: http服务器的域名或者ip地址

port: 指定的端口（默认为80）

path：访问资源的路径

query-string：发送给http服务器的数据

anchor：锚（如 http://localhost:8080/index#top ,这里的锚就是top）。`#`是用来指导浏览器动作的，对服务器端完全无用。所以，HTTP请求中不包括`#`。比如说跳转到顶部，第多少行啥的。

#### 请求报文

我们先来看看 Request 包的结构，Request 包分为 3 部分，第一部分叫 Request line（请求行）, 第二部分叫 Request header（请求头）, 第三部分是 body（主体）。header 和 body 之间有个空行，请求包的例子所示:

![img](https://cdn.nlark.com/yuque/0/2021/png/22813704/1636555980744-2613d37e-ba5a-42cd-a6b1-0a4d144adbc0.png)

#### **响应报文**

![img](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fi.loli.net%2F2020%2F12%2F19%2FEA52cWSMGjivFeL.png&sign=048c5e5a9d6f7302722104a3ea140a4bae8f71c123238ee89d4d42654f300683)



Response 包中的第一行叫做状态行，由 HTTP 协议版本号， 状态码， 状态消息三部分组成。

状态码用来告诉 HTTP 客户端，HTTP 服务器是否产生了预期的 Response。HTTP/1.1 协议中定义了 5 类状态码， 状态码由三位数字组成，第一个数字定义了响应的类别

- 1XX 提示信息 - 表示请求已被成功接收，继续处理（这个常用于内部，比如：当我们发post请求的时候，实际上会发送两次，先发送header，服务器响应100 continue，浏览器再发送body））

- 2XX 成功 - 表示请求已被成功接收，理解，接受（如：200 - 请求成功，204 - 服务器成功处理，但未返回内容 等）

- 3XX 重定向 - 要完成请求必须进行更进一步的处理（如：301 - 永久移动，302 - 临时移动 ，304 - 未更新等）

- 4XX 客户端错误 - 请求有语法错误或请求无法实现（如:404）

- 5XX 服务器端错误 - 服务器未能实现合法的请求（如：500 - 服务器内部错误，503 - 由于超载或系统维护，服务器暂时的无法处理客户端的请求）

  [点击查看更多](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)

#### HTTP缓存

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching

#### HTTP头部

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers

#### HTTP认证

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication

#### 保持状态的方式

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies

**Cookie**:

HTTP 协议是无状态， Cookie 来保存状态信息。Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器之后向同一服务器再次发起请求时被携带上，用于告知服务端两个请求是否来自同一浏览器。由于之后每次请求都会需要携带 Cookie 数据，因此会带来额外的性能开销。

实现方式：写在请求头里面

用途：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）

- 个性化设置（如用户自定义设置、主题等）

- 浏览器行为跟踪（如跟踪分析用户行为，定制个性化广告（突然想起了之前看的一个新闻：谷歌将终止第三方Cookie采集隐私数据，广告商遇“灭顶之灾”？）等）

**Session**:

session是一种记录客户端状态的机制，不同的是cookie保存在客户端浏览器中，而session保存在服务器上。客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上，这就是session。客户端浏览器再次访问时只需要从该session中查找该客户的状态就可以了。

session的原理是第一次访问的时候，随机生成一个sid并生成一个sid对应的对象，然后把这个sid发送到浏览器端。下次客户端把sid带回服务器，从服务器上找到这个sid对应的对象就可以了。

由于使用session时会给cookie存一个sessionId(即加密后的connect.id),所以session是依赖于cookie的。（一般是这样，但是你也可以写在query-string，bodyl里面也行）

#### 注意

- `application/x-www-form-urlencoded`

  表单代码：

  ```html
  <form action="http://localhost:8888/task/" method="POST">
  First name: <input type="text" name="firstName" value="Mickey&"><br>
  Last name: <input type="text" name="lastName" value="Mouse "><br>
  <input type="submit" value="提交">
  </form>
  ```

  通过测试发现可以正常访问接口，在Chrome的开发者工具中可以看出，表单上传编码格式为application/x-www-form-urlencoded(Request Headers中)，参数的格式为key=value&key=value。

  我们可以看出，服务器知道参数用符号&间隔，如果参数值中需要&，则必须对其进行编码。编码格式就是application/x-www-form-urlencoded（将键值对的参数用&连接起来，如果有空格，将空格转换为+加号；有特殊符号，将特殊符号转换为ASCII HEX值）。

  对于Get请求，是将参数转换`?key=value&key=value`格式，连接到url后；如果是post方法
