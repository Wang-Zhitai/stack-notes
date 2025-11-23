## 4-3-2-根文件系统

一个完整的嵌入式 Linux 系统，包含 uboot，Linux 内核，根文件系统三个部分。其启动顺序为，在系统刚一上电的时候，先执行 uboot，由 uboot 引导内核，内核启动成功之后挂载根文件系统。根文件系统挂载成功以后，嵌入式 Linux 系统就启动成功了。在开发板上看到的现象是：用户可以在串口控制台上输入命令与 Linux 系统进行交互。

如何移植 uboot,如何移植 Linux 内核以及驱动程序的编写，之前已经给大家讲过。但是文件系统要如何构建这部分还没有给大家讲解，所以从这一期开始，我们来学习根文件系统相关的知识。

嵌入式 Linux 系统构成图：

![image-20240713152630010](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713152630010.png)

### 根文件系统介绍

要想讲清楚是文件系统，可以将文件系统在字面上分为俩部分来理解，一部分是文件，一部分是系统。那文件系统是不是可以理解成是组织和管理文件的一个系统呢。所以有了文件系统以后，就可以轻松的操作存储在存储介质的文件。比如删除一个文件，新增一个文件等。文件系统的格式（类型）有多种，如 fat32 ext2 ext3 ntfs 等。

#### 文件系统目录

Linux 文件系统中一般由如下的几个目录构成。这些目录用于不同的功能。

**1**  /bin **目录**

该目录下主要是存放 Linux 命令，这些可以被 root 和普通用户所使用，/bin 目录下常用的命令有：cat chgrp chmod cp ls sh kill mount umount 

**2 /sbin** **目录**

该目录下存放的是系统命令，只有系统管理员（俗称最高权限的 root）能够使用的命令，除此之外系统命令还可以存放在/usr/sbin,/usr/local/sbin 目录下。/sbin 目录下常用的命令有：shutdown reboot fdisk fsck init 等。

**3 /dev** **目录**

该目录下存放的是设备接口的文件，设备文件是 Linux 中特有的文件类型，在 Linux 系统下，一切皆文件。所以以文件的方式访问各种设备，即通过读写某个设备文件操作某个具体硬件。比如通过“/dev/ttySAC0”文件可以操作串口 0 等。

**4 /etc** **目录**

该目录下存放着系统主要的配置文件

**5 /lib** **目录**

Lib 是 library 的简称，也就是库的意思，因此此目录下存放着 Linux 所必须的库文件。

**6 /home** **目录**

系统默认的用户文件夹，它是可选的，对于每个普通用户，在/home 目录下都有一个以用户名命名的子目录，里面存放用户相关的配置文件。

**7 /root** **目录**

系统管理员（root）的主文件夹，即是根用户的目录，与此对应，普通用户的目录是/home 下的某个子目录。

**8 /usr** **目录**

usr 是 unix shared resources(共享资源)的缩写，用来存放用户的应用程序和文件。

**9 /var** **目录**

var 是 variable(变量)的缩写，/var 目录中存放可变的数据，如日志文件。

**10 /proc 目录**

这是一个空目录，常作为 proc 文件系统的挂接点

**11 /mnt** **目录**

用于临时挂载某个文件系统的挂接点，通常是空目录，也可以在里面创建一引起空的子目录，比如/mnt/cdram /mnt/hda1 。用来临时挂载光盘、移动存储设备等。

**12 /tmp** **目录**

用于存放临时文件，通常是空目录，一些需要生成临时文件的程序用到的/tmp 目录下。

#### 什么是根文件系统

根文件系统也可以组织和管理文件，所以根文件系统也是文件系统。但是为什么多了一个根字呢？因为在 Linux 系统上没有盘符这个概念,是使用一个类似大树一样的结构来组织和管理的。这颗大树的最底端就叫做根，然后每个目录都是这个树干上的子目录。

Linux 系统上是用“/”来表示根，一般也叫做根目录。在根目录下有非常多的特定目录。这些由特定目录挂载（mount）在根目录下。那这些挂载在根目录下由特定目录（特定目录介绍见 1.3 小节）构成的文件构建系统就叫做文件系统。

所以根文件系统是一个挂载在 Linux 根目录下的，有特定目录组成的文件系统。如下图所示：

![image-20240713152946001](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713152946001.png)

#### 根文件系统制作工具

最简单的根文件系统一般使用 busybox 工具制作。除此之外，还可以使用 buildroot 工具和 yocto 工具构建根文件系统。当然也可以直接使用发行版 Linux 系统，如 Ubuntu 系统，Debian系统。这些文件系统之间的界限没有很明确的界定。以下大致列出一些根文件系统的特点，如下图所示：

![image-20240713153016125](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153016125.png)

接下来我们分别来看下这些文件系统。

**Buildroot**

Buildroot 是 Linux 平台上一个开源的嵌入式 Linux 系统自动构建框架。整个Buildroot 是由 Makefile 脚本和 Kconfig 配置文件构成的。你可以通过 Buildroot 配置编译出一个完整的可以直接烧写到机器上运行的 Linux 系统软件。这里我们只是使用buildroot 编译文件系统。

![image-20240713153048641](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153048641.png)

Buildroot 优势：

1 通过源码构建，有很大的灵活性

2 方便的交叉编译环境，可以进行快速构建

3 方便各系统组件配置及定制开发

**Yocto**

Yocto 和 Buildroot 一样, 是一套构建嵌入式系统的工具,但是两者的风格完全不同。Yocto project 是通过一个个单独的包(meta)来维护,比如有的包负责核心,有的包负责外围。有的包用于跑 Rockchip 的芯片,有的包用于安装上 Qt。

![image-20240713153136934](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153136934.png)

Yocto 的优缺点：

1 支持的架构多：ARM MIPS PPC 和其他架构。

2 通过 QEMU 完全支持各种设备仿真

3 灵活性： 公司以多种不同的方式使用 Yocto 项目。一个示例是创建内部 Linux 发行版，作为公司可以跨多个产品组使用的代码库。通过自定义和分层，项目组可以利用基础 Linux 发行版创建适合其产品需求的分发。

**Ubuntu**

Ubuntu 是一个流行的 Linux 发行版，是基于 Debian 的 unstable 版本加强而来，以“最好的 Linux 桌面系统”而闻名，近些年 Ubuntu 也推出了 Ubuntu Enterprise Linux, 在企业Linux 应用市场占有率也有较大提高。

优点：技术支持较好，用户界面友好，硬件的兼容性好，采用基于 Deb 的 ATP 包管理系统。

缺点：技术支持和更新服务是需要付费的, 服务器软件生态系统的规模和活力方面稍弱。

**Debian**

Debian GNU / Linux 是一款是由 GPL 和其他自由软件许可协议授权的自由软件组成的Linux 操作系统，由 Debian Project 组织维护。以其坚守 Unix 和自由软件的精神，以及其给予用户的众多选择而闻名。

优点：Debian 是极为精简而稳定的 Linux 发行版，有着干净的作业环境，采用基于 Deb的 ATP 包管理系统。

缺点：不提供专门技术支持，不包含封闭源代码软件；发行周期过长，稳定版本中软件过时。中文支持不是很完善。

### Busybox 制作最小文件系统

#### Busybox 工具简介

在制作文件系统的时候，可以使用“Busybox 工具”.

“BusyBox 工具”是一个集成了一百多个最常用 Linux 命令和工具的软件。BusyBox 包含了一些简单的工具，例如 ls、cat 和 echo 命令等等，还包含了一些更大、更复杂的工具，例 grep、find、mount 以及 telnet 命令。有些人将 BusyBox 称为 Linux 工具里的“瑞士军刀”。简单的说 BusyBox 就好像是个大工具箱，它集成压缩了 Linux 的许多工具和命令，也包含了 Android 系统自带的 shell。

“Busybox 工具”的官方网址是“http://www.busybox.net/”，这是一个开源的程序，并且一直在更新中，我们使用的版本是“busybox-1.33.1.tar.bz2”。下面我们来讲解一下如何使用 BusyBox 制作最小文件系统。

#### 设置支持中文

首先拷贝“busybox-1.33.1.tar.bz2”到我们的虚拟机 Ubuntu 系统上，然后在 Ubuntu 命令行，执行“ tar -xvf busybox-1.33.1.tar.bz2”解压命令。

解压完成后，使用“ cd busybox-1.33.1”命令进入到前面解压出来的“busybox-1.33.1”文件夹中。

从 busybox1.17.0 以上之后，对 ls 命令不做修改是无法显示中文的。就算是内核设置了支持中文的话，在 shell 下用 ls 命令也是无法显示中文的，这是因为 busybox1.17.0 以后版本对中文的支持进行了限制。要想让 busybox1.17.0 以上支持中文，需做如下修改。

首先使用以下命令对 printable_string.c 文件进行修改，如下图所示：

> vim libbb/printable_string.c

![image-20240713153511708](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153511708.png)

然后查找函数 printable_string，我们可以看到在第 31,32 行和 45 行的操作，大于 0x7F的字符直接被 break 掉，或者直接被“？”代替了。所以就算是 linux 内核设置了支持中文，也是无法显示出来的。所以我们对这三行进行修改，注释掉对大于 0x7f 字符的相关操作。如图所示：

![image-20240713153533587](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153533587.png)

修改之后保存退出。然后使用命令“vim libbb/unicode.c”，对 unicode.c 文件进行修改，如下图所示：

![image-20240713153558658](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153558658.png)

