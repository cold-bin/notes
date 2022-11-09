---
title: "Go语言unsafe包的使用"
date: 2022-08-28T14:18:10+08:00
tags: [unsafe,零拷贝]
categories: [Golang,Go高性能并发编程]
draft: false
---

# unsafe使用及底层

## unsafe实现原理

在使用之前我们先来看一下unsafe包的源码部分，标准库unsafe包中只提供了3种方法，分别是:

```go
func Sizeof(x ArbitraryType) uintptr
func Offsetof(x ArbitraryType) uintptr
func Alignof(x ArbitraryType) uintptr
```

- `Sizeof(x ArbitrayType)`方法主要作用是用返回类型`x`所占据的字节数，但并不包含`x`所指向的内容的大小，与C语言标准库中的`Sizeof()`方法功能一样
- `Offsetof(x ArbitraryType)`方法主要作用是返回结构体成员在内存中的位置离结构体起始处(结构体的第一个字段的偏移量都是0)的字节数，即偏移量，我们在注释中看一看到其入参必须是一个结构体，其返回值是一个常量。
- `Alignof(x ArbitratyType)`的主要作用是返回一个类型的对齐值，也可以叫做对齐系数或者对齐倍数。对齐值是一个和内存对齐有关的值，合理的内存对齐可以提高内存读写的性能。一般对齐值是2^n^，最大不会超过8(受内存对齐影响). 获取对齐值还可以使用反射包的函数，也就是说：`unsafe.Alignof(x)`等价于`reflect.TypeOf(x).Align()`。对于任意类型的变量`x`，`unsafe.Alignof(x)`至少为1。对于`struct`结构体类型的变量`x`，计算`x`每一个字段`f`的`unsafe.Alignof(x，f)`，`unsafe.Alignof(x)`等于其中的最大值。对于数组类型的变量`x`，`unsafe.Alignof(x)`等于构成数组的元素类型的对齐倍数。没有任何字段的空结构体和没有任何元素的数组占据的内存空间大小为0，不同大小为0的变量可能指向同一块地址。

细心的朋友会发发现这三个方法返回的都是`uintptr`类型，这个目的就是可以和`unsafe.poniter`类型相互转换，因为`*T`是不能计算偏移量的，也不能进行计算，但是`uintptr`是可以的，所以可以使用`uintptr`类型进行计算，这样就可以可以访问特定的内存了，达到对不同的内存读写的目的。三个方法的入参都是`ArbitraryType`类型，代表着任意类型的意思，同时还提供了一个`Pointer`指针类型，即像`void *`一样的通用型指针。

```go
type ArbitraryType int
type Pointer *ArbitraryType
// uintptr 是一个整数类型，它足够大，可以存储指针的值，当然已经不是指针了，不是地址
type uintptr uintptr
```

上面说了这么多，可能会有点懵，在这里对三种指针类型做一个总结：

- `*T`：普通类型指针类型，用于传递对象地址，不能进行指针运算。
- `unsafe.poniter`：通用指针类型，用于转换不同类型的指针，不能进行指针运算，不能读取内存存储的值(需转换到某一类型的普通指针)
- `uintptr`：用于指针运算，GC不把`uintptr`当指针，`uintptr`无法持有对象。**`uintptr`类型的目标会被回收，所以一般不要独立出一个`uintptr`类型的变量，可能莫名其妙就被GC掉了**。

三者关系就是：`unsafe.Pointer`是桥梁，可以让任意类型的指针实现相互转换，也可以将任意类型的指针转换为`uintptr`进行指针运算，也就说`uintptr`是用来与`unsafe.Pointer`打配合，用于指针运算。画个图表示一下：

![img](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/202208282136228.jpeg)

基本原理就说到这里啦，接下来我们一起来看看如何使用~

## **`unsafe.Pointer`基本使用**

在`atomic.Value`源码里，看到`atomic/value.go`中定义了一个`ifaceWords`结构，其中`typ`和`data`字段类型就是`unsafe.Poniter`，这里使用`unsafe.Poniter`类型的原因是传入的值就是`interface{}`类型，使用`unsafe.Pointer`强转成`ifaceWords`类型，这样可以把类型和值都保存了下来，方便后面的写入类型检查。截取部分代码如下：

