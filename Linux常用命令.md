---
title: "Linux常用命令"
date: 2023-01-10T19:46:21+08:00
tags: [Linux命令]
categories: [Linux,服务器]
draft: false
---

# linux常用命令

## 零、linux初窥

![](https://raw.githubusercontent.com/cold-bin/img-for-cold-bin-blog/master/img/linuxFHS%E5%87%86%E5%88%99.png)

## 一、命令行操作体验

在 linux 中，最最重要的就是命令，这就包含了 2 个过程，输入和输出

- 输入：输入当然就是打开终端，然后按键盘输入，然后按回车，输入格式一般就是这类的

```bash
#创建一个名为 file 的文件，touch是一个命令
touch file

#进入一个目录，cd是一个命令
cd /etc/

#查看当前所在目录
pwd
```

- 输出：输出会返回你想要的结果，比如你要看什么文件，就会返回文件的内容。如果只是执行，执行失败会告诉你哪里错了，如果执行成功那么会没有输出，因为 linux 的哲学就是：没有结果就是最好的结果

### 1. 开始

如图，双击桌面上的 `Xfce 终端` 图标打开终端后系统会自动运行 Shell 程序，然后我们就可以输入命令让系统来执行了：

![1](https://doc.shiyanlou.com/document-uid735639labid2timestamp1531471836262.png)

#### 1) 重要快捷键

真正学习命令行之前，你先要掌握几个十分有用、必需掌握的小技巧：

##### [Tab]

使用`Tab`键来进行命令补全，`Tab`键一般是在字母`Q`旁边，这个技巧给你带来的最大的好处就是当你忘记某个命令的全称时可以只输入它的开头的一部分，然后按下`Tab`键就可以得到提示或者帮助完成：

![1](https://doc.shiyanlou.com/document-uid735639labid2timestamp1531472459081.png)

当然不止补全命令，补全目录、补全命令参数都是没问题的：

![1](https://doc.shiyanlou.com/document-uid735639labid2timestamp1531472445671.png)

##### [Ctrl+c]

想想你有没有遇到过这种情况，当你在 Linux 命令行中无意输入了一个不知道的命令，或者错误地使用了一个命令，导致在终端里出现了你无法预料的情况，比如，屏幕上只有光标在闪烁却无法继续输入命令，或者不停地输出一大堆你不想要的结果。你想要立即停止并恢复到你可控的状态，那该怎么办呢？这时候你就可以使用`Ctrl+c`键来强行终止当前程序（你可以放心它并不会使终端退出）。

尝试输入以下命令：

```bash
tail
```

然后你会发现你接下来的输入都没有任何反应了，只是将你输入的东西显示出来，现在你可以使用`Ctrl+c`，来中断这个你目前可能还不知道是什么的程序（在后续课程中我们会具体解释这个`tail`命令是什么）。

又或者输入：

```bash
find /
```

![pic](https://doc.shiyanlou.com/document-uid735639labid2timestamp1531471910323.png)

显然这不是你想的结果，可以使用`Ctrl+c`结束。

虽然这个按着很方便，但不要随便按，因为有时候，当你看到终端没有任何反应或提示，也不能接受你的输入时，可能只是运行的程序需要你耐心等一下，就不要急着按`Ctrl+c`了。

##### 其他一些常用快捷键

| 按键            | 作用                                         |
| --------------- | -------------------------------------------- |
| `Ctrl+d`        | 键盘输入结束或退出终端                       |
| `Ctrl+s`        | 暂停当前程序，暂停后按下任意键恢复运行       |
| `Ctrl+z`        | 将当前程序放到后台运行，恢复到前台为命令`fg` |
| `Ctrl+a`        | 将光标移至输入行头，相当于`Home`键           |
| `Ctrl+e`        | 将光标移至输入行末，相当于`End`键            |
| `Ctrl+k`        | 删除从光标所在位置到行末                     |
| `Alt+Backspace` | 向前删除一个单词                             |
| `Shift+PgUp`    | 将终端显示向上滚动                           |
| `Shift+PgDn`    | 将终端显示向下滚动                           |

#### 2) 利用历史输入命令

很简单，你可以使用键盘上的方向上键`↑`，恢复你之前输入过的命令，你一试便知。

#### 3) 通配符

通配符是一种特殊语句，主要有星号（*）和问号（?），用来对字符串进行模糊匹配（比如文件名、参数名）。当查找文件夹时，可以使用它来代替一个或多个真正字符；当不知道真正字符或者懒得输入完整名字时，常常使用通配符代替一个或多个真正字符。

终端里面输入的通配符是由 Shell 处理的，不是由所涉及的命令语句处理的，它只会出现在命令的“参数值”里（它不能出现在命令名称里， 命令不记得，那就用`Tab`补全）。当 Shell 在“参数值”中遇到了通配符时，Shell 会将其当作路径或文件名在磁盘上搜寻可能的匹配：若符合要求的匹配存在，则进行代换（路径扩展）；否则就将该通配符作为一个普通字符传递给“命令”，然后再由命令进行处理。总之，通配符实际上就是一种 Shell 实现的路径扩展功能。在通配符被处理后， Shell 会先完成该命令的重组，然后继续处理重组后的命令，直至执行该命令。

首先回到用户家目录：

```bash
cd /home/shiyanlou
```

然后使用 touch 命令创建 2 个文件，后缀都为 txt：

```bash
touch asd.txt fgh.txt
```

可以给文件随意命名，假如过了很长时间，你已经忘了这两个文件的文件名，现在你想在一大堆文件中找到这两个文件，就可以使用通配符：

```bash
ls *.txt
```

![1](https://doc.shiyanlou.com/document-uid735639labid2timestamp1531471952505.png)

在创建文件的时候，如果需要一次性创建多个文件，比如：**“love_1_linux.txt，love_2_linux.txt，... love_10_linux.txt”**。在 Linux 中十分方便：

```bash
touch love_{1..10}_shiyanlou.txt
```

![pic](https://doc.shiyanlou.com/document-uid735639labid2timestamp1531471982853.png)

Shell 常用通配符：

| 字符                    | 含义                                       |
| ----------------------- | ------------------------------------------ |
| `*`                     | 匹配 0 或多个字符                          |
| `?`                     | 匹配任意一个字符                           |
| `[list]`                | 匹配 list 中的任意单一字符                 |
| `[^list]`               | 匹配 除 list 中的任意单一字符以外的字符    |
| `[c1-c2]`               | 匹配 c1-c2 中的任意单一字符 如：[0-9][a-z] |
| `{string1,string2,...}` | 匹配 string1 或 string2 (或更多)其一字符串 |
| `{c1..c2}`              | 匹配 c1-c2 中全部字符 如{1..10}            |

#### 4) 在命令行中获取帮助

在 Linux 环境中，如果你遇到困难，可以使用`man`命令，它是`Manual pages`的缩写。

Manual pages 是 UNIX 或类 UNIX 操作系统中在线软件文档的一种普遍的形式， 内容包括计算机程序（包括库和系统调用）、正式的标准和惯例，甚至是抽象的概念。用户可以通过执行`man`命令调用手册页。

你可以使用如下方式来获得某个命令的说明和使用方式的详细介绍：

```bash
man <command_name>
```

比如你想查看 man 命令本身的使用方式，你可以输入：

```bash
man man
```

通常情况下，man 手册里面的内容都是英文的，这就要求你有一定的英文基础。man 手册的内容很多，涉及了 Linux 使用过程中的方方面面。为了便于查找，man 手册被进行了分册（分区段）处理，在 Research UNIX、BSD、OS X 和 Linux 中，手册通常被分为 8 个区段，安排如下：

| 区段 | 说明                                      |
| ---- | ----------------------------------------- |
| 1    | 一般命令                                  |
| 2    | 系统调用                                  |
| 3    | 库函数，涵盖了 C 标准函数库               |
| 4    | 特殊文件（通常是/dev 中的设备）和驱动程序 |
| 5    | 文件格式和约定                            |
| 6    | 游戏和屏保                                |
| 7    | 杂项                                      |
| 8    | 系统管理命令和守护进程                    |

要查看相应区段的内容，就在 man 后面加上相应区段的数字即可，如：

```bash
man 1 ls
```

会显示第一区段中的`ls`命令 man 页面。

所有的手册页遵循一个常见的布局，为了通过简单的 ASCII 文本展示而被优化，而这种情况下可能没有任何形式的高亮或字体控制。一般包括以下部分内容：

**NAME（名称）**

> 该命令或函数的名称，接着是一行简介。

**SYNOPSIS（概要）**

> 对于命令，正式的描述它如何运行，以及需要什么样的命令行参数。对于函数，介绍函数所需的参数，以及哪个头文件包含该函数的定义。

**DESCRIPTION（说明）**

> 命令或函数功能的文本描述。

**EXAMPLES（示例）**

> 常用的一些示例。

**SEE ALSO（参见）**

> 相关命令或函数的列表。

也可能存在其它部分内容，但这些部分没有得到跨手册页的标准化。常见的例子包括：OPTIONS（选项），EXIT STATUS（退出状态），ENVIRONMENT（环境），BUGS（程序漏洞），FILES（文件），AUTHOR（作者），REPORTING BUGS（已知漏洞），HISTORY（历史）和 COPYRIGHT（版权）。

通常 man 手册中的内容很多，你可能不太容易找到你想要的结果，不过幸运的是你可以在 man 中使用搜索`/<你要搜索的关键字>`，查找完毕后你可以使用`n`键切换到下一个关键字所在处，`shift+n`为上一个关键字所在处。使用`Space`（空格键）翻页，`Enter`（回车键）向下滚动一行，或者使用`k`，`j`（vim 编辑器的移动键）进行向前向后滚动一行。按下`h`键为显示使用帮助（因为 man 使用 less 作为阅读器，实为`less`工具的帮助），按下`q`退出。

想要获得更详细的帮助，你还可以使用`info`命令，不过通常使用`man`就足够了。如果你知道某个命令的作用，只是想快速查看一些它的某个具体参数的作用，那么你可以使用`--help`参数，大部分命令都会带有这个参数，如：

```bash
ls --help
```

## 二、用户及文件权限管理

Linux 是一个可以实现多用户登录的操作系统，比如“李雷”和“韩梅梅”都可以同时登录同一台主机，他们共享一些主机的资源，但他们也分别有自己的用户空间，用于存放各自的文件。但实际上他们的文件都是放在同一个物理磁盘上的甚至同一个逻辑分区或者目录里，但是由于 Linux 的 **用户管理** 和 **权限机制**，不同用户不可以轻易地查看、修改彼此的文件。

### 1. 查看用户

请打开终端，输入命令：

```bash
who am i

# 或者

who mom likes
```

![3-2.1-1](https://doc.shiyanlou.com/document-uid735639labid3timestamp1531731170296.png)

输出的第一列表示打开当前伪终端的用户的用户名（要查看当前登录用户的用户名，去掉空格直接使用 `whoami` 即可），第二列的 `pts/0` 中 `pts` 表示伪终端，所谓伪是相对于 `/dev/tty` 设备而言的，还记得上一节讲终端时的那七个使用 `[Ctrl]`+`[Alt]`+`[F1]～[F7]` 进行切换的 `/dev/tty` 设备么，这是“真终端”，伪终端就是当你在图形用户界面使用 `/dev/tty7` 时每打开一个终端就会产生一个伪终端，`pts/0` 后面那个数字就表示打开的伪终端序号，你可以尝试再打开一个终端，然后在里面输入 `who am i`，看第二列是不是就变成 `pts/1` 了，第三列则表示当前伪终端的启动时间。

还有一点需要注意的是，在某些环境中 `who am i` 和 `who mom likes` 命令不会输出任何内容，这是因为当前使用的 SHELL 不是登录时的 SHELL，没有用户与 who 的 stdin 相关联，因此不会输出任何内容。例如我在本地的 Ubuntu 系统上输入这个命令就不会有提示。

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583140204433)

此时我们只需要打开一个登录 SHELL 的终端例如 Tmux，或者通过 ssh 登录到本机，再在新的终端里执行命令即可。

```bash
tmux
```

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583140710447)

`who` 命令其它常用参数

| 参数 | 说明                       |
| ---- | -------------------------- |
| `-a` | 打印能打印的全部           |
| `-d` | 打印死掉的进程             |
| `-m` | 同`am i`，`mom likes`      |
| `-q` | 打印当前登录用户数及用户名 |
| `-u` | 打印当前登录用户登录信息   |
| `-r` | 打印运行等级               |

### 2. 创建及切换用户

在 Linux 系统里， `root` 账户拥有整个系统至高无上的权限，比如新建和添加用户。

> root 权限，系统权限的一种，与 SYSTEM 权限可以理解成一个概念，但高于 Administrator 权限，root 是 Linux 和 UNIX 系统中的超级管理员用户帐户，该帐户拥有整个系统至高无上的权力，所有对象他都可以操作，所以很多黑客在入侵系统的时候，都要把权限提升到 root 权限，这个操作等同于在 Windows 下就是将新建的非法帐户添加到 Administrators 用户组。更比如安卓操作系统中（基于 Linux 内核）获得 root 权限之后就意味着已经获得了手机的最高权限，这时候你可以对手机中的任何文件（包括系统文件）执行所有增、删、改、查的操作。

大部分 Linux 系统在安装时都会建议用户新建一个用户而不是直接使用 root 用户进行登录，当然也有直接使用 root 登录的例如 Kali（基于 Debian 的 Linux 发行版，集成大量工具软件，主要用于数字取证的操作系统）。一般我们登录系统时都是以普通账户的身份登录的，要创建用户需要 root 权限，这里就要用到 `sudo` 这个命令了。不过使用这个命令有两个大前提，一是你要知道当前登录用户的密码，二是当前用户必须在 `sudo` 用户组。shiyanlou 用户也属于 sudo 用户组（稍后会介绍如何查看和添加用户组）。

#### `su`，`su-` 与 `sudo`

**需要注意 Linux 环境下输入密码是不会显示的。**

`su <user>` 可以切换到用户 user，执行时需要输入目标用户的密码，`sudo <cmd>` 可以以特权级别运行 cmd 命令，需要当前用户属于 sudo 组，且需要输入当前用户的密码。`su - <user>` 命令也是切换用户，但是同时用户的环境变量和工作目录也会跟着改变成目标用户所对应的。

现在我们新建一个叫 `lilei` 的用户：

```bash
sudo adduser lilei
```

实验楼的环境目前设置为 shiyanlou 用户执行 sudo 不需要输入密码，通常此处需要按照提示输入 shiyanlou 密码（**Linux 下密码输入是不显示任何内容的，shiyanlou 用户密码可以在右侧环境信息里查看，请勿自行设置密码**）。然后是给 lilei 用户设置密码，后面的选项的一些内容你可以选择直接回车使用默认值。

![3-2.2-1](https://doc.shiyanlou.com/document-uid735639labid3timestamp1531731216215.png)

这个命令不但可以添加用户到系统，同时也会默认为新用户在 /home 目录下创建一个工作目录：

```bash
ls /home
```

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583141675216)

现在你已经创建好一个用户，并且你可以使用你创建的用户登录了，使用如下命令切换登录用户：

```bash
su -l lilei
```

输入刚刚设置的 lilei 的密码，然后输入如下命令并查看输出：

```bash
who am i
whoami
pwd
```

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583142076115)

你发现了区别了吗？这就是上一小节我们讲到的 `who am i` 和 `whoami` 命令的区别。

退出当前用户跟退出终端一样，可以使用 `exit` 命令或者使用快捷键 `Ctrl+D`。

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583142261959)

### 3. 用户组

在 Linux 里面每个用户都有一个归属（用户组），用户组简单地理解就是一组用户的集合，它们共享一些资源和权限，同时拥有私有资源，就跟家的形式差不多，你的兄弟姐妹（不同的用户）属于同一个家（用户组），你们可以共同拥有这个家（共享资源），爸妈对待你们都一样（共享权限），你偶尔写写日记，其他人未经允许不能查看（私有资源和权限）。当然一个用户是可以属于多个用户组的，正如你既属于家庭，又属于学校或公司。

在 Linux 里面如何知道自己属于哪些用户组呢？

#### 方法一：使用 `groups` 命令

```bash
groups shiyanlou
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid3timestamp1454035714557.png)

其中冒号之前表示用户，后面表示该用户所属的用户组。这里可以看到 shiyanlou 用户属于 shiyanlou 用户组，每次新建用户如果不指定用户组的话，默认会自动创建一个与用户名相同的用户组（差不多就相当于家长的意思）。

默认情况下在 sudo 用户组里的可以使用 sudo 命令获得 root 权限。shiyanlou 用户也可以使用 sudo 命令，为什么这里没有显示在 sudo 用户组里呢？可以查看下 `/etc/sudoers.d/shiyanlou` 文件，我们在 `/etc/sudoers.d` 目录下创建了这个文件，从而给 shiyanlou 用户赋予了 sudo 权限：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid3timestamp1454035855554.png)

#### 方法二：查看 `/etc/group` 文件

```bash
cat /etc/group | sort
```

这里 `cat` 命令用于读取指定文件的内容并打印到终端输出，后面会详细讲它的使用。 `| sort` 表示将读取的文本进行一个字典排序再输出，然后你将看到如下一堆输出，你可以在最下面看到 shiyanlou 的用户组信息：

![3-2.3-3](https://doc.shiyanlou.com/document-uid735639labid3timestamp1531731335264.png)

没找到？没关系，你可以使用 `grep` 命令过滤掉一些你不想看到的结果：

```bash
cat /etc/group | grep -E "shiyanlou"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid13labid3timestamp1454035698068.png)

**`/etc/group` 文件格式说明**

/etc/group 的内容包括用户组（Group）、用户组口令、GID（组 ID） 及该用户组所包含的用户（User），每个用户组一条记录。格式如下：

> group_name:password:GID:user_list

你看到上面的 password 字段为一个 `x`，并不是说密码就是它，只是表示密码不可见而已。

这里需要注意，如果用户的 GID 等于用户组的 GID，那么最后一个字段 `user_list` 就是空的，这里的 GID 是指用户默认所在组的 GID，可以使用 `id` 命令查看。比如 shiyanlou 用户，在 `/etc/group` 中的 shiyanlou 用户组后面是不会显示的。lilei 用户，在 `/etc/group` 中的 lilei 用户组后面是不会显示的。

#### 将其它用户加入 sudo 用户组`usermod` 命令

默认情况下新创建的用户是不具有 root 权限的，也不在 sudo 用户组，可以让其加入 sudo 用户组从而获取 root 权限：

```bash
# 注意 Linux 上输入密码是不会显示的
su -l lilei
sudo ls
```

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583145040679)

会提示 lilei 不在 sudoers 文件中，意思就是 lilei 不在 sudo 用户组中，至于 sudoers 文件（/etc/sudoers）你现在最好不要动它，操作不慎会导致比较麻烦的后果。

使用 `usermod` 命令可以为用户添加用户组，同样使用该命令你必需有 root 权限，你可以直接使用 root 用户为其它用户添加用户组，或者用其它已经在 sudo 用户组的用户使用 sudo 命令获取权限来执行该命令。

这里我用 shiyanlou 用户执行 sudo 命令将 lilei 添加到 sudo 用户组，让它也可以使用 sudo 命令获得 root 权限，首先我们切换回 shiyanlou 用户。

```bash
su - shiyanlou
```

此处需要输入 shiyanlou 用户密码，shiyanlou 的密码可以在右侧工具栏的环境信息里看到。

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200616-1592274816723)

当然也可以通过 `sudo passwd shiyanlou` 进行设置，或者你直接关闭当前终端打开一个新的终端。

```bash
groups lilei
sudo usermod -G sudo lilei
groups lilei
```

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583145514469)

