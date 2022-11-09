---
title: "Golang编程思想集锦"
date: 2022-08-10T17:13:38+08:00
tags: [go编程思想]
categories: [Golang]
draft: false
---



## 一、go面向对象

### 封装

结构体与方法

### 多态

接口来实现多态，不同结构体可以实现相同的接口。这样就能拥有同种行为的不同具体状态 --> 多态

### 继承、覆盖

```go
// 标准包里的实现，大概是直接把文件里的所有内容给提取出来
func (f *File) Read(b []byte) (n int, err error) {
	if err := f.checkValid("read"); err != nil {
		return 0, err
	}
	n, e := f.read(b)
	return n, f.wrapErr("read", e)
}

//main.go 
// 为了满足某种业务需求，文件里的内容不是所有都需要读出来，或者需要将某些内容屏蔽掉，这个时候，就凸显出接口的魅力了
func main() {
	file, err := os.Open("./1.txt")
	if err != nil {
		panic(err)
		return
	}
	myFile := NewMyFile(file)
	defer myFile.File.Close()
	
	bs := make([]byte, 10) // read 10 bytes
	if _, err = myFile.Read(bs); err != nil {
		panic(err)
		return
	}
	
	fmt.Println("result: ",string(bs))
}

type MyFile struct {
	*os.File
}

func NewMyFile(f *os.File) *MyFile {
	return &MyFile{
		File: f,
	}
}

// 覆盖掉标准包里的io.File的Read方法，实现自己的逻辑
func (f *MyFile) Read(bs []byte) (n int, err error) {
	// 在原来的读取逻辑修改即可
	// 在系统底层读的时候，监控字节流数据，每取到一个有效的unicode字符，
	// 就监视它的值是否是期望输出的，不是就替换，是的话就通过并写入bs切片
	return
}
//其他未覆盖的方法都被继承
```



## 二、接口

什么是接口呢？接口，顾名思义就是一个插座，一个合适某种行为的规范。在Java里，接口是通过类来显式地调用（`implements`）；而在Go语言里，主要是通过隐式调用：某类型所属方法的名字、入参、出参和某个接口里的方法完全一样（**全部**），那么Go语言就默认实现了该接口。

来看看使用接口与不使用接口作为函数参数的例子：

- 不使用接口

  不使用接口作为一种约定，而使用某种特定类型的参数作为入参，这样做只能实现当前函数的特定功能，也就是说当前函数只能做到一件事情，不能做做到多件事情。

- 使用接口

  使用接口作为函数入参的约定，即要求当前传入的类型，必须具备函数入参接口里约定好的所有行为、方法。这样才能进行调用该函数。

  下面的**接口抽象**

### 接口抽象

对于大多数的动态语言而言，如Java，接口主要是用来抽象解耦代码间的依赖的，即**将约定和具体实现进行分离，或者说，就是让调用者无需关心底层的实现，只需要遵照接口方法里的定义调用即可，当然也可以按照业务需求自己再实现一个覆盖掉原方法实现也行（无缝衔接）。**

**示例**

假设first的实现方法是第一次实现，但是由于业务增长，急需更换另一种实现方式second，如何快速无缝衔接呢？

第一次实现的时候先抽象出某一个行为要完成所需的一系列动作（或者叫方法、函数）。然后自己给这些接口提供一个实现，实现之后，所有的入参或接收者都采用接口类型来接收。

这样做可以达到**更好地解耦**：即下一位觉得你的实现缺乏一些更好地优化，只需要根据自己的逻辑实现这个接口，就可以无缝衔接到代码里，无需大改代码如果不使用接口抽象显然，每个人都有自己的实现，这样会导致类型不一样，就会看底层代码的实现，哪里不对就修改哪里，显然比较繁琐。通过接口进行抽象之后，无需关注实现，只需要给出自己的实现就可以无缝衔接调用

```go
type Retriever interface {
	GetRetriever(url string) (bytes []byte, err error)
}


//first/retriever.go
type First struct{}
//某个团队提供的获取url里内容的一种方式
func (First) GetRetriever(url string) (bytes []byte, err error) {
	res, err := http.Get(url)
	if err != nil {
		return nil, err
	}

	bytes, err = ioutil.ReadAll(res.Body)

	return
}

//second/retriever.go
type Second struct{}
//测试团队提供的一种方式
func (Second) GetRetriever(url string) (bytes []byte, err error) {
	return []byte("test content"), nil
}

//main.go
func main() {
	var r Retriever = first.First{} //想要更换实现，只需修改此处即可，无需关注后面

	bytes, err := r.GetRetriever("https://www.baidu.com")
	if err != nil {
		return
	}

	fmt.Println(string(bytes))
}
```



### 接口组合