打开之后查找函数 unicode_conv_to_printable2，我们可以看到 1022 行和 1030 行也对大于0x7f 的字符有对应操作，同样的我们对这两处做修改，如图所示，修改之后保存退出。

![image-20240713153619632](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153619632.png)

经过以上修改之后，源码就可以支持中文字符了，但还需要配置 busybox 来使能 unicode码，输入“make menuconfig”命令，进入“Busybox”的配置界面。依次选择如下配置项：

> Settings -->
>
> Support Unicode
>
> Check $LC_ALL, $LC_CTYPE and $LANG environment variables

![image-20240713153649020](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153649020.png)

接着使用左右反方向按键，选择“Exit”，回到“Busybox Configuration”界面。然后使用“->”“<-”按键，选择“Exit”，选择“Yes”保存配置

经过以上配置最小系统就可以支持中文了。

#### 配置 busybox

接下来对 Busybox 进行编译配置，输入“ make menuconfig”命令，如下图，出现“Busybox” 的配置界面。

![image-20240713153733485](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153733485.png)

进入“Busybox”的配置界面。依次选择如下配置项：

> Settings -->
>
> Build Options
>
> Cross Compiler prefix

进入“Cross Compiler prefix”配置界面，如下图所示：

![image-20240713153808576](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153808576.png)

如下图，输入交叉编译工具。注意！ 交叉编译器的设置需要根据使用的开发板进行设置， RK3568 开发板，需要设置 RK3568 的交叉编译器，则输入“aarch64-linux-gnu-”。

![image-20240713153831791](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713153831791.png)

按“回车”返回到“Setting”配置界面，这时可以看到刚才我们设置的交叉编译工具.

选中“Installation Options”配置项,设置安装路径为../system

选中“Exit”按“回车”继续退出，直到弹出以下界面，选择“yes”保存退出，如下图所示。至此我们的 busybox 就设置好了。

#### 编译 busybox

保存退出之后，我们需要设置临时环境变量如下图所示：

> export PATH=/usr/local/arm64/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin:$PATH

执行“make”进行编译，编译完成，接下来我们需要把编译生成的“二进制文件”安装到刚才我们指定的“../system”目录里面，安装二进制文件到“../system”目录，如下图，输入命令“ make install”。完成后如下图所示：

![image-20240713154022015](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713154022015.png)

然后使用“ cd ../system”命令，进入“../system”目录，查看里面安装的文件，如下图所示

![image-20240713154035004](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713154035004.png)

至此我们由 busybox 制作的文件系统就制作好了。但是还需要一些步骤来对该文件系统进行完善。

#### 完善最小根文件系统

##### 创建必要文件夹

文件系统还需要新建“dev,etc,lib,mnt,proc,sys,tmp,var”文件夹

> mkdir dev etc lib mnt proc sys tmp var etc/init.d

##### 拷贝 lib 库文件

Busybox 编译生成的二进制文件是以动态链接库的形式运行，所以我们需要拷贝编译器里面的库文件到“lib”目录，使用以下命令：