然后你再切换回 lilei 用户，现在就可以使用 sudo 获取 root 权限了。

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583145591124)

### 4. 删除用户和用户组

#### 1) `deluser`命令

删除用户是很简单的事：

```bash
sudo deluser lilei --remove-home
```

![3-2.4-1](https://doc.shiyanlou.com/document-uid735639labid3timestamp1531731417990.png)

使用 `--remove-home` 参数在删除用户时候会一并将该用户的工作目录一并删除。如果不使用那么系统会自动在 /home 目录为该用户保留工作目录。

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200302-1583146790310)

#### 2）`groupdel` 命令

删除用户组可以使用 `groupdel` 命令，倘若该群组中仍包括某些用户，则必须先删除这些用户后，才能删除群组。

## 三、linux文件权限

### 1. 查看文件权限

#### 1) `ll`或`ls -l`命令

`ll`或`ls -l`查看当前或指定目录下的非隐藏文件的所有属性，包括**文件权限**、**修改时间**、**文件拥有及创建用户**、**大小**、**文件名**等信息

另外，`ls -a`的`-a`参数可以显示**隐藏文件**

#### 2）权限字母表示

![img](https://doc.shiyanlou.com/linux_base/3-9.png)

![img](https://doc.shiyanlou.com/linux_base/3-10.png)

### 2. 变更文件所有者

`chown` 将指定文件的拥有者改为指定的用户或组，用户可以是用户名或者用户 ID；组可以是组名或者组 ID；文件是以空格分开的要改变权限的文件列表，支持通配符。

```
-c 显示更改的部分的信息
-R 处理指定目录及子目录
```

#### `chown`命令

（1）改变拥有者和群组 并显示改变信息

```
chown -c mail:mail log2012.log
```

（2）改变文件群组

```
chown -c :mail t.log
```

（3）改变文件夹及子文件目录属主及属组为 mail

```
chown -cR mail: test/
```

### 3. 修改文件权限

####  `chmod`命令

一种是包含字母和操作符表达式的文字设定法；另一种是包含数字的数字设定法。

每一文件或目录的访问权限都有三组，每组用三位表示，分别为文件属主的读、写和执行权限；与属主同组的用户的读、写和执行权限；系统中其他用户的读、写和执行权限。可使用 ls -l test.txt 查找。

以文件 log2012.log 为例：

```
-rw-r--r-- 1 root root 296K 11-13 06:03 log2012.log
```

第一列共有 10 个位置，第一个字符指定了文件类型。在通常意义上，一个目录也是一个文件。如果第一个字符是横线，表示是一个非目录的文件。如果是 d，表示是一个目录。从第二个字符开始到第十个 9 个字符，3 个字符一组，分别表示了 3 组用户对文件或者目录的权限。权限字符用横线代表空许可，r 代表只读，w 代表写，x 代表可执行。

**常用参数**：

```
-c 当发生改变时，报告处理信息
-R 处理指定目录以及其子目录下所有文件
```

**权限范围**：

```
u ：目录或者文件的当前的用户
g ：目录或者文件的当前的群组
o ：除了目录或者文件的当前用户或群组之外的用户或者群组
a ：所有的用户及群组
```

**权限代号**：

```
r ：读权限，用数字4表示
w ：写权限，用数字2表示
x ：执行权限，用数字1表示
- ：删除权限，用数字0表示
s ：特殊权限
```

（1）增加文件 t.log 所有用户可执行权限

```
chmod a+x t.log
```

（2）减少文件 t.log 所有用户读写权限

```
chmod a-rw t.log
```

（3）撤销原来所有的权限，然后使拥有者具有可读权限,并输出处理信息

```
chmod u=r t.log -c
```

（4）给 file 的属主分配读、写、执行(7)的权限，给file的所在组分配读、执行(5)的权限，给其他用户分配执行(1)的权限

```
chmod 751 t.log -c（或者：chmod u=rwx,g=rx,o=x t.log -c)
```

（5）将 test 目录及其子目录所有文件添加可读权限

```
chmod u+r,g+r,o+r -R text/ -c
```

### 4. 更多

#### `adduser` 和 `useradd` 

答：`useradd` 只创建用户，不会创建用户密码和工作目录，创建完了需要使用 `passwd <username>` 去设置新用户的密码。`adduser` 在创建用户的同时，会创建工作目录和密码（提示你设置），做这一系列的操作。其实 `useradd`、`userdel` 这类操作更像是一种命令，执行完了就返回。而 `adduser` 更像是一种程序，需要你输入、确定等一系列操作。

## 四、Linux 目录结构及文件基本操作

### 1. linux目录结构（FHS准则）

![img](https://doc.shiyanlou.com/linux_base/4-1.png)

### 2. `pwd`命令

打印当前绝对路径

### 3. `cd`命令

用于切换目录，不能切换至某个具体文件下。在 Linux 里面使用 `.` 表示当前目录，`..` 表示上一级目录（**注意，我们上一节介绍过的，以 `.` 开头的文件都是隐藏文件，所以这两个目录必然也是隐藏的，你可以使用 `ls -a` 命令查看隐藏文件**），`-` 表示上一次所在目录，`～` 通常表示当前用户的 `home` 目录。在 Linux 里面使用 `.` 表示当前目录，`..` 表示上一级目录（**注意，我们上一节介绍过的，以 `.` 开头的文件都是隐藏文件，所以这两个目录必然也是隐藏的，你可以使用 `ls -a` 命令查看隐藏文件**），`-` 表示上一次所在目录，`～` 通常表示当前用户的 `home` 目录。

#### 1）绝对路径

关于绝对路径，简单地说就是以根" / "目录为起点的完整路径，以你所要到的目录为终点，表现形式如： `/usr/local/bin`，表示根目录下的 `usr` 目录中的 `local` 目录中的 `bin` 目录。

#### 2）相对路径

相对路径，也就是相对于你当前的目录的路径，相对路径是以当前目录 `.` 为起点，以你所要到的目录为终点，表现形式如： `usr/local/bin` （这里假设你当前目录为根目录）。你可能注意到，我们表示相对路径实际并没有加上表示当前目录的那个 `.` ，而是直接以目录名开头，因为这个 `usr` 目录为 `/` 目录下的子目录，是可以省略这个 `.` 的（以后会讲到一个类似不能省略的情况）；如果是当前目录的上一级目录，则需要使用 `..` ，比如你当前目录为 `/home/shiyanlou` 目录下，根目录就应该表示为 `../../` ，表示上一级目录（ `home` 目录）的上一级目录（ `/` 目录）。

### 4. 新建

#### 1）新建空白文件`touch`命令

```bash
touch file{1..10}.txt#创建十个文件
#用法：touch 文件名
#没有的文件会创建一个新文件，若当前目录存在同名文件，则 touch 命令，则会更改该文件夹的时间戳而不是新建文件。
```

#### 2）新建目录与多级目录mkdir

可用选项：

- -m: 对新建目录设置存取权限，也可以用 chmod 命令设置;
- -p: 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后，系统将自动建立好那些尚不在的目录，即一次可以建立多个目录。

（1）当前工作目录下创建名为 t的文件夹

```
mkdir t
```

（2）在 tmp 目录下创建路径为 test/t1/t 的目录，若不存在，则创建：

```
mkdir -p /tmp/test/t1/t
```

### 5. 复制

#### 1）复制文件`cp`命令

注意：命令行复制，如果目标文件已经存在会提示是否覆盖，而在 shell 脚本中，如果不加 -i 参数，则不会提示，而是直接覆盖！

```
-i 提示
-r 复制目录及目录内所有项目
-a 复制的文件与原文件时间一样
```

（1）复制 a.txt 到 test 目录下，保持原文件时间，如果原文件存在提示是否覆盖。

```
cp -ai a.txt test下
```

（2）为 a.txt 建议一个链接（快捷方式）

```
cp -s a.txt link_a.txt
```

#### 2）复制目录`cp`命令

添加参数`-r`即可

### 6. 删除

#### 1）删除文件`rm`命令

使用 `rm`（remove files or directories）命令删除一个文件：

```bash
rm test
```

有时候你会遇到想要删除一些为只读权限的文件，直接使用 `rm` 删除会显示一个提示，如下：

![4-3.3-1](https://doc.shiyanlou.com/document-uid735639labid59timestamp1531733991692.png)

你如果想忽略这提示，直接删除文件，可以使用 `-f` 参数强制删除：

```bash
rm -f test
```

#### 2）删除目录`rm`命令

跟复制目录一样，要删除一个目录，也需要加上 `-r` 或 `-R` 参数：

```bash
rm -r family
```

遇到权限不足删除不了的目录也可以和删除文件一样加上 `-f` 参数：

```bash
rm -rf family
```

### 7. 移动文件与文件重命名

移动文件或修改文件名，根据第二参数类型（如目录，则移动文件；如为文件则重命令该文件）。

当第二个参数为目录时，第一个参数可以是多个以空格分隔的文件或目录，然后移动第一个参数指定的多个文件到第二个参数指定的目录中。

（1）将文件 test.log 重命名为 test1.txt

```
mv test.log test1.txt
```

（2）将文件 log1.txt,log2.txt,log3.txt 移动到根的 test3 目录中

```
mv llog1.txt log2.txt log3.txt /test3
```

（3）将文件 file1 改名为 file2，如果 file2 已经存在，则询问是否覆盖

```
mv -i log1.txt log2.txt
```

（4）移动当前文件夹下的所有文件到上一级目录

```
mv * ../
```

### 8. 查看

#### 1) `cat`，`tac` 和 `nl` 命令查看文件内容

前两个命令都是用来打印文件内容到标准输出（终端），其中 `cat` 为正序显示，`tac` 为倒序显示。

> 标准输入输出：当我们执行一个 shell 命令行时通常会自动打开三个标准文件，即标准输入文件（stdin），默认对应终端的键盘、标准输出文件（stdout）和标准错误输出文件（stderr），后两个文件都对应被重定向到终端的屏幕，以便我们能直接看到输出内容。进程将从标准输入文件中得到输入数据，将正常输出数据输出到标准输出文件，而将错误信息送到标准错误文件中。

比如我们要查看之前从 `/etc` 目录下拷贝来的 `passwd` 文件：

```bash
cd /home/shiyanlou
cp /etc/passwd passwd
cat passwd
```

可以加上 `-n` 参数显示行号：

```bash
cat -n passwd
```

![4-3.4-1](https://doc.shiyanlou.com/document-uid735639labid59timestamp1531734168883.png)

`nl` 命令，添加行号并打印，这是个比 `cat -n` 更专业的行号打印命令。

这里简单列举它的常用的几个参数：

```bash
-b : 指定添加行号的方式，主要有两种：
    -b a:表示无论是否为空行，同样列出行号("cat -n"就是这种方式)
    -b t:只列出非空行的编号并列出（默认为这种方式）
-n : 设置行号的样式，主要有三种：
    -n ln:在行号字段最左端显示
    -n rn:在行号字段最右边显示，且不加 0
    -n rz:在行号字段最右边显示，且加 0
-w : 行号字段占用的位数(默认为 6 位)
```

![4-3.5-2](https://doc.shiyanlou.com/document-uid735639labid59timestamp1531734186668.png)

你会发现使用这几个命令，默认的终端窗口大小，一屏显示不完文本的内容，得用鼠标拖动滚动条或者滑动滚轮才能继续往下翻页，要是可以直接使用键盘操作翻页就好了，那么你就可以使用下面要介绍的命令。

#### 2) `more` 和 `less` 命令分页查看文件内容

如果说上面的 `cat` 是用来快速查看一个文件的内容的，那么这个 `more` 和 `less` 就是天生用来"阅读"一个文件的内容的，比如说 man 手册内部就是使用的 `less` 来显示内容。其中 `more` 命令比较简单，只能向一个方向滚动，而 `less` 为基于 `more` 和 `vi` （一个强大的编辑器，我们有单独的课程来让你学习）开发，功能更强大。`less` 的使用基本和 `more` 一致，具体使用请查看 man 手册，这里只介绍 `more` 命令的使用。

使用 `more` 命令打开 `passwd` 文件：

```bash
more passwd
```

![4-3.5-3](https://doc.shiyanlou.com/document-uid735639labid59timestamp1531734202525.png)

打开后默认只显示一屏内容，终端底部显示当前阅读的进度。可以使用 `Enter` 键向下滚动一行，使用 `Space` 键向下滚动一屏，按下 `h` 显示帮助，`q` 退出。

#### 3)  `head` 和 `tail` 命令查看文件内容

这两个命令，那些性子比较急的人应该会喜欢，因为它们一个是只查看文件的头几行（默认为 10 行，不足 10 行则显示全部）和尾几行。还是拿 passwd 文件举例，比如当我们想要查看最近新增加的用户，那么我们可以查看这个 `/etc/passwd` 文件，不过我们前面也看到了，这个文件里面一大堆乱糟糟的东西，看起来实在费神啊。因为系统新增加一个用户，会将用户的信息添加到 passwd 文件的最后，那么这时候我们就可以使用 `tail` 命令了：

```bash
tail /etc/passwd
```

甚至更直接的只看一行， 加上 `-n` 参数，后面紧跟行数：

```bash
tail -n 1 /etc/passwd
```

![4-3.5-4](https://doc.shiyanlou.com/document-uid735639labid59timestamp1531734226305.png)

关于 `tail` 命令，不得不提的还有它一个很牛的参数 `-f`，这个参数可以实现不停地读取某个文件的内容并显示。这可以让我们动态查看日志，达到实时监视的目的。

用于显示指定文件末尾内容，不指定文件时，作为输入信息进行处理。常用查看日志文件。

常用参数：

```
-f 循环读取（常用于查看递增的日志文件）
-n<行数> 显示行数（从后向前）
```

（1）循环读取逐渐增加的文件内容

```
ping 127.0.0.1 > ping.log &
```

后台运行：可使用 jobs -l 查看，也可使用 fg 将其移到前台运行。

```
tail -f ping.log
```

（查看日志）

#### 4) `file`命令查看文件类型

```bash
file /bin/ls
```

![4-3.6-1](https://doc.shiyanlou.com/document-uid735639labid59timestamp1531734243413.png)

说明这是一个可执行文件，运行在 64 位平台，并使用了动态链接文件（共享库）。

与 Windows 不同的是，如果你新建了一个 shiyanlou.txt 文件，Windows 会自动把它识别为文本文件，而 `file` 命令会识别为一个空文件。这个前面我提到过，在 Linux 中文件的类型不是根据文件后缀来判断的。当你在文件里输入内容后才会显示文件类型。

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200303-1583215158345)

### 9. 编辑文件

在 Linux 下面编辑文件通常我们会直接使用专门的命令行编辑器比如（emacs，vim，nano），由于涉及 Linux 上的编辑器的内容比较多，且非常重要。

## 五、环境变量与文件查找

### 1. 环境变量

 `/etc/bashrc`（有的 Linux 没有这个文件） 和 `/etc/profile` ，它们分别存放的是 shell 变量和环境变量。还有要注意区别的是每个用户目录下的一个隐藏文件：

```bash
# .profile 可以用 ls -a 查看
cd /home/shiyanlou
ls -a
```

![图片描述](https://doc.shiyanlou.com/courses/uid871732-20200303-1583220161661)

这个 .profile 只对当前用户永久生效。因为它保存在当前用户的 Home 目录下，当切换用户时，工作目录可能一并被切换到对应的目录中，这个文件就无法生效。而写在 `/etc/profile` 里面的是对所有用户永久生效，所以如果想要添加一个全局、永久生效的环境变量（对所有用户都起作用的环境变量），只需要打开 `/etc/profile`，添加环境变量即可。

#### 1）临时生效

使用 export 命令行声明即可，变量在关闭当前 shell 时失效，生效范围是当前的shell，其他shell窗口不生效。

#### 2）永久生效

需要修改配置文件，变量永久生效，生效范围是全局生效。前面我们在 Shell 中修改了一个配置脚本文件之后（比如 zsh 的配置文件 home 目录下的 `.zshrc`），每次都要退出终端重新打开甚至重启主机之后其才能生效，很是麻烦，我们可以使用 `source` 命令来让其立即生效，如：

```bash
cd /home/shiyanlou
source .zshrc
```

### 2. linux上的可执行文件

windows上的可执行文件以`.exe`结尾，linux上则是以文件权限的方式展现是否可以执行。使用`touch`创建的空白文件夹默认权限是`-rw-rw-r--`不论是用户还是用户组还是其他用户及用户组都不具备可执行权，先编辑指定内容，再编译响应程序获得，再使用`chmod`修改编译文件的文件权限为可执行，然后直接调用可执行文件的文件名就可以运行该程序。此过程相当于windows上的编码、编译、生成可执行文件、点击运行即可

你可能很早之前就有疑问，我们在 Shell 中输入一个命令，Shell 是怎么知道去哪找到这个命令然后执行的呢？这是通过环境变量 `PATH` 来进行搜索的，熟悉 Windows 的用户可能知道 Windows 中的也是有这么一个 PATH 环境变量。这个 `PATH` 里面就保存了 Shell 中执行的命令的搜索路径。

查看 `PATH` 环境变量的内容：

```bash
echo $PATH
```

默认情况下你会看到如下输出：

```bash
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

如果你还记得 Linux 目录结构那一节的内容，你就应该知道上面这些目录下放的是哪一类文件了。通常这一类目录下放的都是可执行文件，当我们在 Shell 中执行一个命令时，系统就会按照 PATH 中设定的路径按照顺序依次到目录中去查找，如果存在同名的命令，则执行先找到的那个。

创建一个 Shell 脚本文件，你可以使用 gedit，vim，sublime 等工具编辑。如果你是直接复制的话，建议使用 gedit 或者 sublime，否则可能导致代码缩进混乱。

```bash
cd /home/shiyanlou
touch hello_shell.sh
gedit hello_shell.sh
```

在脚本中添加如下内容，保存并退出。

**注意不要省掉第一行，这不是注释，有用户反映有语法错误，就是因为没有了第一行。**

```bash
#!/bin/bash

for ((i=0; i<10; i++));do
    echo "hello shell"
done

exit 0
```

为文件添加可执行权限，否则执行会报错没有权限：

```bash
chmod 755 hello_shell.sh
```

执行脚本：

```bash
cd /home/shiyanlou
./hello_shell.sh
```

创建一个 C 语言 `hello world` 程序：

```bash
cd /home/shiyanlou
gedit hello_world.c
```

输入如下内容，同样不能省略第一行。

```c
#include <stdio.h>

int main(void)
{
    printf("hello world!\n");
    return 0;
}
```

保存后使用 gcc 生成可执行文件：

```bash
gcc -o hello_world hello_world.c
```

**gcc 生成二进制文件默认具有可执行权限，不需要修改。**

在 `/home/shiyanlou` 家目录创建一个 `mybin` 目录，并将上述 `hello_shell.sh` 和 `hello_world` 文件移动到其中：

```bash
cd /home/shiyanlou
mkdir mybin
mv hello_shell.sh hello_world mybin/
```

现在你可以在 `mybin` 目录中分别运行你刚刚创建的两个程序：

```bash
cd mybin
./hello_shell.sh
./hello_world
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid60timestamp1532339433567.png)

回到上一级目录，也就是 `shiyanlou` 家目录，当再想运行那两个程序时，会发现提示命令找不到，除非加上命令的完整路径，但那样很不方便，如何做到像使用系统命令一样执行自己创建的脚本文件或者程序呢？那就要将命令所在路径添加到 `PATH` 环境变量了。

### 3. 搜索文件

#### 1）`find`命令足矣

用于在文件树中查找文件，并作出相应的处理。

命令格式：

```
find pathname -options [-print -exec -ok ...]
```

命令参数：

```
pathname: find命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
-print：find命令将匹配的文件输出到标准输出。
-exec：find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格。
-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。
```

命令选项：

```
-name 按照文件名查找文件
-perm 按文件权限查找文件
-user 按文件属主查找文件
-group  按照文件所属的组来查找文件。
-type  查找某一类型的文件，诸如：
   b - 块设备文件
   d - 目录
   c - 字符设备文件
   l - 符号链接文件
   p - 管道文件
   f - 普通文件
```

（1）查找 48 小时内修改过的文件

```
find -atime -2
```

（2）在当前目录查找 以 .log 结尾的文件。. 代表当前目录

```
find ./ -name '*.log'
```

（3）查找 /opt 目录下 权限为 777 的文件

```
find /opt -perm 777
```

（4）查找大于 1K 的文件

```
find -size +1000c	
```

(5)查找等于 1000 字符的文件

```
find -size 1000c 
```

`-exec` 参数后面跟的是 `command` 命令，它的终止是以 ; 为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。{} 花括号代表前面find查找出来的文件名。

#### 2）忽略`which`、`whereis`以及`locate`

## 六、文件打包与解压缩

### 1. 概念储备

在 Windows 上最常见的不外乎这两种 `*.zip`，`*.7z` 后缀的压缩文件。而在 Linux 上面常见的格式除了以上两种外，还有 `.rar`，`*.gz`，`*.xz`，`*.bz2`，`*.tar`，`*.tar.gz`，`*.tar.xz`，`*.tar.bz2`，简单介绍如下：

| 文件后缀名 | 说明                           |
| ---------- | ------------------------------ |
| `*.zip`    | zip 程序打包压缩的文件         |
| `*.rar`    | rar 程序压缩的文件             |
| `*.7z`     | 7zip 程序压缩的文件            |
| `*.tar`    | tar 程序打包，未压缩的文件     |
| `*.gz`     | gzip 程序（GNU zip）压缩的文件 |
| `*.xz`     | xz 程序压缩的文件              |
| `*.bz2`    | bzip2 程序压缩的文件           |
| `*.tar.gz` | tar 打包，gzip 程序压缩的文件  |
| `*.tar.xz` | tar 打包，xz 程序压缩的文件    |
| `*tar.bz2` | tar 打包，bzip2 程序压缩的文件 |
| `*.tar.7z` | tar 打包，7z 程序压缩的文件    |

这么多个命令，不过我们一般只需要掌握几个命令即可，包括 `zip`，`tar`。下面会依次介绍这几个命令及对应的解压命令。

### 2. `zip` 压缩打包程序

- 使用 zip 打包文件夹，注意输入完整的参数和路径：

```bash
cd /home/shiyanlou
zip -r -q -o shiyanlou.zip /home/shiyanlou/Desktop
du -h shiyanlou.zip
file shiyanlou.zip
```

上面命令将目录 `/home/shiyanlou/Desktop` 打包成一个文件，并查看了打包后文件的大小和类型。第一行命令中，`-r` 参数表示递归打包包含子目录的全部内容，`-q` 参数表示为安静模式，即不向屏幕输出信息，`-o`，表示输出文件，需在其后紧跟打包输出文件名。后面使用 `du` 命令查看打包后文件的大小（后面会具体说明该命令）。

- 设置压缩级别为 9 和 1（9 最大，1 最小），重新打包：

```bash
zip -r -9 -q -o shiyanlou_9.zip /home/shiyanlou/Desktop -x ~/*.zip
zip -r -1 -q -o shiyanlou_1.zip /home/shiyanlou/Desktop -x ~/*.zip
```

这里添加了一个参数用于设置压缩级别 `-[1-9]`，1 表示最快压缩但体积大，9 表示体积最小但耗时最久。最后那个 `-x` 是为了排除我们上一次创建的 zip 文件，否则又会被打包进这一次的压缩文件中，**注意：这里只能使用绝对路径，否则不起作用**。

我们再用 `du` 命令分别查看默认压缩级别、最低、最高压缩级别及未压缩的文件的大小：

```bash
du -h -d 0 *.zip ~ | sort
```

通过 man 手册可知：

- `-h`， --human-readable（顾名思义，你可以试试不加的情况）
- `-d`， --max-depth（所查看文件的深度）

![图片描述](https://doc.shiyanlou.com/courses/uid600404-20190428-1556438181236)

这样一目了然，理论上来说默认压缩级别应该是最高的，但是由于文件不大，这里的差异不明显（几乎看不出差别），不过你在环境中操作之后看到的压缩文件大小可能跟图上的有些不同，因为系统在使用过程中，会随时生成一些缓存文件在当前用户的家目录中，这对于我们学习命令使用来说，是无关紧要的，可以忽略这些不同。

- 创建加密 zip 包

使用 `-e` 参数可以创建加密压缩包：

```bash
zip -r -e -o shiyanlou_encryption.zip /home/shiyanlou/Desktop
```

**注意：** 关于 `zip` 命令，因为 Windows 系统与 Linux/Unix 在文本文件格式上的一些兼容问题，比如换行符（为不可见字符），在 Windows 为 CR+LF（Carriage-Return+Line-Feed：回车加换行），而在 Linux/Unix 上为 LF（换行），所以如果在不加处理的情况下，在 Linux 上编辑的文本，在 Windows 系统上打开可能看起来是没有换行的。如果你想让你在 Linux 创建的 zip 压缩文件在 Windows 上解压后没有任何问题，那么你还需要对命令做一些修改：

```bash
zip -r -l -o shiyanlou.zip /home/shiyanlou/Desktop
```

需要加上 `-l` 参数将 `LF` 转换为 `CR+LF` 来达到以上目的。

### 3. 使用 `unzip` 命令解压缩 `zip` 文件

将 `shiyanlou.zip` 解压到当前目录：

```bash
unzip shiyanlou.zip
```

使用安静模式，将文件解压到指定目录：

```bash
unzip -q shiyanlou.zip -d ziptest
```

上述指定目录不存在，将会自动创建。如果你不想解压只想查看压缩包的内容你可以使用 `-l` 参数：

```bash
unzip -l shiyanlou.zip
```

**注意：** 使用 unzip 解压文件时我们同样应该注意兼容问题，不过这里我们关心的不再是上面的问题，而是中文编码的问题，通常 Windows 系统上面创建的压缩文件，如果有有包含中文的文档或以中文作为文件名的文件时默认会采用 GBK 或其它编码，而 Linux 上面默认使用的是 UTF-8 编码，如果不加任何处理，直接解压的话可能会出现中文乱码的问题（有时候它会自动帮你处理），为了解决这个问题，我们可以在解压时指定编码类型。

使用 `-O`（英文字母，大写 o）参数指定编码类型：

```bash
unzip -O GBK 中文压缩文件.zip
```

### 4. `tar` 打包、其压缩与解压工具

tar可以实现`*.tar.gz`、`*.tar.xz`和`*tar.bz2`的压缩（xz、gzip 及 bzip2）,zip还是用zip压缩工具

用来压缩和解压文件。tar 本身不具有压缩功能，只具有打包功能，有关压缩及解压是调用其它的功能来完成。

弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件

**注意：**

- **`-f`参数后面需要紧跟文件名，否则无效**
- **使用对应压缩方式打包时，应当加上对应方式压缩解包或者查看**

常用参数：

```markdown
-c 建立新的压缩文件
-f 指定压缩文件名,该参数后面需要紧跟文件名，否则不会生效
-r 添加文件到已经压缩文件包中
-u 添加改了和现有的文件到压缩包中
-x 从压缩包中抽取文件，及解压
-t 显示压缩文件中的内容
-z 支持gzip压缩
-j 支持bzip2压缩
-Z 支持compress解压文件
-v 显示操作过程
```

| 压缩文件格式 | 参数 |
| ------------ | ---- |
| `*.tar.gz`   | `-z` |
| `*.tar.xz`   | `-J` |
| `*tar.bz2`   | `-j` |

有关 gzip 及 bzip2 压缩:

```
gzip 实例：压缩 gzip fileName .tar.gz 和.tgz  解压：gunzip filename.gz 或 gzip -d filename.gz
          对应：tar zcvf filename.tar.gz     tar zxvf filename.tar.gz

bz2实例：压缩 bzip2 -z filename .tar.bz2 解压：bunzip filename.bz2或bzip -d filename.bz2
       对应：tar jcvf filename.tar.gz         解压：tar jxvf filename.tar.bz2
```

（1）将文件全部打包成 tar 包（未压缩）

```
tar -cvf log.tar 1.log,2.log 或tar -cvf log.*
```

（2）将 /etc 下的所有文件及目录打包到指定目录，并使用 gz 压缩

```
tar -zcvf /tmp/etc.tar.gz /etc
```

（3）查看刚打包的文件内容（一定加z，因为是使用 gzip 压缩的）

```
tar -ztvf /tmp/etc.tar.gz
```

（4）要压缩打包 /home, /etc ，但不要 /home/dmtsai

```
tar --exclude /home/dmtsai -zcvf myfile.tar.gz /home/* /etc
```

## 七、文件系统操作与磁盘管理

### 1.  `df`命令

显示磁盘空间使用情况。获取硬盘被占用了多少空间，目前还剩下多少空间等信息，如果没有文件名被指定，则所有当前被挂载的文件系统的可用空间将被显示。

默认情况下，磁盘空间将以 1KB 为单位进行显示，除非环境变量 POSIXLY_CORRECT 被指定，那样将以512字节为单位进行显示：

```
-a 全部文件系统列表
-h 以方便阅读的方式显示信息
-i 显示inode信息
-k 区块为1024字节
-l 只显示本地磁盘
-T 列出文件系统类型
```

（1）显示磁盘使用情况

```
df -l
```

（2）以易读方式列出所有文件系统及其类型

```
df -haT
```

### 2. `du` 命令

du 命令也是查看使用空间的，但是与 df 命令不同的是 Linux du 命令是对文件和目录磁盘使用的空间的查看：

命令格式：

```
du [选项] [文件]
```

常用参数：

```
-a 显示目录中所有文件大小
-k 以KB为单位显示文件大小
-m 以MB为单位显示文件大小
-g 以GB为单位显示文件大小
-h 以易读方式显示文件大小
-s 仅显示总计
-c或--total  除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
```

（1）以易读方式显示文件夹内及子文件夹大小

```
du -h scf/
```

（2）以易读方式显示文件夹内所有文件大小

```
du -ah scf/
```

（3）显示几个文件或目录各自占用磁盘空间的大小，还统计它们的总和

```
du -hc test/ scf/
```

（4）输出当前目录下各个子目录所使用的空间

```
du -hc --max-depth=1 scf/
```

## 八、linux下的帮助命令

### 1. 内建命令与外部命令

一些查看帮助的工具在内建命令与外建命令上是有区别对待的。

> **内建命令**实际上是 shell 程序的一部分，其中包含的是一些比较简单的 Linux 系统命令，这些命令是写在 bash 源码的 builtins 里面的，由 shell 程序识别并在 shell 程序内部完成运行，通常在 Linux 系统加载运行时 shell 就被加载并驻留在系统内存中。而且解析内部命令 shell 不需要创建子进程，因此其执行速度比外部命令快。比如：history、cd、exit 等等。

> **外部命令**是 Linux 系统中的实用程序部分，因为实用程序的功能通常都比较强大，所以其包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调入内存。虽然其不包含在 shell 中，但是其命令执行过程是由 shell 程序控制的。外部命令是在 Bash 之外额外安装的，通常放在/bin，/usr/bin，/sbin，/usr/sbin 等等。比如：ls、vi 等。

简单来说就是：一个是天生自带的天赋技能，一个是后天得来的附加技能。我们可以使用　 type 命令来区分命令是内建的还是外部的。例如这两个得出的结果是不同的

```bash
type exit

type vim
```

得到的是两种结果，若是对 ls 你还能得到第三种结果

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6123timestamp1523930371175.png)

```bash
# 得到这样的结果说明是内建命令，正如上文所说内建命令都是在 bash 源码中的 builtins 的.def中
xxx is a shell builtin
# 得到这样的结果说明是外部命令，正如上文所说，外部命令在/usr/bin or /usr/sbin等等中
xxx is /usr/bin/xxx
# 若是得到alias的结果，说明该指令为命令别名所设定的名称；
xxx is an alias for xx --xxx
```

### 2. 帮助命令的使用

#### 1）`help`命令

只用于查看内建命令的使用方法，无法运用于外部命令的查看，外部命令使用参数往往使用`--help`查看帮助

#### 2）`man`命令

得到的内容比用`help`更多更详细，而且`man`没有内建与外部命令的区分，因为`man`工具是显示系统手册页中的内容

**`man`手册章节含义：**

| 章节数 | 说明                                                |
| ------ | --------------------------------------------------- |
| `1`    | Standard commands （标准命令）                      |
| `2`    | System calls （系统调用）                           |
| `3`    | Library functions （库函数）                        |
| `4`    | Special devices （设备说明）                        |
| `5`    | File formats （文件格式）                           |
| `6`    | Games and toys （游戏和娱乐）                       |
| `7`    | Miscellaneous （杂项）                              |
| `8`    | Administrative Commands （管理员命令）              |
| `9`    | 其他（Linux 特定的）， 用来存放内核例行程序的文档。 |

#### 3）`info`命令

info 来自自由软件基金会的 GNU 项目，是 GNU 的超文本帮助系统，能够更完整的显示出 GNU 信息。所以得到的信息当然更多

如果没有安装`info`，可以执行一下程序安装

```bash
# 安装 info
sudo apt-get update
sudo apt-get install info
# 查看 ls 命令的 info
info ls
```

## 九、Linux 任务计划 crontab

### 1. `crontab`的简介

crontab 命令常见于 Unix 和类 Unix 的操作系统之中（Linux 就属于类 Unix 操作系统），用于设置周期性被执行的指令。

crontab 命令从输入设备读取指令，并将其存放于 crontab 文件中，以供之后读取和执行。通常，crontab 储存的指令被守护进程激活，crond 为其守护进程，crond 常常在后台运行，每一分钟会检查一次是否有预定的作业需要执行。

通过 crontab 命令，我们可以在固定的间隔时间执行指定的系统指令或 shell 脚本。时间间隔的单位可以是分钟、小时、日、月、周的任意组合。

这里我们看一看 crontab 的格式：

```bash
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```

### 2. `crontab`的准备

##### 日志监控

crontab 在本实验环境中需要做一些特殊的准备，首先我们会启动 `rsyslog`，以便我们可以通过日志中的信息来了解我们的任务是否真正的被执行了（在本实验环境中需要手动启动，而在自己本地中 Ubuntu 会默认自行启动不需要手动启动）。

```bash
sudo apt-get install -y rsyslog
sudo service rsyslog start
```

![service-rsyslog-start](https://dn-simplecloud.shiyanlou.com/1135081468201394787)

##### 启动`crontab`

在本实验环境中 crontab 也是不被默认启动的，同时不能在后台由 upstart 来管理，所以需要我们来启动它:

```bash
sudo cron －f &
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6124timestamp1523941816405.png)

### 3. `crontab`的使用

#### 1）`crontab -e`命令

我们通过下面一个命令来添加一个计划任务：

```bash
crontab -e
```

第一次启动会出现这样一个画面，这是让我们选择编辑的工具，选择第二个基本的 vim 就可以了。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6124timestamp1523941985569.png)

##### 编写进程指定时间以及命令

crontab文件的含义：

用户所建立的crontab文件中，每一行都代表一项任务，每行的每个字段代表一项设置，它的格式共分为六个字段，前五段是时间设定段，第六段是要执行的命令段，格式如下：

minute  hour  day  month  week  command

其中：

```txt
minute： 表示分钟，可以是从0到59之间的任何整数。

hour：表示小时，可以是从0到23之间的任何整数。

day：表示日期，可以是从1到31之间的任何整数。

month：表示月份，可以是从1到12之间的任何整数。

week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。

command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。

```

![图片](http://mmbiz.qpic.cn/mmbiz_png/9aPYe0E1fb0j6ZUWqoKEiaahPFNDw1FnJSODfDWvMWmdH516eYzkUb8B3X2TeB9iayFhJ8rWX519DDbpicX0aFtFg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 

在以上各个字段中，还可以使用以下特殊字符：

```txt
星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。

逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”

中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”

正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

在了解命令格式之后，我们通过这样的一个例子来完成一个任务的添加，在文档的最后一排加上这样一排命令，该任务是每分钟我们会在/home/shiyanlou 目录下创建一个以当前的年月日时分秒为名字的空白文件
```

```bash
*/1 * * * * touch /home/shiyanlou/$(date +\%Y\%m\%d\%H\%M\%S)
```

**注意：**

> “ % ” 在 crontab 文件中，有结束命令行、换行、重定向的作用，前面加 ” \ ” 符号转义，否则，“ % ” 符号将执行其结束命令行或者换行的作用，并且其后的内容会被做为标准输入发送给前面的命令。

添加成功后我们会得到最后一排 installing new crontab 的一个提示：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081468203483143)

#### 2）`crontab -l`命令

当然我们也可以通过这样的一个指令来查看我们添加了哪些任务：

```bash
crontab -l
```

通过图中的显示，我们也可以看出，我们正确的保存并且添加成功了该任务的：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081468204230683)

#### 3）检查cron的守护进程是否启动`ps aux | grep cron`

虽然我们添加了任务，但是如果 `cron` 的守护进程并没有启动，它根本都不会监测到有任务，当然也就不会帮我们执行，我们可以通过以下 2 种方式来确定我们的 `cron` 是否成功的在后台启动，默默的帮我们做事，若是没有就得执行上文准备中的第二步了。

```bash
ps aux | grep cron

# or

pgrep cron
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6124timestamp1523942683532.png)

通过下图可以看到任务在创建之后，执行了几次，生成了一些文件，且每分钟生成一个：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6124timestamp1523943532369.png)

#### 4）动态查看日志`tail -f`

我们通过这样一个命令可以查看到执行任务命令之后在日志中的信息反馈：

```bash
sudo tail -f /var/log/syslog
```

从图中我们可以看到分别在 13 点 28、29、30 分的 01 秒为我们在 shiyanlou 用户的家目录下创建了文件。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6124timestamp1523943327065.png)

#### 5）`crontab -r`命令

当我们并不需要这个任务的时候我们可以使用这么一个命令去删除任务：

```bash
crontab -r
```

通过图中我们可以看出我们删除之后再查看任务列表，系统已经显示该用户并没有任务哦。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6124timestamp1523943647348.png)

### 4. `crontab`的深入

每个用户使用 `crontab -e` 添加计划任务，都会在 `/var/spool/cron/crontabs` 中添加一个该用户自己的任务文档，这样目的是为了隔离。

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081468206283987)

如果是系统级别的定时任务，需要 root 权限执行的任务应该怎么处理？

只需要使用 `sudo` 编辑 `/etc/crontab` 文件就可以。

`cron` 服务监测时间最小单位是分钟，所以 `cron` 会每分钟去读取一次 `/etc/crontab` 与 `/var/spool/cron/crontabs` 里面的內容。

在 `/etc` 目录下，`cron` 相关的目录有下面几个：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081468206856712)

每个目录的作用：

1. `/etc/cron.daily`，目录下的脚本会每天执行一次，在每天的 6 点 25 分时运行；
2. `/etc/cron.hourly`，目录下的脚本会每个小时执行一次，在每小时的 17 分钟时运行；
3. `/etc/cron.monthly`，目录下的脚本会每月执行一次，在每月 1 号的 6 点 52 分时运行；
4. `/etc/cron.weekly`，目录下的脚本会每周执行一次，在每周第七天的 6 点 47 分时运行；

系统默认执行时间可以根据需求进行修改。

## 十、命令执行顺序控制与管道

### 1. 顺序执行多条命令:`;`

多条命令使用分号隔开，达到顺序执行的目的。

```bash
sudo apt-get update;sudo apt-get install some-tool;some-tool # 让它自己运行
```

### 2. 有选择执行多条命令`&&`,`||`命令

`&&`表示如果前面的命令执行结果（不是表示终端输出的内容，而是表示命令执行状态的结果）返回 0 则执行后面的，否则不执行，你可以从 `$?` 环境变量获取上一次命令的返回结果：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid1labid63timestamp1544148440172.png)

学习过 C 语言的用户应该知道在 C 语言里面 `&&` 表示逻辑与，而且还有一个 `||` 表示逻辑或，同样 Shell 也有一个 `||`，它们的区别就在于，shell 中的这两个符号除了也可用于表示逻辑与和或之外，就是可以实现这里的命令执行顺序的简单控制。`||` 在这里就是与 `&&` 相反的控制效果，当上一条命令执行结果为 `≠0(\$?≠0)` 时则执行它后面的命令：

```bash
which cowsay>/dev/null || echo "cowsay has not been install, please run 'sudo apt-get install cowsay' to install"
```

除了上述基本的使用之外，我们还可以结合着 `&&` 和 `||` 来实现一些操作，比如：

```bash
which cowsay>/dev/null && echo "exist" || echo "not exist"
#注意顺序：&&在前，表示前面执行结果不为0表示执行后面的操作，如果为0，不执行&&后面的而执行||后面的命令；||在前，表示如果命令返回0时，表示直接执行||后的语句
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid63timestamp1532414664955.png)

我画个流程图来解释一下上面的流程：

![1](https://doc.shiyanlou.com/linux_base/8-3.png)

### 3. 管道

**管道是一种通信机制，通常用于进程间的通信（也可通过 socket 进行网络通信），它表现出来的形式就是将前面每一个进程的输出（stdout）直接作为下一个进程的输入（stdin）。**

管道又分为匿名管道和具名管道（这里将不会讨论在源程序中使用系统调用创建并使用管道的情况，它与命令行的管道在内核中实际都是采用相同的机制）。我们在使用一些过滤程序时经常会用到的就是匿名管道，在命令行中由 `|` 分隔符表示，`|` 在前面的内容中我们已经多次使用到了。具名管道简单的说就是有名字的管道，通常只会在源程序中用到具名管道。

#### 1）管道的含义适用`|`

先试用一下管道，比如查看 `/etc` 目录下有哪些文件和目录，使用 `ls` 命令来查看：

```bash
ls -al /etc
```

有太多内容，屏幕不能完全显示，这时候可以使用滚动条或快捷键滚动窗口来查看。不过这时候可以使用管道：

```bash
ls -al /etc | less
```

通过管道将前一个命令(`ls`)的输出作为下一个命令(`less`)的输入，然后就可以一行一行地看。

#### 2）`cut` 命令，打印每一行的某一字段

打印 `/etc/passwd` 文件中以 `:` 为分隔符的第 1 个字段和第 6 个字段分别表示用户名和其家目录：

```
cut /etc/passwd -d ':' -f 1,6
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid63timestamp1532414685006.png)

打印 `/etc/passwd` 文件中每一行的前 N 个字符：

```bash
# 前五个（包含第五个）
cut /etc/passwd -c -5
# 前五个之后的（包含第五个）
cut /etc/passwd -c 5-
# 第五个
cut /etc/passwd -c 5
# 2 到 5 之间的（包含第五个）
cut /etc/passwd -c 2-5
```

#### 3）`grep` 命令，在文本中或 stdin 中查找匹配字符串

`grep` 命令是很强大的，也是相当常用的一个命令，它结合正则表达式可以实现很复杂却很高效的匹配和查找，不过在学习正则表达式之前，这里介绍它简单的使用，而关于正则表达式后面将会有单独一小节介绍到时会再继续学习 `grep` 命令和其他一些命令。

`grep` 命令的一般形式为：

```bash
grep [命令选项]... 用于匹配的表达式 [文件]...
```

还是先体验一下，我们搜索`/home/shiyanlou`目录下所有包含"shiyanlou"的文本文件，并显示出现在文本中的行号：

```bash
grep -rnI "shiyanlou" ~
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid63timestamp1532414709836.png)

`-r` 参数表示递归搜索子目录中的文件，`-n` 表示打印匹配项行号，`-I` 表示忽略二进制文件。这个操作实际没有多大意义，但可以感受到 `grep` 命令的强大与实用。

当然也可以在匹配字段中使用正则表达式，下面简单的演示：

```bash
# 查看环境变量中以 "yanlou" 结尾的字符串
export | grep ".*yanlou$"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid63timestamp1532414725827.png)