接口组合类似与多继承，就是可以将一些接口组合在一起，这样就拥有这些接口里的方法，当然也可以定义自己方法，实现了子接口，也就实现了部分方法。

接口的组合就是将一些接口整合在一起，有点像多继承一样。可以直接看下边的例子

```go
type Retriver interface {
    Get(url string) string
}

type Poster interface {
    Post(url string, form map[string]string) string
}

func download(r Retriver) string {
    return r.Get("<http://www.baidu.com>")
}

func post(poster Poster)  {
    poster.Post("<http://ww.baidu.com>", map[string]string{
        "name": "shulv",
        "age": "twenty",
    })
}
```

上边定义了两个接口，并且分别有这两个接口的调用者，`download`和`post`。现在假设有个`session`函数，它需要一个参数，即是一个`Retriver`，又是一个`Poster`。此时就可以用到接口的组合

```go
type RetrieverPoster interface {
    Retriver
    Poster
    //Connect(host string) //当然还可以添加别的方法
}

func session(s RetrieverPoster) string {
    s.Post("<http://www.baidu.com>", map[string]string{
        "Contents": "facked",
    })
    return s.Get("<http://www.baidu.com>")
}
```

上边的代码中定义了一个组合接口`RetrieverPoster`，它直接包含了`Retriver`和`Poster`这两个接口，也就是说他可以直接使用这两个接口中的方法。当然，这个组合接口中还可以增加自己的方法。有没有发现它和结构体的嵌套非常的像（不记得结构体嵌套的，可以点[这里](https://juejin.cn/post/6996479088852467726#heading-4)）

下边为了方便理解，我直接在一个文件中定义结构体，并实现结构体的一些方法

```go
type Mock struct {
    Contains string
}

func (r Mock) Get(url string) string {
    return r.Contains
}

func (r *Mock) Post(url string, form map[string]string) string {
    r.Contains = form["Contents"]
    return "ok"
}

type Real struct {
    UserAgent string
    TimeOut time.Duration
}

func (r Real) Get(url string) string {
    resp, err := http.Get(url)
    if err != nil {
        panic(err)
    }

    result, err := httputil.DumpResponse(resp, true)

    resp.Body.Close()

    if err != nil {
        panic(err)
    }

    return string(result)
}
```

上边定义了两个结构体，分别是`Mock`和`Real`，并且实现了对应的方法（可以看到，`Mock`实现了组合接口`RetrieverPoster`）。下边在main函数中调用`session`函数

```go
func main() {
    retriver := Mock{"just a test"}
    fmt.Println(session(&retriever))
}

//输出：facked（因为在Post方法中对Mock.Content进行了修改，说明session里边调用了Mock的Post方法）
```

其实在Go语言的库中，有很多接口组合的例子，比如`io`包里的`ReadWriteCloser`，它就由多个接口组合而成的一个接口，内容如下:

```go
// ReadWriteCloser is the interface that groups the basic Read, Write and Close methods.
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

可以看到它是由`Reader`、`Writer`和`Closer`这三个接口组合而成的。当然，你可以进到`io`这个包里边，里边有很多这样的组合接口

上边就是接口的组合，下边分享一些Go语言中标准的接口



### 常见系统接口

#### Stringer

在`fmt`这个包中，有一个接口叫`Stringer`，它里边只有一个`String`函数，所以一个类型只要实现了`String`函数，就实现了`Stringer`接口

```go
package fmt

type Stringer interface {
    String() string
}
```

下边定义一个结构体类型，该结构体中实现了`String`方法

```go
package main

import "fmt"

type Mock struct {
    Contents string
}

func (m Mock) String() string {
    return fmt.Sprintf("Mock: {Contents = %s}", m.Contents)
}

func main() {
    m := Mock{"a test, too"}
    fmt.Println(m.String())
}

//输出：Mock: {Contents = a test, too}
```



#### Reader

在`io`这个包中有一个`Reader`接口，它里边只有一个`Read`接口。实现`Read`方法的可以是一个文件，可以从文件中读取内容，放入到byte（当然不止有文件，还有网络、`slice`等等，都可以）

```go
package io

type Reader interface {
    Read(p []byte) (n int, err error)
}
```

可以看一下`os.Open`方法的内部实现，它的返回值是一个`File`的结构，然后进入到`File`中可以看到它是一个结构体类型，并且可以发现这个结构体中内嵌了一个`file`的结构体，而这个`file`实际上是 `*File`的真实表示。而 `*File`又实现了`Read`方法，所以它实际上就实现了`Reade`r接口

```go
//file.go
package os

func Open(name string) (*File, error) {
    return OpenFile(name, O_RDONLY, 0)
}

//types.go
package os

type File struct {
    *file // os specific
}
....

// file is the real representation of *File.
type file struct {
    pfd         poll.FD
    name        string
    dirinfo     *dirInfo // nil unless directory being read
    nonblock    bool     // whether we set nonblocking mode
    stdoutOrErr bool     // whether this is stdout or stderr
    appendMode  bool     // whether file is opened for appending
}

而*File实现了Read接口，内容如下:
//file.go
package os

func (f *File) Read(b []byte) (n int, err error) {
    if err := f.checkValid("read"); err != nil {
        return 0, err
    }
    n, e := f.read(b)
    return n, f.wrapErr("read", e)
}
```

在`*File`中可以发现，它不会说我实现了哪些接口。实现了哪些接口是用的人来说的，它只需要实现某些方法就行了

所以我们在获取文件中的内容的时候调用的`bufio.NewScanner()`方法，它需要的参数是一个`io.Reader`类型（所以`Open`返回的`File`类型，是可以直接传递给`NewScanner`，因为它实现了`Reader`接口）

```go
filename := "a.txt"
file, err := os.Open(filename)
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println(scanner.Text())//逐行读取文件中的内容
}