> cp -r  gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/aarch64-linux-gnu/libc/lib/* . /lib/

##### 创建 rcS 文件

然后我们需要创建/etc/init.d/rcS 文件 ，在 system 目录下使用“vim etc/init.d/rcS”命令建立“rcS”文件，并在“rcS”文件输入下面的内容。

```shell
#! /bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin:$PATH
runlevel=S
prevlevel=N
umask 022
export PATH runlevel prevlevel
/bin/hostname woniu
mount -a
mkdir /dev/pts
mount -t devpts devpts /dev/pts
echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s
mkdir -p /var/empty
mkdir -p /var/log
mkdir -p /var/log/boa
mkdir -p /var/lock
mkdir -p /var/run
mkdir -p /var/tmp
syslogd
/etc/rc.d/init.d/netd start
mkdir /mnt/disk
sleep 1
echo "*************************************" >/dev/ttyFIQ0
echo "http://woniuxy.com" > /dev/ttyFIQ0
echo "*************************************" > /dev/ttyFIQ0
```

下面我们来分析每一句命令的作用。

> 第 1 行表示该文件为一个 shell 脚本。
>
> 第 2 行和后面的第 6 行指定了 PATH 环境变量，PATH 这个环境变量是 linux 系统内部定义的一个环境变量，含义是操作系统去执行程序时会默认到 PATH 指定的各个目录下去寻找。如果找不到就认定这个程序不存在，如果找到了就去执行它。将一个可执行程序的目录导出到 PATH，可以让我们不带路径来执行这个程序。
>
> rcS 中为什么要先导出 PATH？就是因为我们希望一旦进入命令行下时，PATH 环境变量中就有默认的/bin /sbin /usr/bin /usr/sbin 这几个常见的可执行程序的路径，这样我们进入命令行后就可以 ls、cd 等直接使用了。
>
> 第 3 行和第 6 行指定了 runlevel 环境变量，runlevel=S 表示将系统设置为单用户模式。
>
> 第 5 行指定当前用户在创建文件时的默认权限。
>
> 第 6 行导出环境变量
>
> 第 7 行设置机器名字为 iTOP-RK3568
>
> 第 8 行 mount -a 是挂载所有的应该被挂载的文件系统，在 busybox 中 mount -a 时 busybox会去查找一个文件/etc/fstab 文件，这个文件按照一定的格式列出来所有应该被挂载的文件系统（包括了虚拟文件系统），在这里我们还没有创建，下面会进行创建。
>
> 第 9 和 10 行，创建目录/dev/pts，然后将 devpts 挂载到/dev/pts 目录中。
>
> 第 11 和 12 行，使用 mdev 来管理热插拔设备，Linux 内核就可以在/dev 目录下自动创建设备节点. 
>
> 第 13 行到 18 行，在对应的目录下创建相应的文件。
>
> 第 19 行 syslogd 记录系统或应用程序产生的各种信息，并把信息写到日志中
>
> 第 20 行启动 netd 服务，我们在之后会创建这个文件
>
> 第 21 行在 mnt 下创建 disk 文件夹，方便外部设备（U 盘，TF 卡）的挂载。
>
> 第 22 行设置本地回环 lo 为 127.0.0.1。
>
> 第 23 行执行 ifconfig-eth0 脚本配置网络，在下面的步骤中会创建这个脚本。

修改完成之后，我们需要使用命令“chmod 777 rcS”，来给予文件可执行权限。

##### 创建 fstab 文件

文件 fstab 包含了你的电脑上的存储设备及其文件系统的信息。它是决定一个硬盘（分区）被怎样使用或者说整合到整个系统中的唯一文件。

这个文件的全路径是/etc/fstab。它只是一个文本文件，你能够用你喜欢的编辑器打开它，但是必须是 root 用户才能编辑它。同时 fsck、mount、umount 的等命令都利用该程序。

具体来说： 用 fstab 可以自动挂载各种文件系统格式的硬盘、分区、可移动设备和远程设备等。 对于 Windows 与 arch 双操作系统用户，用 fstab 挂载 FAT 格式和 NTFS 格式的分区，可以在 Linux 中共享 windows 系统下的资源。

让我们对 fstab 的用法进行一个详细的了解。

/etc/fstab 由下面的 fields 组成 (fields 之间以空格或 tab 分开):

> <file system> <dir> <type> <options> <dump> <pass>

> <file systems> - 存储设备的标识 (i.e. /dev/sda1). 
>
> <dir> - 告诉 mount 命令应该将文件设备挂载到哪里。
>
> <type> - 定义了要挂载的设备或是分区的文件系统类型，支持许多种不同的文件系统，如ext2, ext3, reiserfs, xfs, jfs, smbfs, iso9660, vfat, ntfs, swap 以及 auto。 'auto' 类型使 mount 命令对这文件系统类型进行猜测，这对于如 CDROM 和 DVD 之类的可移动设备是非常有用的。
>
> <options> - 定义了不同文件系统的特殊参数，不同文件系统的参数不尽相同。其中一些比较通用的参数有以下这些：
>
> <dump> dump utility 用来决定何时作备份. 安装之后 ( ArchLinux 默认未安装 ), dump会检查其内容，并用数字来决定是否对这个文件系统进行备份。允许的数字是 0 和 1 。0 表示忽略， 1 则进行备份。大部分的用户是没有安装 dump 的 ，对他们而言 <dump> 应设为0。
>
> <pass> fsck 读取 <pass> 的数值来决定需要检查的文件系统的检查顺序。允许的数字是 0, 1, 和 2。根目录应当获得最高的优先权 1, 其它所有需要被检查的设备设置为 2. 0 表示设备不会被 fsck 所检查。

在对 fstab 有了一定了解之后，我们使用命令“vim etc/fstab”创建 fstab 文件，并写入以下内容：

> \#<file system> <dir> <type> <options> <dump> <pass>
>
> proc /proc proc  defaults 0 0
>
> tmpfs /tmp tmpfs defaults 0 0
>
> sysfs /sys sysfs defaults 0 0

![image-20240713154649816](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713154649816.png)

修改完成之后，我们需要使用命令“chmod 777 fstab”，来给予文件可执行权限.

##### 创建 inittab 文件

init 进程为内核加载文件系统的第一个进程，一般称为 0 号进程，init 进程启动后会自动读取 inittab。

inittab 文件中每一行的格式如下所示：（busybox 的根目录下的 example 文件夹下有详尽的inittab 文件范例）

> <id>:<runlevel>:<action>:<process>

尽管此格式与传统的 Sytem V init 类似，但是，id 在 BusyBox 的 init 中具有不同的意义。对 BusyBox 而言，id 用来指定启动进程的控制 tty。如果所启动的进程并不是可以交互的 shell，例如 BusyBox 的 sh（ash），应该会有个控制 tty，如果控制 tty 不存在，Busybox 的 sh 会报错。BusyBox 将会完全忽略 runlevel 字段，所以空着它就行了，你也许会问既然没用保留着它干吗，我想大概是为了和传统的 Sytem V init 保持一致的格式吧。process 字段用来指定所执行程式的路径，包括命令行选项。action 字段用来指定下面表中 8 个可应用到 process 的动作之一。

> **sysinit**   为 init 提供初始化命令行的路径
>
> **respawn**   每当相应的进程终止执行便会重新启动
>
> **askfirst**     类似 respawn，不过它的主要用途是减少系统上执行的终端应用程序的数量。它将会促使 init 在控制台上显示“Please press Enter to active this console ”的信息，并在重新启动之前等待用户按下 enter 键
>
> **wait** 告诉 init 必须等到相应的进程完成之后才能继续执行
>
> **once** 仅执行相应的进程一次，而且不会等待它完成
>
> **ctraldel** 当按下 Ctrl+Alt+Delete 组合键时，执行相应的进程
>
> **shutdown** 当系统关机时，执行相应的进程
>
> **restart** 当 init 重新启动时，执行相应的进程，通常此处所执行的进程就是 init 本身

文件根据此配置文件来执行后续的步骤。我们在/etc 目录下使用命令“vim etc/inittab”命令，创建 inittab 文件，在该文件中我们输入以下内容：

```shell
::sysinit:/etc/init.d/rcS
console::askfirst:-/bin/sh
::restart:/sbin/init
::shutdown:/bin/umount -a -r
::shutdown:/sbin/swapoff -a
```

第 1 行，系统启动以后运行/etc/init.d/rcS 这个脚本文件。

第 2 行，将 console 作为控制台终端，也就是 ttySTM0。

第 3 行，重启的话运行/sbin/init。

第 4 行，关机的时候执行/bin/umount，也就是卸载各个文件系统。

第 5 行，关机的时候执行/sbin/swapoff，也就是关闭交换分区。

保存退出之后，使用命令“chmod 777 inittab”，修改该文件的权限

##### 创建 passwd 文件

然后使用“ vim etc/passwd”命令在“etc”目录建立“passwd”文件，passwd 用来是用来保存用户的信息的，在“passwd”文件中输入下面的内容。

```shell
root:x:0:0:root:/:/bin/sh
bin:*:1:1:bin:/bin:
daemon:*:2:2:daemon:/sbin:
nobody:*:99:99:Nobody:/:
```

passwd 文件是用来保存用户的信息的，是一个纯文本文件。

> name：用户名。
>
> password：密码（已经加密）
>
> uid：UID（用户标识）,操作系统自己用的
>
> gid：GID 组标识。
>
> comment：用户全名或本地帐号
>
> home：用户的主目录的绝对路径
>
> shell：登录使用的 Shell，就是对登录命令进行解析的工具

保存退出之后，使用命令“chmod 777 passwd”，修改该文件的权限

##### 创建 profile 文件

然后使用“ vim etc/profile”命令在“etc”目录建立“profile”文件，profile 用来设置我们的环境变量，在“profile”文件中输入下面的内容。

```
USER="`id -un`" LOGNAME=$USER
PS1='[$USER@$HOSTNAME]:$PWD# '
PATH=$PATH
HOSTNAME=`/bin/hostname` 
export USER LOGNAME PS1 PATH
```

保存并退出“profile”文件，使用“chmod 777 profile”命令修改“profile”文件的权限

##### 创建 eth0-setting 和 eth1-setting 文件

在 etc 目录下，使用命令“vim eth0-setting”，创建 eth0-setting 文件，并输入以下内容：

```
IP=192.168.1.230
Mask=255.255.255.0
Gateway=192.168.1.1
DNS=192.168.1.1
MAC=08:90:90:90:90:90
```

然后在 etc 目录下，使用命令“vim eth1-setting”，创建 eth1-setting 文件，并输入以下内容：

```
IP=192.168.1.231
Mask=255.255.255.0
Gateway=192.168.1.1
DNS=192.168.1.1
MAC=BE:C8:1D:14:82:1D
```

> IP：开发板启动后默认设置的 IP 地址，IP 地址为 192.168.1.230 和 192.168.1.231
>
> Mask：开发板启动后默认设置的掩码，掩码为 255.255.255.0
>
> Gateway：开发板启动后默认配置的网关，网关为 192.168.1.1
>
> DNS：开发板启动后默认设置的 DNS，DNS 为 192.168.1.1
>
> MAC：开发板启动后默认设置的 MAC 地址，MAC 地址为 08:90:90:90:90:90
>
> MAC 地址和作者保持一致即可，其余网络信息大家依据自己的网络情况设置即可，

保存并退出“eth0-setting”文件，使用“chmod 777 eth0-setting”命令修改“profile”文件的权限

##### 创建 ifconfig-eth0 和 ifconfig-eth1 文件

在/etc 目录下使用“cd init.d”命令进入 init.d 文件夹

然后使用命令“vim ifconfig-eth0”创建 ifconfig-eth0 文件，并写入以下内容

```shell
#!/bin/sh
echo -n Try to bring eth0 interface up......>/dev/ttyFIQ0
if [ -f /etc/eth0-setting ] ; then
  source /etc/eth0-setting
  if grep -q "^/dev/root / nfs " /etc/mtab ; then
    echo -n NFS root ... > /dev/ttyFIQ0
  else
      ifconfig eth0 down
      ifconfig eth0 hw ether $MAC
      ifconfig eth0 $IP netmask $Mask up
      route add default gw $Gateway
	fi
	echo nameserver $DNS > /etc/resolv.conf
else
  if grep -q "^/dev/root / nfs " /etc/mtab ; then
  	echo -n NFS root ... > /dev/ttyFIQ0
  else
  	/sbin/ifconfig eth0 192.168.253.12 netmask 255.255.255.0 up
  fi
fi
echo Done >/dev/ttyFIQ0
```

然后使用命令“vim ifconfig-eth1”创建 ifconfig-eth1 文件，并写入以下内容：

```shell
#!/bin/sh
echo -n Try to bring eth0 interface up......>/dev/ttyFIQ0
if [ -f /etc/eth1-setting ] ; then
  source /etc/eth1-setting
  if grep -q "^/dev/root / nfs " /etc/mtab ; then
    echo -n NFS root ... > /dev/ttyFIQ0
  else
    ifconfig eth1 down
    ifconfig eth1 hw ether $MAC
    ifconfig eth1 $IP netmask $Mask up
    route add default gw $Gateway
  fi
  echo nameserver $DNS > /etc/resolv.conf
else
  if grep -q "^/dev/root / nfs " /etc/mtab ; then
    echo -n NFS root ... > /dev/ttyFIQ0
  else
    /sbin/ifconfig eth0 192.168.253.12 netmask 255.255.255.0 up
  fi
fi
echo Done >/dev/ttyFIQ0
```

ifconfig-eth0 和 ifconfig-eth1 是网络配置文件，工作流程就是读取我们创建的 eth0-setting和 eth1-setting 网络信息文件来设置网络，如果 eth0-setting 或者 eth1-setting 配置文件不存在，就使用默认配置。

保存退出之后，使用命令“chmod 777 ifconfig-eth0 ifconfig-eth1“，修改文件的权限

##### 创建 medv.conf 文件

在 etc 目录下，使用命令“vim mdev.conf”新建 mdev.conf 文件,因为我们在 rcS 文件里面配置了热拔插功能，但是还需要 mdev.conf 配合，mdev.conf 文件是 U 盘 sd 卡的热拔插文件，我们在 mdev.conf 输入以下内容：

```
sd[a-z][0-9] 0:0 666 @ /etc/hotplug/udisk_inserting

sd[a-z] 0:0 666 $ /etc/hotplug/udisk_removing
```

接着使用命令“mkdir hotplug”在 etc 目录下创建 hotplug 文件夹.

然后使用命令“cd hotplug”进入 hotplug 文件夹，然后在 hotplug 文件夹下使用命令“vim udisk_inserting”建立 udisk_inserting 文件夹并在里面输入以下内容：

```shell
#!/bin/sh
echo "usbdisk insert!" > /dev/console
if [ -e "/dev/$MDEV" ] ; then
  mkdir -p /mnt/usbdisk/$MDEV
  mount /dev/$MDEV /mnt/usbdisk/$MDEV
  echo "/dev/$MDEV mounted in /mnt/usbdisk/$MDEV">/dev/console
fi
```

保存退出后，在 hotplug 文件夹下使用命令“vim udisk_removing”建立 udisk_removing 文件夹并在里面输入以下内容：

```shell
#!/bin/sh
echo "usbdisk remove!" > /dev/console
umount -l /mnt/usbdisk/sd*
rm -rf /mnt/usbdisk/sd*
```

保存退出后，在 hotplug 目录下使用命令“chmod 777 *”修改文件权限。

##### 创建 mtab 文件

返回 etc 目录下，我们需要在 etc 目录下创建 mtab 文件，/etc/mtab 是当前的分区挂载情况，记录的是当前系统已挂载的分区。每次挂载/卸载分区时会更新/etc/mtab 文件中的信息。所以在 ifconfig-eth0 文件里面我们就是通过这个/etc/mtab 来判断当前的文件系统是不是 nfs 网络文件系统。

mtab 文件是个链接文件，我们在 etc 目录下输入命令“ln -s /proc/mounts mtab”把他链接到/proc/mounts

##### 创建 netd 文件

netd 文件是用用来配合 profile 文件里面的/etc/rc.d/init.d/netd start 的这行代码来启动 netd服务的。这里我们就来一起创建这个文件。

（1）在 etc 目录使用命令“mkdir rc.d”建立文件夹“rc.d”然后使用“cd rc.d” 命令进入到刚才建立的“rc.d” 文件夹 ，接下来在“rc.d” 目录下使用“mkdir init.d” 命令建立“init.d” 文件夹， 使用“cd init.d” 命令进入到刚才建立的“init.d” 文件夹。

（2）在“ init.d”目录下， 使用“vi netd” 命令建立“ netd” 文件，然后使用“chmod 777 netd” 命令修改“ netd” 文件的权限为 777。

```shell
#!/bin/sh
base=inetd
# See how we were called.
case "$1" in
  start)
  	/usr/sbin/$base
  ;;
  stop)
    pid=`/bin/pidof $base`
    if [ -n "$pid" ]; then
    	kill -9 $pid
    fi
  ;;
esac
exit 0
```

#### 构建文件系统镜像

返回上一目录，使用命令“mkdir rootfs”,   rootfs 用来挂载之后制作制作出来的 rootfs.ext4，

然后使用命令“du system”命令来查看 system 文件大小，可以看到该文件系统的大小为18M，如下图所示：

![image-20240713160257295](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713160257295.png)

所以我们镜像大小在这里设置为 30M，使用命令如下

> dd if=/dev/zero of=rootfs.ext4 bs=1M count=30
>
> mkfs.ext4 -L rootfs rootfs.ext4

建立一个大小为 30M 的 ext4 磁盘

接下来使用 mount 命令将 rootfs.ext4 挂载到 rootfs 目录下

> mount ./rootfs.ext4 rootfs

然后拷贝我们制作的文件系统到 rootfs 目录下

> cp -r ./system/* ./rootfs

拷贝完成后，使用命令“umount rootfs”，进行解除挂载

> umount rootfs

最后将 rootfs.ext4 文件拷贝成 rootfs.img

> mv rootfs.ext4 rootfs.img

![image-20240713160421343](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713160421343.png)

接一来烧录运行测试。

### 最小文件系统移植 QT 库

我们使用 busybox 工具制作了一个最简单的根文件系统，如果我们想要在此基础上运行 Qt 程序，我们需要移植 QT 库，本章节将一步步移植 Qt 库，并运行 QT 程序。

#### 编译触摸驱动

使用命令“tar -vxf tslib-1.4.tar.gz”解压源码 tslib-1.4.tar.gz，然后执行如下命令。

```
cd tslib
./configure CC=aarch64-linux-gnu-gcc CXX=aarch64-linux-gnu-g++ --host=aarch64-linux-gnu --prefix=/opt/tslib1.4 ac_cv_func_malloc_0_nonnull=yes
make
make install
```

上述命令用于指定编译器 aarch64-linux-gnu，和生成文件的路径 /opt/tslib1.4。

然后编译和安装，执行完成后使用命令

> vim /opt/tslib1.4/etc/ts.conf

修改第二行如下

![image-20240713162236894](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713162236894.png)

注意：（ts.conf 文件里边的每条语句最前边不能有空格。）

保存，退出。

#### 编译 QtE5.15 库

使用如下命令解压 QtE5.15.2 源码，并进入解压生成“qt-everywhere-opensource-src-5.15.2” 目录。

> tar -vxf qt-everywhere-src-5.15.2.tar.xz

在“qt-everywhere-src-5.15.2”目录下，使用如下命令，打开 qmake.conf 文件。

> vi qtbase/mkspecs/linux-arm-gnueabi-g++/qmake.conf

并将 qmake.conf 修改为如下所示内容。

```
#
# qmake configuration for building with arm-linux-gnueabi-g++
#
MAKEFILE_GENERATOR = UNIX
CONFIG += incremental
QMAKE_INCREMENTAL_STYLE = sublib
include(../common/linux.conf)
include(../common/gcc-base-unix.conf)
include(../common/g++-unix.conf)
QT_QPA_DEFAULT_PLATFORM = linuxfb
QMAKE_CFLAGS_RELEASE += -O2 -march=armv8-a -lts
QMAKE_CXXFLAGS_RELEASE += -O2 -march=armv8-a -lts
# modifications to g++.conf
QMAKE_CC = aarch64-linux-gnu-gcc
QMAKE_CXX = aarch64-linux-gnu-g++
QMAKE_LINK = aarch64-linux-gnu-g++
QMAKE_LINK_SHLIB = aarch64-linux-gnu-g++
# modifications to linux.conf
QMAKE_AR = aarch64-linux-gnu-ar cqs
QMAKE_OBJCOPY = aarch64-linux-gnu-objcopy
QMAKE_NM = aarch64-linux-gnu-nm -P
QMAKE_STRIP = aarch64-linux-gnu-strip
load(qt_config)
```

然后保存退出，使用“vi autoconfigure.sh”命令，新建“autoconfigure.sh”脚本。脚本内容如下所示。

```shell
#!/bin/sh
./configure \
-v \
-prefix /opt/qt5.15.2 \
-confirm-license \
-opensource \
-release \
-xplatform linux-aarch64-gnu-g++ \
-optimized-qmake \
-c++std c++11 \
--rpath=no \
-pch \
-skip qt3d \
-skip qtactiveqt \
-skip qtandroidextras \
-skip qtcanvas3d \
-skip qtconnectivity \
-skip qtdatavis3d \
-skip qtdoc \
-skip qtgamepad \
-skip qtlocation \
-skip qtmacextras \
-skip qtnetworkauth \
-skip qtpurchasing \
-skip qtremoteobjects \
-skip qtscript \
-skip qtscxml \
-skip qtsensors \
-skip qtspeech \
-skip qtsvg \
-skip qttools \
-skip qttranslations \
-skip qtwayland \
-skip qtwebengine \
-skip qtwebview \
-skip qtwinextras \
-skip qtx11extras \
-skip qtxmlpatterns \
-make libs \
-make examples \
-nomake tools -nomake tests \
-gui \
-widgets \
-dbus-runtime \
--glib=no \
--iconv=no \
--pcre=qt \
--zlib=qt \
-no-openssl \
--freetype=qt \
--harfbuzz=qt \
-no-opengl \
-linuxfb \
--xcb=no \
--libpng=qt \
--libjpeg=qt \
--sqlite=qt \
-plugin-sql-sqlite \
-I/opt/tslib1.4/include \
-L/opt/tslib1.4/lib \
-recheck-all
exit
```

保存脚本，退出。使用命令“chmod 777 autoconfigure.sh”修改脚本权限，然后使用命令“./autoconfigure.sh”执行脚本，

配置完成之后，接着使用编译命令“make -j12”，

然后使用安装命令“make install”。

完成后，可以在“/opt”目录下可以查看到生成的“qt5.15.2”文件

#### 生成文件系统

进入 文件系统 system 目录后使用命令”mkdir opt”，新建文件夹 opt ，

将刚才编译生成的“qt5.15.2”文件和“tslib1.4”触摸文件，拷贝到“/home/自己制作的文件系统目录/opt”目录下。

然后将字库文件“fonts”拷贝到“opt/qt5.15.2/lib/”目录下，并使用 “unzip -o -d ./ fonts.zip”解压。

![image-20240713162834117](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713162834117.png)

将“lib.tar.gz”拷贝到文件系统的“lib”目录下解压

在 system 目录下，使用以下命令，修改环境变量

> vim etc/profile

在/etc/profile 中添加 tslib 和 qt 的环境变量，添加完成如下所示：

```shell
# Ash profile
# vim: syntax=sh
# No core files by default
ulimit -S -c 0 > /dev/null 2>&1
USER="`id -un`"
LOGNAME=$USER
PS1='[$USER@$HOSTNAME]:$PWD# ' 
PATH=$PATH
HOSTNAME=`/bin/hostname` 
export USER LOGNAME PS1 PATH
EVENT=$(cat /proc/bus/input/devices | grep -E 'ft5x06|goodix-gt911' -A4 | tail -n1 | head -c 95 | cut -c13-18)
export TSLIB_ROOT=/opt/tslib1.4
export QT_ROOT=/opt/qt5.15.2
export TSLIB_TSDEVICE=/dev/input/$EVENT
export TSLIB_TSEVENTTYPE=input
export TSLIB_CONFFILE=/opt/tslib1.4/etc/ts.conf
export TSLIB_PLUGINDIR=/opt/tslib1.4/lib/ts
export TSLIB_CONSOLEDEVICE=none
export TSLIB_FBDEVICE=/dev/fb0
export LD_PRELOAD=$TSLIB_ROOT/lib/libts.so
export QWS_MOUSE_PROTO=tslib:/dev/input/$EVENT
export LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib:$QT_ROOT/lib:$TSLIB_ROOT/lib:$TSLIB_ROOT/lib/
export QT_QPA_PLATFORM_PLUGIN_PATH=$QT_ROOT/plugins
export QT_QPA_PLATFORM=linuxfb:tty=/dev/fb0
export QT_QPA_FONTDIR=$QT_ROOT/lib/fonts
export QT_QPA_GENERIC_PLUGINS=tslib
```

下面我们对该内容进行部分解释

自动获取触摸的设备节点，变量 EVENT 的值可能是 event0，event1.......等。

> EVENT=$(cat /proc/bus/input/devices | grep -E 'ft5x06|goodix-gt911' -A4 | tail -n1 | head -c 95 | cut -c13-18)

触摸库所在的位置

> export TSLIB_ROOT=/opt/tslib1.4

触摸屏设备文件

> export TSLIB_TSDEVICE=/dev/input/$EVENT

tslib 模块配置文件

> export TSLIB_CONFFILE=/opt/tslib1.4/etc/ts.conf

指定触摸屏校准文件 pintercal 的存放位置

> export TSLIB_CALIBFILE=/etc/pointercal

设定控制台设备为 none ，否则默认为 /dev/tty ，这样可以避免出现“ open consoledevice: No such file or directory KDSETMODE: Bad file descriptor ” 的错误

> export TSLIB_CONSOLEDEVICE=none

qt 库所在的路径

> export QT_ROOT=/opt/qt5.15.2

qt 字库的目录

> export QT_QPA_FONTDIR=$QT_ROOT/lib/fonts

qt 插件的目录

> export QT_QPA_PLATFORM_PLUGIN_PATH=$QT_ROOT/plugins

指定帧缓冲设备/dev/fb0

> export QT_QPA_PLATFORM=linuxfb:fb=/dev/fb0

添加 QT 和触摸库的环境变量

> export LD_LIBRARY_PATH= /lib:/usr/lib:/usr/local/lib:$QT_ROOT/lib:$TSLIB_ROOT/lib/

保存退出，使用命令“du system”命令来查看 system 文件大小，可以看到该文件系统的大小为 505M，

![image-20240713163310383](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713163310383.png)

所以我们镜像大小在这里设置为 700M，使用命令如下：

> dd if=/dev/zero of=rootfs.ext4 bs=1M count=700
>
> mkfs.ext4 -L rootfs rootfs.ext4

建立一个大小为 700M 的 ext4 磁盘

![image-20240713163353614](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713163353614.png)

然后我们使用命令“mkdir rootfs”,rootfs 用来挂载之前制作制作出来的 rootfs.ext4，

接下来使用 mount 命令将 rootfs.ext4 挂载到 rootfs 目录下

> mount ./rootfs.ext4 rootfs

然后拷贝我们制作的文件系统到 rootfs 目录下，

> cp -r ./system/* ./rootfs

拷贝完成后，使用命令“umount rootfs”，进行解除挂载，

最后将 rootfs.ext4 文件拷贝为 rootfs.img,

#### 编译运行 Qt 程序

> //因为官方提供的这个QT程序是32位的，所以需要做一下兼容处理
>
> chmod 777 install.sh
>
> ./install.sh
>
> /opt/qt5.15.2/bin/qmake

然后输入以下命令编译 QT 程序

> make

编译完生成可执行程序 QLed，

将编译生成的可执行文件 QLed 拷贝到开发板上运行，屏幕上显示 QT 程序

![image-20240713163641482](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713163641482.png)

### Buildroot 系统构建

在上一个章节我们学习了使用 busybox 来构建文件系统，但是 busybox 构建的文件系统很多东西需要自己交叉编译添加，某些软件需要自己去移植，所以移植过程是非常繁琐的。本章节我们来学习另外一种非常实用的根文件系统构建方法——使用 buildroot 来构建根文件系统。buildroot 不仅集成了 busybox，而且还集成了各种常见的第三方库和软件，需要什么软件，选择哪个软件就好，buildroot 上手门槛低，极大的方便了嵌入式 Linux 开发人员构建根文件系统。

#### buildroot 基本介绍

Buildroot 是一个可以使用交叉编译简单且自动化地为嵌入式系统构建完整 Linux 系统的工具。Buildroot 可以为你的系统生成交叉编译工具链，根文件系统，Linux 内核镜像和引导加载程序。Buildroot 可以独立地用于这些选项的任意组合（比如，你可以使用现有的交叉编译工具链，并且仅仅只编译文件系统）。

Buildroot 主要用于使用嵌入式系统的开发者，嵌入式系统通常使用的处理器并不是 PC 电脑上的X86处理器，他们可以是PowerPc处理器，MIPS处理器或者ARM处理器等等。Buildroot支持很多处理器，它还具有一些可用目标板的默认配置。除此之外，许多第三方项目的 SDK都是基于 buildroot 开发。

构建开源软件包工作流程大致如下所示：

> 1 获取：获取软件包的源码
>
> 2 解压：解压软件包源码
>
> 3 补丁：针对缺陷修复和增加的功能应用补丁
>
> 4 配置：根据环境准备构建过程
>
> 5 安装：复制二进制和辅助文件到目标目录
>
> 6 打包：为在其他系统上安装而打包二进制和辅助文件

如上所示，我们可以看到构建每个软件包的工作流程几乎是相同的，buildroot 主要的功能就是把这些重复操作自动化，用户只需要勾选上所需软件包，便自动完成以上所有操作。Buildroot 生成 uboot Linux Kernel 的编译工作流程是差不多的，只要配置好自定义的参数，在buildroot 配置好编译生成即可。

#### 获取 buildroot 官方源码

Buildroot 每隔三个月发布一次版本，2 月 5 月 8 月 11 月各发布一次。版本号格式是YYYY.MM，比如 2019.02、2019.08 等等。

使用浏览器打开以下网址下载 buildroot 最新的源码。

https://buildroot.org/download.html

本教程中使用的 buildroot 版本是 Buildroot-2022.05.tar.bz2，建议大家和本教程保持一致。

点击 buildroot-2022.05.tar.bz2 进行下载。下载之后使用 SSH 拷贝到 ubuntu 虚拟机中。

输入以下命令解压压缩包：

我们进入 buildroot-2022.05 目录，

![image-20240713164058507](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164058507.png)

#### buildroot 目录简介

Buildroot 基本上是一组 Makefile 文件，可以使用正确的选项对所需的软件进行下载，配置和编译。它还包含各种软件包的补丁，主要是那些涉及交叉编译工具链（gcc,bin utils 和uClibc）的软件包。每个软件包基本只有一个 Makefile 文件，它们以.mk扩展名命名。

Buildroot 编译之前的目录如下所示：

![image-20240713164142964](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164142964.png)

Buildroot 编译之后会多出来俩个文件夹，一个是 dl 文件夹，一个是 output 目录文件夹，最终生成的板子镜像在 output/images 目录下

目录介绍

> buildroot/arch:目录存放 CPU 架构相关的配置脚本，如 arm/mips/x86 ，这些 CPU 相关的配置，在制作工具链，编译 boot 和内核时很关键。
> buildroot/board:存放某个具体单板紧密相关的文件，比如内核配置文件、sd 卡制作脚本、rootfs覆盖文件等。
> buildroot/boot:存放各种 Bootloaders 相关的的补丁*.patch、校验文件*.hash、构建脚本*.mk、配置选项 Config.in
> buildroot/configs: 存放各个单板的 buildroot 配置文件
> buildroot/dl:目录存放从官网上下载的开源软件包，第一次下载后，下次就不会再去从官网下载了，而是从 dl/目录下拿开源包，以节约时间。本身下载通常都是很慢的,你可以手动找到相关包下载后放到这里就 OK 了,make 时会自动检测这个目录. buildroot/docs: 存放 pdf,html 格式的 buildroot 详细说明。
> buildroot/fs:放各种文件系统的源代码
> buildroot/fs/skeleton:放生成文件系统镜像的地方，及板子里面的系统*
> *buildroot/linux:存放 Linux 的构建脚本*.mk 和配置选项 Config.in
> buildroot/package/: 此目录下放着应用软件的配置文件，每个应用软件的配置文件有 Config.in 和软件包名字.mk。软件包名字.mk 文件可以下载应用软件包，是 Makefile 脚本的自动构建脚本。
> buildroot/output/: 是编译的输出文件夹，里面的 build/目录存放着解压后的各种软件包编译完后的现场。
> build:所有源码包解压出来的文件存放地和编译的发生地
> staging:包含了根文件系统的层次结构，编译生成的所有头文件和库，以及其他开发文件，体积较大；
> target:目录是用来制作 rootfs 的，里面放着 Linux 系统基本的目录结构，以及各种编译好的应用库和 bin 可执行文件。
> Images:目录下就是最终生成的可烧写到板子上的各种 image。
> host:是由各类源码编译后在你主机上运行的工具的安装目录,如 arm-linux-gcc 就是安装在这里;编译出来的主机工具在 host/usr 下;根目录所需要的库及一些基本目录就在
> host/<tuple>/sysroot/或 host/usr/<tuple>/sysroot/里
> buildroot/support:存放一些为 bulidroot 提供功能支持的脚本、配置文件
> buildroot/system:这里就是根目录的主要骨架了和相关的启动初始化配置,当制作根目录时就是将此处的文件 cp 到 output 里去.然后再安装 toolchain 的动态库和你勾选的 package 的可执行文件之类的. buildroot/toolchain:存放制作各种交叉编译工具链的构建脚本*.mk 和配置选项 Config.in
> buildroot/utils:存放一些 buildroot 的实用脚本和工具。
> buildroot/CHANGES:buildroot 的修改日志。
> buildroot/.config:make menuconfig 后生成的最终配置文件。
> buildroot/Config.in ：所有 Config.in 的入口，也是 Build options 的提供者
> buildroot/Config.in.legacy ：Legacy config options 的提供者
> buildroot/DEVELOPERS ：开发人员列表，N 后面是开发人员名字，F 后面是开发的软件
> buildroot/Makefile ：顶层 Makefile
> buildroot/Makefile.legacy ：旧的 Makefile，为了支持向后兼容
> buildroot/README ：buildroot 简单说明。

#### buildroot 帮助命令

在 buildroot 根目录执行 make help，即可获得 buildroot 常用命令的提示信息

> root@ubuntu:/home/topeet/Linux/buildroot-2022.05# make help
> Cleaning: #清理
> clean - 删除编译产生的文件
> distclean - 删除所有非源码文件（包括.config）
> Build: #编译
> all - 编译所有
> toolchain - 编译工具链
> sdk - 编译 sdk
> Configuration: #配置
> menuconfig - 基于 curses 的 buildroot 配置界面(常用)
> nconfig - 基于 ncursesbuildroot 配置界面
> xconfig - 基于 Qt 的 buildroot 配置界面
> gconfig - 基于 GTK 的 buildroot 配置界面
> oldconfig - 解决所有.config 中未解决的符号问题(symbols)
> syncconfig - 和 oldconfig 类似，但会额外更新依赖
> olddefconfig - 和 syncconfig 类似，但会将新的 symbols 设为默认值
> randconfig - 所有选项随机配置
> defconfig - 所有选项都询问，如果设置有 BR2_DEFCONFIG，则使用它的配置
> savedefconfig - 把当前配置保存到 BR2_DEFCONFIG
> update-defconfig - 类似 savedefconfig
> allyesconfig - 所有配置选项都选择 yes
> allnoconfig - 所有配置选项都选择 no
> alldefconfig - 所有配置选项都选择 default
> randpackageconfig - 软件包选项都选择随机
> allyespackageconfig - 软件包选项都选择 yes
> allnopackageconfig - 软件包选项都选择 no
> Package-specific: #具体的包操作
> <pkg> -编译、安装该 pkg 以及其依赖
> <pkg>-source -下载该 pkg 所有文件
> <pkg>-extract -解压该 pkg(解压后放在 output/build/pkg 名字目录下)
> <pkg>-patch - 给该 pkg 打补丁
> <pkg>-depends - 编译 pkg 的依赖
> <pkg>-configure - 编译 pkg 到配置这一步
> <pkg>-build - 编译 pkg 到构造这一步
> <pkg>-show-info - 显示该 pkg 信息
> <pkg>-show-depends - 显示该 pkg 的所有依赖
> <pkg>-show-rdepends - 显示依赖该 pkg 的所有包
> <pkg>-show-recursive-depends - 递归显示该 pkg 的所有依赖
> <pkg>-show-recursive-rdepends - 递归显示依赖该 pkg 的所有包
> <pkg>-graph-depends - 图形化显示该 pkg 的所有依赖
> <pkg>-graph-rdepends - 图形化显示依赖该 pkg 的所有包
> <pkg>-dirclean - 清除该 pkg 目录(清除解压目录 output/build/pkg 名字)
> <pkg>-reconfigure - 从配置这一步开始重新编译 pkg
> <pkg>-rebuild - 从构造这一步开始重新编译 pkg
> <pkg>-reinstall - 从安装这一步开始重新编译 pkg
> Documentation: -文档
> manual -生成各种格式的帮助手册
> manual-html - 生成 HTML 格式的帮助手册
> manual-split-html - 生成 split HTML 格式的帮助手册
> manual-pdf - 生成 PDF 格式的帮助手册
> manual-text - 生成 text 格式的帮助手册
> manual-epub - 生成 ePub 格式的帮助手册
> graph-build -生成图形化查看编译时间文件
> graph-depends - 生成图形化查看所有依赖文件
> graph-size -生成图形化查看文件系统大小文件
> list-defconfigs - 显示拥有的默认配置的单板列表
> Miscellaneous: - 杂项
> source - 下载所有要离线编译的源码到 dl 路径
> external-deps - 列出使用的所有外部包(make show-targets 的详细版)
> legal-info - 显示所有包的 license 合规性
> show-info - generate info about packages, as a JSON blurb
> pkg-stats - generate info about packages as JSON and HTML
> missing-cpe - generate XML snippets for missing CPE identifiers
> printvars - 打印所有内部指定的变量值
> show-vars - dump all internal variables as a JSON blurb; use VARS=... to limit the list to variables names matching that pattern
> make V=0|1 - 设置编译打印信息(0:安装编译 1:打印编译信息)
> make O=dir - 指定所有文件(包括.config)输出目录
> For further details, see README, generate the Buildroot manual, or consult
> it on-line at http://buildroot.org/docs.html

#### 安装编译环境

通过查看官方手册可知构建系统需要的工具有如下图所示， 这些工具需要我们在 ubuntu虚拟机上安装。官方手册地址可以点击以下网址：https://buildroot.org/downloads/manual/manual.html#requirement

![image-20240713164536362](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164536362.png)

我们需要安装这些编译必须工具，不然编译过程会出错。我们可以使用以下命令来安装这些工具，

> sudo apt install -y sed make binutils build-essential gcc g++ patch gzip bzip2 perl tar cpio unzip rsync file bc wget

#### Buildroot 配置

Buildroot 可以和配置内核一样，使用 menuconfig 进行图形化配置。输入“make menuconfig” 命令可以打开图形化配置界面，

![image-20240713164626147](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164626147.png)



#### Buildroot 构建

##### 配置 Target option

接下来我们来一步步配置 buildroot，使用“make menuconfig”命令打开配置界面。首先我们来配置 Target option 选项，

![image-20240713164724827](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164724827.png)

Target options 参数与芯片架构相关。如下图所示，Rk3568 Rk3399 imx6ull 芯片信息如下所示：

![image-20240713164744797](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164744797.png)

在 menuconfig 中需要配置的项目和对应的内容如下所示，如果配置的是 RK3568 开发板，配置项目如下所示，此项配置需要根据芯片架构进行修改。

> Target Architecture (AArch64（little endian)) ---> #设置架构和大小端
>
> Target Binary Format(ELF)--> #设置二进制格式
>
> Target Architecture Variant(cortex-A55)--> #设置芯片架构
>
> Floating point strategy(FP-ARMv8)--> #设置浮点数的策略

![image-20240713164807385](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713164807385.png)

##### Build options 选项

Build options 选项主要是一些编译时用到的选项,比如 dl 的路径,下载代码包使用的路径, 同时运行多个编译的上限,是否使能编译器缓冲区等等,这里按照默认就行了，此配置极少需要改动。Build options 选项具体含义如下所示：

> Commands --->
>
> │ │ ($(CONFIG_DIR)/defconfig) Location to save buildroot config \#保存 buildroot config
>
> │ │ ($(TOPDIR)/dl) Download dir #下载目录-buildroot/dl(通常使用默认配置)
>
> │ │ ($(BASE_DIR)/host) Host dir \#主机目录-buildroot/output/host(通常使用默认配置)
>
> │ │ Mirrors and Download locations ---> #镜像和下载位置
>
> │ │ (0) Number of jobs to run simultaneously (0 for auto) #同时要运行的作业数,0 代表自动
>
> │ │ [ ] Enable compiler cache #启用编译器缓存-可以加快编译速度
>
> │ │ [ ] build packages with debugging symbols #使用调试符号生成包
>
> │ │ [ ] build packages with runtime debugging info
>
> │ │ [*] strip target binaries #剥离目标二进制文件，会使文件变小
>
> │ │ () executables that should not be stripped #应该剥离的可执行文件
>
> │ │ () directories that should be skipped when stripping #剥离时应跳过的目录
>
> │ │ gcc optimization level (optimize for size) ---> #gcc 优化级别
>
> │ │ libraries (shared only) --->
>
> │ │ ($(CONFIG_DIR)/local.mk) location of a package override file #包覆盖文件的位置
>
> │ │ () global patch directories #全局修补程序目录
>
> │ │ Advanced --->
>
> │ │ ** Security Hardening Options *** #安全强化选项
>
> │ │ -*- Build code with PIC/PIE #使用 PIC/PIE 构建代码
>
> │ │ Stack Smashing Protection needs a toolchain w/ SSP *** #堆叠粉碎保护需要一个带有 SSP 的工具链
>
> │ │ RELRO Protection (Full) --->
>
> │ │ *** Fortify Source needs a glibc toolchain and optimization *

##### 配置 Toolchain

Toolchain 主要介绍如何使用 buildroot 配置内部交叉编译工具链以及外部交叉编译工具链还有如何添加自己的交叉编译工具链

> Toolchain type (Buildroot toolchain) --->
>
> ( ) Buildroot toolchain //内部交叉编译工具链
>
> ( ) External toolchain //外部交叉编译工具链

如上所示，Toolchain type 有俩个选项，选择 Buildroot toolchain 则是使用 buildroot 默认的自动化脚本从零开始制作交叉编译工具链，如果是选择 External toolchain 则是使用外部制作好的工具链。

接下来我们以设置外部交叉编译器进行演示，将交叉编译器压缩包拷贝到 Ubuntu 的 usr/local 目录下，解压缩后如下图所示：

![image-20240713165158511](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713165158511.png)

iTOP-RK3568 使用的内核镜像是 4.19.232 版本，在 buildroot 中要配置一致，即为 4.19.232，转换为 16 进制为 413E8，对应的十进制为 267240。我们进入到/usr/local/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu 目录下，修改aarch64-none-linux-gnu/libc/usr/include/linux/version.h 文件，修改为如下图所示，然后保存修改。

![image-20240713165220837](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713165220837.png)

需要配置的项目和对应的内容如下所示：

> Toolchain type (External toolchain) ---> #设置交叉编译器的类型为外部交叉编译器
> *** Toolchain External Options ***
> Toolchain (Custom toolchain) ---> #设置为 Custom toolchain，使用用户自己的交叉编译器
> Toolchain origin (Pre-installed toolchain) ---> # 设置为 Pre-installed toolchain，使用预装的交叉编译器
> (/usr/local/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu) Toolchain path (NEW)#设置外部交叉编译的路径，注意这里要写绝对路径，根据实际编译器的路径进行修改
> (aarch64-none-linux-gnu) Toolchain prefix (NEW) #设置 toolchain 的前缀，在编译器
> /usr/local/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu/bin 目录下查看
> External toolchain gcc version (9.x) ---> # 设置外部编译器 GCC 版本选择为 gcc9.x
> External toolchain kernel headers series (4.19.x) ---> # 内核头文件 Linux 4.19.x kernelheaders
> External toolchain C library (glibc/eglibc) ---> # C 库选择 选择 glibc
> [ ] Toolchain has WCHAR support? (NEW)
> [ ] Toolchain has locale support? (NEW)
> [*] Toolchain has threads support? (NEW)
> [*] Toolchain has threads debugging support? (NEW)
> [*] Toolchain has NPTL threads support? (NEW)
> [ *] Toolchain has SSP support? (NEW)
> [ *] Toolchain has RPC support? (NEW)
> [ *] Toolchain has C++ support? (NEW)
> [ ] Toolchain has D support? (NEW)
> [ ] Toolchain has Fortran support? (NEW)
> [ ] Toolchain has OpenMP support? (NEW)
> [ ] Copy gdb server to the Target (NEW)
> *** Host GDB Options ***
> [ ] Build cross gdb for the host
> *** Toolchain Generic Options ***
> () Extra toolchain libraries to be copied to target
> () Target Optimizations 不选
> () Target linker options 不选
> [ ] Register toolchain within Eclipse Buildroot plug-in #eclipse 插件支持，不选

##### 配置 System configuration

System configuration 页面主要是配置系统主机名，登录欢迎信息 root 密码，以及构建系统镜像版本，根文件系统覆盖。需要配置的项目和对应的内容如下所示：

> Root FS skeleton (default target skeleton) --->
>
> (iTOP-RK3568) System hostname \# 系统主机名，写 iTOP-RK3568
>
> (Welcome to Buildroot) System banner #系统欢迎语
>
> Passwords encoding (sha-256) ---> # 密码编码方式 sha-256
>
> Init system (BusyBox) ---> #系统初始化，选择 BusyBox
>
> \#设备文件管理，选择 Dynamic using devtmpfs + mdev，即使用 mdev 动态加载设备节点的方式/dev management (Dynamic using devtmpfs + mdev) --->\#设备节点的配置表设置，一定要选择 system/device_table.txt，否则后面在 dev 目录下就不会生成各种设备节点。当然我们也可以手动配置该文件，添加必要的节点或删除不需要的节点。
>
> (system/device_table.txt) Path to the permission tables
>
> [ ] support extended attributes in device tables
>
> [ ] Use symlinks to /usr for /bin, /sbin and /lib
>
> [*] Enable root login with password
>
> () Root password # 进入 linux 控制台终端后的密码，为空则登录时不需要密码，默认登录用户名为 root,密码为空。
>
> /bin/sh (busybox' default shell) --->
>
> [*] Run a getty (login prompt) after boot ---> #保持默认，默认为选中
>
> [*] remount root filesystem read-write during boot
>
> () Network interface to configure through DHCP
>
> (/bin:/sbin:/usr/bin:/usr/sbin) Set the system's default PATH
>
> [*] Purge unwanted locales
>
> (C en_US) Locales to keep
>
> () Generate locale data (NEW)
>
> [ ] Enable Native Language Support (NLS) (NEW)
>
> [ ] Install timezone info
>
> () Path to the users tables
>
> () Root filesystem overlay directories
>
> () Custom scripts to run before commencing the build
>
> () Custom scripts to run before creating filesystem images
>
> () Custom scripts to run inside the fakeroot environment
>
> () Custom scripts to run after creating filesystem images

##### Kernel 选项

Kernel 选项主要介绍了在线下载内核源码并自动编译到根文件系统的方法，**这里不选**

> [*] Linux Kernel
>
> Kernel version (Latest version (5.17)) ---> #内核版本，选择用户自定义
>
> () Custom kernel patches (NEW) \# 自定义的内核补丁
>
> Kernel configuration (Using an in-tree defconfig file) #内核配置
>
> () Defconfig name (NEW) #defconfig 文件名字
>
> () Additional configuration fragment files (NEW)
>
> () Custom boot logo file path (NEW)
>
> Kernel binary format (Image) ---> # 内核二进制文件格式
>
> Kernel compression format (gzip compression) ---> # 内核压缩格式
>
> [ ] Build a Device Tree Blob (DTB) (NEW)
>
> [ ] Install kernel image to /boot in target (NEW)
>
> [ ] Needs host OpenSSL (NEW)
>
> [ ] Needs host libelf (NEW)
>
> [ ] Needs host pahole (NEW)
>
> Linux Kernel Extensions ---> #内核扩展，默认不选择
>
> Linux Kernel Tools ---> #内核工具，默认不选择

##### **buildroot** **编译**

在上面的小节中，我们配置了 build root 的基础功能，然后我们输入 make 命令进行编译，注意编译过程中需要保证虚拟机 ubuntu 是联网状态。如下图所示：

![image-20240713165612753](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713165612753.png)

编译完之后

![image-20240713165624772](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713165624772.png)

我们使用命令“ls output/images/”即可看到编译出来的文件系统。此时编译出来的文件系统是最精简的文件系统。接下来将教大家如何添加软件包。

#### 配置 Target packages

Target packages 选项主要介绍在目标开发板上使用根文件系统所需要的软件包的添加方法，以及配置方法，大家可以根据需求来自行添加。

![image-20240713165804406](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713165804406.png)

![image-20240713165823549](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713165823549.png)

详细的选项介绍如下所示：

> Audio and video applications ---> # 主要介绍 buildtroot 支持的音频和视频应用功能介绍以及如何配置使用
> Compressors and decompressors ---> # 主要介绍 buildtroot 支持的压缩和解压应用简介以及如何配置使用
> Debugging, profiling and benchmark ---> # 主要介绍 buildroot 调试、分析和基准测试应用的简介以及如何配置使用。
> Development tools ---> # 主要介绍 buildroot 支持在目标系统上的开发工具应用的简介以及如何配置使用。
> Filesystem and flash utilities ---> #主要介绍 buildroot 支持在目标系统上的文件系统和闪存实用程序的简介以及如何配置使用。
> Fonts, cursors, icons, sounds and themes ---> #主要介绍 buildroot 支持在目标系统上的字体，游标，图标，声音和主题的简介以及如何配置使用。
> Games ---> # 主要介绍 buildroot 支持在目标系统上的游戏以及如何配置使用。
> Graphic libraries and applications (graphic/text) ---> #主要介绍 buildroot 支持在目标系统上的图形库和应用程序(图形/文本)的简介以及如何配置使用。
> Hardware handling ---> # 主要介绍buildroot支持在目标系统上的硬件处理应用/工具的简介以及如何配置使用。
> Interpreter languages and scripting ---># 主要介绍 buildroot 支持在目标系统上的编程语言和脚本的简介以及如何配置使用。
> Libraries ---> #主要介绍 buildroot 支持在目标系统上的库的简介以及如何配置使用。
> Mail ---> #主要介绍 buildroot 支持在目标系统上的邮箱应用的简介以及如何配置使用。
> Miscellaneous --->#主要介绍 buildroot 支持在目标系统上的一些杂项应用的简介以及如何配置使用。
> Networking applications ---> #主要介绍 buildroot 支持在目标系统上的网络应用程序的简介以及如何配置使用。
> Package managers ---> #主要介绍 buildroot 支持在目标系统上的包管理应用的简介以及如何配置使用。
> Real-Time ---> #主要介绍 buildroot 支持在目标系统上的实时时钟的简介以及如何配置使用。
> Security ---> #
> Shell and utilities ---> #主要介绍 buildroot 支持在目标系统上的 Shell 和程序的简介以及如何配置使用。
> System tools ---> # 主要介绍 buildroot 支持在目标系统上的系统工具的简介以及如何配置使用。
> Text editors and viewers --->#主要介绍 buildroot 支持在目标系统上的 文版编辑和浏览工具的简介以及如何配置使用。

以上了解了target packages的基本功能，接下来我们就为文件系统添加一些基本的软件包，进一步增强文件系统的功能。

##### **支持** **linux** **磁盘工具**

如果读者，想要的文件系统支持 FAT32 格式的分区，则需要参考下述操作进行配置dosfstools 工具包。

依次选择如下配置项：

> -> Target packages
>
> -> Filesystem and flash utilities
>
> ->dosfstools

![image-20240713170126132](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170126132.png)

如果读者，想要的文件系统支持 ext2 ext3 ext4 文件系统分区，请参考下图配置增加e2fsprogs 工具包。

依次选择如下配置项：

> -> Target packages
>
> -> Filesystem and flash utilities
>
> -> e2fsprogs

![image-20240713170140369](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170140369.png)

如果读者，想要格式化 spi nandflash norflash 等块设备，就参考下述配置增加 mtd tools。依次选择如下配置项：

> -> Target packages
>
> -> Filesystem and flash utilities

![image-20240713170151538](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170151538.png)

##### 支持 **nfs** **挂载工具**

如果读者想要使用 nfs 挂载工具，依次选择如下配置项：

> -> Target packages
>
> -> Filesystem and flash utilities

![image-20240713170235779](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170235779.png)

##### **支持** **v4l2** **框架工具**

如果读者想要使用摄像头等设备，需要查看摄像头数据或者设置摄像头的配置，则需要安装 v4l-utils 工具，请参考如下述配置路径来进行增加相应的包。

> -> Target packages
>
> -> Audio and video applications

![image-20240713170322746](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170322746.png)

依次选择如下配置项：

> -> Target packages
>
> -> Libraries
>
> -> Hardware handling

![image-20240713170359904](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170359904.png)

##### **支持** **can** **工具**

如果读者，想要测试 can 设备，参考下述配置进行安装 can-utils 来进行 can 数据的发送和读写，测试 can 之前还需要安装 iproute2 命令来初始化配置 can 设备。

依次选择如下配置项：

> -> Target packages
>
> -> Networking applications

![image-20240713170448280](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170448280.png)

##### **支持** **ssh** **访问工具**

如果读者，想通过 ssh 工具来远程登录开发板，或者使用开发板远程登录其他 ssh，则需要安装 openssh 工具包，配置过程如下：

> -> Target packages
>
> -> Networking applications

![image-20240713170522733](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170522733.png)

##### **支持** **4G** **拨号上网工具**

 RK 3568 开发板可以直接安装配套的 pcie 接口 4G 上网模块，可以通过 ppp 协议进行拨号上网，如果需要使用 ppp 协议则需要安装 pppd 工具包，配置方式参考下图。

> -> Target packages
>
> -> Networking applications

![image-20240713170602036](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170602036.png)

##### **支持** **hci** **蓝牙工具**

由于我们的开发板上蓝牙模块，如果使用蓝牙模块功能则要安装对应的工具包才可以，参考下述配置进行配置安装 bluez-utils 5.x 以及 bluez-tools 。

> -> Target packages
>
> -> Networking applications

![image-20240713170633464](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170633464.png)

##### **支持** **alsa** **声卡工具**

ALSA 是 Advanced Linux Sound Architecture，高级 Linux 声音架构的简称，它在 Linux操作系统上提供了音频和 MIDI（Musical Instrument Digital Interface，音乐设备数字化接口）的支持。在 2.6 系列内核中，ALSA 已经成为默认的声音子系统，用来替换 2.4 系列内核中的OSS（Open Sound System，开放声音系统）。

ALSA 的主要特性包括：高效地支持从消费类入门级声卡到专业级音频设备所有类型的音频接口，完全模块化的设计， 支持对称多处理（SMP）和线程安全，对 OSS 的向后兼容，以及提供了用户空间的 alsa-lib 库来简化应用程序的开发。需要使用 aplay 等命令，请参考如下配置进行配置。

> Target packages
>
> ---> Audio and video applications
>
> ---> [*] alsa-utils --->

![image-20240713170719319](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170719319.png)

> Target packages
>
> -> Libraries
>
> -> Audio/Sound
>
> -> -*- alsa-lib ---> 此配置项下的文件全部选中

![image-20240713170740816](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170740816.png)

##### **支持** **wpa WIFI** **工具**

如果需要使用开发板上的 wifi 模块进行上网等相关测试，则需要安装 iw 包来获取无线路由设备信息，并通过 wpa_supplicant 进行网络链接。

> -> Target packages
>
> -> Networking applications

![image-20240713170809176](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170809176.png)

> -> Target packages
>
> -> Networking applications

![image-20240713170830825](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170830825.png)

##### **支持** **Qt** **配置**

**支持** **multimedia**

凭借 GStreamer，程序员可以很容易地创建各种多媒体功能组件，包括简单的音频回放，音频和视频播放，录音，流媒体和音频编辑。基于流水线设计，可以创建诸如视频编辑器、流媒体广播和媒体播放器等等的很多多媒体应用，我们需要参考如下配置目录进行配置依次选择如下配置项：

> -> Target packages
>
> -> Audio and video applications

![image-20240713170935535](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713170935535.png)

**支持** **tslib** **触摸**

Tslib 是嵌入式经常使用的第三方库，可以优化触摸识别，如果读者需要，可以按下图配置：

依次选择如下配置项：

> -> Target packages
>
> -> Libraries
>
> -> Hardware handling

![image-20240713171013734](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171013734.png)

**支持** **qt5**

升级虚拟机ubuntu的gcc

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install gcc-9 g++-9
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 90 --slave /usr/bin/g++ g++ /usr/bin/g++-9
gcc --version
```

我们可以直接使用 buildroot 来制作一个具有 qt 环境的文件系统，配置如下图：

依次选择如下配置项：

> -> Target packages
>
> -> Graphic libraries and applications (graphic/text)
>
> -> Qt5

![image-20240713171038779](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171038779.png)

下面的勾选只是举一个例子请根据个人需求进行配置

![image-20240713171053818](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171053818.png)

支持虚拟键盘：

![image-20240713171108241](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171108241.png)

支持 SQLite：

> -> Target packages
>
> -> Graphic libraries and applications (graphic/text)
>
> -> Qt5
>
> ->SQLoite 3 support (QT SQLite) ---->

![image-20240713171142212](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171142212.png)

##### **支持** **python3**

Python 是一种面向对象的解释型语言,读者可以根据自己的需求选择编译安装 python 的模块，配置过程如下：

> -> Target packages
>
> -> Interpreter languages and scripting

![image-20240713171219028](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171219028.png)

##### **支持** **MQTT** **库**

Mosqitto 是一款实现了消息推送协议 MQTT v1.1 的开源消息代理软件，提供轻量级的，支持可发布/可订阅的的消息推送模式，使设备对设备之间的短消息通信变得简单。读者可以根据需求是否需要来选择安装，其配置过程如下

> -> Target packages
>
> -> Networking applications

![image-20240713171254938](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171254938.png)

##### **支持** **opencv**

> -> Target packages
>
> Libraries --->
>
> Graphics --->
>
> [*] opencv4 --->

![image-20240713171318682](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171318682.png)

##### **Buildroot** **下配置** **Busybox**

busybox 也是基于 buildroot 制作的文件系统，所以我们 buildroot 里的 busybox 也可以进行配置。首先我们在 buildroot 源码目录下，使用命令“make busybox-menuconfig”，如下图所示：

![image-20240713171401748](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171401748.png)

在这里我们可以看到 busybox 版本是 1.35.0，然后就是我们之前看到过的 busybox 配置界面了，用户可根据自己需要配置，这里的配置方法同使用 busybox 构建文件系统方法一致，

![image-20240713171426923](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171426923.png)

然后我们使用命令“make busybox”即可单独编译 busybox。

![image-20240713171442118](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171442118.png)

最后编译完成以后使用命令“make”重新编译 buildroot，主要是对其进行打包，

![image-20240713171459991](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171459991.png)

重新编译完成以后查看 output/images 目录下 rootfs.tar 的创建时间是否为刚刚编译的，如果不是的话就删除掉 rootfs.tar，然后重新执行“make”重新编译一下即可。

##### 支持 自动获取IP地址网络

>  Target packages ---->
>
> Networking applications  --->
>
> dhcpcd

##### Buildroot 编译

接着我们使用命令“make”编译 buildroot，期间他会自行下载一些软件源码，请保证网络可用。如下图所示：

![image-20240713171532162](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171532162.png)

最后编译完成

![image-20240713171550832](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171550832.png)

我们使用命令“ls output/images/”即可看到编译出来的文件系统。

![image-20240713171602201](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171602201.png)

如果想要下载默认支持的源码包，（**一般不下**），输入以下命令：

> make source

##### 制作文件系统镜像

在上面的章节中我们已经编译出了 rootfs.tar 文件系统，接下来我们就需要将文件系统源码打包成可以烧写到开发板的镜像。这里以 RK3568 开发板为例，我们将 rootfs.tar 拷贝到 123文件夹中，然后输入以下命令“tar -vxf rootfs.tar ”解压缩，如下图所示：

![image-20240713171645611](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171645611.png)

解压完毕，如下所示：

![image-20240713171700013](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171700013.png)

删除 rootfs.tar，如下所示，这样 123 文件夹里面包含了文件系统。

> rm -rf rootfs.tar

![image-20240713171722010](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171722010.png)

然后在 123 文件夹的同级目录，输入以下命令。制作自己的根文件系统，大小依据自己的根文件系统而定，注意依据 文件夹的大小来修改 count 值，：

> mkdir rootfs
>
> dd if=/dev/zero of=rootfs.img bs=1M count=1024
>
> mkfs.ext4 rootfs.img
>
> mount rootfs.img rootfs/
>
> cp -rfp 123/* rootfs/
>
> umount rootfs/
>
> e2fsck -p -f rootfs.img
>
> resize2fs -M rootfs.img

![image-20240713171752625](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171752625.png)

![image-20240713171759607](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171759607.png)

![image-20240713171805520](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713171805520.png)

这样 rootfs.img 就是最终的根文件系统映像文件了。