其中`$`就表示一行的末尾。

#### 4）`wc `命令，简单小巧的计数工具

wc 命令用于统计并输出一个文件中行、单词和字节的数目，比如输出 `/etc/passwd` 文件的统计信息：

```bash
wc /etc/passwd
```

分别只输出行数、单词数、字节数、字符数和输入文本中最长一行的字节数：

```bash
# 行数
wc -l /etc/passwd
# 单词数
wc -w /etc/passwd
# 字节数
wc -c /etc/passwd
# 字符数
wc -m /etc/passwd
# 最长行字节数
wc -L /etc/passwd
```

**注意：对于西文字符来说，一个字符就是一个字节，但对于中文字符一个汉字是大于 或等于2 个字节的，具体数目是由字符编码决定的。**

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid63timestamp1532414808838.png)

再来结合管道来操作一下，下面统计 /etc 下面所有目录数：

```bash
ls -dl /etc/*/ | wc -l
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6125timestamp1523946094712.png)

#### 5）`sort` 排序命令

这个命令前面我们也是用过多次，功能很简单就是将输入按照一定方式排序，然后再输出，它支持的排序有按字典排序，数字排序，按月份排序，随机排序，反转排序，指定特定字段进行排序等等。

默认为字典排序：

```bash
cat /etc/passwd | sort
```

反转排序：

```bash
cat /etc/passwd | sort -r
```

按特定字段排序：

```bash
cat /etc/passwd | sort -t':' -k 3
```

上面的`-t`参数用于指定字段的分隔符，这里是以":"作为分隔符；`-k 字段号`用于指定对哪一个字段进行排序。这里`/etc/passwd`文件的第三个字段为数字，默认情况下是以字典序排序的，如果要按照数字排序就要加上`-n`参数：

```bash
cat /etc/passwd | sort -t':' -k 3 -n
```

注意观察第二个冒号后的数字： ![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid63timestamp1532414849333.png)

#### 6）uniq 去重命令

`uniq` 命令可以用于过滤或者输出重复行。

- 过滤重复行

我们可以使用 `history` 命令查看最近执行过的命令（实际为读取 `${SHELL}_history` 文件，如我们环境中的 `.zsh_history` 文件），不过你可能只想查看使用了哪个命令而不需要知道具体干了什么，那么你可能就会要想去掉命令后面的参数然后去掉重复的命令：

```bash
history | cut -c 8- | cut -d ' ' -f 1 | uniq
```

然后经过层层过滤，你会发现确是只输出了执行的命令那一列，不过去重效果好像不明显，仔细看你会发现它确实去重了，只是不那么明显，之所以不明显是因为 `uniq` 命令只能去连续重复的行，不是全文去重，所以要达到预期效果，我们先排序：

```bash
history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq
# 或者
history | cut -c 8- | cut -d ' ' -f 1 | sort -u
```

这就是 Linux/UNIX 哲学吸引人的地方，大繁至简，一个命令只干一件事却能干到最好。

- 输出重复行

```bash
# 输出重复过的行（重复的只输出一个）及重复次数
history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq -dc
# 输出所有重复的行
history | cut -c 8- | cut -d ' ' -f 1 | sort | uniq -D
```

文本处理命令还有很多，下一节将继续介绍一些常用的文本处理的命令。

## 十一、简单的文本处理

### 1. `tr`命令

**`tr` 命令可以用来删除一段文本信息中的某些文字。或者将其进行转换**。

**使用方式**

```bash
tr [option]...SET1 [SET2]
```

**常用的选项有**

| 选项 | 说明                                                         |
| ---- | ------------------------------------------------------------ |
| `-d` | 删除和 set1 匹配的字符，注意不是全词匹配也不是按字符顺序匹配 |
| `-s` | 去除 set1 指定的在输入文本中连续并重复的字符                 |

**操作举例**

```bash
# 删除 "hello shiyanlou" 中所有的'o'，'l'，'h'
$ echo 'hello shiyanlou' | tr -d 'olh'
# 将"hello" 中的ll，去重为一个l
$ echo 'hello' | tr -s 'l'
# 将输入文本，全部转换为大写或小写输出（正则表达式）
$ echo 'input some text here' | tr '[:lower:]' '[:upper:]'
# 上面的'[:lower:]' '[:upper:]'你也可以简单的写作'[a-z]' '[A-Z]'，当然反过来将大写变小写也是可以的
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid337timestamp1532414877239.png)

