---
title: "Go杂烩"
date: 2022-07-15T14:19:32+08:00
tags: []
categories: [Golang]
draft: false
---

## Golang杂烩

### 一、结构体

和 C/C++ 的结构体类似，Go 语言的结构体 struct 是一种聚合的数据类型，可以包含任意类型的值。

#### 方法

- 值接收者（go语言特有）

  所谓值接收者，指的是，当调用值接收者的方法时，会临时copy一份当前对象的副本，然后在这个副本的基础上再操作。**显然，这个副本已经与原来的对象没有指向关系了，修改副本属性，无法更改原对象的值。**

  应用场景：当对象主要运用于读数据，而非写数据时；当前对象的内存开销比较小

- 指针接收者

  应用场景：当对象需要修改对象属性时，需要使用指针接收者；当对象内存开销比较大。

  注：不能一味使用指针接收者，有时候，指针接收者带来的开销可能比值接收者还要大

- 两者实现某接口时的区别

  **指针接收者实现某接口时，在类型转换上，只能使用指针类型的接收者；而值接收者实现某接口时，在类型转换上，无论是值接收者还是指针接收者都可以**

  ```go
  package main
  import (
  	"fmt"
  )
  type AnimalInterface interface {
  	bake(string) error
  }
  //Dog ...
  type Dog struct {
  	name string
  }
  func (dog Dog) bake(w string) error {
  	fmt.Printf("%s bake %s \n", dog.name, w)
  	return nil
  }
  type Cat struct {
  	name string
  }
  func (cat *Cat) bake(w string) error {
  	fmt.Printf("%s bake %s \n", cat.name, w)
  	return nil
  }
  func main() {
  	var dogBig AnimalInterface = Dog{
  		name: "大黄",
  	}
  	dogBig.bake("吴奇隆")
  	var dogLittle AnimalInterface = &Dog{
  		name: "小黄",
  	}
  	dogLittle.bake("李易峰")
  	// cannot use Cat literal (type Cat) as type AnimalInterface in assignment:Cat does not implement AnimalInterface (bake method has pointer receiver)
  	var catHua AnimalInterface = Cat{
  		name: "小花",
  	}
  	catHua.bake("刘亦菲")
  	var catBlue AnimalInterface = &Cat{
  		name: "小蓝",
  	}
  	catBlue.bake("张园园")
  }
  ```

#### 自动转换

先定义一个 struct

```Go
type Person struct {
    Name string
    Age int
}
```

初始化

```Go
func main() {
    me := &Person{
        Name: "HF",
        Age:  1,
    }

    fmt.Printf("type of me: %T\n", me)
    fmt.Printf("my name: %s, type of my name: %T\n", (*me).Name, (*me).Name)
    fmt.Printf("my age: %d, type of my age: %T\n", (*me).Age, (*me).Age)

    fmt.Printf("my name: %s, type of my name: %T\n", me.Name, me.Name)
    fmt.Printf("my age: %d, type of my Age: %T\n", me.Age, me.Age)
}
```

执行命令源码文件后输出：

```Bash
type of me: *main.Person
my name: HF, type of my name: string
my age: 1, type of my age: int
my name: HF, type of my name: string
my age: 1, type of my Age: int
```

`me` 是一个指针型变量，访问 `(*me).XXX` 和 `me.XXX` 的效果是一样的，说明编译器会将 `me.XXX` 自动转换成 `(*me).XXX`，前者稍显笨拙。

#### 内存分配

```Go
import (
    "fmt"
    "unsafe"
)

func main() {
    me := &Person{
        Name: "HF",
        Age:  1,
    }

    fmt.Printf("pointer of me: %p, size of me: %d\n", me, unsafe.Sizeof(me))
    fmt.Printf("pointer of my name: %p, size of my name: %d\n", &me.Name, unsafe.Sizeof(me.Name))
    fmt.Printf("pointer of my age: %p, size of my age: %d\n", &me.Age, unsafe.Sizeof(me.Age))
}
```

执行命令源码文件后输出：

