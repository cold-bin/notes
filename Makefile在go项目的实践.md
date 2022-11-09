---
title: "Makefile在go项目的实践"
date: 2022-07-16T22:13:26+08:00
tags: [Makefile]
categories: [Golang]
draft: false
---

# Make介绍

**make命令**是GNU的工程化编译工具，用以实现工程化的管理，提高开发效率。

Make解释[Makefile](https://so.csdn.net/so/search?q=Makefile&spm=1001.2101.3001.7020) 中的指令（应该说是规则）。在Makefile文件中描述了整个工程所有文件的编译顺序、编译规则。Makefile 有自己的书写格式、关键字、函数。像C 语言有自己的格式、关键字和函数一样。而且在Makefile 中可以使用系统shell所提供的任何命令来完成想要的工作。

# Makefile文件

构建规则都写在Makefile文件里面，要学会如何Make命令，就必须学会如何编写Makefile文件。

## 文件格式

Makefile文件由一系列规则（rules）构成。

每条规则要说明构建的依赖条件，及怎么样去构建。那么格式如下，

```txt
# rule
<target> : <prerequisites> 
[tab]  <commands>
```

target: 目标

prerequisites： 先决条件，或者说依赖条件

tab: 使用tab来缩进

command: 要执行的命令（可以说是小型的shell代码块）

### target目标

一个目标，可以是文件名，也可以是某个操作的名字（伪目标），这个名字由自己定义，用来指明要构建的对象。

```makefile
create:
    touch newfile
```

比如上面这条规则，伪目标为create，命令作用为创建一个文件。要想构建这个操作，调用`make create`(指定目标进行构建编译)即可。

**但是如果目录下，存在一个文件名为create，那么构建命令就不会去执行。为了解决这个问题，当使用伪目标时，可以明确声明create是“伪目标“，告诉make跳过文件检查。**

```makefile
.PHONY: clean
create:
    touch newfile
```

如果Make命令运行时没有指定目标，默认会执行Makefile文件的第一个目标。

### prerequisites先决条件

先决条件，通常是文件名，多个名字用空格分隔。

它定义了一个是否进行重新构建的判断标准： 如果有任何一个先决文件发生改变（时间戳更新），就要重新构建。

```makefile
result.txt: source.txt
    cp source.txt result.txt
```

上面代码中，构建 result.txt 的前置条件是 source.txt 。如果当前目录中，source.txt 已经存在，那么`make result.txt`可以正常运行，否则必须再写一条规则，来生成 source.txt 。

```makefile
source.txt:
    echo "this is the source" > source.txt
```

上面代码中，source.txt后面没有前置条件，就意味着它跟其他文件都无关，只要这个文件还不存在，每次调用`make source.txt`，它都会生成。

```bash
make result.txt
make result.txt
```

上面命令连续执行两次`make result.txt`。第一次执行会先新建 source.txt，然后再新建 result.txt。第二次执行，make发现 source.txt 没有变动（时间戳晚于 result.txt），就不会执行任何操作，result.txt 也不会重新生成。

如果需要生成多个文件，往往采用下面的写法。

```makefile
source: file1 file2 file3
```

上面代码中，source 是一个伪目标，只有三个前置文件，没有任何对应的命令。

```bash
make source
```

执行`make source`命令后，就会一次性生成 file1，file2，file3 三个文件。这比下面的写法要方便很多。

```bash
make file1
make file2
make file3
```

**如果先决条件也是伪目标，即不是一个实实在在的、真实存在的文件，而是仅仅是一个目标的标识符。那么在当前目标执行前，就会先去先决条件对应的伪目标去递归执行。当前目标被指定make时，会先调用先决条件的目标，如果先决条件也具备这样的依赖时，也会如此递归调用下去**

> 
>
> target: target1 target2
>
> ​	echo > "this tartget"
>
> 
>
> target1:
>
> ​	echo > "this is target1"
>
> 
>
> target2:
>
> ​	echo > "this is target2"
>
> 

### commands命令

命令是构建目标时具体执行的指令，由一行或多行shell组成。每行命令之前必须有一个tab键缩进。如果想用其他键缩进，可以用内置变量.RECIPEPREFIX声明。

```makefile
.RECIPEPREFIX = >
hello:
> echo Hello, world
```

> 需要注意的是，每行shell在一个单独的bash进程中执行，多进程间没有继承关系。

```makefile
var:
    export name=wangpeng
    echo "myname is $name"
```

运行上面的构建 ，发现变量name是取不到的，因为两行shell在两个独立的bash中运行。

最直接的方法就是将两行shell写到一行中，

```makefile
var:
    export name=wangpeng; echo "myname is $name"
```

第二种办法，在换行前加反斜杠\转义，

```makefile
var:
    export name=wangpeng \
    echo "myname is $name"
```

还有第三种办法是使用`。ONESHELL`内置命令。

```makefile
.ONESHELL:
var:
    export name=wangpeng
    echo "myname is $name"
```

## 文件语法

### 注释

行首井号（#）表示注释。

### 回显

回显是指，在执行到每行命令前，将命令本身打印出来。

```makefile
test:
    # 这是测试
```

执行上面构建会输出

```bash
make test
# 这是测试
```

在命令的前面加上@，就可以关闭回声。

```makefile
test:
    @# 这是测试
```

这下构建时就不会有任何输出。

### 通配符

Makefile 的通配符与 Bash 一致，主要有星号（*）、问号（？）.比如* .text 表示所有后缀名为text的文件。

### 模式匹配

Make命令允许对文件名，进行类似正则运算的匹配，主要用到的匹配符是%。比如，假定当前目录下有 f1.c 和 f2.c 两个源码文件，需要将它们编译为对应的对象文件。

```makefile
%.o: %.c
```

等同于下面的写法。

```makefile
f1.o: f1.c
f2.o: f2.c
```

使用匹配符%，可以将大量同类型的文件，只用一条规则就完成构建。

### 变量和赋值符

Makefile 允许自定义变量。

```makefile
txt = Hello World
test:
    @echo $(txt)
```

调用shell中的变量，需要使用两个美元符号$$。

Makefile一共提供了四个赋值运算符 （=、:=、?=、+=），它们的区别请看[StackOverflow](http://stackoverflow.com/questions/448910/makefile-variable-assignment)。

```makefile
VARIABLE = value
# 在执行时扩展，允许递归扩展。

VARIABLE := value
# 在定义时扩展。

VARIABLE ?= value
# 只有在该变量为空时才设置值。

VARIABLE += value
# 将值追加到变量的尾端。
```

### 内置变量

Make命令提供一系列内置变量，比如，$(CC)指向当前使用的编译器，$(MAKE) 指向当前使用的Make工具。这主要是为了跨平台的兼容性，详细的内置变量清单见[手册](https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html)。

```makefile
output:
    $(CC) -o output input.c
```

### 判断和循环

Makefile使用 Bash 语法，完成判断和循环。

```makefile
ifeq ($(CC),gcc)
  libs=$(libs_for_gcc)
else
  libs=$(normal_libs)
endif
```

上面代码判断当前编译器是否 gcc ，然后指定不同的库文件。

```makefile
LIST = one two three
all:
    for i in $(LIST); do \
        echo $$i; \
    done

# 等同于

all:
    for i in one two three; do \
        echo $i; \
    done
```

上面代码的运行结果。

```txt
one
two
three
```

# Makefile例子演示

## 1. 执行多个目标

```makefile
.PHONY: cleanall cleanobj cleandiff

cleanall: cleanobj cleandiff
        rm all

cleanobj:
        rm *.o

cleandiff:
        rm *.diff
```

上面代码可以调用不同目标，删除不同后缀名的文件，也可以调用一个目标（cleanall），删除所有指定类型的文件。

## 2. 构建golang项目

以下Makefile仅供参考，[项目仓库](https://github.com/liuhaibin123456789/web-app/blob/master/Makefile)

```makefile
# 伪目标，如果未指定终极目标，将默认使用Makefile里的第一个目标，
# 即Makefile里最靠前的规则（本Makefile是all这个目标）
# 因此，当执行make时，会默认第一个目标为终极目标，所以all后面的依赖都会执行下去
# 因此，可以将all后面放置一些必须编译的默认选项，在只使用make时；当然也可以使用make指定终极目标进行构建
.PHONY: openssl format build build_linux build_win clean swag \ 
docker-build help format test run
all: openssl format build

# 声明编译项目的文件名
BUILD_NAME=web_app

# swagger接口文档初始化
swag:
	@swag init

# 在./config目录，签发自建的tls证书
# 或者使用go标准库：go run $GOROOT/src/crypto/tls/generate_cert.go --host localhost
openssl:
	@openssl genrsa -out ./config/key.pem 2048;openssl req -new -x509 -key ./config/key.pem -out ./config/cert.pem -days 3650

# 使用Dockerfile对项目打包编译出镜像
docker-build: swag
	@docker build -t ${BUILD_NAME}:1.0

# 格式化项目
format:
	@go fmt ./
	@go vet ./

# 测试代码
test: swag
	@go test -v #回归测试

# 直接运行项目根目录下已经编译好的二进制文件
run:
	./${BUILD_NAME}

# 默认编译
build: test
	@go build -o ${BUILD_NAME} ${SOURCE}

# 交叉编译--适应linux系统
build_linux: test
	@CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ${BUILD_NAME} .

# 交叉编译--适应windows系统
build_win: test
    @CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o ${BUILD_NAME} .

# 清除编译文件
clean:
	@go clean

# 帮助命令
help:
	@echo "make - 格式化 Go 代码、更新swagger文档、生成tls证书、测试代码、编译生成二进制文件"
	@echo "make docker-build - 构建本项目的Docker镜像"
	@echo "make build - 编译 Go 代码, 生成当前环境默认的二进制文件"
	@echo "make build_linux - 编译 Go 代码, 生成linux环境二进制文件"
	@echo "make build_win - 编译 Go 代码, 生成windows环境二进制文件"
	@echo "make run - 直接运行 Go 代码"
	@echo "make clean - 移除二进制文件和 vim swap files"
	@echo "make format - 运行 Go 工具 'fmt' and 'vet'"
```



另一个例子：

```makefile
.PHONY: all build clean run check cover lint docker help
BIN_FILE=hello
all: check build
build:
    @go build -o "${BIN_FILE}"
clean:
    @go clean
    rm --force "xx.out"
test:
    @go test
check:
    @go fmt ./
    @go vet ./
cover:
    @go test -coverprofile xx.out
    @go tool cover -html=xx.out
run:
    ./"${BIN_FILE}"
lint:
    golangci-lint run --enable-all
docker:
    @docker build -t leo/hello:latest .
help:
    @echo "make 格式化go代码 并编译生成二进制文件"
    @echo "make build 编译go代码生成二进制文件"
    @echo "make clean 清理中间目标文件"
    @echo "make test 执行测试case"
    @echo "make check 格式化go代码"
    @echo "make cover 检查测试覆盖率"
    @echo "make run 直接运行程序"
    @echo "make lint 执行代码检查"
    @echo "make docker 构建docker镜像"
```

这样就很方便地通过一个make命令完成对项目的构建。