更多 `tr` 的使用，你可以使用`--help`或者`man tr`获得。

### 2. `col`命令

col 命令可以将`Tab`换成对等数量的空格键，或反转这个操作。

**使用方式**

```bash
col [option]
```

**常用的选项有**

| 选项 | 说明                          |
| ---- | ----------------------------- |
| `-x` | 将`Tab`转换为空格             |
| `-h` | 将空格转换为`Tab`（默认选项） |

**操作举例**

```bash
# 查看 /etc/protocols 中的不可见字符，可以看到很多 ^I ，这其实就是 Tab 转义成可见字符的符号
cat -A /etc/protocols
# 使用 col -x 将 /etc/protocols 中的 Tab 转换为空格，然后再使用 cat 查看，你发现 ^I 不见了
cat /etc/protocols | col -x | cat -A
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid337timestamp1532414886554.png)

### 3. `join`命令

这个命令就是用于将两个文件中包含相同内容的那一行合并在一起。

**使用方式**

```bash
join [option]... file1 file2
```

**常用的选项有**

| 选项 | 说明                                                 |
| ---- | ---------------------------------------------------- |
| `-t` | 指定分隔符，默认为空格                               |
| `-i` | 忽略大小写的差异                                     |
| `-1` | 指明第一个文件要用哪个字段来对比，默认对比第一个字段 |
| `-2` | 指明第二个文件要用哪个字段来对比，默认对比第一个字段 |

**操作举例**

```bash
cd /home/shiyanlou
# 创建两个文件
echo '1 hello' > file1
echo '1 shiyanlou' > file2
join file1 file2
# 将 /etc/passwd 与 /etc/shadow 两个文件合并，指定以':'作为分隔符
sudo join -t':' /etc/passwd /etc/shadow
# 将 /etc/passwd 与 /etc/group 两个文件合并，指定以':'作为分隔符，分别比对第4和第3个字段
sudo join -t':' -1 4 /etc/passwd -2 3 /etc/group
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid337timestamp1532414902443.png) ![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid337timestamp1532414948354.png)

