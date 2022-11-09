---
title: "Flag包的在go项目的实践"
date: 2022-07-17T10:16:08+08:00
tags: [命令行参数解析]
categories: [Golang]
draft: false
---

阅读本文前，需具备[flag基础](https://liuhaibin123456789.github.io/post/Go%E8%AF%AD%E8%A8%80%E6%A0%87%E5%87%86%E5%BA%93flag%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8/)

通常我们有种需求：可以在不修改程序源码的情况下，控制一些程序内部的变化。比如配置文件，我们可以手动更改配置文件，也可以在程序启动时，通过命令行参数在程序运行时，给程序提供一些配置选项，这样可以方便地不需要更改程序源码就可以达到在外部控制程序内部一些变化。

以下是通过命令行操作给项目地配置文件进行动态指定，这样就可以不用手动修改配置文件了（比较在非图形化的 Linux 上部署服务，直接通过命令行实现配置文件内容变化算是比直接修改要方便一点点

```go
// @author cold bin
// @date 2022/7/17

package main

import (
	"flag"
	"fmt"
	"log"
	"time"
)

/*
	命令行操作结合配置文件进行操作,命令行提供默认操作
*/

func main() {
	var port, mode, StartTime, loglevel string
	//使用命令前需要先解析
	flag.Parse()
	args := flag.Args()
	fmt.Println(args)
	if len(args) <= 0 {
		log.Println("请输入命令参数")
		return
	}

	//子命令
	switch args[0] {
	case "conf":
		cmdSet := flag.NewFlagSet("conf", flag.ExitOnError)

		cmdSet.StringVar(
			&port,
			"port",
			"8080",
			"指定项目运行监听端口",
		)
		cmdSet.StringVar(
			&port,
			"p",
			"8080",
			"指定项目运行监听端口",
		)

		cmdSet.StringVar(
			&mode,
			"mode",
			"debug",
			"指定项目启动模式，默认debug为开发模式，release为发布模式，test为测试模式",
		)
		cmdSet.StringVar(
			&mode,
			"m",
			"debug",
			"指定项目启动模式，默认debug为开发模式，release为发布模式，test为测试模式",
		)

		cmdSet.StringVar(
			&StartTime,
			"time",
			time.Now().Format("2006-01-02"),
			"项目的启动时间，未指定时，默认当前系统时间",
		)
		cmdSet.StringVar(
			&StartTime,
			"t",
			time.Now().Format("2006-01-02"),
			"项目的启动时间，未指定时，默认当前系统时间",
		)

		cmdSet.StringVar(
			&loglevel,
			"logLevel",
			"debug",
			"默认日志是warning级别，还有debug、error级别",
		)
		cmdSet.StringVar(
			&loglevel,
			"l",
			"debug",
			"默认日志是warning级别，还有debug、error级别",
		)
		//使用命令前需要解析
		_ = cmdSet.Parse(args[1:])

		fmt.Println(port, mode, StartTime, loglevel)
	case "java":
		log.Println("java命令")
	default:
		log.Fatal("命令错误")
	}
}
```



当然也可以将出现地常量或字符串进行更好的迁移，使用有意义地变量名代替阅读观感更佳
