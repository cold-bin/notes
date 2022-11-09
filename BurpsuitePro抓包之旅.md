---
title: "BurpsuitePro抓包之旅"
date: 2022-10-05T17:34:18+08:00
tags: []
categories: [抓包]
draft: false
---

## burpsuitepro的模块

**Proxy**

提供一个直观、友好的用户界面，它的代理服务器**包含非常详细的拦截规则**，并能**准确分析 HTTP 消息的结构与内容**。

**Spide**

爬行蜘蛛工具，可以用来**抓取目标网站**，以显示网站的内容，基本结构，和其他功能。

**Scanner**

Web 应用程序的**安全漏洞进行自动发现工具**。它被设计**用于渗透测试**，并密切与您现有的技术和方法，以适应执行手动和半自动化的 Web 应用程序渗透测试。这个对于抓包分析来说用处不大。

**Repeater**

可让您**手动重新发送单个HTTP请求**以查看服务器回应。这个对于重新构造请求头和请求体比较方便。

**Intruder**

是burp套件的优势,它提供一组特别有用的功能。它可以**自动实施各种定制攻击**，包括资源枚举、数据提取、模糊测试等常见漏洞等。在各种有效的扫描工具中，它能够以最细化、最简单的方式访问它生产的请求与响应，允许组合利用个人智能与该工具的控制优点。

**Sequencer**

对会话令牌，会话标识符或其他出于安全原因需要随机产生的键值的可预测性进行分析。

**Comparer**

是一个简单的工具，执行比较数据之间的任何两个项目（一个可视化的“差异”）。在攻击一个Web 应用程序的情况下，这一要求通常会出现当你想快速识别两个应用程序的响应之间的差异（例如，入侵者攻击的过程中收到的两种反应之间之间，或登录失败的反应使用有效的和无效的用户名）之间，或两个应用程序请求（例如，确定不同的行为引起不同的请求参数）。

**Decoder**

使用各种编码绕过服务器端输入过滤，smart decode 自动识别编码格式。

## google浏览器网页抓包

### 下载证书

burpsuitepro默认的证书地址在`120.0.0.1:8080`端口，当然也可以更改。这样在本地就可以访问`127.0.0.1:8080`地址获取CA证书了。如果发生端口冲突，可以手动修改burpsuitepro的代理地址端口，这样只需要在对应代理端口访问即可。然后点击 **CA Certificate**即可下载证书![image-20221006113634706](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061136901.png)

### 安装证书

设置--->隐私设置和安全性--->更多--->管理证书

![b737fc74c5a1698bd8745ef1a2320a14.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061125310.png)

**导入证书**

![7c4d257c0186e489d33d37a15c01ced0.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061126216.png)

下一步到浏览本地证书位置。选所有文件，不然可能你找不到你的证书!! 选择证书后打开进入下一步。

![0e1b1951d7488b6a596593b4f9452e3f.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061126793.png)


按下图位置设置进入下一步，完成。

![a548a0ffefcd751bd8cc76f61d7cd946.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061126621.png)



**最后设置证书信任**

按图示操作找到刚才安装的证书。

![ba7853dac1e9af6f9546f951f38607ce.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061126828.png)


选中证书点高级

![8ac0a0cdb877d6d64c65037a1acf48d0.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061126607.png)

按下图勾选，确认。最后重启浏览器即可。

![0673ec779b4a47d586f06047e79e8f7b.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061126046.png)

### 抓包

#### 挂代理

需要使用google浏览器抓包时，需要将burpsuitepro的proxy端口地址开个代理。可以全局代理，也建议局部代理，这里推荐一款好用的插件 SwitchyOmega ,它可以定制代理规则，将浏览器的请求转发到对应地址上。这里我们需要将burpsuitepro监听的端口地址，使用代理进行转发，这样浏览器发出的请求就会被转到代理地址，就可以收到请求报文信息。

#### 打开拦截

proxy->intercept-->intercept is off=>intercept is on

![image-20221006114647653](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061146506.png)

打开代理和拦截后，那么对满足代理规则的网址的请求都会拦截并解析。

![image-20221006115823412](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202210061158626.png)

#### 选项解释

这里说说，Forward Drop Action intercept is off/on 这几个选项

**Forward**

拦截并编辑请求之后，发送请求到服务器或浏览器

**Drop**

不想要发送本次请求，可以点击**Drop**放弃这个拦截请求

**intercept is off/on**

关闭/开启所有拦截.如果时 on ，表示请求和响应将被拦截或自动转发根据配置的拦截规则配置的代理选项。如果显示 off 则显示拦截之后的所有信息将自动转发。相当于没有对这个请求做任何事情。

**Action**

对当前拦截的请求做一些操作，如**Repeater**可以篡改请求之后再发送请求。

## 微信小程序抓包

由于目前微信升级了。最新版本的微信小程序应用使用代理，请求却无法走代理。目前好像不能抓包。微信小程序的请求不走代理端口地址。

## APP抓包

todo