### 4. `paste`命令

`paste`这个命令与`join` 命令类似，它是在不对比数据的情况下，简单地将多个文件合并一起，以`Tab`隔开。

**使用方式**

```bash
paste [option] file...
```

**常用的选项有**

| 选项 | 说明                         |
| ---- | ---------------------------- |
| `-d` | 指定合并的分隔符，默认为 Tab |
| `-s` | 不合并到一行，每个文件为一行 |

**操作举例**

```bash
echo hello > file1
echo shiyanlou > file2
echo www.shiyanlou.com > file3
paste -d ':' file1 file2 file3
paste -s file1 file2 file3
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid337timestamp1532414967936.png)

## 十二、数据流重定向

重定向操作：

```bash
echo 'hello shiyanlou' > redirect
echo 'www.shiyanlou.com' >> redirect
cat redirect
```

> 当然前面没有用到的 `<` 和 `<<` 操作也是没有问题的，如你理解的一样，它们的区别在于重定向的方向不一致而已，`>` 表示是从左到右，`<` 右到左。

 Linux 默认提供了三个特殊设备，用于终端的显示和输出，分别为 `stdin`（标准输入，对应于你在终端的输入），`stdout`（标准输出，对应于终端的输出），`stderr`（标准错误输出，对应于终端的输出）。

| 文件描述符 | 设备文件      | 说明     |
| ---------- | ------------- | -------- |
| `0`        | `/dev/stdin`  | 标准输入 |
| `1`        | `/dev/stdout` | 标准输出 |
| `2`        | `/dev/stderr` | 标准错误 |

> 文件描述符：文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。在程序设计中，一些涉及底层的程序编写往往会围绕着文件描述符展开。但是文件描述符这一概念往往只适用于 UNIX、Linux 这样的操作系统。

### 1. 重定向与管道区别

我们可以这样使用这些文件描述符。例如默认使用终端的标准输入作为命令的输入和标准输出作为命令的输出：

```bash
cat # 按 Ctrl+C 退出
```

将 cat 的连续输出（heredoc 方式）重定向到一个文件：

```bash
mkdir Documents
cat > Documents/test.c <<EOF
#include <stdio.h>