```go
// ifaceWords is interface{} internal representation.
type ifaceWords struct {
 typ  unsafe.Pointer
 data unsafe.Pointer
}
// Load returns the value set by the most recent Store.
// It returns nil if there has been no call to Store for this Value.
func (v *Value) Load() (x interface{}) {
 vp := (*ifaceWords)(unsafe.Pointer(v))
  for {
  typ := LoadPointer(&vp.typ) // 读取已经存在值的类型
    /**
    ..... 中间省略
    **/
    // First store completed. Check type and overwrite data.
  if typ != xp.typ { //当前类型与要存入的类型做对比
   panic("sync/atomic: store of inconsistently typed value into Value")
  }
}
```

上面就是源码中使用`unsafe.Pointer`的一个例子，有一天当你准备读源码时，`unsafe.pointer`的使用到处可见。好啦，接下来我们写一个简单的例子，看看`unsafe.Pointer`是如何使用的。

```go
func main()  {
 number := 5
 pointer := &number
 fmt.Printf("number:addr:%p, value:%d\n",pointer,*pointer)

 float32Number := (*float32)(unsafe.Pointer(pointer))
 *float32Number = *float32Number + 3

 fmt.Printf("float64:addr:%p, value:%f\n",float32Number,*float32Number)
}
```

运行结果：

```text
number:addr:0xc000018090, value:5
float64:addr:0xc000018090, value:3.000000
```

由运行可知使用`unsafe.Pointer`强制类型转换后指针指向的地址是没有改变，只是类型发生了改变。这个例子本身没什么意义，正常项目中也不会这样使用。

总结一下基本使用：先把`*T`类型转换成`unsafe.Pointer`类型，然后在进行强制转换转成你需要的指针类型即可。

## `Sizeof`、`Alignof`、`Offsetof`三个函数的基本使用

先看一个例子：

```go
type User struct {
 Name string
 Age uint32
 Gender bool // 男:true 女：false
}

func func_example()  {
 // sizeof
 fmt.Println(unsafe.Sizeof(true))
 fmt.Println(unsafe.Sizeof(int8(0)))
 fmt.Println(unsafe.Sizeof(int16(10)))
 fmt.Println(unsafe.Sizeof(int(10)))
 fmt.Println(unsafe.Sizeof(int32(190)))
 fmt.Println(unsafe.Sizeof("asong"))
 fmt.Println(unsafe.Sizeof([]int{1,3,4}))
 // Offsetof
 user := User{Name: "Asong", Age: 23,Gender: true}
 userNamePointer := unsafe.Pointer(&user)

 nNamePointer := (*string)(unsafe.Pointer(userNamePointer))
 *nNamePointer = "Golang梦工厂"

 nAgePointer := (*uint32)(unsafe.Pointer(uintptr(userNamePointer) + unsafe.Offsetof(user.Age)))
 *nAgePointer = 25

 nGender := (*bool)(unsafe.Pointer(uintptr(userNamePointer)+unsafe.Offsetof(user.Gender)))
 *nGender = false

 fmt.Printf("u.Name: %s, u.Age: %d,  u.Gender: %v\n", user.Name, user.Age,user.Gender)
 // Alignof
 var b bool
 var i8 int8
 var i16 int16
 var i64 int64
 var f32 float32
 var s string
 var m map[string]string
 var p *int32

 fmt.Println(unsafe.Alignof(b))
 fmt.Println(unsafe.Alignof(i8))
 fmt.Println(unsafe.Alignof(i16))
 fmt.Println(unsafe.Alignof(i64))
 fmt.Println(unsafe.Alignof(f32))
 fmt.Println(unsafe.Alignof(s))
 fmt.Println(unsafe.Alignof(m))
 fmt.Println(unsafe.Alignof(p))
}
```

为了省事，把三个函数的使用示例放到了一起。

