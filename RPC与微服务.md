---
title: "RPC与微服务"
date: 2022-08-21T13:52:34+08:00
tags: [GRPC,微服务]
categories: [Golang]
draft: false
---

# RPC与微服务

## 一、RPC

### 1. 什么是RPC

1. RPC(即：Remote Procedure Call) **远程过程调用**，简单地理解是一个节点请求另一个节点提供的服务。当然这两个节点可能部署在不同的主机上，也可能是相同的主机上，但是两个节点是**进程隔离级别**。

2. 对应RPC的是，**本地过程调用**。调用本地代码里的某个函数就是最常见的本地过程调用，显然本地过程调用不是进程隔离级别，而是共享在一个进程。

3. 将本地过程调用编程远程过程调用会面临各种问题

   - Call的id映射

     假设现在我们要RPC调用Multiply函数实现功能。那我们怎么告诉远程机器我们要调用Multiply，而不是Add或者FooBar呢？在本地过程调用中，函数体是直接通过函数指针来指定的，我们调用Multiply，编译器就自动帮我们调用它相应的函数指针。但是在远程调用中，函数指针是不行的，因为两个进程的地址空间是完全不一样的。所以，**在RPC中，所有的函数都必须有自己的一个ID，这个ID在所有进程中都是唯一确定的。客户端在做远程过程调用时，必须附上这个ID。**然后我们还需要在客户端和服务端分别维护一个 (函数 <=> Call ID) 的对应表。两者的表不一定需要完全相同，但相同的函数对应的Call ID必须相同。当客户端需要进行远程调用时，它就查一下这个表，找出相应的Call ID，然后把它传给服务端，服务端也通过查表，来确定客户端需要调用的函数，然后执行相应函数的代码。

   - 序列化与反序列化

     客户端怎么把参数值传给远程的函数呢？在本地调用中，我们只需要把参数压到栈里，然后让函数自己去栈里读就行。但是在远程过程调用时，客户端跟服务端是不同的进程，不能通过内存来传递参数，甚至有时候客户端和服务端使用的都不是同一种语言（比如服务端用C++，客户端用Java或者Python）。这时候就需要客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。同理，从服务端返回的值也需要序列化反序列化的过程。

     **序列化**：将某种格式的数据转化为字节流

     **反序列化**：将字节流数据转化为某种格式的数据

     例如json序列化与反序列化，json序列化指先将json格式的数据序列化未字节流数据；json反序列化指将字节流数据提取出来并解读成json这种格式

   - 网络传输

     远程调用往往用在网络上，客户端和服务端是通过网络连接的。所有的数据都需要通过网络传输，因此就需要有一个网络传输层。网络传输层需要把Call ID和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。只要能完成这两者的，都可以作为传输层使用。因此，它所使用的协议其实是不限的，能完成传输就行。尽管大部分RPC框架都使用TCP协议，但其实UDP也可以，而gRPC干脆就用了HTTP2。Java的Netty也属于这层的东西。

4. 解决了上面三个机制，就能实现RPC了，具体过程如下：

   1. client端RPC过程

      ```txt
      1. 将这个调用映射为Call ID。这里假设用最简单的字符串当Call ID的方法
      2. 将Call ID和传入的参数序列化。可以直接将它们的值以二进制形式打包
      3. 把步骤2中得到的数据包发送给ServerAddr，这需要使用网络传输层
      4. 等待服务器返回结果
      4. 如果服务器调用成功，那么就将结果反序列化
      ```

   2. server端RPC过程

      ```txt
      1. 在本地维护一个Call ID到函数指针的映射call_id_map，可以用dict完成
      2. 等待请求，包括多线程的并发处理能力，能够并发地RPC
      3. 得到一个请求后，将其数据包反序列化，得到Call ID
      4. 通过在call_id_map中查找，得到相应的函数指针
      5. 将参数反序列化后，在本地调用Call ID对应的函数，得到结果
      6. 将结果序列化后通过网络返回给Client
      ```

5. 对于上面三个机制地实现可以参考

   - Call ID映射可以直接使用函数字符串，也可以使用整数ID，保证所有进程里的函数ID都唯一即可。映射表一般就是一个哈希表，毕竟哈希表的查询时间复杂度为O(1)。
   - 序列化反序列化可以自己写，也可以使用Protobuf（这种方式序列化比http传输数据更加轻量与高效）或者FlatBuffers之类的。
   - 网络传输库可以自己写socket，或者用tcp、udp或者http之类的。

6. 实际上真正的开发过程中，除了上面的基本功能（三个机制）以外还需要更多的细节：

   链路追踪、服务发现、服务熔断、请求限流、服务降级、网络错误、流量控制、超时和重试、动态扩容、DDD领域驱动设计等。



### 2. rpc、http以及restful之间的区别

- restful只能说是一种设计风格，还没有形成一种协议、一种约束，是一种针对资源划分的更好的设计风格。

- rpc是框架，既不是某种风格。也不是某种协议或规范。rpc调用时，在网络上数据传输的格式，可以是http协议的格式，还可以是json格式，还可以是xml格式，当然还可以是更加轻量、更加高效的protobuf协议格式数据

- http其实是一种网络传输协议，基于tcp，**规定了数据传输的格式**。现在客户端浏览器与服务端通信基本都是采用http协议。也可以用来进行远程服务调用。缺点是消息封装臃肿。现在热门的Restful风格，就可以通过http协议来实现



### 3. 基于json的rpc

**服务端：**

```go
package main

import (
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

type HelloService struct {}

func (s *HelloService) Hello(request string, reply *string) error {
	*reply = "hello "+ request
	return nil
}

func main(){
	rpc.RegisterName("HelloService", new(HelloService))
	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		panic("启动错误")
	}
	for {
		conn, err := listener.Accept()
		if err != nil {
			panic("接收")
		}
		go rpc.ServeCodec(jsonrpc.NewServerCodec(conn)) // 服务端使用json编解码
	}
}
```

