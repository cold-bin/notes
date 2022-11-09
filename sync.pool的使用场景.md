---
title: "sync.pool的使用场景"
date: 2022-08-29T10:40:55+08:00
tags: [sync.pool]
categories: [Golang,Go高性能并发编程]
draft: false
---

## `sync.Pool` 的使用场景

> 一句话总结：保存和复用临时对象，减少内存分配，降低GC压力。

举个简单的例子：

```go
type Student struct {
	Name   string
	Age    int32
	Remark [1024]byte
}

var buf, _ = json.Marshal(Student{Name: "Geektutu", Age: 25})

func unmarsh() {
	stu := &Student{}
	json.Unmarshal(buf, stu)
}
```

`json`的反序列化在文本解析和网络通信过程中非常常见，当程序并发度非常高的情况下，短时间内需要创建大量的临时对象。而这些对象是都是分配在堆上的，会给GC造成很大压力，严重影响程序的性能。

~~注：`json`反序列化没有使用`sync.Pool`进行池化复用对象~~

> 参考：[垃圾回收(GC)的工作原理](https://geektutu.com/post/qa-golang-2.html#Q5-简述-Go-语言GC-垃圾回收-的工作原理)

Go语言从1.3版本开始提供了对象重用的机制，即`sync.Pool`。`sync.Pool`是可伸缩的，同时也是并发安全的，其大小仅受限于内存的大小。`sync.Pool`用于存储那些被分配了但是没有被使用，而未来可能会使用的值。这样就可以不用再次经过内存分配，可直接复用已有对象，减轻GC的压力，从而提升系统的性能。

`sync.Pool`的大小是可伸缩的，高负载时会动态扩容，存放在池中的对象如果不活跃了会被自动清理。

## 如何使用

`sync.Pool`的使用方式非常简单

### 声明对象池

只需要实现`New`函数即可。对象池中没有对象时，将会调用`New`函数创建。

```gp
var studentPool = sync.Pool{
    New: func() interface{} { 
        return new(Student) 
    },
}
```

### `Get`&`Put`

```go
stu := studentPool.Get().(*Student)
json.Unmarshal(buf, stu)
studentPool.Put(stu)
```

- `Get()` 用于从对象池中获取对象，因为返回值是 `interface{}`，因此需要类型转换。
- `Put()` 则是在对象使用完毕后，返回对象池。

## 性能测试

### `struct`反序列化

```go
func BenchmarkUnmarshal(b *testing.B) {
	for n := 0; n < b.N; n++ {
		stu := &Student{}
		json.Unmarshal(buf, stu)
	}
}

func BenchmarkUnmarshalWithPool(b *testing.B) {
	for n := 0; n < b.N; n++ {
		stu := studentPool.Get().(*Student)
		json.Unmarshal(buf, stu)
		studentPool.Put(stu)
	}
}
```

测试结果如下：

```bash
go test -bench . -benchmem
```

在这个例子中，因为`Student`结构体内存占用较小，内存分配几乎不耗时间。而标准库`json`反序列化时利用了反射，效率是比较低的，占据了大部分时间，因此两种方式最终的执行时间几乎没什么变化。但是内存占用差了一个数量级，使用了 `sync.Pool` 后，内存占用仅为未使用的 `234/5096 = 1/22`，对GC的影响就很大了。

### `bytes.Buffer`

```go
var bufferPool = sync.Pool{
	New: func() interface{} {
		return &bytes.Buffer{}
	},
}

var data = make([]byte, 10000)

func BenchmarkBufferWithPool(b *testing.B) {
	for n := 0; n < b.N; n++ {
		buf := bufferPool.Get().(*bytes.Buffer)
		buf.Write(data)
		buf.Reset()
		bufferPool.Put(buf)
	}
}

func BenchmarkBuffer(b *testing.B) {
	for n := 0; n < b.N; n++ {
		var buf bytes.Buffer
		buf.Write(data)
	}
}
```

测试结果如下：

这个例子创建了一个`bytes.Buffer`对象池，而且每次只执行一个简单的`Write`操作，存粹的内存搬运工，耗时几乎可以忽略。而内存分配和回收的耗时占比较多，因此对程序整体的性能影响更大。

## 标准库中的应用

### `fmt.Printf`

Go语言标准库也大量使用了`sync.Pool`，例如`fmt`和`encoding/json`。

以下是`fmt.Printf`的源代码(`go/src/fmt/print.go`)：

```go
// go 1.13.6

// pp is used to store a printer's state and is reused with sync.Pool to avoid allocations.
type pp struct {
    buf buffer
    ...
}

var ppFree = sync.Pool{
	New: func() interface{} { return new(pp) },
}

// newPrinter allocates a new pp struct or grabs a cached one.
func newPrinter() *pp {
	p := ppFree.Get().(*pp)
	p.panicking = false
	p.erroring = false
	p.wrapErrs = false
	p.fmt.init(&p.buf)
	return p
}

// free saves used pp structs in ppFree; avoids an allocation per invocation.
func (p *pp) free() {
	if cap(p.buf) > 64<<10 {
		return
	}

	p.buf = p.buf[:0]
	p.arg = nil
	p.value = reflect.Value{}
	p.wrappedErr = nil
	ppFree.Put(p)
}

func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintf(format, a)
	n, err = w.Write(p.buf)
	p.free()
	return
}

// Printf formats according to a format specifier and writes to standard output.
// It returns the number of bytes written and any write error encountered.
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}
```

`fmt.Printf` 的调用是非常频繁的，利用 `sync.Pool` 复用 pp 对象能够极大地提升性能，减少内存占用，同时降低 GC 压力。

### `json.Marshal`

```go
func Marshal(v interface{}) ([]byte, error) {
	e := newEncodeState()

	err := e.marshal(v, encOpts{escapeHTML: true})
	if err != nil {
		return nil, err
	}
	buf := append([]byte(nil), e.Bytes()...)
    
    // 放回对象池里
	encodeStatePool.Put(e)

	return buf, nil
}
...
var encodeStatePool sync.Pool

func newEncodeState() *encodeState {
    // 取出对象池里的 encodeState 对象，避免减少内存空间分配，从而降低GC的压力
	if v := encodeStatePool.Get(); v != nil {
		e := v.(*encodeState)
		e.Reset()
		if len(e.ptrSeen) > 0 {
			panic("ptrEncoder.encode should have emptied ptrSeen via defers")
		}
		e.ptrLevel = 0
		return e
	}
	return &encodeState{ptrSeen: make(map[interface{}]struct{})}
}

```

在后端开发里`json.Marshal`的调用次数会很多很多，经常遇到需要将要发给前端的数据先调用`json.Marshal`方法序列化为`json`格式的数据，然后回传前端响应。显然，官方库里的池化复用`json`解码器`encodeState`对象，可以减少内存分配，减轻GC压力
