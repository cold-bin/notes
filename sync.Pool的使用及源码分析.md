---
title: "Sync.Pool的使用及源码分析"
date: 2022-09-24T15:09:27+08:00
tags: [sync.pool]
categories: [Golang,Go高性能并发编程]
draft: false
---

## `sync.Pool`使用及源码浅析

### `sync.Pool`使用

#### 背景

“频繁创建对象，频繁销毁对象”是在项目开发里算比较常见。`sync.Pool`的出现就是为了解决这个问题。

Go语言从1.3版本开始提供了对象重用的机制，即`sync.Pool`。`sync.Pool`是可伸缩的，同时也是并发安全的，其大小仅受限于内存的大小。`sync.Pool`用于存储那些被分配了但是没有被使用，而未来可能会使用的值。这样就可以不用再次经过内存分配，可直接复用已有对象，减轻GC的压力，从而提升系统的性能。

`sync.Pool`的大小是可伸缩的，高负载时会动态扩容，存放在池中的对象如果不活跃了会被自动清理。

> GC是一种自动内存管理机制，回收不再使用的对象的内存。
>
> 需要注意的是，`sync.Pool` 缓存的对象随时可能被无通知的清除，因此不能将 `sync.Pool` 用于存储持久对象的场景。

#### 声明对象池

只需要实现`New`函数即可。对象池中没有对象时，将会调用`New`函数创建。

```go
var machinePool = sync.Pool{
    New: func() interface{} { 
        return new(Machine) 
    },
}
```

#### `Get`&`Put`

```go
m := machinePool.Get().(*Machine)
json.Unmarshal(buf, m)
machinePool.Put(m)
```

- `Get()` 用于从对象池中获取对象，因为返回值是 `interface{}`，因此需要类型转换。
- `Put()` 则是在对象使用完毕后，返回对象池。

#### 性能测试

##### `struct`反序列化

```go
func BenchmarkUnmarshal(b *testing.B) {
	for n := 0; n < b.N; n++ {
		m := &Machine{}
		if err := json.Unmarshal([]byte("{\"A\":2}"), m); err != nil {
			log.Println(err)
			return
		}
	}
}

func BenchmarkUnmarshalWithPool(b *testing.B) {
	for n := 0; n < b.N; n++ {
		m := machinePool.Get().(*Machine)
		if err := json.Unmarshal([]byte("{\"A\":2}"), m); err != nil {
			log.Println(err)
			return
		}
		machinePool.Put(m)
	}
}
```

测试：

```bash
$ go test -bench . -benchmem
goos: windows
goarch: amd64
pkg: demo
cpu: Intel(R) Core(TM) i5-10210U CPU @ 1.60GHz
BenchmarkUnmarshal-8             2188940               528.5 ns/op           232 B/op          6 allocs/op
BenchmarkUnmarshalWithPool-8     2264995               518.5 ns/op           224 B/op          5 allocs/op
PASS
ok      demo    3.805s
```

可以从结果看到：使用池化复用对象的方式要比不使用具有更好的性能。当然，本例之中的结构体比较小，没能凸显出较大区别。

##### `bytes.Buffer`

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

测试：

```bash
$ go test -bench . -benchmem
goos: windows
goarch: amd64
pkg: demo
cpu: Intel(R) Core(TM) i5-10210U CPU @ 1.60GHz
BenchmarkBufferWithPool-8       10175354               109.2 ns/op             0 B/op          0 allocs/op
BenchmarkBuffer-8                 750318              1669 ns/op           10240 B/op          1 allocs/op
PASS
ok      demo    2.879s
```

使用池化进行对象的复用和不使用有明显的性能差异

#### 标准库应用

- `json.Marshal`

  `encodeState`对象的复用

- `fmt.Printf`

  对`pp`的复用

### `sync.Pool`浅析

参考博客 -> https://www.cyhone.com/articles/think-in-sync-pool/

```go
// A Pool must not be copied after first use.
type Pool struct {
	noCopy noCopy

	local     unsafe.Pointer // local fixed-size per-P pool, actual type is [P]poolLocal
	localSize uintptr        // size of the local array

	victim     unsafe.Pointer // local from previous cycle
	victimSize uintptr        // size of victims array

	// New optionally specifies a function to generate
	// a value when Get would otherwise return nil.
	// It may not be changed concurrently with calls to Get.
	New func() interface{}
}
```

#### `no copy`

`no copy`的原因是为了安全，因为结构体对象中包含引用类型的话，直接赋值拷贝是浅拷贝，是不安全的。因为浅拷贝之后，就相当于有两个指针指向同一个地址上的对象，任意一个指针引起的更新删除等操作都会影响到另一个指针。

`no copy`的实现也很简单，只需要有实现`sync.Locker`接口，然后再把实现的类型嵌入目标结构体，就可以实现。这种实现不是直接禁掉复制这个功能，嵌入了`no copy`字段的程序依然可以正常执行。通过`go vet`分析，拷贝了嵌入`no copy`字段的类型时会报错，提示不能对当前类型进行值拷贝（goland也能在一定程度上提示，变黄）。