```Bash
pointer of me: 0xc00000a060, size of me: 8
pointer of my name: 0xc00000a060, size of my name: 16
pointer of my age: 0xc00000a070, size of my age: 8
```

指针变量 `me` 指向从 0xc00000a060 开始的内存区域，也是 Person 结构体 Name 字段的内存起始地址。Name 字段是一个 string 类型的变量，占用16字节内存，所以 Age 字段的内存起始地址是 0xc00000a060 + 0x10 也就是 0xc00000a070。而 me 变量只是存放了结构体变量的内存地址，所以占用8个字节的内存而不是24字节。

```Go
import (
    "fmt"
    "unsafe"
)

func main() {
    me := &Person{
        Name: "HF",
        Age:  1,
    }

    fmt.Printf("pointer of me: %p, size of real struct: %d\n", me, unsafe.Sizeof(*me))
}
```

执行命令源码文件后输出：

```Bash
pointer of me: 0xc00000a060, size of real struct: 24
```

这里真正的结构体变量确实占用了16 + 8也就是24字节的内存。

#### 结构体比较

**如果结构体的全部成员都是可以比较的，那么结构体也是可以比较的。**

```Go
func main() {
    me := Person{Name: "HF"}
    he := Person{}

    fmt.Println(me == he) // false
    fmt.Println(me.Name == he.Name && me.Age == he.Age) // false
}
```

上下两种写法是等价的。

#### 继承（嵌入）--匿名结构体

一个结构体可以继承多个结构体。 **继承的结构体不能有同名字段。**

```Go
import (
    "net/http"
)
type Request struct{}
type T struct {
    http.Request // field name is "Request"
    Request // field name is "Request" 
}
```

编译时会报错 `duplicate field Request`

Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字，叫匿名成员。**匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。**

```Go
type Name struct {
    FirstName string
    LastName  string
}

type Person struct {
    Name
    Age int
}

func main() {
    me := new(Person)
    me.Name.FirstName = "Fluoride"
    me.Name.LastName = "Hydrogen"
    fmt.Printf("me: %#v", *me) // me: &main.Person{Name:main.Name{FirstName:"Fluoride", LastName:"Hydrogen"}, Age:0}
}
```

**但是这样类似文件目录递进的访问形式略显繁琐，得益于匿名成员的特性，直接访问子字段而不必给出完整的路径**：

```Go
func main() {
    me := new(Person)
    me.FirstName = "Fluoride"
    me.LastName = "Hydrogen"
    fmt.Printf("me: %#v", *me)
}
```

所有的内置类型和自定义类型都是可以作为匿名字段的，但是结构体面值并不支持省略写法。

##### 嵌套带有方法的匿名结构体

- 示例

  嵌入的匿名结构的方法将被父结构体拥有；而且，切入的匿名结构的方法，还可以被父级结构体再实现一遍，但是实现过后，会被覆盖

  ```go
  type Name struct {
      FirstName string
      LastName  string
  }
  
  func (n *Name) string() string{
      return n.FirstName+" "+n.LastName
  }
  
  type Person struct {
      Name
      Age int
  }
  
  // 覆盖匿名结构体方法
  func (p *Person) string() string{
      return "已被覆盖"
  }
  ```

- 好处

  上面的例子还没体现出匿名结构体的用途。

  试想这样一个场景：已经开发好了某一个工具类A，但是由于业务增长、扩容，原有的部分实现遇到性能瓶颈，那么怎么更好的修改这块代码呢？

  - 一种方案是：直接在原有的实现基础上，将有性能瓶颈的代码块进行更改，但是对代码侵入性比较大；
  - 另一种方案是：依然保留原方案的实现，在原方案的基础上，提供一个新的方案，将源对象作为匿名结构体嵌入到新方案的结构体，然后“重写”那些有性能瓶颈的方法，以实现更新换代。

#### 反射

Go 语言通过 reflect 包实现运行时的反射，允许程序操作任意类型的对象。 典型用法是用静态类型 interface{} 保存一个值，通过调用 `TypeOf()` 方法获取其动态类型信息，返回一个 Type 类型值。调用 `ValueOf `方法返回一个 Value 类型值，该值代表运行时的数据。