//下边是NewScanner内部实现
func NewScanner(r io.Reader) *Scanner {
    return &Scanner{
        r:            r,
        split:        ScanLines,
        maxTokenSize: MaxScanTokenSize,
    }
}
```



#### Writer

跟`Reader`接口一样，它也在`io`包中，接口中只有一个`Write`方法。实现`Write`方法的可以是一个文件，将提供的byte数据写入到文件中

```go
package io

type Writer interface {
    Write(p []byte) (n int, err error)
}

```

在`fmt`包中有一个`Fprintf`函数（`fmt`中各种输出函数的使用及作用，可以点[这里](https://juejin.cn/post/6993138029858652191)），`Fprintf`的作用是返回一个格式化后的字符串。它的第一个参数是一个`io.Writer`类型，所以任何实现了这个`Writer`接口的，都能够传给它来用

```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
    p := newPrinter()
    p.doPrintf(format, a)
    n, err = w.Write(p.buf)
    p.free()
    return
}
```



### 示例：读取文件内容

以读取文件内容为例，通常的做法如下

```go
package main

import (
    "bufio"
    "fmt"
    "os"
)

func printFile(filename string)  {
    file, err := os.Open(filename)
    if err != nil {
        panic(err)
    }

    scanner := bufio.NewScanner(file)
    for scanner.Scan() {
        fmt.Println(scanner.Text())//逐行读取文件中的内容
    }
}

func main() {
    printFile("a.txt")
}
```

这样的做法缺点就是，`printFile`只能接收一个文件名字符串，只能读取文件中的内容。**下边对他进行优化，让他不仅能够打印文件内容，并且还能够通过像打印文件一样打印字符串**

```go
package main

import (
    "bufio"
    "fmt"
    "io"
    "os"
    "strings"
)

func printFile(filename string)  {
    file, err := os.Open(filename)
    if err != nil {
        panic(err)
    }

    printFileContents(file)
}

// 使用接口作为函数入参，达到解耦
func printFileContents(read io.Reader)  { 
    scanner := bufio.NewScanner(read)
    for scanner.Scan() {
        fmt.Println(scanner.Text())//逐行读取文件中的内容
    }
}

func main() {
    printFile("a.txt")
    s := `uiieo
    "dgsjg"
    s
d
3
	`
    printFileContents(strings.NewReader(s))
}
```

可以看到将实际的打印内容工作交由`printFileContents`方法来做，它接收一个`Reader`类型的参数，所以可以利用它来打印文件内容，也能将字符串类型以文件的方式打印，这样它就变得更加通用



## 三、函数式编程

### 函数式选项模式`Option`

作为一个类库作者，迟早会面临到接口变更问题。或者是因为外部环境变化，或者是因为功能升级而扩大了外延，或者是因为需要废弃掉过去的不完善的设计，或者是因为个人水平的提升，无论哪一种理由，你都可能会发现必须要修改掉原有的接口，替换之以一个更完美的新接口。

#### 旧的方式

想象下有一个早期的类库：

```go
package tut

func New(a int) *Holder {
  return &Holder{
    a: a,
  }
}

type Holder struct {
  a int
}
```

后来，我们发现需要增加一个布尔量 b，于是修改 tut 库为：

```go
package tut

func New(a int, b bool) *Holder {
  return &Holder{
    a: a,
    b: b,
  }
}

type Holder struct {
  a int
  b bool
}
```

没过几天，现在我们认为有必要增加一个字符串变量，tut 库不得不被修改为：

```go
package tut

func New(a int, b bool, c string) *Holder {
  return &Holder{
    a: a,
    b: b,
    c: c,
  }
}