**客户端：**

```go
package main

import (
	"fmt"
	"net"
	"net/rpc"
	"net/rpc/jsonrpc"
)

func main(){
	conn, err := net.Dial("tcp", "localhost:1234")
	if err != nil {
		panic("连接错误")
	}
	client := rpc.NewClientWithCodec(jsonrpc.NewClientCodec(conn))// 客户端需要使用json编解码
	var reply string
	err = client.Call("HelloService.Hello", "imooc", &reply)
	if err != nil {
		panic("调用错误")
	}
	fmt.Println(reply)
}
```



### 4. 基于http的rpc

**服务端：**

```go
func main() {
    rpc.RegisterName("HelloService", new(HelloService))
    http.HandleFunc("/jsonrpc", func(w http.ResponseWriter, r *http.Request) {
        var conn io.ReadWriteCloser = struct {
            io.Writer
            io.ReadCloser
        }{
            ReadCloser: r.Body,
            Writer:     w,
        }
        rpc.ServeRequest(jsonrpc.NewServerCodec(conn)) // 将单次请求转化为rpc
    })
    http.ListenAndServe(":1234", nil)
}
```



> 显然，以上基于json、http的rpc封装性不是很好，缺乏一个更好的框架。



## 二、GRPC和protobuf

### 1. 什么是GRPC、protobuf

GRPC是一个高性能、开源和通用的 RPC 框架，面向移动和 HTTP/2 设计。

GRPC调用如下图：

![image.png](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202208201128237.png)

protoBuf是**结构数据序列化的方法**，可简单类比于XML，其具有以下特点：

- **语言无关、平台无关**。即 protoBuf 支持 Java、C++、Python 等多种语言，支持多个平台
- **高效**。即比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单
- **扩展性、兼容性好**。你可以更新数据结构，而不影响和破坏原有的旧程序



### 2. protobuf语法指南

现阶段比较流行的是proto3。通过编写少许的proto代码，然后通过插件就可以自动生成服务端的GRPC调用框架，只需添加服务的具体实现逻辑即可。