```Go
import (
    "fmt"
    "reflect"
)

type Person struct {
    Name string
    Age  int
}

func getFieldString(person *Person, field string) string {
    r := reflect.ValueOf(person)
    f := reflect.Indirect(r).FieldByName(field)
    return f.String()
}

func getFieldInteger(person *Person, field string) int {
    r := reflect.ValueOf(person)
    f := reflect.Indirect(r).FieldByName(field)
    return int(f.Int())
}

func main() {
    me := &Person{Name: "HF", Age: 1}
    fmt.Printf("my name: %s\n", getFieldString(me, "Name")) // my name: HF
    fmt.Printf("my age: %d\n", getFieldInteger(me, "Age")) // my age: 1
}
```

#### 标签

在 Go 语言中首字母大小写有特殊的语法含义，小写包外无法引用。由于需要和其它的系统进行数据交互，例如转换成 JSON。这时如果用属性名来作为键值可能不一定会符合项目要求。tag 在转换成其它数据格式的时候，强制使用自定义的字段作为键值。

```Go
import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func main() {
    me := &Person{Name: "HF", Age: 1}
    json, err := json.Marshal(me)
    if err != nil {
        panic(err)
    }
    fmt.Printf("JSON of me: %s\n", string(json)) // JSON of me: {"name":"HF","age":1}
}
```

如果不带上自定义 tag，编组后的 JSON 是 `{"Name":"HF","Age":1}`，直接使用 Person 结构体的字段名作为 JSON 字段名。

tag 甚至支持更丰富的自定义项，比如 `omitempty`：

```Go
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"`
}