int main()
{
    printf("hello world\n");
    return 0;
}

EOF
```

将一个文件作为命令的输入，标准输出作为命令的输出：

```bash
cat Documents/test.c
```

将 echo 命令通过管道传过来的数据作为 cat 命令的输入，将标准输出作为命令的输出：

```bash
echo 'hi' | cat
```

将 echo 命令的输出从默认的标准输出重定向到一个普通文件：

```bash
echo 'hello shiyanlou' > redirect
cat redirect
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid346timestamp1532415296335.png)

初学者这里要注意不要将管道和重定向混淆，**管道默认是连接前一个命令的输出到下一个命令的输入**，而重定向通常是需要一个文件来建立两个命令的连接，你可以仔细体会一下上述第三个操作和最后两个操作的异同点。

### 2. 标准错误重定向

重定向标准输出到文件，这是一个很实用的操作，另一个很实用的操作是将标准错误重定向，标准输出和标准错误都被指向伪终端的屏幕显示，所以我们经常看到的一个命令的输出通常是同时包含了标准输出和标准错误的结果的。比如下面的操作：

```bash
# 使用cat 命令同时读取两个文件，其中一个存在，另一个不存在
cat Documents/test.c hello.c
# 你可以看到除了正确输出了前一个文件的内容，还在末尾出现了一条错误信息
# 下面我们将输出重定向到一个文件
cat Documents/test.c hello.c > somefile
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6127timestamp1523951670892.png)

遗憾的是，这里依然出现了那条错误信息，这正是因为如我上面说的那样，标准输出和标准错误虽然都指向终端屏幕，实际它们并不一样。那有的时候我们就是要隐藏某些错误或者警告，那又该怎么做呢。这就需要用到我们前面讲的文件描述符了：

```bash
# 将标准错误重定向到标准输出，再将标准输出重定向到文件，注意要将重定向到文件写到前面
cat Documents/test.c hello.c >somefile  2>&1
# 或者只用bash提供的特殊的重定向符号"&"将标准错误和标准输出同时重定向到文件
cat Documents/test.c hello.c &>somefilehell
```

**注意你应该在输出重定向文件描述符前加上`&`，否则 shell 会当做重定向到一个文件名为 1 的文件中**

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6127timestamp1523951876075.png)

### 3. 使用 `tee` 命令同时重定向到多个文件

你可能还有这样的需求，除了需要将输出重定向到文件，也需要将信息打印在终端。那么你可以使用 `tee` 命令来实现：

```bash
echo 'hello shiyanlou' | tee hello
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid735639labid346timestamp1532415315324.png)

### 4. 永久重定向

当然不需要，我们可以使用 `exec` 命令实现永久重定向。`exec` 命令的作用是使用指定的命令替换当前的 Shell，即使用一个进程替换当前进程，或者指定新的重定向：

