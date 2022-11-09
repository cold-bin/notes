---
title: "MySQL数据库经验总结"
date: 2022-08-22T12:05:55+08:00
tags: [数据库]
categories: [MySQL,数据库]
draft: true

---

# MySQL数据库经验总结

## 数据库表设计

### 字段设计要点

- 建表时最好不要出现`null`字段，因为MySQL的很多运算符遇到`null`值时都会失效，诸如`like`、`in`、`=`等
- 主键设计
- 外键设计

## 查询要点

- `distinct`和`group by`效率？

- 聚合函数可以放在哪里？

  select字段，还有是having的过滤条件。where里不能使用聚合函数作为条件

- 