func main() {
    me := &Person{Name: "HF"}
    json, err := json.Marshal(me)
    if err != nil {
        panic(err)
    }
    fmt.Printf("JSON of me: %s\n", string(json)) // JSON of me: {"name":"HF"}
}
```

如果结构体的某个字段为零值或者空值，JSON 编组时将跳过这个字段。

关于tag的映射，gin框架集成了validator库v10版的反射校验，可以通过设置结构体tag，从而达到参数校验的目的，省去诸多业务代码。



### 二、java与go的并发比对

很多有经验的工程师在使用基于 JVM 的语言时，都会看到这样的错误：

```txt
[error] (run-main-0) java.lang.OutOfMemoryError: unable to create native thread:
[error] java.lang.OutOfMemoryError: unable to create native thread:
[error] 	at java.base/java.lang.Thread.start0(Native Method)
[error] 	at java.base/java.lang.Thread.start(Thread.java:813)
...
[error] 	at java.base/java.lang.Thread.run(Thread.java:844)
```

呃，这是由线程所造成的`OutOfMemory`。在我的笔记本电脑上运行 Linux 操作系统时，仅仅创建 11500 个线程之后，就会出现这个错误。

如果你在 Go 语言上做相同的事情，启动永远处于休眠状态的 Goroutines，那么你会看到非常不同的结果。在我的笔记本电脑上，在我觉得实在乏味无聊之前，我能够创建七千万个 Goroutines。那么，为什么 Goroutines 的数量能够远远超过线程呢？要揭示问题的答案，我们需要一直向下沿着操作系统进行一次往返旅行。这不仅仅是一个学术问题，它对你如何设计软件有现实的影响。在生产环境中，我曾经多次遇到 JVM 线程的限制，有些是因为糟糕的代码泄露线程，有的则是因为工程师没有意识到 JVM 的线程限制。

#### 那到底什么是线程？

术语“线程”可以用来描述很多不同的事情。在本文中，我会使用它来代指一个逻辑线程。也就是：按照线性顺序的一系列操作；一个执行的逻辑路径。CPU 的每个核心只能真正并发同时执行一个逻辑线程。这就带来一个固有的问题：如果线程的数量多于内核的数量，那么有的线程必须要暂停以便于其他的线程来运行工作，当再次轮到自己的执行的时候，会将任务恢复。为了支持暂停和恢复，线程至少需要如下两件事情：

> 1. 某种类型的指令指针。也就是，当我暂停的时候，我正在执行哪行代码？
> 1. 一个栈。也就是，我当前的状态是什么？栈中包含了本地变量以及指向变量所分配的堆的指针。同一个进程中的所有线程共享相同的堆。


鉴于以上两点，系统在将线程调度到 CPU 上时就有了足够的信息，能够暂停某个线程、允许其他的线程运行，随后再次恢复原来的线程。这种操作通常对线程来说是完全透明的。从线程的角度来说，它是连续运行的。线程能够感知到重新调度的唯一方式是测量连续操作之间的计时。

回到我们最原始的问题：我们为什么能有这么多的 Goroutines 呢？

#### JVM 使用操作系统线程

尽管并非规范所要求，但是据我所知所有的现代、通用 JVM 都将线程委托给了平台的操作系统线程来处理。在接下来的内容中，我将会使用“用户空间线程（user space thread）”来代指由语言进行调度的线程，而不是内核 /OS 所调度的线程。操作系统实现的线程有两个属性，这两个属性极大地限制了它们可以存在的数量；任何将语言线程和操作系统线程进行 1:1 映射的解决方案都无法支持大规模的并发。

#### 在 JVM 中，固定大小的栈

**使用操作系统线程将会导致每个线程都有固定的、较大的内存成本**

采用操作系统线程的另一个主要问题是每个 OS 线程都有大小固定的栈。尽管这个大小是可以配置的，但是在 64 位的环境中，JVM 会为每个线程分配 1M 的栈。你可以将默认的栈空间设置地更小一些，但是你需要权衡内存的使用，因为这会增加栈溢出的风险。代码中的递归越多，就越有可能出现栈溢出。如果你保持默认值的话，那么 1000 个线程就将使用 1GB 的 RAM。虽然现在 RAM 便宜了很多，但是几乎没有人会为了运行上百万个线程而准备 TB 级别的 RAM。

#### Go 的行为有何不同：动态大小的栈

Golang 采取了一种很聪明的技巧，防止系统因为运行大量的（大多数是未使用的）栈而耗尽内存：Go 的栈是动态分配大小的，随着存储数据的数量而增长和收缩。这并不是一件简单的事情，它的设计经历了多轮的迭代。我并不打算讲解内部的细节（关于这方面的知识，有很多的博客文章和其他材料进行了详细的阐述），但结论就是每个新建的 Goroutine 只有大约 4KB 的栈。每个栈只有 4KB，那么在一个 1GB 的 RAM 上，我们就可以有 250 万个 Goroutine 了，相对于 Java 中每个线程的 1MB，这是巨大的提升。

#### 在 JVM 中：上下文切换的延迟

**从上下文切换的角度来说，使用操作系统线程只能有数万个线程**

因为 JVM 使用了操作系统线程，所以依赖操作系统内核来调度它们。操作系统有一个所有正在运行的进程和线程的列表，并试图为它们分配“公平”的 CPU 运行时间。当内核从一个线程切换至另一个线程时，有很多的工作要做。新运行的线程和进程必须要将其他线程也在同一个 CPU 上运行的事实抽象出去。我不会在这里讨论细节问题，但是如果你对此感兴趣的话，可以阅读更多的材料。这里比较重要的就是，切换上下文要消耗 1 到 100 微秒。这看上去时间并不多，相对现实的情况是每次切换 10 微秒，如果你想要每秒钟内至少调度每个线程一次的话，那么每个核心上只能运行大约 10 万个线程。这实际上还没有给线程时间来执行有用的工作。

#### Go 的行为有何不同：在一个操作系统线程上运行多个 Goroutines

Golang 实现了自己的调度器，允许众多的 Goroutines 运行在相同的 OS 线程上。就算 Go 会运行与内核相同的上下文切换，但是它能够避免切换至ring-0以运行内核，然后再切换回来，这样就会节省大量的时间。但是，这只是纸面上的分析。为了支持上百万的 Goroutines，Go 需要完成更复杂的事情。

即便 JVM 将线程放到用户空间，它也无法支持上百万的线程。假设在按照这样新设计系统中，新线程之间的切换只需要 100 纳秒。即便你所做的只是上下文切换，如果你想要每秒钟调度每个线程十次的话，你也只能运行大约 100 万个线程。更重要的是，为了完成这一点，我们需要最大限度地利用 CPU。要支持真正的大并发需要另外一项优化：当你知道线程能够做有用的工作时，才去调度它。如果你运行大量线程的话，其实只有少量的线程会执行有用的工作。Go 通过集成通道（channel）和调度器（scheduler）来实现这一点。如果某个 Goroutine 在一个空的通道上等待，那么调度器会看到这一点并且不会运行该 Goroutine。Go 更近一步，将大多数空闲的线程都放到它的操作系统线程上。通过这种方式，活跃的 Goroutine（预期数量会少得多）会在同一个线程上调度执行，而数以百万计的大多数休眠的 Goroutine 会单独处理。这样有助于降低延迟。

除非 Java 增加语言特性，允许调度器进行观察，否则的话，是不可能支持智能调度的。但是，你可以在“用户空间”中构建运行时调度器，它能够感知线程何时能够执行工作。这构成了像 Akka 这种类型的框架的基础，它能够支持上百万的 Actor。

### 结论

操作系统线程模型与轻量级、用户空间的线程模型之间的转换在不断发生，未来可能还会继续。对于高度并发的用户场景来说，这是唯一的选择。然而，它具有相当的复杂性。如果 Go 选择采用 OS 线程而不是采用自己的调度器和递增的栈模式的话，那么他们能够在运行时中减少数千行的代码。对于很多用户场景来说，这确实是更好的模型。复杂性可以被语言和库的编写者抽象出去，这样软件工程师就能编写大量并发的程序了。



### 三、接口

Go 语言内置了以下这些基础类型：

- 布尔型：bool
- 整型：int、byte、uint 等
- 浮点型：float32、float64
- 字符串：string
- 字符：rune
- 错误：error

这些都是具体的类型，Go 语言中还存在一种抽象类型，也就是接口类型，它不会暴露它所代表的对象的内部值的结构，而是展示出它们自己方法。简单来说，接口是**一组方法的集合**。

Go 语言的接口引入了“非侵入式”概念，不需要使用“显式”的首发去实现。而 Java 和 C++ 通常会这样来“显示”地创建一个类去实现接口：

```Java
// Java
interface Animal {
    public void eat();
    public void sleep();
}