```bash
# 先开启一个子 Shell
zsh
# 使用exec替换当前进程的重定向，将标准输出重定向到一个文件
exec 1>somefile
# 后面你执行的命令的输出都将被重定向到文件中，直到你退出当前子shell，或取消exec的重定向（后面将告诉你怎么做）
ls
exit
cat somefile
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid6127timestamp1523952144929.png)

## 十三、软件包管理（centos、乌班图）

                包的组成：
                    二进制文件、库文件、配置文件、帮助文件
                程序包管理器：
                    debian： deb文件, dpkg包管理器
                    redhat： rpm文件, rpm包管理器
                             rpm：Redhat Package Manager
                                  RPM Package Manager
                包之间：可能存在依赖关系，甚至循环依赖
    
                解决依赖包管理工具：
                    yum：rpm包管理器的前端工具
                    apt-get：deb包管理器前端工具
                    zypper: suse上的rpm前端管理工具
                    dnf: Fedora 18+ rpm包管理器前端管理工具
                查看二进制程序所依赖的库文件：
                    ldd /PATH/TO/BINARY_FILE
                管理及查看本机装载的库文件：
                    ldconfig :加载配置文件中指定的库文件
                    /sbin/ldconfig -p :显示本机已经缓存的所有可用库文件名及文件路径映射关系
                    配置文件: /etc/ld.so.conf   /etc/ld.so.conf.d/*.conf
                    缓存文件：/etc/ld.so.cache 
    
                程序包管理器：
                    功能：将编译好的应用程序的各组成文件打包一个或几个程序包文件，从而方便快捷地实现程序包的安装、卸载、查询、升级和校验等管理操作
                数据库(公共)：/var/lib/rpm
                    程序包名称及版本
                    依赖关系
                    功能说明
                    包安装后生成的各文件路径及校验码信息
                获取程序包的途径：
                    (1) 系统发版的光盘或官方的服务器
                        CentOS镜像：
                        https://www.centos.org/download/
                        http://mirrors.aliyun.com
                        http://mirrors.sohu.com
                        http://mirrors.163.com
                    (2) 项目官方站点
                    (3) 第三方组织：
                            Fedora-EPEL：
                                Extra Packages for Enterprise Linux
                            Rpmforge:RHEL推荐，包很全
                            搜索引擎：
                                http://pkgs.org
                                http://rpmfind.net
                                http://rpm.pbone.net
                                https://sourceforge.net/
                    (4) 自己制作
                    注意：第三方包建议要检查其合法性来源合法性,程序包的完整性
    
                CentOS系统上使用rpm命令管理程序包：
                    安装、卸载、升级、查询、校验、数据库维护
                    安装：
                    rpm {-i|--install} [install-options] PACKAGE_FILE… 
                        -v: verbose
                        -vv: 
                        -h: 以#显示程序包管理执行进度
                    rpm -ivh PACKAGE_FILE ...
    
                    [install-options]
                        --test: 测试安装，但不真正执行安装，即dry run模式
                        --nodeps：忽略依赖关系
                        --replacepkgs | replacefiles
                        --nosignature: 不检查来源合法性
                        --nodigest：不检查包完整性
                        --noscripts：不执行程序包脚本
                            %pre: 安装前脚本 --nopre
                            %post: 安装后脚本 --nopost
                            %preun: 卸载前脚本 --nopreun
                            %postun: 卸载后脚本 --nopostun
                    升级：
                        rpm {-U|--upgrade} [install-options] PACKAGE_FILE...
                        rpm {-F|--freshen} [install-options] PACKAGE_FILE...
                            upgrade：安装有旧版程序包，则“升级”
                                     如果不存在旧版程序包，则“安装”
                            freshen：安装有旧版程序包，则“升级”
                                    如果不存在旧版程序包，则不执行
                            rpm -Uvh PACKAGE_FILE ...
                            rpm -Fvh PACKAGE_FILE ...
                            --oldpackage：降级
                            --force: 强制安装
    
                    注意：
                        (1) 不要对内核做升级操作；Linux支持多内核版本并存，因此，对直接安装新版本内核
                        (2) 如果原程序包的配置文件安装后曾被修改，升级时，新版本的提供的同一个配置文件并不会直接覆盖老版本的配置文件，而把新版本的文件重命名(FILENAME.rpmnew)后保留


                    包查询：
                        rpm {-q|--query} [select-options] [query-options]
                        [select-options]
                            -a: 所有包
                            -f: 查看指定的文件由哪个程序包安装生成
                            -p rpmfile：针对尚未安装的程序包文件做查询操作
                            --whatprovides CAPABILITY：查询指定的CAPABILITY由哪个包所提供
                            --whatrequires CAPABILITY：查询指定的CAPABILITY被哪个包所依赖
                        常用查询用法：
                            -qi PACKAGE, -qf FILE, -qc PACKAGE, -ql PACKAGE, -qd PACKAGE
                            -qpi PACKAGE_FILE, -qpl PACKAGE_FILE, ...
                            -qa
                    rpm2cpio 包文件|cpio –itv 预览包内文件
                    rpm2cpio 包文件|cpio –id “*.conf” 释放包内文件
    
                    包校验：
                        rpm {-V|--verify} [select-options] [verify-options]
                            S file Size differs
                            M Mode differs (includes permissions and file type)
                            5 digest (formerly MD5 sum) differs
                            D Device major/minor number mismatch
                            L readLink(2) path mismatch
                            U User ownership differs
                            G Group ownership differs
                            T mTime differs
                            P capabilities differ
                        包来源合法性验正及完整性验证
                            完整性验证：SHA256
                            来源合法性验证：RSA
                        公钥加密
                            对称加密：加密、解密使用同一密钥
                            非对称加密：密钥是成对儿的
                                public key: 公钥，公开所有人
                                secret key: 私钥, 不能公开
                        导入所需要公钥
                            rpm -K|checksig rpmfile 检查包的完整性和签名
                            rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
                            CentOS 7发行版光盘提供：RPM-GPG-KEY-CentOS-7
                            rpm -qa “gpg-pubkey*”
    
                    rpm数据库：
                        数据库重建：
                            /var/lib/rpm
                        rpm {--initdb|--rebuilddb}
                            initdb: 初始化
                                如果事先不存在数据库，则新建之
                                否则，不执行任何操作
                            rebuilddb：重建已安装的包头的数据库索引目录
    
                CentOS系统上使用yum命令：
                    YUM: Yellowdog Update Modifier，rpm的前端程序，可解决软件包相关依赖性，可在多个库之间定位软件包，up2date的替代工具
                        yum repository: yum repo，存储了众多rpm包，以及包的相关的元数据文件（放置于特定目录repodata下）
                        文件服务器：
                            http://
                            https://
                            ftp://
                            file://
    
                    yum配置文件：
                        yum客户端配置文件：
                            /etc/yum.conf：为所有仓库提供公共配置
                            /etc/yum.repos.d/*.repo：为仓库的指向提供配置
                            仓库指向的定义：
                                [repositoryID]
                                name=Some name for this repository
                                baseurl=url://path/to/repository/
                                enabled={1|0}
                                gpgcheck={1|0}
                                gpgkey=URL
                                enablegroups={1|0}
                                failovermethod={roundrobin|priority}
                                        roundrobin：意为随机挑选，默认值
                                        priority:按顺序访问
                                cost= 默认为1000
    
                            yum的repo配置文件中可用的变量：
                                $releasever: 当前OS的发行版的主版本号
                                $arch: 平台，i386,i486,i586,x86_64等
                                $basearch：基础平台；i386, x86_64
                                $YUM0-$YUM9:自定义变量
    
                            yum源：
                                阿里云repo文件
                                    http://mirrors.aliyun.com/repo/
                                CentOS系统的yum源
                                    阿里云：https://mirrors.aliyun.com/centos/$releasever/os/x86_64/
                                    清华大学：https://mirrors.tuna.tsinghua.edu.cn/centos/$releaseve
                                EPEL的yum源
                                    阿里云：https://mirrors.aliyun.com/epel/$releasever/x86_64
                                    阿里巴巴开源软件https://opsx.alibaba.com/
    
                            yum命令：
                                yum的命令行选项：
                                    --nogpgcheck：禁止进行gpg check
                                    -y: 自动回答为“yes”
                                    -q：静默模式
                                    --disablerepo=repoidglob：临时禁用此处指定的repo
                                    --enablerepo=repoidglob：临时启用此处指定的repo
                                    --noplugins：禁用所有插件
    
                                yum命令的用法：
                                    yum [options] [command] [package ...]
                                显示仓库列表：
                                    yum repolist [all|enabled|disable
                                显示程序包：
                                    yum list
                                    yum list [all | glob_exp1] [glob_exp2] [...]
                                    yum list {available|installed|updates} [glob_exp1] [...]
                                安装程序包：
                                    yum install package1 [package2] [...]
                                    yum reinstall package1 [package2] [...] (重新安装)
                                升级程序包：
                                    yum update [package1] [package2] [...]
                                    yum downgrade package1 [package2] [...] (降级)
                                卸载程序包：
                                    yum remove | erase package1 [package2] [...]
    
                                查看程序包information：
                                    yum info [...]
                                查看指定的特性(可以是某文件)是由哪个程序包所提供：
                                    yum provides | whatprovides feature1 [feature2] [...]
                                清理本地缓存：
                                    清除/var/cache/yum/$basearch/$releasever缓存
                                    yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]
                                构建缓存：
                                    yum makecache
    
                                查看yum事务历史：
                                    yum history [info|list|packages-list|packages-info|
                                    summary|addon-info|redo|undo|
                                    rollback|new|sync|stats]
                                    yum history
                                    yum history info 6
                                    yum history undo 6 
    
                                    日志 ：/var/log/yum.log
    
                                安装及升级本地程序包：
                                    yum localinstall rpmfile1 [rpmfile2] [...]
                                         (用install替代)
                                    yum localupdate rpmfile1 [rpmfile2] [...]
                                         (用update替代)
    
                                包组管理的相关命令：
                                    yum groupinstall group1 [group2] [...]
                                    yum groupupdate group1 [group2] [...]
                                    yum grouplist [hidden] [groupwildcard] [...]
                                    yum groupremove group1 [group2] [...]
                                    yum groupinfo group1 [...]
    
                Ubuntu软件管理
                    dpkg常见用法： man dpkg
                        dpkg -i package.deb 安装包
                        dpkg -r package 删除包，不建议，不自动卸载依赖于它的包
                        dpkg -P package 删除包（包括配置文件）
                        dpkg -l 列出当前已安装的包，类似rpm -qa
                        dpkg -l package 显示该包的简要说明，类似rpm –qi
                        dpkg -L package 列出该包中所包含的文件，类似rpm –ql
                        dpkg -S <pattern> 搜索包含pattern的包，类似rpm –qf
                        dpkg -s package 列出该包的状态，包括详细信息，类似rpm –qi
                        dpkg --configure package 配置包，-a 使用，配置所有没有配置的软件包
                        dpkg -c package.deb 列出 deb 包的内容
                    dpkg示例：
                        列出系统上安装的所有软件包
                            dpkg -log
                        列出软件包安装的文件
                            dpkg -L bash
                        查看/bin/bash来自于哪个软件包
                            dpkg -S /bin/bash
                        安装本地的 .deb 文件
                            dpkg -i /mnt/cdrom/pool/main/z/zip/zip_3.0-11build1_amd64.deb
                        卸载软件包
                            dpkg -r zip
    
                    apt命令：
                        apt与apt-get命令对比：
                            apt 命令         被取代的命令             命令的功能
                            apt install     apt-get install     安装软件包
                            apt remove         apt-get remove         移除软件包
                            apt purge         apt-get purge         移除软件包及配置文件
                            apt update         apt-get update         刷新存储库索引
                            apt upgrade     apt-get upgrade     升级所有可升级的软件包
                            apt autoremove     apt-get autoremove     自动删除不需要的包
                            apt full-upgrade apt-get dist-upgrade 在升级软件包时自动处理依赖关系
                            apt search         apt-cache search     搜索应用程序
                            apt show         apt-cache show         显示安装细节
    
                        apt 特有的命令：
                            apt list             列出包含条件的包（已安装，可升级等）
                            apt edit-sources     编辑源列表
                        apt命令操作（如安装和删除软件包）记录在/var/log/dpkg.log日志文
                        APT包索引来自/etc/apt/sources.list文件和/etc/apt/sources.list.d目录中定义的存储库的可用包的数据库。要使用存储库中所做的最新更改来更新本地程序包索引
    
                        安装包：
                            apt install tree zip
                        删除包：
                            apt remove tree zip
                            说明：apt remove中添加--purge选项会删除包配置文件，谨慎使用
                        更新包索引：
                            apt update

### yum 命令

yum（ Yellow dog Updater, Modified）是一个在Fedora和RedHat以及SUSE中的Shell前端软件包管理器。

基於RPM包管理，能够从指定的服务器自动下载RPM包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软体包，无须繁琐地一次次下载、安装。

yum提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

1. 列出所有可更新的软件清单命令：yum check-update
2. 更新所有软件命令：yum update
3. 仅安装指定的软件命令：yum install <package_name>
4. 仅更新指定的软件命令：yum update <package_name>
5. 列出所有可安裝的软件清单命令：yum list
6. 删除软件包命令：yum remove <package_name>
7. 查找软件包 命令：yum search
8. 清除缓存命令:

- yum clean packages: 清除缓存目录下的软件包
- yum clean headers: 清除缓存目录下的 headers
- yum clean oldheaders: 清除缓存目录下旧的 headers
- yum clean, yum clean all (= yum clean packages; yum clean oldheaders) :清除缓存目录下的软件包及旧的headers

安装 pam-devel

```bash
yum install pam-devel
```

## 十四、进程概念

### 1. 概念的理解

首先程序与进程是什么？程序与进程又有什么区别？

#### 1）程序

> **程序**（procedure）：不太精确地说，程序就是执行一系列有逻辑、有顺序结构的指令，帮我们达成某个结果。就如我们去餐馆，给服务员说我要牛肉盖浇饭，她执行了做牛肉盖浇饭这么一个程序，最后我们得到了这么一盘牛肉盖浇饭。它需要去执行，不然它就像一本武功秘籍，放在那里等人翻看。

#### 2）进程


> **进程**（process）：进程是程序在一个数据集合上的一次执行过程，在早期的 UNIX、Linux 2.4 及更早的版本中，它是系统进行资源分配和调度的独立基本单位。同上一个例子，就如我们去了餐馆，给服务员说我要牛肉盖浇饭，她执行了做牛肉盖浇饭这么一个程序，而里面做饭的是一个进程，做牛肉汤汁的是一个进程，把牛肉汤汁与饭混合在一起的是一个进程，把饭端上桌的是一个进程。它就像是我们在看武功秘籍这么一个过程，然后一个篇章一个篇章地去练。

简单来说，程序是为了完成某种任务而设计的软件，比如 vim 是程序。什么是进程呢？进程就是运行中的程序。

程序只是一些列指令的集合，是一个静止的实体，而进程不同，进程有以下的特性：

- 动态性：进程的实质是一次程序执行的过程，有创建、撤销等状态的变化。而程序是一个静态的实体。
- 并发性：进程可以做到在一个时间段内，有多个程序在运行中。程序只是静态的实体，所以不存在并发性。
- 独立性：进程可以独立分配资源，独立接受调度，独立地运行。
- 异步性：进程以不可预知的速度向前推进。
- 结构性：进程拥有代码段、数据段、PCB（进程控制块，进程存在的唯一标志）。也正是因为有结构性，进程才可以做到独立地运行。

> **并发：**在一个时间段内，宏观来看有多个程序都在活动，有条不紊的执行（每一瞬间只有一个在执行，只是在一段时间有多个程序都执行过）

> **并行：**在每一个瞬间，都有多个程序都在同时执行，这个必须有多个 CPU 才行

引入进程是因为传统意义上的程序已经不足以描述 OS 中各种活动之间的动态性、并发性、独立性还有相互制约性。程序就像一个公司，只是一些证书，文件的堆积（静态实体）。而当公司运作起来就有各个部门的区分，财务部，技术部，销售部等等，就像各个进程，各个部门之间可以独立运作，也可以有交互（独立性、并发性）。

而随着程序的发展越做越大，又会继续细分，从而引入了线程的概念，当代多数操作系统、Linux 2.6 及更新的版本中，进程本身不是基本运行单位，而是线程的容器。就像上述所说的，每个部门又会细分为各个工作小组（线程），而工作小组需要的资源需要向上级（进程）申请。

> **线程**（thread）是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。因为线程中几乎不包含系统资源，所以执行更快、更有效率。

简而言之，一个程序至少有一个进程，一个进程至少有一个线程。线程的划分尺度小于进程，使得多线程程序的并发性高。另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。就如下图所示：

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469062947147)

### 2. 进程的分类

大概明白进程是个什么样的存在后，我们需要进一步了解的就是进程分类。可以从两个角度来分：

- 以进程的功能与服务的对象来分；
- 以应用程序的服务类型来分；

第一个角度来看，我们可以分为用户进程与系统进程：

- 用户进程：通过执行用户程序、应用程序或称之为内核之外的系统程序而产生的进程，此类进程可以在用户的控制下运行或关闭。
- 系统进程：通过执行系统内核程序而产生的进程，比如可以执行内存资源分配和进程切换等相对底层的工作；而且该进程的运行不受用户的干预，即使是 root 用户也不能干预系统进程的运行。

第二角度来看，我们可以将进程分为交互进程、批处理进程、守护进程：

- 交互进程：由一个 shell 终端启动的进程，在执行过程中，需要与用户进行交互操作，可以运行于前台，也可以运行在后台。
- 批处理进程：该进程是一个进程集合，负责按顺序启动其他的进程。
- 守护进程：守护进程是一直运行的一种进程，在 Linux 系统启动时启动，在系统关闭时终止。它们独立于控制终端并且周期性的执行某种任务或等待处理某些发生的事件。例如 httpd 进程，一直处于运行状态，等待用户的访问。还有经常用的 cron（在 centOS 系列为 crond）进程，这个进程为 crontab 的守护进程，可以周期性的执行用户设定的某些任务。

### 3. 进程组与 Sessions

每一个进程都会是一个进程组的成员，而且这个进程组是唯一存在的，他们是依靠 PGID（process group ID）来区别的，而每当一个进程被创建的时候，它便会成为其父进程所在组中的一员。

一般情况，进程组的 PGID 等同于进程组的第一个成员的 PID，并且这样的进程称为该进程组的领导者，也就是领导进程，进程一般通过使用 `getpgrp()` 系统调用来寻找其所在组的 PGID，领导进程可以先终结，此时进程组依然存在，并持有相同的 PGID，直到进程组中最后一个进程终结。

与进程组类似，每当一个进程被创建的时候，它便会成为其父进程所在 Session 中的一员，每一个进程组都会在一个 Session 中，并且这个 Session 是唯一存在的，

Session 主要是针对一个 tty 建立，Session 中的每个进程都称为一个工作(job)。每个会话可以连接一个终端(control terminal)。当控制终端有输入输出时，都传递给该会话的前台进程组。Session 意义在于将多个 jobs 囊括在一个终端，并取其中的一个 job 作为前台，来直接接收该终端的输入输出以及终端信号。 其他 jobs 在后台运行。

> **前台**（foreground）就是在终端中运行，能与你有交互的

> **后台**（background）就是在终端中运行，但是你并不能与其任何的交互，也不会显示其执行的过程

### 4. 工作管理

bash(Bourne-Again shell)支持工作控制（job control），而 sh（Bourne shell）并不支持。

并且每个终端或者说 bash 只能管理当前终端中的 job，不能管理其他终端中的 job。比如我当前存在两个 bash 分别为 bash1、bash2，bash1 只能管理其自己里面的 job 并不能管理 bash2 里面的 job

我们都知道当一个进程在前台运作时我们可以用 `ctrl + c` 来终止它，但是若是在后台的话就不行了。

#### 1）`&`命令

我们可以通过 `&` 这个符号，让我们的命令在后台中运行：

```bash
ls &
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469036077882)

