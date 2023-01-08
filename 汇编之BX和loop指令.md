---
title: "汇编之BX和loop指令"
date: 2022-12-09T21:58:07+08:00
tags: [汇编BX和loop指令]
categories: [汇编]
draft: false
---

## [bx]

1、[bx]是什么呢？

和[0]类似，[0]表示[内存](https://so.csdn.net/so/search?q=内存&spm=1001.2101.3001.7020)单元，它的偏移地址是0；


2、内存单元的描述

我们要完整地描述一个内存单元，需要两种信息：
（1）内存单元的地址；
（2）内存单元的长度（类型）；

用[0]表示一个内存单元时，0表示单元的偏移地址，段地址默认在 ds 中，单元的长度（类型）可以由具体指令中的其他操作对象（比如说[寄存器](https://so.csdn.net/so/search?q=寄存器&spm=1001.2101.3001.7020)）指出；

[bx]同样也表示一个内存单元，它的偏移地址在bx中，比如下面的指令：

- mov ax,[bx]
- mov al,[bx]
  

3、 描述性符号“()”

为了描述上的简洁，以后将使用一个描述性的符号 “() ”来表示一个寄存器或一个内存单元中的内容。比如：

（1）ax 中的内容为0010H，我们可以这样来描述：`(ax)=0010H`；
（2）2000:1000 处的内容为0010H，我们可以这样来描述：`(21000H)=0010H`；
（3）对于 `mov ax,[2]` 的功能，我们可以这样来描述：`(ax)=((ds)*16+2)`；
（4）对于 `mov [2],ax` 的功能，我们可以这样来描述：`((ds)*16+2)=(ax)`；
（5）对于 `add ax,2` 的功能，我们可以这样来描述：`(ax)=(ax)+2`；
（6）对于 `add ax,bx` 的功能，我们可以这样来描述：`(ax)=(ax)+(bx)`；
（7）对于 `push ax` 的功能，我们可以这样来描述：`(sp) = (sp)-2 ((ss)*16＋(sp))=(ax)`
（8）对于 `pop ax` 的功能，我们可以这样来描述：`(ax)=((ss)*16+(sp)) (sp)=(sp)+2`


4、约定符号 idata 表示常量

我们在 Debug 中写过类似的指令：`mov ax,[0]`，表示将 `ds:0` 处的数据送入 ax 中。指令中，在“[…]”里用一个常量0表示内存单元的偏移地址。以后，我们用 `idata` 表示常量。

例如：

- `mov ax,[idata]` 就代表 `mov ax,[1]`、`mov ax,[2]`、`mov ax,[3]` 等；
- `mov bx,idata`就代表 `mov bx,1`、`mov bx,2`、`mov bx,3` 等；
- `mov ds,idata`就代表 `mov ds,1`、`mov ds,2` 等；

我们之前的用法都是非法指令；


5、[bx]

```asm
mov [bx],ax
```

功能：bx 中存放的数据作为一个偏移地址 EA，段地址 SA 默认在 ds 中，将 ax 中的数据送入内存 SA:EA 处，即：`(ds*16 +(bx)) = (ax)`；


问题：程序和内存中的情况如下图所示，写出程序执行后，21000H~21007H 单元中的内容。
![img](https://img-blog.csdnimg.cn/07c78c66e7594fdbbffc174d1b0fcebe.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAaWQxMHQu,size_20,color_FFFFFF,t_70,g_se,x_16)


（1）先看一下程序的前三条指令：

```asm
mov ax,2000H
mov ds,ax
mov bx,1000H
```

这三条指令执行后，`ds=2000H,bx=1000H`；


（2）再看第四条指令：`mov ax,[bx]`，

- 指令执行前：
  - `ds=2000H`，`bx=1000H`，则 `mov ax,[bx]` 将把内存2000:1000处的字型数据送入 ax 中；
- 指令执行后:
  - `ax=00beH`；
    

（3）再看第五六两条指令：

```asm
inc bx
inc bx
```

- 指令执行前：
  - `bx=1000H`；
- 指令执行后:
  - `bx=0002H`；
    

（4）再看第七条指令：`mov [bx],ax`，

- 指令执行前：
  - `ds=2000H`，`bx=1002H`，则 `mov [bx],ax` 将把 ax 中的数据送入内存2000:1002处；
- 指令执行后:
  - 2000:1002单元的内容为`BE`，2000:1003单元的内容为`00`；

之后的操作类似，最终结果：
![img](https://img-blog.csdnimg.cn/a22a7d5d910d4ba189e30eb6d710bd99.png) 

## Loop指令

这个指令和循环有关；

1、指令的格式是：loop 标号，CPU 执行 loop 指令的时候，要进行两步操作：

- `(cx)=(cx)-1`;
- 判断 cx 中的值，若不为零，则转至标号处执行；程序若为零，则向下执行。
  

2、通常，loop 指令实现循环，cx 中存放循环的次数；

问题：编程计算212；

```asm
 assume cs:code
code segment
    mov ax,2
    mov cx,11
s:  add ax,ax
    loop s
    mov ax,4c00h
    int 21h
code ends
end
```

分析：

（1）标号：在汇编语言中，标号代表一个地址，此程序中有一个标号 s 。它实际上标识了一个地址，这个地址处有一条指令：`add ax,ax`；

（2）`loop s`：CPU 执行 `loop s` 的时候，要进行两步操作：

- `(cx)=(cx)-1`；
- 判断 cx 中的值，不为0则转至标号 s 所标识的地址处执行(这里的指令是 `add ax,ax`)，如果为0则执行下一条指令(下一条指令是 `mov ax,4c00h`)；

总结：

（1）在 cx 中存放循环次数；
（2）loop 指令中的标号所标识地址要在前面；
（3）要循环执行的程序段，要写在标号和 loop 指令的中间；

用 cx 和 loop 指令相配合实现循环功能的程序框架如下：

```
mov cx,循环次数
s:
循环执行的程序段
loop s
```

 

3、在 Debug 中跟踪供 loop 指令实现的循环程序

注意：在汇编程序中，数据不能以字母开头，如果要输入像 FFFFH 这样的数，则要在前面添加一个0；

在 debug 程序中引入 G 命令和 P 命令：

1. G命令
   - G命令如果后面不带参数，则一直执行程序，直到程序结束；
   - G命令后面如果带参数，则执行到 ip 为那个参数地址停止；
   
2. P命令
   - T命令相当于单步进入（step into）；
   - P命令相当于单步通过（step over）；
     

## Debug 和汇编编译器 Masm 对指令的不同处理

1、在 debug 中，可以直接用指令 `mov ax,[0]` 将偏移地址为0号单元的内容赋值给 ax；

2、但通过 masm 编译器，`mov ax,[0]` 会被编译成 `mov ax,0`；

- 要写成这样才能实现：`mov ax,ds:[0]`；

- 也可以写成这样：

  ```ASM
  mov bx,0
  mov ax,[bx]
  ```

  或者

   

  ```
  mov ax,ds:[bx]
  ```

## loop 和 [bx] 的联合应用

计算 ffff:0~ffff:b 单元中的数据的和，结果存储在 dx 中：

1、注意两个问题：

- 12个8位数据加载一起，最后的结果可能会超出8位（越界），故要用16位寄存器存放结果；
- 将一个8位的数据加入到16位寄存器中，类型不匹配，8位的数据不能与16位相加；

2、【解决办法】

- 把原来8位的数据，先通过通用寄存器 ax，将它们转化成16位的；

3、代码如下：

```asm
assume cs:codesg
codesg segment
start:
	;指定数据段
	mov ax,0ffffh
	mov ds,ax

	;初始化
	mov ax,0
	mov dx,0
	mov bx,0

	;指定循环次数 12次
	mov cx,0ch
circ:
	;把8位数据存入al中,即ax中存放的是[bx]转化之后的16位数据，前8位都是0
	mov al,[bx]
	;进行累加
	add dx,ax
	;bx自增，变化内存的偏移地址
	inc bx
	loop circ

	;程序返回
	mov ax,4c00h
	int 21H
codesg ends
end start
```

 

## 段前缀

1. 指令 `mov ax,[bx]` 中，内存单元的偏移地址由 bx 给出，而段地址默认在 ds 中；
   
2. 我们可以在访问内存单元的指令中，显式地给出内存单元的段地址所在的段寄存器，比如 `mov ax,ds:[0]`，`mov ax,ds:[bx]` 这里的 ds 就叫做【段前缀】；
   

## 一段安全的空间

1. 8086模式中，随意向一段内存空间写入内容是很危险的，因为这段空间中可能存放着【重要的系统数据或代码】；
   
2. 在一般的 PC 机中，DOS 方式下，DOS 和其他合法的程序一般都不会使用【0:200~0:2FF】的256个字节的空间。所以，我们使用这段空间是安全的；
   

## 段前缀的使用

将内存 ffff:0~ffff:b 段元中的数据拷贝到 0:200~0:20b 单元中；

```asm
assume cs:code
code segment

    mov bx,0            ; (bx)=0，偏移地址从0开始
    mov cx,12           ; (cx)=12，循环12次

s:  mov ax,offffh
    mov ds,ax           ; (ds)=0ffffh
    mov dl,[bx]         ; (dl)= ((ds)*16+(bx))，将ffff:bx中的数据送入dl

    mov ax,0020h
    mov ds,ax           ; (ds)=0020h
    mov [bx],dl         ; ((ds)*16+(bx))=(dl)，将中dl的数据送入0020:bx

    inc bx              ; (bx)=(bx)+1
    loop s

    mov ax,4c00h
    int 21h

code ends
end
```

由于上述效率不高，因此进行如下优化：

```asm
assume cs:code
code segment

    mov ax,offffh
    mov ds,ax           ; (ds)=0ffffh

    mov ax,0020h
    mov es,ax           ; (es)=0020h

    mov bx,0            ; (bx)=0，此时ds:bx指向ffff:0，es:bx指向0020:0
    mov cx,12           ; (cx)=12，循环12次

s:  mov dl,[bx]         ; (dl)= ((ds)*16+(bx))，将ffff:bx中的数据送入dl
    mov es:[bx],dl      ; ((es)*16+(bx))=(dl)，将中dl的数据送入0020:bx
    inc bx              ; (bx)=(bx)+1
    loop s

    mov ax,4c00h
    int 21h

code ends
end
```