type Holder struct {
  a int
  b bool
  c string
}
```

想象一下，tut 库的使用者在面对三次接口 New() 的升级时，那么势必会导致：由于New方法更改函数的入口，所以，任何使用该方法创建对象的地方都必须修改。显然这样是很麻烦的，有没有一种方式，可以不改变New方法的入参，及时需要更新迭代对象版本时（增添改属性），New方法在调用的时候，即使不变也可以维护以前的所有属性及功能。答案就是，**函数选项模式**

对此我们需要 Functional Options 模式来解救之。

#### 新的方式

这里以`colly`框架源码为例介绍**函数选项模式**

```bash
go get -u github.com/gocolly/colly
```

`colly`提供了这样一个`Collector`实例构建方法

```go
// NewCollector creates a new Collector instance with default configuration
func NewCollector(options ...func(*Collector)) *Collector {
	c := &Collector{}
	c.Init()

	for _, f := range options {
		f(c)
	}

	c.parseSettingsFromEnv()

	return c
}
```

可以看到，本实例初始化函数的入参是一个`...func(*Collector)`类型。为什么这样做呢？即，为什么要将初始化函数的`Colletor`指针作为当前函数的入参呢？这个函数又有什么作用呢？继续往下看。

在`colly`的源码里，可以翻阅到一些函数的返回值类型为`func(*Collector)`的函数，这些函数是干嘛的呢？

```go
// AllowURLRevisit instructs the Collector to allow multiple downloads of the same URL
func AllowURLRevisit() func(*Collector) {
	return func(c *Collector) {
		c.AllowURLRevisit = true
	}
}
// CacheDir specifies the location where GET requests are cached as files.
func CacheDir(path string) func(*Collector) {
	return func(c *Collector) {
		c.CacheDir = path
	}
}

...
```

我们可以看到：这些函数主要是将`Colletor`实例的一些属性进行更新，而且，翻阅源码，可以看到：`Colly`几乎为每一个`Colletor`的字段属性都提供了一个这样的函数。而在初始化实例的时候，可以通过提供这些函数（即选项）来配置当前的`Collector`实例.

**实例**

```go
package main

type Option func(h *Holder)

func NewHolder(opts ...Option) *Holder {
	h := &Holder{a: -1}
	for _, opt := range opts {
		opt(h)
	}
	return h
}

type Holder struct {
	a int
	b string
	c bool
	// 业务需求，更改参数
	d interface{}
}

func (h Holder) GetA() int { return h.a }

func (h Holder) GetB() string { return h.b }

func (h Holder) GetC() bool { return h.c }

func OptionWithA(a int) Option {
	return func(h *Holder) { h.a = a }
}

func OptionWithB(b string) Option {
	return func(h *Holder) { h.b = b }
}

// Deprecated:已弃用的标志
func OptionWithC(c bool) Option {
	return func(h *Holder) { h.c = c }
}

func OptionWithD(d interface{}) Option {
	return func(h *Holder) { h.d = d }
}

func main() {
	h := NewHolder(
		OptionWithB("b"),
		OptionWithC(false),
	)
	println(h.GetA())
	println(h.GetB())
	println(h.GetC())
}
```

**好处**

1. 这样做虽然看似，增添了很多无用代码，但是这样做可以使得，`colly`的`NewCollector`在创建实例对象时具有更好的灵活性： 这样做，以后在更新版本时，不需要更改原有的New方法，即，这对于调用方来说，这样做的意义在于**向前兼容性**（增添一些属性不会影响原来的调用方的逻辑代码，一般不会更改或减少原来的元素）
2. 当配置某个实例时，**配置项很多，而且有一些字段不是必须的，是可选的**，就可以考虑使用函数选项模式

## 四、并发

先看一个最简单的并发问题

```go
func main(){
	for i := 0; i < 9; i++ {
		go func() {
			fmt.Print(i)
		}()
	}
    time.Sleep(2 * time.Second)
}
// 运行结果：296667939
// 可以看到有并发问题出现
```

**解决思路**

匿名函数可以捕获父函数的成员变量。这里的匿名函数里的`i`指的是多次循环里的同一个地址里的`i`，开多个协程来操作同一个地址上的变量，显然存在**并发问题**

解决方法：不再使得一个地址的变量被多个goroutine操作，在创建协程的作用域上，cv一份变量，这样就可以得到一个goroutine只操作一个地址的变量

```go
func main (){
    for i := 0; i < 9; i++ {
     	go func(i int) {
	    	fmt.Println(i)
        }(i)
    }
}
// or
func main(){
	for i := 0; i < 9; i++ {
		i := i
		go func() {
			fmt.Print(i)
		}()
	}
    time.Sleep(2 * time.Second)
}
```