图中所显示的 `[1] 236`分别是该 job 的 job number 与该进程的 PID，而最后一行的 Done 表示该命令已经在后台执行完毕。

#### 2）`ctrl + z` 命令

我们还可以通过 `ctrl + z` 使我们的当前工作停止并丢到后台中去

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469036715105)

被停止并放置在后台的工作我们可以使用这个命令来查看：

```bash
jobs
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469037134869)

其中第一列显示的为被放置后台 job 的编号，而第二列的 `＋` 表示最近(刚刚、最后)被放置后台的 job，同时也表示预设的工作，也就是若是有什么针对后台 job 的操作，首先对预设的 job，`-` 表示倒数第二（也就是在预设之前的一个）被放置后台的工作，倒数第三个（再之前的）以后都不会有这样的符号修饰，第三列表示它们的状态，而最后一列表示该进程执行的命令。

我们可以通过这样的一个命令将后台的工作拿到前台来：

```bash
# 后面不加参数提取预设工作，加参数提取指定工作的编号
# ubuntu 在 zsh 中需要 %，在 bash 中不需要 %
fg [%jobnumber]
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469037555070)

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469037666320)

之前我们通过 `ctrl + z` 使得工作停止放置在后台，若是我们想让其在后台运作我们就使用这样一个命令：

```bash
#与fg类似，加参则指定，不加参则取预设
bg [%jobnumber]
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469037983282)

#### 3）`kill`命令

发送指定的信号到相应进程。不指定型号将发送`SIGTERM（15）`终止指定进程。如果仍无法终止该程序可用"-KILL" 参数，其发送的信号为`SIGKILL(9)` ，将强制结束进程，使用`ps`命令或者`jobs` 命令可以查看进程号。root用户将影响用户的进程，非root用户只能影响自己的进程。

常用参数：

```
-l  信号，若果不加信号的编号参数，则使用“-l”参数会列出全部的信号名称
-a  当处理当前进程时，不限制命令名和进程号的对应关系
-p  指定kill 命令只打印相关进程的进程号，而不发送任何信号
-s  指定发送信号
-u  指定用户
```

（1）先使用`ps`查找进程pro1，然后用`kill`杀掉

```
kill -9 $(ps -ef | grep pro1)
```

#### 4）`ps` 命令

`ps` 也是我们最常用的查看进程的工具之一，我们通过这样的一个命令来了解一下，它能给我们带来哪些信息：

```bash
ps -aux
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469086224826)

```bash
ps axjf
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469086474033)

我们来总体了解下会出现哪些信息给我们，这些信息又代表着什么（更多的 keywords 大家可以通过 `man ps` 了解）。

| 内容        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| `F`         | 进程的标志（process flags），当 flags 值为 1 则表示此子程序只是 fork 但没有执行 exec，为 4 表示此程序使用超级管理员 root 权限 |
| `USER`      | 进程的拥有用户                                               |
| `PID`       | 进程的 ID                                                    |
| `PPID`      | 其父进程的 PID                                               |
| `SID`       | session 的 ID                                                |
| `TPGID`     | 前台进程组的 ID                                              |
| `%CPU`      | 进程占用的 CPU 百分比                                        |
| `%MEM`      | 占用内存的百分比                                             |
| `NI`        | 进程的 NICE 值                                               |
| `VSZ`       | 进程使用虚拟内存大小                                         |
| `RSS`       | 驻留内存中页的大小                                           |
| `TTY`       | 终端 ID                                                      |
| `S or STAT` | 进程状态                                                     |
| `WCHAN`     | 正在等待的进程资源                                           |
| `START`     | 启动进程的时间                                               |
| `TIME`      | 进程消耗 CPU 的时间                                          |
| `COMMAND`   | 命令的名称和参数                                             |

> **TPGID**栏写着-1 的都是没有控制终端的进程，也就是守护进程

> **STAT**表示进程的状态，而进程的状态有很多，如下表所示

| 状态 | 解释                               |
| ---- | ---------------------------------- |
| `R`  | Running.运行中                     |
| `S`  | Interruptible Sleep.等待调用       |
| `D`  | Uninterruptible Sleep.不可中断睡眠 |
| `T`  | Stoped.暂停或者跟踪状态            |
| `X`  | Dead.即将被撤销                    |
| `Z`  | Zombie.僵尸进程                    |
| `W`  | Paging.内存交换                    |
| `N`  | 优先级低的进程                     |
| `<`  | 优先级高的进程                     |
| `s`  | 进程的领导者                       |
| `L`  | 锁定状态                           |
| `l`  | 多线程状态                         |
| `+`  | 前台进程                           |

> 其中的 D 是不能被中断睡眠的状态，处在这种状态的进程不接受外来的任何 signal，所以无法使用 kill 命令杀掉处于 D 状态的进程，无论是 `kill`，`kill -9` 还是 `kill -15`，一般处于这种状态可能是进程 I/O 的时候出问题了。

ps 工具有许多的参数，下面给大家解释部分常用的参数。

使用 `-l` 参数可以显示自己这次登录的 bash 相关的进程信息罗列出来：

```bash
ps -l
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469088103140)

相对来说我们更加常用下面这个命令，他将会罗列出所有的进程信息：

```bash
ps -aux
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469089342412)

若是查找其中的某个进程的话，我们还可以配合着 `grep` 和正则表达式一起使用：

```bash
ps -aux | grep zsh
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469089837027)

此外我们还可以查看时，将连同部分的进程呈树状显示出来：

```bash
ps axjf
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469090040956)

当然如果你觉得使用这样的此时没有把你想要的信息放在一起，我们也可以是用这样的命令，来自定义我们所需要的参数显示：

```bash
ps -afxo user,ppid,pid,pgid,command
```

![实验楼](https://dn-simplecloud.shiyanlou.com/1135081469004994601)

这是一个简单而又实用的工具，想要更灵活的使用，想要知道更多的参数我们可以使用 man 来获取更多相关的信息。

`ps(process status)`，用来查看当前运行的进程状态，一次性查看，如果需要动态连续结果使用 `top linux`上进程有5种状态:

1. 运行(正在运行或在运行队列中等待)
2. 中断(休眠中, 受阻, 在等待某个条件的形成或接受到信号)
3. 不可中断(收到信号不唤醒和不可运行, 进程必须等待直到有中断发生)
4. 僵死(进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放)
5. 停止(进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行)

`ps` 工具标识进程的5种状态码:

```txt
D 不可中断 uninterruptible sleep (usually IO)
R 运行 runnable (on run queue)
S 中断 sleeping
T 停止 traced or stopped
Z 僵死 a defunct (”zombie”) process
```

命令参数：

```txt
-A 显示所有进程
a 显示所有进程
-a 显示同一终端下所有进程
c 显示进程真实名称
e 显示环境变量
f 显示进程间的关系
r 显示当前终端运行的进程
-aux 显示所有包含其它使用的进程
```

（1）显示当前所有进程环境变量及进程间关系

```bash
ps -ef
```

（2）显示当前所有进程

```bash
ps -A
```

（3）与grep联用查找某进程

```bash
ps -aux | grep apache
```

（4）找出与 cron 与 syslog 这两个服务有关的 PID 号码

```bash
ps aux | grep '(cron|syslog)'
```

## 十五、网络通讯命令

### 1. ifconfig 命令

- ifconfig 用于查看和配置 Linux 系统的网络接口。
- 查看所有网络接口及其状态：ifconfig -a 。
- 使用 up 和 down 命令启动或停止某个接口：ifconfig eth0 up 和 ifconfig eth0 down 。

### 2. `iptables` 命令

iptables ，是一个配置 Linux 内核防火墙的命令行工具。功能非常强大，对于我们开发来说，主要掌握如何开放端口即可。例如：

- 把来源 IP 为 192.168.1.101 访问本机 80 端口的包直接拒绝：iptables -I INPUT -s 192.168.1.101 -p tcp --dport 80 -j REJECT 。
- 开启 80 端口，因为web对外都是这个端口

```bash
iptables -A INPUT -p tcp --dport 80 -j ACCEP
```

- 另外，要注意使用 iptables save 命令，进行保存。否则，服务器重启后，配置的规则将丢失。

### 3. `netstat` 命令

Linux netstat命令用于显示网络状态。

利用netstat指令可让你得知整个Linux系统的网络情况。

语法

```bash
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

参数说明：

- -a或–all 显示所有连线中的Socket。
- -A<网络类型>或–<网络类型> 列出该网络类型连线中的相关地址。
- -c或–continuous 持续列出网络状态。
- -C或–cache 显示路由器配置的快取信息。
- -e或–extend 显示网络其他相关信息。
- -F或–fib 显示FIB。
- -g或–groups 显示多重广播功能群组组员名单。
- -h或–help 在线帮助。
- -i或–interfaces 显示网络界面信息表单。
- -l或–listening 显示监控中的服务器的Socket。
- -M或–masquerade 显示伪装的网络连线。
- -n或–numeric 直接使用IP地址，而不通过域名服务器。
- -N或–netlink或–symbolic 显示网络硬件外围设备的符号连接名称。
- -o或–timers 显示计时器。
- -p或–programs 显示正在使用Socket的程序识别码和程序名称。
- -r或–route 显示Routing Table。
- -s或–statistice 显示网络工作信息统计表。
- -t或–tcp 显示TCP传输协议的连线状况。
- -u或–udp 显示UDP传输协议的连线状况。
- -v或–verbose 显示指令执行过程。
- -V或–version 显示版本信息。
- -w或–raw 显示RAW传输协议的连线状况。
- -x或–unix 此参数的效果和指定"-A unix"参数相同。
- –ip或–inet 此参数的效果和指定"-A inet"参数相同。

**如何查看系统都开启了哪些端口？**

```bash
[root@centos6 ~ 13:20 #55]# netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1035/sshd
tcp        0      0 :::22                       :::*                        LISTEN      1035/sshd
udp        0      0 0.0.0.0:68                  0.0.0.0:*                               931/dhclient
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node PID/Program name    Path
unix  2      [ ACC ]     STREAM     LISTENING     6825   1/init              @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     8429   1003/dbus-daemon    /var/run/dbus/system_bus_socket
```

**如何查看网络连接状况？**

```bash
[root@centos6 ~ 13:22 #58]# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN
tcp        0      0 192.168.147.130:22          192.168.147.1:23893         ESTABLISHED
tcp        0      0 :::22                       :::*                        LISTEN
udp        0      0 0.0.0.0:68                  0.0.0.0:*
```

**如何统计系统当前进程连接数？**

- 输入命令 netstat -an | grep ESTABLISHED | wc -l 。
- 输出结果 177 。一共有 177 连接数。

**用 netstat 命令配合其他命令，按照源 IP 统计所有到 80 端口的 ESTABLISHED 状态链接的个数？**

> 严格来说，这个题目考验的是对 awk 的使用。

首先，使用 netstat -an|grep ESTABLISHED 命令。结果如下：

```bash
tcp        0      0 120.27.146.122:80       113.65.18.33:62721      ESTABLISHED
tcp        0      0 120.27.146.122:80       27.43.83.115:47148      ESTABLISHED
tcp        0      0 120.27.146.122:58838    106.39.162.96:443       ESTABLISHED
tcp        0      0 120.27.146.122:52304    203.208.40.121:443      ESTABLISHED
tcp        0      0 120.27.146.122:33194    203.208.40.122:443      ESTABLISHED
tcp        0      0 120.27.146.122:53758    101.37.183.144:443      ESTABLISHED
tcp        0      0 120.27.146.122:27017    23.105.193.30:50556     ESTABLISHED
```

### 4. `ping` 命令

Linux ping命令用于检测主机。

执行ping指令会使用ICMP传输协议，发出要求回应的信息，若远端主机的网络功能没有问题，就会回应该信息，因而得知该主机运作正常。

指定接收包的次数

```bash
ping -c 2 www.baidu.com
```

### 5. `telnet` 命令

Linux telnet命令用于远端登入。

执行telnet指令开启终端机阶段作业，并登入远端主机。

语法

```bash
telnet [-8acdEfFKLrx][-b<主机别名>][-e<脱离字符>][-k<域名>][-l<用户名称>][-n<记录文件>][-S<服务类型>][-X<认证形态>][主机名称或IP地址<通信端口>]
```

参数说明：

- -8 允许使用8位字符资料，包括输入与输出。
- -a 尝试自动登入远端系统。
- -b<主机别名> 使用别名指定远端主机名称。
- -c 不读取用户专属目录里的.telnetrc文件。
- -d 启动排错模式。
- -e<脱离字符> 设置脱离字符。
- -E 滤除脱离字符。
- -f 此参数的效果和指定"-F"参数相同。
- -F 使用Kerberos V5认证时，加上此参数可把本地主机的认证数据上传到远端主机。
- -k<域名> 使用Kerberos认证时，加上此参数让远端主机采用指定的领域名，而非该主机的域名。
- -K 不自动登入远端主机。
- -l<用户名称> 指定要登入远端主机的用户名称。
- -L 允许输出8位字符资料。
- -n<记录文件> 指定文件记录相关信息。
- -r 使用类似rlogin指令的用户界面。
- -S<服务类型> 设置telnet连线所需的IP TOS信息。
- -x 假设主机有支持数据加密的功能，就使用它。
- -X<认证形态> 关闭指定的认证形态。

登录远程主机

```bash
# 登录IP为 192.168.0.5 的远程主机
telnet 192.168.0.5 
```
