---
title: "MongoDB基础篇"
date: 2022-11-25T20:20:04+08:00
tags: [MongoDB]
categories: [数据库,MongoDB]
draft: false
---

## MongoDB背景

### **1、MongoDB是什么？**

MongoDB是一款为web应用程序和互联网基础设施设计的数据库管理系统。没错MongoDB就是数据库，是NoSQL类型的数据库

### **2、为什么要用MongoDB？**

（1）MongoDB提出的是文档、集合的概念，使用BSON（类JSON）作为其数据模型结构，其结构是面向对象的而不是二维表，存储一个用户在MongoDB中是这样子的。

```text
{
    username:'123',
    password:'123'
｝
```

使用这样的数据模型，使得MongoDB能在生产环境中提供高读写的能力，吞吐量较于mysql等SQL数据库大大增强。

（2）易伸缩，自动故障转移。易伸缩指的是提供了分片能力，能对数据集进行分片，数据的存储压力分摊给多台服务器。自动故障转移是副本集的概念，MongoDB能检测主节点是否存活，当失活时能自动提升从节点为主节点，达到故障转移。

（3）数据模型因为是面向对象的，所以可以表示丰富的、有层级的数据结构，比如博客系统中能把“评论”直接怼到“文章“的文档中，而不必像myqsl一样创建三张表来描述这样的关系。

### **3、主要特性**

> （1）文档数据类型

SQL类型的数据库是正规化的，可以通过主键或者外键的约束保证数据的完整性与唯一性，所以SQL类型的数据库常用于对数据完整性较高的系统。MongoDB在这一方面是不如SQL类型的数据库，且MongoDB没有固定的Schema，正因为MongoDB少了一些这样的约束条件，可以让数据的存储数据结构更灵活，存储速度更加快。

> （2）即时查询能力

MongoDB保留了关系型数据库即时查询的能力，保留了索引（底层是基于B tree）的能力。这一点汲取了关系型数据库的优点，相比于同类型的NoSQL redis 并没有上述的能力。

> （3）复制能力

MongoDB自身提供了副本集能将数据分布在多台机器上实现冗余，目的是可以提供自动故障转移、扩展读能力。

> （4）速度与持久性

MongoDB的驱动实现一个写入语义 fire and forget ，即通过驱动调用写入时，可以立即得到返回得到成功的结果（即使是报错），这样让写入的速度更加快，当然会有一定的不安全性，完全依赖网络。

MongoDB提供了Journaling日志的概念，实际上像mysql的bin-log日志，当需要插入的时候会先往日志里面写入记录，再完成实际的数据操作，这样如果出现停电，进程突然中断的情况，可以保障数据不会错误，可以通过修复功能读取Journaling日志进行修复。

> （5）数据扩展

MongoDB使用分片技术对数据进行扩展，MongoDB能自动分片、自动转移分片里面的数据块，让每一个服务器里面存储的数据都是一样大小。

### **4、C/S服务模型**

MongoDB核心服务器主要是通过mongod程序启动的，而且在启动时不需对MongoDB使用的内存进行配置，因为其设计哲学是内存管理最好是交给操作系统，缺少内存配置是MongoDB的设计亮点，另外，还可通过mongos路由服务器使用分片功能。

MongoDB的主要客户端是可以交互的js shell 通过mongo启动，使用js shell能使用js直接与MongoDB进行交流，像使用sql语句查询mysql数据一样使用js语法查询MongoDB的数据，另外还提供了各种语言的驱动包，方便各种语言的接入。

### **5、完善的命令行工具**

mongodump和mongorestore,备份和恢复数据库的标准工具。输出BSON格式，迁移数据库。

mongoexport和mongoimport，用来导入导出JSON、CSV和TSV数据，数据需要支持多格式时有用。mongoimport还能用与大数据集的初始导入，但是在导入前顺便还要注意一下，为了能充分利用好mongoDB通常需要对数据模型做一些调整。

mongosniff,网络嗅探工具，用来观察发送到数据库的操作。基本就是把网络上传输的BSON转换为易于人们阅读的shell语句。

因此，可以总结得到，MongoDB结合键值存储和关系数据库的最好特性。因为简单，所以数据极快，而且相对容易伸缩，还提供复杂查询机制的数据库。MongoDB需要跑在64位的服务器上面，且最好单独部署，因为是数据库，所以也需要对其进行热备、冷备处理。

### **6、几个shell实操**

因为本篇文章不是API手册，所有这里对shell的使用也是基础的介绍什么功能可以用什么语句，主要是为了展示使用MongoDB shell的方便性，如果需要知道具体的MongoDB shell语法可以查阅官方文档。

> 1、切换数据库

```text
use dba
```

创建数据库并不是必须的操作，数据库与集合只有在第一次插入文档时才会被创建，与对数据的动态处理方式是一致的。简化并加速开发过程，而且有利于动态分配命名空间。如果担心数据库或集合被意外创建，可以开启严格模式

> 2、插入语法

```text
1 db.users.insert({username:"smith"})
2 db.users.save({username:"smith"})
```

> 3、查找语法

```text
1 db.users.find()
2 db.users.count()
```

> 4、更新语法

```text
 1 db.users.update({username:"smith"},{$set:{country:"Canada"}})
 2 //把用户名为smith的用户的国家改成Canada
 3
 4 db.users.update({username:"smith"},{$unset:{country:1}})
 5 //把用户名为smith的用户的国家字段给移除
 6
 7 db.users.update({username:"jones"},{$set:{favorites:{movies:["casablance","rocky"]}}})
 8 //这里主要体现多值修改，在favorties字段中添加多个值
 9
10 db.users.update({"favorites.movies":"casablance"},{$addToSet:{favorites.movies:"the maltese"}},false,true)
11 //多项更新
```

> 5、删除语法

```text
1 db.foo.remove() //删除所有数据
2 db.foo.remove({favorties.cities:"cheyene"}) //根据条件进行删除
3 db.drop() //删除整个集合
```

> 6、索引相关语法

```text
1 db.numbers.ensureIndex({num:1})
2 //创建一个升序索引
3 db.numbers.getIndexes()
4 //获取全部索引
```

> 7、基本管理语法

```text
 1 show dbs
 2 //查询所有数据库
 3 show collections
 4 //显示所有表
 5 db.stats()
 6 //显示数据库状态信息
 7 db.numbers.stats()
 8 //显示集合表状态信息
 9 db,shutdownServer()
10 //停止数据库
11 db.help()
12 //获取数据库操作命令
13 db.foo.help()
14 //获取表操作命令
15 tab 键 //能自动帮我们补全命令
```

## MongoDB基础操作