1. **首先看`sizeof`方法**

   > `Sizeof` 采用任何类型的表达式 x，并返回假设变量 v 的大小（以字节为单位），就好像 v 是通过 `var v = x` 声明的一样。该大小不包括 x 可能引用的任何内存。例如，如果 x 是切片，则 `Sizeof` 返回切片描述符的大小，而不是切片引用的内存的大小。

   我们可以知道基本类型所占字节大小。这里重点说一下`int`、`string`、`[]byte`类型。

   - Go语言中的`int`类型的具体大小是跟机器的CPU位数相关的。如果CPU是32位的，那么`int`就占4字节，如果CPU是64位的，那么`int`就占8字节，这里我的电脑是64位的，所以结果就是8字节。

   - Go语言里的`string`类型，其实是不可变的字符串结构。字符串运行时的结构如下

     ```go
     type StringHeader struct {
      Data uintptr // 示操作系统cpu而定，64位 --> 64位
      Len  int	  // 示操作系统cpu而定，64位 --> 64位
     }
     ```

     因此，使用`sizeof`取`string`的大小时，其实是对一个结构体的`sizeof`，始终都是`8+8=16`个字节。

   - Go语言里的`[]byte`类型。切片运行时的结构如下

     ```go
     type SliceHeader struct {
      Data uintptr
      Len  int
      Cap  int
     }
     ```

     因此，使用`sizeof`取`[]byte`的大小时，同`string`理，始终都是`8+8+8=24`个字节。当然，数组就不会了。

2. `Offsetof`函数

   我想要修改结构体中成员变量，第一个成员变量是不需要进行偏移量计算的，直接取出指针后转换为`unsafe.pointer`，再强制给他转换成字符串类型的指针值即可。如果要修改其他成员变量，需要进行偏移量计算，才可以对其内存地址修改，所以`Offsetof`方法就可返回成员变量在结构体中的偏移量，也就是返回结构体初始位置到成员变量之间的字节数。

   看代码时大家应该要住`uintptr`的使用，不可以用一个临时变量存储`uintptr`类型，前面我们提到过用于指针运算，`GC`不把`uintptr`当指针，`uintptr`无法持有对象。`uintptr`类型的目标会被回收，所以你不知道他什么时候会被`GC`掉，那样接下来的内存操作会发生什么样的错误，咱也不知道。比如这样一个例子：

   ```go
   // 切记不要这样使用
   p1 := uintptr(userNamePointer)
   nAgePointer := (*uint32)(unsafe.Pointer(p1 + unsafe.Offsetof(user.Age)))
   ```

3. 最后看一下`Alignof`函数，主要是获取变量的对齐值. 

除了`int、uintptr`这些依赖CPU位数的类型，基本类型的对齐值都是固定的，结构体中对齐值取他的成员对齐值的最大值，结构体的对齐涉及到内存对齐。

## 经典应用：string与[]byte的相互转换

实现`string`与`byte`的转换，正常情况下，我们可能会写出这样的标准转换：

```go
// string to []byte
str1 := "Golang梦工厂"
by := []byte(s1)

// []byte to string
str2 := string(by)
```

使用这种方式进行转换都会涉及**底层数值的拷贝**，所以想要实现**零拷贝**，我们可以使用`unsafe.Pointer`来实现，通过强转换直接完成指针的指向，从而使`string`和`[]byte`指向同一个底层数据。在reflect包中有`string`和`slice`对应的结构体，他们的分别是：

```go
type StringHeader struct {
 Data uintptr
 Len  int
}

type SliceHeader struct {
 Data uintptr
 Len  int
 Cap  int
}
```

`StringHeader`代表的是`string`运行时的表现形式(`SliceHeader`同理)，通过对比`string`和`slice`运行时的表达可以看出，他们只有一个`Cap`字段不同，所以他们的内存布局是对齐的，所以可以通过`unsafe.Pointer`进行转换，因为可以写出如下代码：

```go
func stringToByte(s string) []byte {
 header := (*reflect.StringHeader)(unsafe.Pointer(&s))

 newHeader := reflect.SliceHeader{
  Data: header.Data,
  Len:  header.Len,
  Cap:  header.Len,
 }

 return *(*[]byte)(unsafe.Pointer(&newHeader))
}

func bytesToString(b []byte) string{
 header := (*reflect.SliceHeader)(unsafe.Pointer(&b))

 newHeader := reflect.StringHeader{
  Data: header.Data,
  Len:  header.Len,
 }

 return *(*string)(unsafe.Pointer(&newHeader))
}
```

上面的代码我们通过重新构造`slice header`和`string header`完成了类型转换，其实`[]byte`转换成`string`可以省略掉自己构造`StringHeader`的方式，直接使用强转就可以，因为`string`的底层也是`[]byte`，强转会自动构造，省略后的代码如下：

```go
func bytesToString(b []byte) string {
 return *(* string)(unsafe.Pointer(&b))
}
```

虽然这种方式更高效率，但是不推荐大家使用，前面也提高到了，这要是不安全的，使用当不当会出现极大的隐患，一些严重的情况`recover`也不能捕获。