当然除了使用`no copy`字段来约束类型不能出现复制以外，还可以在代码逻辑层面实现（不是范式，不能总结）。

#### `local & local size`

- `local` 是个数组，长度为 P 的个数。其元素类型是 `poolLocal`。这里面存储着各个 P 对应的本地对象池。可以近似的看做 `[P]poolLocal`。（P，指的是GMP里的Processor）
- `localSize`。代表 local 数组的长度。因为 P 可以在运行时通过调用 `runtime.GOMAXPROCS` 进行修改, 因此我们还是得通过 `localSize` 来对应 `local` 数组的长度。

由于每个 P 都有自己的一个本地对象池 `poolLocal`，`Get` 和 `Put` 操作都会优先存取本地对象池。由于 P 的特性，操作本地对象池的时候整个并发问题就简化了很多，可以尽量避免并发冲突。

我们再看下本地对象池 `poolLocal` 的定义，如下：

```go
// 每个 P 都会有一个 poolLocal 的本地
type poolLocal struct {
	poolLocalInternal

	pad [128 - unsafe.Sizeof(poolLocalInternal{})%128]byte 
    // 128 - unsafe.Sizeof(poolLocalInternal{})%128 + unsafe.Sizeof(poolLocalInternal{}) = n*128
}

type poolLocalInternal struct {
	private interface{}
	shared  poolChain
}
```

`pad` 变量的作用在下文会讲到，这里暂时不展开讨论。我们可以直接看 `poolLocalInternal` 的定义，其中每个本地对象池，都会包含两项：

- `private` 私有变量。`Get` 和 `Put` 操作都会优先存取 `private` 变量，如果 `private` 变量可以满足情况，则不再深入进行其他的复杂操作。
- `shared`。其类型为 `poolChain`，这个是链表结构，这个就是 P 的本地对象池了。

#### `Get`方法

```go
func (p *Pool) Get() interface{} {
	if race.Enabled {
		race.Disable()
	}
    // 禁掉M调度，固定住P，并拿到当前P的poolLocal数组
	l, pid := p.pin()
    // 途径一：拿私有
	x := l.private
	l.private = nil
    // 途径二：私有没有，就拿公共存储区shared双端队列缓存
	if x == nil {
		x, _ = l.shared.popHead()
		if x == nil {
            // 途径三：还没有，在当前P进行地址偏移，获取数组里其他所有P的公共存储区shared双端队列缓存；还没有就取 pool.victim
			x = p.getSlow(pid)
		}
	}
    // 取消P的固定
	runtime_procUnpin()
	if race.Enabled {
		race.Enable()
		if x != nil {
			race.Acquire(poolRaceAddr(x))
		}
	}
    // 途径四：实在没得，就内存分配一个对象
	if x == nil && p.New != nil {
		x = p.New()
	}
	return x
}
```

#### `Put`方法

```go
// Put adds x to the pool.
func (p *Pool) Put(x interface{}) {
	if x == nil {
		return
	}
    // 竟态检测
	if race.Enabled {
		if fastrand()%4 == 0 {
			// Randomly drop x on floor.
			return
		}
		race.ReleaseMerge(poolRaceAddr(x))
		race.Disable()
	}
    // 先禁用M调度，固定当前G
	l, _ := p.pin()
    // 如果私有为空，就放到私有上
	if l.private == nil {
		l.private = x
		x = nil
	}
    // 私有已经有了，就放到公共缓存区
	if x != nil {
		l.shared.pushHead(x)
	}
    // 取消固定
	runtime_procUnpin()
	if race.Enabled {
		race.Enable()
	}
}
```

#### 清理对象

每个被使用的 `sync.Pool`，都会在初始化阶段被添加到全局变量 `allPools []*Pool` 对象中。Golang 的 `runtime` 将会在 **每轮 GC 前**，触发调用 `poolCleanup` 函数，清理 `allPools`。

```go
func poolCleanup() {
	// Drop victim caches from all pools.
	for _, p := range oldPools {
		p.victim = nil
		p.victimSize = 0
	}

	// Move primary cache to victim cache.
	for _, p := range allPools {
		p.victim = p.local
		p.victimSize = p.localSize
		p.local = nil
		p.localSize = 0
	}
    
	oldPools, allPools = allPools, nil
}

var (
	allPoolsMu Mutex
	allPools []*Pool // get 或 put时，会将pool对象放到allPools里
	oldPools []*Pool
)

func init() {
	runtime_registerPoolCleanup(poolCleanup)
}
```

可以看到，每次GC前，都会将当前p里的local放到victim里，这样，需要两次GC才能将`sync.Pool`里的对象池的对象，完全清掉。

### `sync.Pool`结构

![image-20220923225531884](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202209232255837.png)