public class Mammal implements Animal {
    @Override
    public void eat() {
        System.out.println("Mammal eats.");
    }

    @Override
    public void sleep() {
        System.out.println("Mammal sleeps.");
    }
}
```



```c++
// C++
class Animal {
    public:
        virtual void eat() = 0;
        virtual void sleep() = 0;
}

class Mammal: public Animal {
    public:
        void eat() {
            cout << "Mammal eats." << endl;
        }

        void sleep() {
            cout << "Mammal sleeps." << endl;
        }
}
```

在实现接口前必须要先定义接口，并且将类型和接口紧密绑定（明确声明实现类继承了某个接口），这种就是“侵入式”接口。修改接口会影响到所有实现了该接口的类型。

而在 Go 语言中：

```Go
type Animal interface {
    eat()
    sleep()
}
type Mammal struct {}

func (m *Mammal) eat() {
    fmt.Println("Mammal eats.")
}

func (m *Mammal) sleep() {
    fmt.Println("Mammal sleeps.")
}
```

以上第二段代码本身也是完整可用的，和第一段没有直接关系。但是有了上面的接口类型定义后，Mammal 结构确实也拥有了 Animal 接口定义的所有方法，那么 Mammal 类型也就自动实现了 Animal 接口。

所以接口指定的规则非常简单：表达一个类型属于某个接口只要这个类型实现这个接口。

```Go
func main() {
    var animal Animal = new(Mammal)
    animal.eat()
    animal.sleep()
}
```

接口和类型可以直接转换，**这种松散的对应关系系可以大幅降低因为接 口调整而导致的大量代码调整工作**。

#### 接口值

接口赋值在 Go 语言中分为两种:

1. 将对象实例赋值给接口
2. 将一个接口赋值给另一个接口

将某种类型的对象实例赋值给接口，要求该对象实例实现了接口要求的所有方法。上面的代码对应的是第一种情况，`Mammal` 结构体类型实现了 `eat()` 和 `sleep()` 两个 `Animal` 接口要求的方法。

定义两个接口，一个叫 `Animal`，一个叫 `Mammal`，两个接口都定义了 `eat()` 和 `sleep()` 方法。

```Go
type Animal interface {
    eat()
    sleep()
}