[proto3](https://developers.google.com/protocol-buffers/docs/proto3)



### 3. GRPC demo

#### 安装grpc和protobuf

用go mod一键sync一下两个库

```go
import(
    "google.golang.org/grpc"
    "google.golang.org/protobuf"
)
```

#### protocol buffer编译器

这个编译器可以单独下载，但我们也可以使用Goland里面的`protocol buffer`的编译器插件

```plain
file->settings->plugins->搜protocol
```

#### 安装protoc

到[protobuf release](https://github.com/protocolbuffers/protobuf/releases)，选择适合自己操作系统的压缩包文件

将解压后得到的`protoc`二进制文件移动到$GOPATH/bin里

不会有人不知道$GOPATH/bin，也没有将这个文件夹加到path环境变量里吧

#### go的protoc编辑器插件

具体可以看[其他网上教程](https://www.cnblogs.com/hongjijun/p/13724738.html)

```plain
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```

#### demo

我们把讲protobuf语法时的示例文件作为例子，将其改名为`login.proto`并执行以下命令：

```latex
protoc --go_out=. ./login.proto
protoc --go-grpc_out=. ./login.proto
```

这两条命令会生成两个文件`login.pb.go`和`login_grpc.pb.go`(放在项目的proto文件夹里)，服务端和客户端都需要这两个文件。

`login.pb.go`放置了将`login.proto`里的结构翻译成go语言结构体和相关东西。

```go
type LoginReq struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	UserName string `protobuf:"bytes,1,opt,name=UserName,proto3" json:"UserName,omitempty"`
	PassWord string `protobuf:"bytes,2,opt,name=PassWord,proto3" json:"PassWord,omitempty"`
}

type LoginResp struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	OK bool `protobuf:"varint,1,opt,name=OK,proto3" json:"OK,omitempty"`
}
```

`login_grpc.pb.go`放置了gRPC框架封装好的逻辑

```go
func NewBiliClient(cc grpc.ClientConnInterface) BiliClient {
	return &biliClient{cc}
}

func (c *biliClient) Login(ctx context.Context, in *LoginReq, 
                           opts ...grpc.CallOption) (*LoginResp, error) {
	out := new(LoginResp)
	err := c.cc.Invoke(ctx, "/grpcinclass.Bili/Login", in, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

type BiliServer interface {
	Login(context.Context, *LoginReq) (*LoginResp, error)
	mustEmbedUnimplementedBiliServer()
}

func RegisterBiliServer(s grpc.ServiceRegistrar, srv BiliServer) {
	s.RegisterService(&Bili_ServiceDesc, srv)
}
```

接下来使用下面的server代码起一个rpc服务端：

```go
//server（被调用rpc的一方）
package main

import (
	"context"
	"google.golang.org/grpc"
	"log"
	"net"
)

const (
	port = ":50051"
)

func main() {
    // 监听端口
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	s := grpc.NewServer() //获取新服务示例
	proto.RegisterBiliServer(s, &server{})

    // 开始处理
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}

type server struct {
	proto.UnimplementedBiliServer // 用于实现proto包里BiliServer接口
}

func (s *server) Login(ctx context.Context, req *proto.LoginReq) (*proto.LoginResp, error) {
	resp := &proto.LoginResp{}
	log.Println("recv:", req.UserName, req.PassWord)
	if req.PassWord != GetPassWord(req.UserName) {
		resp.OK = false
		return resp, nil
	}
	resp.OK = true
	return resp, nil
}

func GetPassWord(userName string) (password string) {
	return userName + "123456"
}
```

接下来使用下面的client代码起一个rpc客户端：

```go
//client（调用rpc的一方）
package main

import (
	"context"
	"fmt"
	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"log"
)

const (
	address = "localhost:50051"
)

func main() {
	//建立链接
	conn, err := grpc.Dial(address, grpc.WithTransportCredentials(insecure.NewCredentials()))
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	c := proto.NewBiliClient(conn)

	for {
		//这段不重要
		fmt.Println("input username&password:")
		iptName := ""
		_, _ = fmt.Scanln(&iptName)
		iptPassword := ""
		_, _ = fmt.Scanln(&iptPassword)

		loginResp, _ := c.Login(context.Background(), &proto.LoginReq{
			UserName: iptName,
			PassWord: iptPassword,
		})

		if loginResp.OK {
			fmt.Println("success")
			break
		}
		fmt.Println("retry")
	}
}
```

> protobuf 3中还有一种数据类型——steam（流），其用于传输流式数据，感兴趣可以参考[这篇文章](https://segmentfault.com/a/1190000016503114)(这篇文章是我随便找的用于简要了解)



### 4. protobuf类型补充

前面的proto官网语法介绍里包含了许多丰富的类型，但是并没有将所有常用的类型包含，包括时间戳、时间段、还有任意类型消息。类型足够丰富，才能更好地描述一次行为产生的数据。

#### Timestamp类型

**源码包：**

`google/protobuf/timestamp.proto`里

```protobuf
message Timestamp {
	int64 seconds = 1; // 时间戳秒数
	int32 nanos = 2; // 纳秒数
}
```

可以先导入这个消息定义的源码包后再引用，这是protobuf协议里官方自己给出的时间戳表示法，需要使用到时间类型时，可以考虑嵌入该类型。

```protobuf
message Msg {
	int64 id=1;
	string content=2;
	google.protobuf.Timestamp at_time=3;
}
```

使用插件编译自动生成代码时，时间类型`*timestamp.Timestamp`只是用来表示时间的结构体，和go表示时间的基本类型显然**不一致**。当然，google官方提供了转化方法。

```go
type Msg struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields
    ...//其他字段
    AtTime *timestamp.Timestamp `protobuf:"bytes,1,opt,name=birthday,proto3" json:"birthday,omitempty"`
}
```



#### Duration类型

与Timestamp类型类似



### 5. GRPC的metadata和RPC自定义认证

#### metadata介绍

在http请求当中我们可以设置header用来传递数据，grpc底层采用http2协议也是支持传递数据的，采用的是metadata。 Metadata 对于 gRPC 本身来说透明， 它使得 client 和 server 能为对方提供本次调用的信息。就像一次 http 请求的 RequestHeader 和 ResponseHeader，http header 的生命周期是一次 http 请求， Metadata 的生命周期则是一次 RPC调用。

在http/1.1协议里，header是明确存在请求报文里，那么GRPC的header在哪里呢？

答案：GRPC的header是metadata，metadata在代码里是放在上下文中存储数据，在网络传输上是放在HEADERS帧字段的几个字节上。

在 gRPC 中，Metadata 实际上就是一个 map 结构，其原型如下：

```go
type MD map[string][]string
```

是一个字符串与字符串切片的映射结构。

#### metadata创建

在 `google.golang.org/grpc/metadata` 中分别提供了两个方法来创建 metadata，第一种是 `metadata.New` 方法，如下：

```go
metadata.New(map[string]string{"go": "programming", "tour": "book"})
```

使用 New 方法所创建的 metadata，将会直接被转换为对应的 MD 结构，参考结果如下：

```go
go:   []string{"programming"}
tour: []string{"book"}
```

第二种是 `metadata.Pairs` 方法，如下：

```go
metadata.Pairs(
    "go", "programming",
    "tour", "book",
    "go", "eddycjy",
)
```

使用 Pairs 方法所创建的 metadata，将会以奇数来配对，并且所有的 Key 都会被默认转为小写，若出现同名的 Key，将会追加到对应 Key 的切片（slice）上，参考结果如下：

```go
go:   []string{"programming", "eddycjy"}
tour: []string{"book"}
```

#### 设置/获取metadata

```go
ctx := context.Background()
md := metadata.New(map[string]string{"go": "programming", "tour": "book"})

newCtx1 := metadata.NewIncomingContext(ctx, md)
newCtx2 := metadata.NewOutgoingContext(ctx, md)
```

在 gRPC 中对于 metadata 进行了区别，分为了传入和传出用的 metadata，这是为了防止 metadata 从入站 RPC 转发到其出站 RPC 的情况，针对此提供了两种方法来分别进行设置，如下：

- NewIncomingContext：创建一个附加了所传入的 md 新上下文，仅供自身的 gRPC 服务端内部使用。
- NewOutgoingContext：创建一个附加了传出 md 的新上下文，可供外部的 gRPC 客户端、服务端使用。

因此相对的在 metadata 的获取上，也区分了两种方法，分别是 FromIncomingContext 和 NewOutgoingContext，与设置的方法所相对应的含义，如下：

```go
md1, _ := metadata.FromIncomingContext(ctx)
md2, _ := metadata.FromOutgoingContext(ctx)
```

那么总的来说，这两种方法在实现上有没有什么区别呢，我们可以一起深入看看：

```go
type mdIncomingKey struct{}
type mdOutgoingKey struct{}

func NewIncomingContext(ctx context.Context, md MD) context.Context {
	return context.WithValue(ctx, mdIncomingKey{}, md)
}

func NewOutgoingContext(ctx context.Context, md MD) context.Context {
	return context.WithValue(ctx, mdOutgoingKey{}, rawMD{md: md})
}
```

实际上主要是在内部进行了 Key 的区分，以所指定的 Key 来读取相对应的 metadata，以防造成脏读，其在实现逻辑上本质上并没有太大的区别。另外大家可以看到，其对 Key 的设置，是用一个结构体去定义的，这是 Go 语言官方一直在推荐的写法，建议大家也这么写。

#### 实际使用场景

在上面我们已经介绍了关键的 metadata 以及其相对的 IncomingContext、OutgoingContext 类别的相关方法.

那么我们回过来想，假设我现在有一个 ServiceA 作为服务端，然后有一个 Client 去调用 ServiceA，我想传入我们自定义的 metadata 信息，那我们应该怎么写才合适，流程图如下：

![image](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202208201527778.jpeg)

在常规情况下，我们在 `ServiceA` 的服务端，应当使用 `metadata.FromIncomingContext` 方法进行读取，如下：

```go
func (t *ServiceA) GetXXX(ctx context.Context, r *pb.GetXXXRequest) (*pb.GetXXXReply, error) {
	md, _ := metadata.FromIncomingContext(ctx)
	log.Printf("md: %+v", md)
	...
}
```

而在 Client，我们应当使用 `metadata.AppendToOutgoingContext` 方法，如下：

```go
func main() {
	ctx := context.Background()
	newCtx := metadata.AppendToOutgoingContext(ctx, "cookie", "asdfghjkll")
	
	clientConn, _ := GetClientConn(newCtx, ...)
	defer clientConn.Close()
    
	xxxClient := pb.NewXXXClient(clientConn)
	resp, _ := xxxClient.GetXXX(newCtx, &pb.GetXXXRequest{Name: "xxx"})
	...
}
```

这里需要注意一点，在新增 metadata 信息时，务必使用 Append 类别的方法，否则如果直接 New 一个全新的 md，将会导致原有的 metadata 信息丢失（除非你确定你希望得到这样的结果）。



### 6. GRPC拦截器

我想在每一个RPC方法的前面或后面做某些操作，我想对RPC方法进行鉴权校验，我想对RPC方法进行上下文的超时控制，我想对每个RPC方法的请求都做日志记录，我想对每个RPC请求做一些头部帧数据监控，避免恶意爬虫...等等一些针对某个业务模块的RPC方法进行**统一的特殊处理**。（类似于gin框架中间件的意思）

这诸如类似的一切需求的答案在拦截器（Interceptor）上，你能够借助它实现许许多多的定制功能且不直接侵入业务代码。

#### 拦截器类型

1. **客户端**

   1. **一元拦截器**

      客户端的一元拦截器类型为 `UnaryClientInterceptor`，方法原型如下：

      ```go
      type UnaryClientInterceptor func(
          ctx context.Context, // 上下文
          method string, //调用方法
          req, //请求参数
          reply interface{},//响应结果数据 
          cc *ClientConn, //连接指针实体
          invoker UnaryInvoker, //调用程序的实体
          opts ...CallOption,// 调用的配置
      ) error
      ```

      一元拦截器的实现通常可以分为三个部分：预处理，调用RPC方法和后处理。

   2. **流拦截器**

      客户端的流拦截器类型为 `StreamClientInterceptor`，方法原型如下：

      ```go
      type StreamClientInterceptor func(
          ctx context.Context, 
          desc *StreamDesc, 
          cc *ClientConn, 
          method string, 
          streamer Streamer, 
          opts ...CallOption,
      ) (ClientStream, error)
      ```

      流拦截器的实现包括预处理和流操作拦截，并不能在事后进行 RPC 方法调用和后处理，而是拦截用户对流的操作.

2. **服务端**

   1. **一元拦截器**

      服务端的一元拦截器类型为 `UnaryServerInterceptor`，方法原型如下：

      ```go
      type UnaryServerInterceptor func(
          ctx context.Context, 
          req interface{}, 
          info *UnaryServerInfo, 
          handler UnaryHandler,
      ) (resp interface{}, err error)
      ```

      其一共包含四个参数，分别是RPC上下文、RPC方法的请求参数、RPC方法的所有信息、RPC方法本身。

   2. **流拦截器**

      服务端的流拦截器类型为 `StreamServerInterceptor`，方法原型如下：

      ```go
      type StreamServerInterceptor func(
          srv interface{}, 
          ss ServerStream, 
          info *StreamServerInfo, 
          handler StreamHandler,
      ) error
      ```

#### 实现一个拦截器

如何实现一个拦截器呢？在启动一个GRPC服务端时，可以通过配置GRPC服务端的GRPC拦截器，拦截器的具体逻辑只需要自己实现即可。

```go
opts := []grpc.ServerOption{
	grpc.UnaryInterceptor(XXXInterceptor),
}
s := grpc.NewServer(opts...)// 函数选项模式

// 实现XXXInterceptor逻辑
func XXXInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, 
                    handler grpc.UnaryHandler) (interface{}, error) {
	log.Println("be pending") //RPC调用前
    // 还可以获取rpc请求头的元数据是否合乎预期。例如可以做：认证与鉴权、还可以识别RPC请求来源并做一些限制等等
	resp, err := handler(ctx, req) // RPC的处理方法
	log.Println("be processed") // RPC调用后
	return resp, err
}
```

上面代码就是配置了一个`XXXInterceptor`的GRPC一元拦截器。`XXXInterceptor`的逻辑需要自己实现。

对流的拦截器逻辑一样，实现对应的接口方法即可。

#### 如何使用多个拦截器

理论上是不能直接多次配置单个拦截器来实现多拦截，**以下代码会报错**

```go
opts := []grpc.ServerOption{
	grpc.UnaryInterceptor(XXX1Interceptor),
    grpc.UnaryInterceptor(XXX2Interceptor),
    grpc.UnaryInterceptor(XXX3Interceptor),
}
s := grpc.NewServer(opts...)// 函数选项模式
```

这里需要使用第三方库来实现多个拦截器的逻辑，安装：

```bash
go get -u github.com/grpc-ecosystem/go-grpc-middleware
```

使用

```go
import grpc_middleware "github.com/grpc-ecosystem/go-grpc-middleware"
...
opts := []grpc.ServerOption{
		grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(
			XXX1Interceptor,
			XXX2Interceptor,
            XXX3Interceptor,
		)),
	}
s := grpc.NewServer(opts...)
...
```

第三方库实现的原理

O_o 没看懂

GRPC社区生态还有很多好用的middlware，诸如：`grpc_zap`、`grpc_auth`、`grpc_recovery`等等

github速通 --> https://github.com/grpc-ecosystem/go-grpc-middleware



#### GRPC自定义认证

关于GRPC token认证鉴权，其实可以参照http/1.1来进行设计：在metadata里放入token信息，每次请求都需要携带这个metadata token信息，服务端可以写一个拦截器专门用来token拦截认证。

当然这样的逻辑，官方已经封装好了如下接口

```go
type PerRPCCredentials interface {
    GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error)// 获取当前请求认证所需的元数据（metadata）
    RequireTransportSecurity() bool// 是否需要基于 TLS 认证进行安全传输（https）
}
```

实现如上两个接口的结构体`auth`，就可以在客户端拨号连接服务端时，配置一个认证的`grpc.WithPerRPCCredentials(&auth)`的连接选项。这样，每次拨号连接的时候，会将每个RPC上下文里塞入想要的认证信息。服务端只需要统一拦截请求，检查请求的metadata是否存在需要的认证信息即可。

**客户端代码**

```go
type Auth struct {
	AppKey    string
	AppSecret string
}

func (a *Auth) GetRequestMetadata(ctx context.Context, uri ...string) (map[string]string, error) {
	return map[string]string{"app_key": a.AppKey, "app_secret": a.AppSecret}, nil
}

func (a *Auth) RequireTransportSecurity() bool {
	return false
}

func main() {
	auth := Auth{
		AppKey:    "auth",
		AppSecret: "XXXXXXXXXX",// 例如token
	}
	ctx := context.Background()
	opts := []grpc.DialOption{grpc.WithPerRPCCredentials(&auth)}
	clientConn, err := GetClientConn(ctx, "localhost:8004", opts)
	if err != nil {
		log.Fatalf("err: %v", err)
	}
	defer clientConn.Close()
	...
}
...
```

**服务端代码**：

貌似使用拦截器进行认证时，意味着需要对所有RPC请求做出统一拦截和限制。这与gin框架的中间件有点区别，gin的中间件可以针对部分路由实现拦截，而GRPC的拦截器只能对所有的RPC请求做出限制，不能只对某一部分RPC请求作出限制。所以，也可以不用拦截器来实现服务端的同意拦截与认证。可以侵入式地给每一个需要认证的RPC补上认证的逻辑即可。

```go
// 实现AuthInterceptor逻辑，用于配置服务端token认证拦截器
func AuthInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, 
                    handler grpc.UnaryHandler) (res interface{},err error) {
    //拦截普通方法请求，验证token
	if err = auth(ctx);err != nil {
		return
	}
	// 继续处理请求
	return handler(ctx, req)
}

func auth(ctx context.Context) error {
	md, ok := metadata.FromIncomingContext(ctx)
	if !ok {
		return fmt.Errorf("missing credentials")
	}
	var auth string
	var token string

	if val, ok := md["app_key"]; ok {
		auth = val[0]
	}
	if val, ok := md["app_secret"]; ok {
		token = val[0]
	}

	if auth != "auth" || token != "XXXXXXXXXX" {
		return status.Errorf(codes.Unauthenticated, "客户端请求的token不合法")
	}
	return nil
}
```



### 7. GRPC验证器

github速通用法 --> https://github.com/mwitkow/go-proto-validators

类似于gin的`validator`验证器，有了验证器，我们可以在用户发起请求并携带参数时，可以更加**优雅地**检查入参是否符合预期。

#### 上手使用

这里使用第三方插件[go-proto-validators](https://github.com/mwitkow/go-proto-validators)自动生成验证规则。

```bash
go get github.com/mwitkow/go-proto-validators
```

1. 新建simple.proto文件

   ```protobuf
   syntax = "proto3";
   
   package proto;
   
   import "github.com/mwitkow/go-proto-validators/validator.proto";
   
   message InnerMessage {
     // some_integer can only be in range (1, 100).
     int32 some_integer = 1 [(validator.field) = {int_gt: 0, int_lt: 100}];
     // some_float can only be in range (0;1).
     double some_float = 2 [(validator.field) = {float_gte: 0, float_lte: 1}];
   }
   
   message OuterMessage {
     // important_string must be a lowercase alpha-numeric of 5 to 30 characters (RE2 syntax).
     string important_string = 1 [(validator.field) = {regex: "^[a-z]{2,5}$"}];
     // proto3 doesn't have `required`, the `msg_exist` enforces presence of InnerMessage.
     InnerMessage inner = 2 [(validator.field) = {msg_exists : true}];
   }
   
   service Simple{
     rpc Route (InnerMessage) returns (OuterMessage){};
   }
   ```

   代码`import "github.com/mwitkow/go-proto-validators/validator.proto"`，文件`validator.proto`需要`import "google/protobuf/descriptor.proto";`包，不然会报错。`google/protobuf`地址：https://github.com/protocolbuffers/protobuf/tree/master/src/google/protobuf/descriptor.proto。把`src`文件夹中的`protobuf`目录下载到GOPATH目录下。

2. 编译simple.proto文件

   ```bash
   go get github.com/mwitkow/go-proto-validators/protoc-gen-govalidators
   ```

   指令编译：`protoc --govalidators_out=. --go-grpc_out=./ --go_out=./ ./simple.proto`

   编译完成后，自动生成`simple.pb.go`和`simple.validator.pb.go`文件，`simple.pb.go`文件不再介绍，我们看下`simple.validator.pb.go`文件，里面自动生成了`message`中属性的验证规则。

3. 然后把`grpc_validator`验证拦截器添加到服务端

   ```go
   grpcServer := grpc.NewServer(
   	grpc.StreamInterceptor(grpc_middleware.ChainStreamServer(
   			grpc_validator.StreamServerInterceptor(),
   	        grpc_auth.StreamServerInterceptor(auth.AuthInterceptor),
   			grpc_zap.StreamServerInterceptor(zap.ZapInterceptor()),
   			grpc_recovery.StreamServerInterceptor(recovery.RecoveryInterceptor()),
   		)),
   	grpc.UnaryInterceptor(grpc_middleware.ChainUnaryServer(
   		    grpc_validator.UnaryServerInterceptor(),
   		    grpc_auth.UnaryServerInterceptor(auth.AuthInterceptor),
   			grpc_zap.UnaryServerInterceptor(zap.ZapInterceptor()),
               grpc_recovery.UnaryServerInterceptor(recovery.RecoveryInterceptor()),
   		)),
   )
   ```

4. 运行后，当输入数据验证失败后，会有以下错误返回

   ```bash
   Call Route err: rpc error: code = InvalidArgument desc = invalid field SomeInteger: value '101' must be less than '100'
   ```

5. 为了更友好的针对参数错误的处理，针对所有RPC调用返回的错误，如果是validator的参数不匹配的错误，则表明参数出现不符合服务端预期，所以，需要返回一个对应的提示。这个可以结合拦截器对所有RPC，做一个统一的校验处理.

   ```go
   type Validator interface{
       Validate() error
   }
   
   // 实现ValidatorInterceptor逻辑，用于验证器优雅地拦截
   func ValidatorInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, 
                       handler grpc.UnaryHandler) (res interface{},err error) {
       //拦截普通方法请求，只针对使用validator规则的rpc进行校验
       if r,ok:=req.(Validator);ok{ // 将空接口断言成具有Validate方法的实体类型
           if err:=r.Validate();err!=nil{// 参数有问题
               return nil,status.Error(codes.InvalidArgument,err.Error())// 错误的兼容处理
           }
       }
       
   	// 继续处理请求
   	return handler(ctx, req)
   }
   
   ```

   

#### 其他类型验证规则设置

`enum`验证

```protobuf
Copysyntax = "proto3";
package proto;
import "github.com/mwitkow/go-proto-validators/validator.proto";// 也可以将这个proto文件拉下来

message SomeMsg {
  Action do = 1 [(validator.field) = {is_in_enum : true}];
}

enum Action {
  ALLOW = 0;
  DENY = 1;
  CHILL = 2;
}
```

`UUID`验证

```protobuf
syntax = "proto3";
package proto;
import "github.com/mwitkow/go-proto-validators/validator.proto";

message UUIDMsg {
  // user_id must be a valid version 4 UUID.
  string user_id = 1 [(validator.field) = {uuid_ver: 4, string_not_empty: true}];
}
```

### 8. GRPC状态码及错误处理

#### GRPC状态码

那我们更细致来看，这些 gRPC 的内部状态又分别有哪些呢，目前官方给出的全部状态响应码如下：

| Code | Status              | Notes                                        |
| ---- | ------------------- | -------------------------------------------- |
| 0    | OK                  | 成功                                         |
| 1    | CANCELLED           | 该操作被调用方取消                           |
| 2    | UNKNOWN             | 未知错误                                     |
| 3    | INVALID_ARGUMENT    | 无效的参数                                   |
| 4    | DEADLINE_EXCEEDED   | 在操作完成之前超过了约定的最后期限。         |
| 5    | NOT_FOUND           | 找不到                                       |
| 6    | ALREADY_EXISTS      | 已经存在                                     |
| 7    | PERMISSION_DENIED   | 权限不足                                     |
| 8    | RESOURCE_EXHAUSTED  | 资源耗尽                                     |
| 9    | FAILED_PRECONDITION | 该操作被拒绝，因为未处于执行该操作所需的状态 |
| 10   | ABORTED             | 该操作被中止                                 |
| 11   | OUT_OF_RANGE        | 超出范围，尝试执行的操作超出了约定的有效范围 |
| 12   | UNIMPLEMENTED       | 未实现                                       |
| 13   | INTERNAL            | 内部错误                                     |
| 14   | UNAVAILABLE         | 该服务当前不可用。                           |
| 15   | DATA_LOSS           | 不可恢复的数据丢失或损坏。                   |

那么对应在我们刚刚的调用结果，状态码是 UNKNOWN，这是为什么呢，我们可以查看底层的处理源码，如下：

```go
func FromError(err error) (s *Status, ok bool) {
	...
	if se, ok := err.(interface {// 匿名接口
		GRPCStatus() *Status
	}); ok {
		return se.GRPCStatus(), true
	}
	return New(codes.Unknown, err.Error()), false
}
```

我们可以看到，实际上若不是含有`GRPCStatus`方法的类型，都是默认返回`codes.Unknown`，也就是未知。也就是说，我们需要做一层兼容处理，让原生的`Error`类型转化为对应的类型。可以使用`status.Error(codes.InvalidArgument,err.Error())`来进行兼容处理。同时，也可以自己是实现一个`MyError`类型实现`GRPCStatus`、`Error`等方法，就可以实现无缝衔接（隐式接口的魅力）。

#### 错误处理

如上，我们可以自定义自己的错误码无缝衔接至GRPC框架中使用。

### 9. GRPC的超时机制

超时时间的设置和适当控制，是在微服务架构中非常重要的一个**保全项**。

我们假设一个应用场景，你有多个服务，他们分别是 A、B、C、D，他们之间是最简单的关联依赖，也就是 A=>B=>C=>D。在某一天，你有一个需求上线了，修改的代码内容正正好就是与服务 D 相关的，恰好这个需求就对应着一轮业务高峰的使用，但你发现不知道为什么，你的服务 A、B、C、D 全部都出现了响应缓慢，整体来看，开始出现应用系统雪崩….这到底是怎么了？

从根本上来讲，是服务D出现了问题，所导致的这一系列上下游服务出现**连锁反应**，因为在服务调用中默认没有设置超时时间，或者所设置的超时时间过长，都会导致多服务下的整个调用链雪崩，导致非常严重的事故，因此任何调用的默认超时时间的设置是非常有必要的，在gRPC中更是强调 TL;DR（Too long, Don’t read）并建议始终设定截止日期. **应当给每一个rpc都设置调用的超时时间**.

站在巨人的肩膀上-->go语言标准包`context`.

```go
// 客户端超时拦截器，控制整个调用链的调用时间，如果未在指定时间内返回结果，表明RPC过程中存在超时或不可用的服务
func UnaryContextTimeout() grpc.UnaryClientInterceptor {
	return func(ctx context.Context, method string, req, resp interface{}, cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
		ctx, cancel := defaultContextTimeout(ctx)
		if cancel != nil {
			defer cancel()
		}

		return invoker(ctx, method, req, resp, cc, opts...)
	}
}
// 默认超时设置
func defaultContextTimeout(ctx context.Context) (context.Context, context.CancelFunc) {
	var cancel context.CancelFunc
	if _, ok := ctx.Deadline(); !ok {
		defaultTimeout := 60 * time.Second
		ctx, cancel = context.WithTimeout(ctx, defaultTimeout)
	}

	return ctx, cancel
}
```



### 10. 重试

gRPC生态圈中的[grpc_retry拦截器](https://github.com/grpc-ecosystem/go-grpc-middleware).

### 11. GRPC gateway

grpc-gateway是protoc的一个插件 。它读取GRPC服务定义，并生成反向代理服务器，将RESTful JSON API请求转换为GRPC的方式调用。简单来说，给客户端的调用提供了双协议的支持，即**http1.1协议**和**protobuf协议**，可以使用http请求发送json数据，也可以使用RPC请求发送protobuf二进制数据帧来达到调用的目的。（方便测试、方便前端）

![preload](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202208211729914.png)

#### grpc-gateway 介绍和安装

我们需要安装 grpc-gateway 的 protoc-gen-grpc-gateway 插件，安装命令如下：

```shell
$ go get -u github.com/grpc-ecosystem/grpc-gateway/protoc-gen-grpc-gateway@v1.14.5
```

将所编译安装的 Protoc Plugin 的可执行文件从 `$GOPATH` 中移动到相应的 bin 目录下，例如：

```shell
$ mv $GOPATH/bin/protoc-gen-grpc-gateway /usr/local/go/bin/
```

这里的命令操作并非是绝对必须的，主要目的是将二进制文件 protoc-gen-grpc-gateway 移动到 bin 目录下，让其可以执行，确保在 `$PATH` 下，只要达到这个效果就可以了。

#### Proto 文件的处理

##### Proto 文件修改和编译

那么针对 grpc-gateway 的使用，我们需要调整项目 proto 命令下的 tag.proto 文件，修改为如下：

```protobuf
syntax = "proto3";

package proto;

import "proto/common.proto";
import "google/api/annotations.proto";

service TagService {
    rpc GetTagList (GetTagListRequest) returns (GetTagListReply) {
        option (google.api.http) = {
            get: "/api/v1/tags"
        };
    }
}
// 其他http方法和数据的写法，在proto文件源码可以找到："google/api/http.proto"里的注释有
...
```

我们在 proto 文件中增加了 `google/api/annotations.proto` 文件的引入，并在对应的 RPC 方法中新增了针对 HTTP 路由的注解。接下来我们重新编译 proto 文件，在项目根目录执行如下命令：

```shell
$ protoc -I/usr/local/include -I. \
       -I$GOPATH/src \
       -I$GOPATH/src/github.com/grpc-ecosystem/grpc-gateway/third_party/googleapis \
       --grpc-gateway_out=logtostderr=true:. \
       ./proto/*.proto
```

执行完毕后将生成 `tag.pb.gw.go` 文件，也就是目前 `proto` 目录下用.pb.go 和.pb.gw.go 两种文件，分别对应两类功能支持。

我们这里使用到了一个新的 protoc 命令选项 `-I` 参数，它的格式为：`-IPATH, --proto_path=PATH`，作用是指定 `import` 搜索的目录（也就是 Proto 文件中的 import 命令），可指定多个，如果不指定则默认当前工作目录。

另外在实际使用场景中，还有一个较常用的选项参数，`M` 参数，例如 protoc 的命令格式为：`Mfoo/bar.proto=quux/shme`，则在生成、编译 Proto 时将所指定的包名替换为所要求的名字（如：`foo/bar.proto` 编译后为包名为 `quux/shme`），更多的选项支持可执行 `protoc --help` 命令查看帮助文档。

##### annotations.proto 是什么

我们刚刚在 grpc-gateway 的 proto 文件生成中用到了 `google/api/annotations.proto` 文件，实际上它是 googleapis 的产物，在前面的章节我们有介绍过。

另外你可以结合 grpc-gateway 的 protoc 的生成命令来看，你会发现它在 grpc-gateway 的仓库下的 third_party 目录也放了个 googleapis，因此在引用 annotations.proto 时，用的就是 grpc-gateway 下的，这样子可以保证其兼容性和稳定性（版本可控）。

那么 `annotations.proto` 文件到底是什么，又有什么用呢，我们一起看看它的文件内容，如下：

```protobuf
syntax = "proto3";

package google.api;

import "google/api/http.proto";
import "google/protobuf/descriptor.proto";
...
extend google.protobuf.MethodOptions {
  HttpRule http = 72295728;
}
```

查看核心使用的 `http.proto` 文件中的一部分内容，如下：

```protobuf
message HttpRule {
  string selector = 1;
  oneof pattern {
    string get = 2;
    string put = 3;
    string post = 4;
    string delete = 5;
    string patch = 6;
    CustomHttpPattern custom = 8;
  }

  string body = 7;
  string response_body = 12;
  repeated HttpRule additional_bindings = 11;
}
```

总的来说，主要是针对的 HTTP 转换提供支持，定义了 Protobuf 所扩展的 HTTP Option，在 Proto 文件中可用于定义 API 服务的 HTTP 的相关配置，并且可以指定每一个 RPC 方法都映射到一个或多个 HTTP REST API 方法上。

因此如果你没有引入 `annotations.proto` 文件和在 Proto 文件中填写相关 HTTP Option 的话，执行生成命令，不会报错，但也不会生成任何东西。

#### 服务逻辑实现

接下来我们开始实现基于 grpc-gateway 的在同端口下同 RPC 方法提供 gRPC（HTTP/2）和 HTTP/1.1 双流量的访问支持，我们打开项目根目录下的启动文件 main.go，修改为如下代码：

```go
var port string

func init() {
	flag.StringVar(&port, "port", "8004", "启动端口号")
	flag.Parse()
}
```

##### 不同协议的分流

我们调整了这个案例的服务启动端口号，然后继续在 main.go 中写入如下代码：

```go
func grpcHandlerFunc(grpcServer *grpc.Server, otherHandler http.Handler) http.Handler {
	return h2c.NewHandler(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		if r.ProtoMajor == 2 && strings.Contains(r.Header.Get("Content-Type"), "application/grpc") {
			grpcServer.ServeHTTP(w, r)
		} else {
			otherHandler.ServeHTTP(w, r)
		}
	}), &http2.Server{})
}
```

这是一个很核心的方法，重要的分流和设置一共有两个部分，如下：

- gRPC 和 HTTP/1.1 的流量区分：
  - 对 ProtoMajor 进行判断，该字段代表客户端请求的版本号，客户端始终使用 HTTP/1.1 或 HTTP/2。
  - Header 头 Content-Type 的确定：grpc 的标志位 `application/grpc` 的确定。
- gRPC 服务的非加密模式的设置：关注代码中的"h2c"标识，“h2c” 标识允许通过明文 TCP 运行 HTTP/2 的协议，此标识符用于 HTTP/1.1 升级标头字段以及标识 HTTP/2 over TCP，而官方标准库 `golang.org/x/net/http2/h2c` 实现了 HTTP/2 的未加密模式，我们直接使用即可。

在整体的方法逻辑上来讲，我们可以看到关键之处在于调用了 `h2c.NewHandler` 方法进行了特殊处理，`h2c.NewHandler` 会返回一个 `http.handler`，其主要是在内部逻辑是拦截了所有 `h2c` 流量，然后根据不同的请求流量类型将其劫持并重定向到相应的 `Hander` 中去处理，最终以此达到同个端口上既提供 HTTP/1.1 又提供 HTTP/2 的功能了。

##### Server 实现

完成了不同协议的流量分发和处理后，我们需要实现其 Server 的具体逻辑，继续在 main.go 文件中写入如下代码：

```go
import (
    "github.com/grpc-ecosystem/grpc-gateway/runtime"
    ...
)

func RunServer(port string) error {
	httpMux := runHttpServer()
	grpcS := runGrpcServer()
	gatewayMux := runGrpcGatewayServer()

	httpMux.Handle("/", gatewayMux)

	return http.ListenAndServe(":"+port, grpcHandlerFunc(grpcS, httpMux))
}

func runHttpServer() *http.ServeMux {
	serveMux := http.NewServeMux()
	serveMux.HandleFunc("/ping", func(w http.ResponseWriter, r *http.Request) {
		_, _ = w.Write([]byte(`pong`))
	})

	return serveMux
}

func runGrpcServer() *grpc.Server {
	s := grpc.NewServer()
	pb.RegisterTagServiceServer(s, server.NewTagServer())
	reflection.Register(s)

	return s
}

func runGrpcGatewayServer() *runtime.ServeMux {
	endpoint := "0.0.0.0:" + port
	gwmux := runtime.NewServeMux()
	dopts := []grpc.DialOption{grpc.WithInsecure()}
	_ = pb.RegisterTagServiceHandlerFromEndpoint(context.Background(), gwmux, endpoint, dopts)

	return gwmux
}
```

在上述代码中，与先前的案例中主要差异在于 RunServer 方法中的 grpc-gateway 相关联的注册，核心在于调用了 `RegisterTagServiceHandlerFromEndpoint` 方法去注册 TagServiceHandler 事件，其内部会自动转换并拨号到 gRPC Endpoint，并在上下文结束后关闭连接。

另外在注册 TagServiceHandler 事件时，我们在 `grpc.DialOption` 中通过设置 `grpc.WithInsecure` 指定了 Server 为非加密模式，否则程序在运行时将会出现问题，因为 gRPC Server/Client 在启动和调用时，必须明确其是否加密。

##### 运行和验证

接下来我们编写 main 启动方法，调用 RunServer 方法，如下：

```go
func main() {
	err := RunServer(port)
	if err != nil {
		log.Fatalf("Run Serve err: %v", err)
	}
}
```

完成服务的再启动后我们进行 RPC 方法的验证，如下：

```shell
$ curl http://127.0.0.1:8004/ping
$ curl http://127.0.0.1:8004/api/v1/tags
$ grpcurl -plaintext localhost:8004 proto.TagService.GetTagList 
```

正确的情况下，都会返回响应数据，分别对应心跳检测、RPC 方法的 HTTP/1.1 和 RPC 方法的 gRPC（HTTP/2）的响应。



### 12. 生成接口文档

https://golang2.eddycjy.com/posts/ch3/07-api-doc/