type Mammal interface {
    sleep()
    eat()
}

type Human struct {}

func (h *Human) eat() {
    fmt.Println("Human eats.")
}

func (h *Human) sleep() {
    fmt.Println("Human sleeps.")
}

func main() {
    var human1 *Human = new(human)
    var human2 *Mammal = human1
    var human3 *Animal = human2
   
}
```

两个接口的拥有的方法完全相同（次序不同也没关系），那两者就是相同的，可以相互赋值。

#### 空接口 interface

空接口不包含任何方法，对实现不做任何要求，类似 Java/C# 中所有类的基类：

```Go
type Any interface{}
```

**所以空接口可以赋值任何类型的实例**

最常用到的 fmt 标准库中的 Print() 系列方法，可以接受任意类型的参数：

```Go
func Printf(format string, a ...interface{}) (n int, err error) {
    return Fprintf(os.Stdout, format, a...)
}

func Fprintln(w io.Writer, a ...interface{}) (n int, err error) {
    p := newPrinter()
    p.doPrintln(a)
    n, err = w.Write(p.buf)
    p.free()
    return
}
```

#### 类型断言

类型断言检查它操作对象的动态类型是否和断言的类型匹配

```Go
t := i.(T)
```

检查 `i` 的类型是否为 `T`，如果是的话返回 `i` 的动态值，如果不是，类型断言失败产生运行时错误。

**确保被断言的接口值不是 nil。**

```Go
type Animal interface {
    eat()
    sleep()
}

type Mammal struct {}

func (m *Mammal) eat() {
    fmt.Println("Mammal eats.")
}

func (m *Mammal) sleep() {
    fmt.Println("Mammal sleeps.")
}

func main() {
    var animal Animal = new(Mammal)
    mammal, ok := animal.(*Mammal)
    if ok {
        mammal.eat()
        mammal.sleep()
    }
}
```

#### error 接口

`error` 类型实质上就是一个接口：

```Go
type error interface {
    Error() string
}
```

对照 `error` 接口实现一个自定义的错误类型：

```Go
type CustomError struct {
    Op string
    Err error
}

func (customErr *CustomError) Error() string {
    return fmt.Sprintf("op: %s, err: %s\n", customErr.Op, customErr.Err.Error())
}
```

错误处理：

```Go
err := op() // 某个操作
if err != nil {
    if customErr, ok := err.(*CustomError); ok && e.Err != nil {
        // 错误处理
    }
}
```

#### 类型开关

```Go
func newPrint(x interface{}) string {
	switch x := x.(type) {
	case nil:
		return "NULL"
	case int:
		return fmt.Sprintf("%d is int", x)
	case string:
		return fmt.Sprintf("%s is string", x)
	case bool:
		return fmt.Sprintf("%v is bool", x)
	case uint:
		return fmt.Sprintf("%d is uint", x)
	default:
		return fmt.Sprintf("unknown type!")
	}
}

func main() {
	fmt.Println(newPrint(1))
	fmt.Println(newPrint("hello"))
	fmt.Println(newPrint(true))
	fmt.Println(newPrint(1.0))
}
```

一个类型开关使用 `switch` 选择语句判断 `x.(type)`，每个 `case` 有至少一种类型。每一个 `case` 会被顺序的进行考虑，如果匹配就会执行 `case` 中的语句。

#### 建议

> 因为在 Go 语言中只有当两个或更多的类型实现一个接口时才使用接口，它们必定会从任意特 定的实现细节中抽象出来。结果就是有更少和更简单方法的更小的接口。当新的类型出现时，小的接口更容易满足。对于接口设计的一 个好的标准就是 ask only for what you need（只考虑你需要的东西）。



### 四、接口与反射的关系

作为 Go 语言中为抽象而生的基本工具之一，接口在分配值时存储类型信息，而反射则是一种在运行时检查类型和值信息的方法。

**`reflect` 包提供了检查接口在运行时，检查类型、修改值的方法，Go 使用它实现了反射。**

#### 向接口分配值

接口打包了三样东西：

- 值
- 方法集
- 存储的值的类型

如下图： ![img](https://pic.crazytaxii.com/interface.svg)

图中可以清楚地看到接口的三个部分：`_type` 是类型信息，`*data` 是指向真实值的指针，`itab` 包含方法集。

当一个函数接受了一个接口作为参数时，传递给函数的接口打包了值、方法集和类型。

#### 通过反射包在运行时检查接口数据

一旦值存储在接口中，可以使用 `reflect` 包来检查它的各个部分。我们不能直接检查接口结构，而反射包维护一份有权限访问的接口结构的副本。

甚至通过反射对象访问接口，也是和底层接口直接相关的。

`reflect.Type` 和 `reflect.Value` 提供访问接口成分的方法。`reflect.Type` 暴露接口的 `_type` 部分（有关类型的信息），而 `reflect.Value` 允许程序员检查和操作值。

#### reflect.Type （检查类型）

`reflect.TypeOf()` 函数用来提取值的类型。既然它唯一的参数是空接口，传递给它的参数将被分配成一个接口，因此就有了类型、方法集和值。

`reflect.TypeOf()` 函数返回 `reflect.Type`，它有些方法可以让你查出值的类型。

![img](https://pic.crazytaxii.com/reflect-type.svg)

```Go
package main

import (
    "fmt"
    "log"
    "reflect"
)

type Gift struct {
    Sender    string
    Recipient string
    Number    uint
    Contents  string
}

func main() {
    g := Gift{
        Sender:    "Hank",
        Recipient: "Sue",
        Number:    1,
        Contents:  "Scarf",
    }

    t := reflect.TypeOf(g)

    if kind := t.Kind(); kind != reflect.Struct {
        log.Fatalf("This program expects to work on a struct; we got a %v instead.", kind)
    }

    for i := 0; i < t.NumField(); i++ {
        f := t.Field(i)
        fmt.Printf("Field %03d: %-10.10s %v", i, f.Name, f.Type.Kind())
    }
}
```

这段代码的目的是打印 `Gift` 结构的字段。当 `g` 作为参数传递给 `reflect.TypeOf()` 函数，`g` 就被分配成一个接口，编译器自动填充类型、方法集和值三样东西。这就允许我们遍历接口结构中类型部分的 `[]fields`：

```txt
Field 000: Sender     string
Field 001: Recipient  string
Field 002: Number     uint
Field 003: Contents   string
```

#### reflect.Method（检查方法集）

`reflect.Type` 类型也允许访问 `itab` 部分来提取接口的方法信息。

![img](https://pic.crazytaxii.com/reflect-method.svg)

```Go
package main

import (
    "log"
    "reflect"
)

type Reindeer string

func (r Reindeer) TakeOff() {
    log.Printf("%q lifts off.", r)
}

func (r Reindeer) Land() {
    log.Printf("%q gently lands.", r)
}

func (r Reindeer) ToggleNose() {
    if r != "rudolph" {
        panic("invalid reindeer operation")
    }
    log.Printf("%q nose changes state.", r)
}

func main() {
    r := Reindeer("rudolph")

    t := reflect.TypeOf(r)

    for i := 0; i < t.NumMethod(); i++ {
        m := t.Method(i)
        log.Printf("%s", m.Name)
    }
}
```

这段代码迭代 `itab` 中存储的数据并展示了每个方法的名称：

```txt
Land
TakeOff
ToggleNose
```

#### reflect.Value（检查值）

到此为止我们已经讨论了如何查看类型和方法，最后 `reflect.Value` 提供了接口中存储的真实值的信息。

与 `reflect.Value` 相关的方法必然将类型信息与真实值联合。举个例子，要提取结构体的字段，反射包必须结合接口的布局信息——尤其是字段的信息和 `_type` 中存储的字段偏移量还有 `*data` 指向的真实值。

![img](https://pic.crazytaxii.com/reflect-value.svg)

```Go
package main

import (
    "fmt"
    "log"
    "reflect"
)

type Child struct {
    Name  string
    Grade int
    Nice  bool
}

type Adult struct {
    Name       string
    Occupation string
    Nice       bool
}

// Search a slice of structs for Name field that is "Hank" and set its Nice field to true.
func nice(i interface{}) {
    // Retrieve the underlying value of i. We know that i is an interface.
    v := reflect.ValueOf(i)

    // we're only interested in slices to let's check what kind of value v is. If it isn't a slice, return immediately.
    if v.Kind() != reflect.Slice {
        return
    }

    // v is a slice. Now let's ensure that it is a slice of structs. If not, return immediately.
    if e := v.Type().Elem(); e.Kind() != reflect.Struct {
        return
    }

    // Determine if our struct has a Name field of type string and a Nice field of type bool
    st := v.Type().Elem()

    if nameField, found := st.FieldByName("Name"); found == false || nameField.Type.Kind() != reflect.String {
        return
    }

    if niceField, found := st.FieldByName("Nice"); found == false || niceField.Type.Kind() != reflect.Bool {
        return
    }

    // Set any Nice fields to true where the Name is "Hank"
    for i := 0; i < v.Len(); i++ {
        e := v.Index(i)
        name := e.FieldByName("Name")
        nice := e.FieldByName("Nice")

        if name.String() == "Hank" {
            nice.SetBool(true)
        }
    }
}

func main() {
    children := []Child{
        {Name: "Sue", Grade: 1, Nice: true},
        {Name: "Ava", Grade: 3, Nice: true},
        {Name: "Hank", Grade: 6, Nice: false},
        {Name: "Nancy", Grade: 5, Nice: true},
    }

    adults := []Adult{
        {Name: "Bob", Occupation: "Carpenter", Nice: true},
        {Name: "Steve", Occupation: "Clerk", Nice: true},
        {Name: "Nikki", Occupation: "Rad Tech", Nice: false},
        {Name: "Hank", Occupation: "Go Programmer", Nice: false},
    }

    fmt.Printf("adults before nice: %v", adults)
    nice(adults)
    fmt.Printf("adults after nice: %v", adults)

    fmt.Printf("children before nice: %v", children)
    nice(children)
    fmt.Printf("children after nice: %v", children)
}
```

```txt
adults before nice: [{Bob Carpenter true} {Steve Clerk true} {Nikki Rad Tech false} {Hank Go Programmer false}]
adults after nice: [{Bob Carpenter true} {Steve Clerk true} {Nikki Rad Tech false} {Hank Go Programmer true}]
children before nice: [{Sue 1 true} {Ava 3 true} {Hank 6 false} {Nancy 5 true}]
children after nice: [{Sue 1 true} {Ava 3 true} {Hank 6 true} {Nancy 5 true}]
```

注意，`nice()` 可以更改你传递给它的任意切片的值，不管它接收到了什么类型的参数。

#### 结论

Go 中的反射使用接口和 `reflect` 反射包实现，没啥黑科技——使用反射时就是直接访问接口里面的内容。

**接口更像是一面镜子，允许程序来检查自身。**

尽管 Go 是一门静态语言，但是反射和接口相结合非常强大，这个通常动态语言才有。

要想更多关于反射的信息，可以看看包的文档。



## 五、Go 避免在堆中的高开销垃圾回收

