## 4-3-1-安卓系统&uboot

### 安卓介绍

安卓（Android）是一种基于 Linux 内核的自由及开放源代码的操作系统。主要使用于移动设备，如智能手机和平板电脑，由美国 Google 公司和开放手机联盟领导及开发。

Android 系统最初是由安迪·鲁宾等人开发制作，最初开发这个系统的目的是创建一个数码相机的先进操作系统，后来发现市场需求不够大，加上智能手机市场快速成长，于是 Android 被改造为一款面向智能手机的操作系统。于 2005 年 8月被美国科技企业 Google 收购。2007 年 11 月，Google 和 84 家硬件制造商，软件开发商及电信运营商组建开放手机联盟共同研发改良 Android 系统，并发布Android 的源代码。第一部 Android 智能手机发布于 2008 年 10 月，由 HTC 公司制造。Android 逐渐扩展到平板电脑及其他领上，如电视，数码相机，游戏机，智能手表，车载大屏，智能家居等，并逐渐成为了人们日常生活中不可或缺的系统软件。2011 年第一季度，Android 在全球的市场份额首次超过塞班系统，跃居全球第一。2013 年的第四季度，Android 平台手机的全球市场份额已经达到78.1%。2013 年 09 月 24 日谷歌开发的操作系统 Android 迎来了 5 岁生日，全世界采用这款系统的设备数量已经达到十亿台。2019 年，谷歌官方宣布全世界有25 亿活跃的 Android 设备，还不包括大多数中国设备。

Android 几乎每年都要发布一个大版本，技术的更新迭代非常之快，Android几个主要版本发布时间如下表所示：

![image-20240713095606599](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713095606599.png)

Android *14*（代号：翻转蛋糕）是谷歌*发布的*操作系统。Android *14*相关的首批提交代码出现在Android Gerrit上。美国当地时间2023年10月4日，谷歌在纽约举行的“Made by Google”活动上，正式发布了适用于 Google Pixel 手机等设备的 Android *14*

#### 如何学习安卓

Linux内核的鼻祖Linus在回答一个Minix系统的问题时，第一句话便是“Read The Fucking Source Code”。这句话虽然颇有调侃的意味，但是却道出了阅读源码的重要性。由于 Android 系统的源码是开放的，因此，学习 Android 系统的最好的方法是阅读 Android 源码。

Android 源码获取有三种途径：

> 第一种：从谷歌网站获取 Android 源码
>
> 第二种：从芯片原厂获取 Android 源码
>
> 第三种：从方案厂商获取 Android 源码

这三种获取方式是递进关系，对应着不同的需求。对于第一种方式来说，从谷歌网站获取 Android 源码可以第一时间研究最新的技术，但是对于个人学习者来说，想要移植 android 源码到具体的芯片上是非常困难的事情。

对于第二种方式，芯片原厂从谷歌网站上下载源码，然后在此基础上对自己的 CPU 进行适配，适配完毕之后，再对方案厂商提供修改好的 Android 源码。从芯片原厂获取Android源码可以很容易地运行在某一个硬件上，但是更新较慢。

对于第三种方式，方案厂商从芯片原厂获取到 Android 源码，方案厂商根据开发板底板外设进行适配，适配完毕，此时可以交给用户使用了。

#### Android 体系结构

早期的 Android 结构如下图所示：

![image-20240713095808437](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713095808437.png)

Android 是一种基于 Linux 的开放源代码软件栈，为各类设备和机型而创建。官方网站最新的 Android 平台架构图如下所示：

![image-20240713095827562](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713095827562.png)

1. 应用程序层（System Apps）

   Android 会和同一系列核心应用程序一起发布，应用程序包括 email 客户端，SMS 短消息程序，日历，地图，浏览器，联系人管理程序等。所有的应用程序通常都是使用 JAVA 语言编写的。

2. 应用框架层（Java API Framework）

   开发 Android 应用程序就是面向底层的应用程序框架进行设计的，从此角度来说，Android 上的应用程序是完全平等的。Android 应用程序框架提供了大量的 API 供开发者使用，该应用程序的架构设计简化了组件的重用，任何一个应用程序都可以发布它的功能块并且任何其他的应用程序都可以使用其所发布的功能块（不过得遵循框架的安全性限制）。同样，该应用程序重用机制也使用户可以方便的替换程序组件。应用框架层包括如下内容：

   > 丰富又可扩展的视图（Views）:可以用来构建应用程序，它包括列表（Lists）, 网格（grids）,文本框（text boxes）,按钮（buttons），甚至可嵌入的 web 浏览器。
   >
   > 内容提供器（Content Providers）:使得应用程序可以访问另一个应用程序的数据（比如联系人数据），或者共享他们自己的数据。
   >
   > 资源管理器（Rescource Manager）:提供非代码资源的访问，如本地字符串，图形，和布局文件（layout files）
   >
   > 通知管理器（Notification Manager）:使得应用程序可以在状态栏中显示自定义的提示信息
   >
   > 活动管理器（Activity Manager）：用来管理应用程序生命周期并提供常用的道行回退功能。

3. 函数库（原生 C/C++）

   Android 包含一些被不同组件所使用的 C、C++库的集合，一般不能直接调用C、C++库集，但是可以通过 Android 应用程序框架来调用这些库。函数库主要包含以下内容：

   > Bionic 系统 C 库：标准 C 系统函数库（libc）,它是专门为嵌入式 Linux 设备定制的。
   >
   > 媒体库：支持多种常用的音频，视频格式回放和录制，同时支持静态图像文件。编码格式包括 MPEG4，H.264，MP3，AAC，AMR，JPG，PNG。
   >
   > Surface Manager：对显示子系统的管理，并且为多应用程序提供了 2D 和 3D图层的无缝融合。
   >
   > Webkit,LibWebCore:一个全新的 Web 浏览器引擎，支持 Android 浏览器和一个可嵌入的 Web 视图。Webview 完全可以嵌入到开发者自己的应用程序中。
   >
   > SGL：底层的 2D 图形引擎
   >
   > 3D libraries: 基于 OpenGL ES1.0 API 实现，该库可以使用硬件 3D 加速或者使用高度优化的 3D 软件加速。
   >
   > Free Type：位图和矢量字体显示
   >
   > SQLite：一个对于所有应用程序可用，功能强劲的轻型关系型数据库引擎。

4. Android 运行时（Android Runtime）

   Android 运行时由俩部分组成：Android 核心库和 ART（Android RunTime），其中核心库提供了 Java 语言核心库所使用的绝大部分功能，而虚拟机则负责运行 Android 应用程序。

   Android5.0 以前，Android 运行时由 Dalvik 虚拟机和 Android 核心库组成，但是由于 Dalvik 采用 JIT 的解释器进行动态编译并执行，应用每次运行的时候，字节码都需要通过 JIT 转换为机器码，这会拖慢应用的运行效率，因此导致Android APP 运行比较缓慢，而 ART 模式是在用户安装 APP 时进行预编译，将原本在程序运行时的编译动作提前到应用安装时，这样使得运行时可以减少动态编译的开销，从而提升 Android APP 的运行效率。但是 ART 需要在安装 APP 时进行 AOT 处理，因此 ART 需要占用更多的存储空间，应用安装和系统启动时间会延长不少，ART 还支持 ARM X86 和 MIPS 架构，而且完全兼容 64 位系统，必然会带来更好的用户体验。

5. 硬件抽象层（HAL）

   硬件抽象层（HAL）提供标准界面，向更高级别的 java API 框架显示设备硬件功能。HAL 包含多个库模块，其中每个模块都为特定类型的硬件组件实现一个界面，例如相机或蓝牙模块。当框架 API 要求访问设备硬件时，Android 系统将为该硬件组件加载库模块。

6. Linux 内核（Linux kernel）

   Android 系统建立在 Linux 2.6 之上，如安全性，内存管理，进程管理， 网络协议栈和驱动模型。 Linux 内核也同时作为硬件和软件栈之间的抽象层。其外还对其做了部分修改，主要涉及两部分修改：

   > Binder (IPC)：提供有效的进程间通信，虽然 linux 内核本身已经提供了这些功能，但 Android 系统很多服务都需要用到该功能，为了某种原因其实现了自己的一套。
   >
   > 电源管理：主要是为了省电，毕竟是手持设备嘛，低耗电才是我们的追求。

#### 安卓架构中调用关系

最新的 Android 平台架构图采用静态分层方式的架构划分，众所周知，程序代码是死的，系统运转是活的，各模块代码运行在不同的进程(线程)中，相互之间进行着各种错终复杂的信息传递与交互流，从这个角度来说此图并没能体现Android 整个系统的内部架构、运行机理，以及各个模块之间是如何衔接与配合工作的。

为了更深入地掌握 Android 整个架构思想以及各个模块在 Android 系统所处的地位与价值，以Android系统启动过程为主线，以进程的视角来诠释Android 系统全貌，如下图所示：

![image-20240713102045638](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713102045638.png)

从上面的图中可以找到 A2ndroid 官方给出的 5 个层次的影子（Application，Framework（java），库和运行时（native），HAL，Kernel），java 层和 native层通过 JNI 调用打通，native 层和 kernel 层通过 Syscall 调用打通。这个图是启动android 手机时 Android 系统的启动过程，下面分别介绍：

1. 启动电源以及系统启动

   加载引导程序 Bootloader 到 RAM，然后执行

2. 引导程序 Bootloader

   主要作用是把系统 OS 拉起来并运行。

3. Linux 内核启动

   在系统文件中寻找 init.rc 文件，并启动 init 进程。

4. init 进程启动

   init 进程是 Linux 系统的用户进程，它的 pid=1，是所有用户进程的鼻祖，它是由许多源码文件组成的，它对应的源码目录在/system/core/init 中。init 进程有许多重要的职责：

   > 孵化出许多用户空间的守护进程（ueventd、logd、healthd、installd、adbd、lmkd）启动 ServiceManager(binder 服务管家)、bootanim(开机动画)等重要服务
   >
   > 孵化出 Media Server 进程，负责启动和管理整个 C++ framework，包含AudioFlinger，Camera Service 等服务。
   >
   > 孵化出 Zygote 进程

5. Zygote 进程启动

   Zygote 进程是 Android 系统的第一个 Java 进程(即虚拟机进程)，Zygote 是所有 Java 进程的父进程，它是由 init 进程通过解析 init.rc 文件后 fork 生成的，Zygote的启动脚本放在/system/core/rootdir 目录中。Zygote 进程的职责主要有：

   > 创建 JVM 虚拟机，并为 JVM 注册 JNI 方法
   >
   > 响应 AMS 的请求去创建新的应用进程
   >
   > 孵化出 SystemServer 进程

6. SystemServer 进程启动

   SystemServer 进程是 Zygote 孵化的第一个进程，负责创建系统服务如ActivityManagerService ， WindowManagerService ， PackageManagerService ，InputManagerService 等服务，和管理整个 Java framework。SystemServer 进程的职责主要有：

   > 创建 Binder 线程池，这样就可以与其他进程进行跨进程通信
   >
   > 启动 SystemServiceManger，它用来对系统服务进行创建、管理和启动
   >
   > 通过 SystemServiceManger 启动各种系统服务：引导服务（如 AMS，PMS），核心服务，其他服务（如 WMS，IMS）

7. Launcher 启动

   SystemServer 进程启动的过程中会启动 PMS 和 AMS，PMS 会把系统中的应用程序安装完成，然后 AMS 会请求 Zygote 将 Launcher 启动起来，这就是用户看到的 app 桌面，然后 Launcher 会将已经安装了的应用的应用图标显示出来。Launcher 进程是 Zygote 进程孵化出来的第一个 App 进程。

   至此 Android 系统已经启动完毕，用户就可以点进桌面上的应用图标进入app，对于普通的 app 进程,跟 SystemServer 进程的启动过来有些类似，不同的是app 进程是先发消息给 SystemServer 进程，由 SystemServer 向 Zygote 发出创建进程的请求，而 SystemServer 是由 Zygote 直接 fork 出来，前面已经说过 Zygote是所有 Java 进程的父进程，SystemServer 和所有的 app 进程都是由 Zygote 进程的子进程。

### 编译AOSP

AOSP Android 源码下载的时候，下载完毕大约是 200G 左右，建议使用虚拟机进行编译。

电脑硬件要求如下所述：

> CPU：X86 CPU，核数越多越好
>
> 内存：不少于 16GB（针对虚拟机）
>
> 存储: 不少于 300GB
>
> Ubuntu 版本：Ubuntu18.04
>
> 操作系统中交换分区：6G 以上

如果物理内存只有16G，需要设置交换内存：

在开始之前，使用命令检查一下您的 ubuntu 的 swap 分区。

> sudo swapon --show

通过以下命令创建一个用于 swap 的文件

> sudo swapoff -a
>
> sudo fallocate -l 5G /swapfile

执行以下命令为 swapfile 文件设置正确的权限：

> sudo chmod 600 /swapfile

使用 mkswap 实用程序在文件上设置 Linux SWAP 区域：

> sudo mkswap /swapfile

> sudo vim /etc/fstab

在/etc/fstab 文件最后添加如下内容：

> /swapfile swap swap defaults 0 0

使用以下命令激活 swap 文件：

> sudo swapon /swapfile

使用 swapon 或 free 命令验证 SWAP 是否处于活动状态，如下所示:

> sudo swapon --show
>
> sudo free -h

![image-20240713104047626](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713104047626.png)

#### 下载源码

AOSP 全名为 Android Open Source Project，意为安卓开源项目，开源即开放源代码。

中文官网是：https://source.android.google.cn/

官方网站：https://developer.android.com/  （需要翻墙，所以一般参考下面这 2 个）

https://developer.android.google.cn/

Android 开源项目：https://source.android.google.cn/

##### repo 工具下载和安装

1. 使用命令“sudo apt-get install git”,输入密码，安装过程中询问信息“y”继续，安装 git.

2. 设置 Git，执行如下命令，填写名字和邮箱。

   > git config --global user.name “woniu” 
   >
   > git config --global user.email “woniu@com”

3. 使用以下命令下载 repo，注意！ 这里有可能会因为网络问题下载不成功。如果不能访问，可以换中科大的下载源。

   > git clone https://gerrit-googlesource.lug.ustc.edu.cn/git-repo

4. 使用命令“mkdir .repo”创建一个隐藏文件夹，然后使用“ll”命令查看。

5. 接下来使用“mv git-repo .repo/repo”命令，将“git-repo”移动到刚刚创建的“.repo” 文件中，并将其名称改为“repo”。

6. 使用命令“cp .repo/repo/repo ./”，将 repo 复制到当前目录，接着使用命令“chmod u+x repo”,修改 repo 操作权限。

7. 这里我们下载 AOSP 版本为 Android11，对应版本标记可以下 Android 官网找到，Android 官网链接为：https://source.android.google.cn/setup/start/build-numbers这里我们选择 android-security-11.0.0_r55，并记下这个标记在下载的时候会用到。

   ![image-20240713104620189](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713104620189.png)

##### 下载源码

1. 选定 android-security-11.0.0_r55 版本进行下载，AOSP 源码可以去清华或者中科大的数据源中进行下载。

   中科大： https://mirrors.ustc.edu.cn/aosp

   清华：https://mirrors.tuna.tsinghua.edu.cn

   这里以中科大下载源为例，下载 Android11 源码。

   在 repo 所在路径下输入以下命令：

   > ./repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-security-11.0.0_r55

2. 接着在 repo 所在路径输入命令下载源码

   > ./repo sync -j2

3. 下载完成后，使用命令“du -sh”，显示源码大小为 106G

   ![image-20240713104824596](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713104824596.png)

#### 编译AOSP源码

在上一小节中，我们顺利的将 AOSP 源码下载了下来，很多时候我们不仅仅需要查看源码，还有以下几个需求：

1. 动态调试 Android 系统源码

2. 定制 Android 系统

3. 将 Android 系统刷入自己的 Android 设备中。

为了实现这些需求，所以需要去编译 Android 系统源码。以下是编译步骤：

1. 输入以下命令初始化编译环境，使用 build 目录下的 envsetup.sh 脚本初始化环境，这个脚本会引入其他的执行脚本。

   > source build/envsetup.sh

2. 输入以下命令选择编译目标，lunch 命令是 envsetup.sh 里面定义的一个命令，用来让用户选择编译目标。会有以下信息输出：

   > lunch

   选择编译目标的格式，编译目标的格式组成为

   BUILD-BUILDTYPE，比如 aosp_arm-eng 的 BUILD 为 aosp_arm，BUILDTYPE为 eng。

   其中 BUILD 表示编译出的镜像可以运行在什么环境，aosp 代表 Android 开源项目，arm 表示系统是运行在 arm 架构的处理器上。BUILDTYPE 指的是编译类型，有以下三种：

   > user 用来正式发布到市场的版本，权限受限，如没有 root 权限，不能 debug,adb 默认处于停用状态。
   >
   > userdebug  在 user 版本的基础上开放了 root 权限和 debug 权限，adb 默认处于启动状态，一般用于调试真机。
   >
   > eng  开发工程师的版本，拥有最大的权限（root 等），具有额外调试工具的开发配置，一般用于模拟器

   也可以直接指定编译目标:lunch aosp_x86_64_eng,或者直接输入对应的序号。

3. 开始编译，输入以下命令，通过-jN 参数来设置编译的并行任务数，这里我们设置为 12 个并行任务进行编译。

   > make -j12

   ![image-20240713110349042](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713110349042.png)

4. 最终会在 out/target/product/generic_x86_64/目录下生成了三个重要的镜像文件：system.img、userdata.img、ramdisk.img。大概介绍下这三个镜像文件：

   > system.img :系统镜像，里面包含了 Android 系统主要的目录和文件，通过 init.c进行解析并 mount 挂载到/system 目录下。
   >
   > Userdata.img:用户镜像，是 Android 系统中存放用户数据的，通过 init.c 进行解析并 mount 挂载到/data 目录下。
   >
   > ramdisk.img:根文件系统镜像，包含一些启动 Android 系统的重要文件，比如init.rc。

#### 运行镜像

Android 源码编译出来的镜像，可以通过 Android 自带的 emulator 来运行。emulator 就是模拟器，当用户没有真机时，可以使用模拟器来对系统和程序进行调试和开发。在上一小节中，我们编译成了 x86 64 位 ，使用模拟器启动。

1. 首先点击虚拟机“设置”->处理器->勾选虚拟化 InterVT-x/EPT 或者AMD-V/RVI(V)，如下图所示：

   ![image-20240713110531585](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713110531585.png)

2. 切换 root 用户

3. 在编译完成之后，就可以通过以下命令运行 Android 虚拟机了，命令如下

   > emulator

   ![image-20240713110701407](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713110701407.png)

## 编译瑞星微源码

1. 输入以下命令设置 java 版本为 1.8 版本，确认 java 版本是 1.8 版本之后，才可以进行下一步编译，如下图所示：

   > source javaenv.sh
   >
   > java -version

2. 输入命令配置 Android 分支

   > source build/envsetup.sh
   >
   > lunch 
   >
   > rk3568_r-userdebug

3. 编译

   > ./build.sh -AUCKu

   ![image-20241223170851979](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20241223170851979.png)

4. 在编译内核的过程中，会提示电源域检查，如下图所示：

   ![image-20240713111053919](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713111053919.png)

   设备树中关于 IO-Domain 的默认配置，如下所示：

   ```
   &pmu_io_domains {
     status = "okay";
     pmuio1-supply = <&vcc3v3_pmu>;
     pmuio2-supply = <&vcc3v3_pmu>;
     vccio1-supply = <&vccio_acodec>;
     vccio3-supply = <&vccio_sd>;
     vccio4-supply = <&vcc_3v3>;
     vccio5-supply = <&vcc_3v3>;
     vccio6-supply = <&vcc_3v3>;
   	vccio7-supply = <&vcc_3v3>;
   };
   ```

   弹出这个对话框目的是检查实际硬件原理图和软件设备树中的 GPIO 电压是否匹配，需要根据硬件原理图的实际设计电压来选择（对话框中选择的值不会保存到设备树中，设备树需要手动修改）。当你正确确认 GPIO 电压后，这个对话框不会再弹出（输入值和设备树配置的值相同），如果设备树名字或者设备树里面的 io-domain 发生变化，则会继续弹出进行确认。

   默认配置全部为 3.3V，在弹出来的对话框中全部选择 3.3V 即可。

#### 瑞芯微原厂源码目录介绍

##### 顶层目录分析

![image-20240713111538071](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713111538071.png)

> **art/**   Android RunTime(Android 运行环境)，Android5.0 之后 java 虚拟机就使用 ART
>
> **bionic/**   基础 C 库源代码，Android 改造的 C/C++库，比传统的 glibc 更精 简 ，不 受 GPL 限 制 ，支 持 pthread_cancel(), 不 支 持 C++exception 和 C++ STL 模板编程。
>
> **bootLoader**  包含了 recovery 程序（恢复出厂/升级）的代码。
>
> **build.sh**  瑞芯微提供的编译脚本
>
> **cts**  Android 兼容性测试套件，CTS 是一个自动化测试套件，包含PC 端和设备端（被测设备），对手机的硬件，软件，接口，性能进行测试，测试项目包含平台 API 测试，平台 Intent，蓝牙设备的连接情况，相机拍照功能等，通过 make cts 编译，out/host/linux-x86/cts/下生成 android-cts 文件夹。由于 Android 是开源的，对于 Google Android 的开发联盟中的 Motorola、Samsung、Qualcomm 、 Sony Ericsson、中国移动、ZTE、华为等，底层的代码也是开放的。手机制造商及运营商可以在 Android 上打造，定制自己特有的手机操作系统，这势必在源码级别上对 Android 系统进行代码的添加和更改。如果不规范这些更改则会给上层的应用开放的移植带来问题（那个时候你会看到 MOTO 上跑的愤怒的小鸟会在三星的Android 手机上运行不了，每个应用需要都要发布不同厂商的Android 手机的版本），只有通过 CTS 测试的 Android 手机系统，Google 才会颁发许可， 以保证不同生产商之间的Android 系统的兼容。
>
> **dalvik**  Android Dalvik 虚拟机相关内容，主要是提供了一下工具，如dexdump， vm 主代码已经移除
>
> **developers** 开发者目录， 一些应用程序的参考实例 demo 代码
>
> **development**  提供应用开发的工具， 应用例子，monkey 命令，shell 脚本和
>
> python 脚本：
>
>  development/scripts/stack：用于展开进程异常时的堆栈信息，配合 tombstone 文件
>
>  development/scripts/gdbclient.py: 用于 gdb 调试
>
>  development/scripts/native_heapdump_viewer.py 脚本将dumpheap 进程结果转换成更易读的 html 格式
>
>  development/tools/make_keys： 系统签名生成器
>
>  development/tools/idegen: 用于生成 android.ipr、android.iml IEDA 工程配置文件，可以用 android studio 来导入android 源码
>
> **device**   设备产品定制目录， 该目录是厂商和产品公司定制文件比较多的地方
>
> **external**   外部第三方开源的库和工具，比如 ppp, wpa_supplicant, libz, libcurl 等
>
> **frameworks** Android 应用框架层核心代码， 包括各种系统资源， 后台服务， SystemUI， 大部分都是 java 代码， 同时也包含了各种子系统的 native 代码， 如 av, net 等
>
> **hardware** 硬件抽象层代码， 如 gps, wifi, viberate
>
> **kernel**  内核源码，厂商可以将内核源码集成到该目录
>
> **libcore**  Android 系统中支持众多 Java 标准的库，例如 java.lang 包,libart 库，JSON 库
>
> **libnativehelper**  JNI 编程时调用的各种接口实现代码， 如 findClass()， 最终编译成 libnativehelper.so
>
> **packages**  系统内置的 app 源码， 如设置， 墙纸，输入法，屏保等。
>
> apps：核心应用程序
>
> inputmethods：输入法目录
>
> providers：内容提供者目录
>
> screensavers：屏幕保护
>
> services：通信服务
>
> wallpapers：墙纸
>
> **pdk**  Platform Development Kit 的缩写，平台开发套件， PDK 的目标是帮助芯片组供应商和原始设备制造商迁移到新的版本。
>
> **sdk** 编译 sdk 规则目录，Android 的 SDK 包含该平台为应用程序开发人员提供的开发工具，主要是所有公开 API 的集合，应用程序开发人员可以借助 SDK 中的 API 快速的进行应用的开发.
>
> **platform_testing**  Android 平台测试程序
>
> **prebuilts** 预编译文件夹，x86 和 arm 架构下预编译的一些资源，如 ndk工具包, gcc 编译器等
>
> **system** Android 底层文件系统库、应用和组件，如 busybox， init 进程
>
> **test** Android Vendor 测试框架，如 vts(vendor test suit) ，MainlineTest Suite (MTS)， mlts
>
> **toolchain**  没什么内容，都是一些 python 测试脚本
>
> **tools**  一些特殊命令， 如 acloud 命令用于连接谷歌云端工作站的，apkzlib 用于 apk 打包的。

##### out 目录

编译 android 完毕，会生成 out 目录，目录下主要有俩个子目录，host 当前编译主机需要用到的工具和库，一般都是 x86 架构的。

另外一个就是 target 目录，就是目标机器运行所需要二进制文件，普遍都是arm 架构的二进制文件，该目录一般都是存放编译的中间文件，如.o 文件，以及目标文件，包含 strip 和 unstrip 的。同时对这些文件进行分门别类的进行存放。生成的中间文件:

> tree -L out/target/product/rk3568_r/obj
>
> ├── APPS :内置 app 的中间编译文件
>
> ├── CONFIG
>
> ├── ETC ：etc 的中间编译文件，一般都会最终生成到 system/etc 或者 vendor/etc目录
>
> ├── EXECUTABLES ：可执行的中间编译文件
>
> ├── FAKE
>
> ├── JAVA_LIBRARIES ：java jar 库的中间编译文件
>
> ├── PACKAGING ：镜像打包时的中间编译文件
>
> ├── RENDERSCRIPT_BITCODE
>
> ├── SHARED_LIBRARIES ：动态库的中间编译文件
>
> └── STATIC_LIBRARIES：静态库的中间编译文件

##### system 目录

Android 作为 Linux 系统，在构建最小系统时，需要祖先 init 进程，依赖库，二进制工具 linux 命令，以及各种守护进程等，system 目录就提供了这些东西，该目录大部分都是 C/C++代码。

> ├── apex：apexd 守护进程源码，负责处理 apex 安装包的。
>
> ├── bpf
>
> ├── bpfprogs
>
> ├── bt：Android bluedroid 协议栈
>
> ├── ca-certificates
>
> ├── chre：Context Hub Runtime Environment (CHRE),用于平衡大小核运行不同程序的场景， 提供 api，保证小 型的本机应用程序（称为 nanoapps ）在低功耗处理器上执行。
>
> ├── connectivity：包含 wificond 进程，该进程通过标准 nl80211 命令与 WLAN驱动程序进行通信
>
> ├── core：包含各种依赖库，adb，祖先 init 进程源码，lmkd，logcat, toolbox
>
> ├── extras：各种额外的命令工具，比如 su，memtrack(追踪 graphic 相关内存)，playwav 命令，以及一些测 试代码
>
> ├── gatekeeper：防护程序，如锁屏密码等。
>
> ├── gsid：通用系统镜像守护进程
>
> ├── hardware：网络相关的 HIDL 描述文件。
>
> ├── hwservicemanager：HAL 服务管理中心，负责管理系统中的所有 HAL 服务，由 init 进程启动，属于 vendor 的 binder 通信机制，类似 framework 中的binder. 
>
> ├── incremental_delivery
>
> ├── iorap：iorap 用于缩短应用程序的启动时间，该目录包含 iorapd 守护进程和相应的库文件。
>
> ├── keymaster
>
> ├── libartpalette
>
> ├── libfmq
>
> ├── libhidl:硬件接口定义语言依赖库
>
> ├── libhwbinder:hwbinder 依赖库
>
> ├── libsysprop
>
> ├── libufdt
>
> ├── libvintf
>
> ├── linkerconfig
>
> ├── media:包括 alsa、audio、camera 相关的头文件
>
> ├── memory
>
> ├── netd:Android 中专门负责网络管理和控制的后台守护进程，如管理 DNS，设置防火墙，带宽控制
>
> ├── nfc
>
> ├── nvram
>
> ├── security
>
> ├── sepolicy:selinux 权限策略配置目录
>
> ├── server_configurable_flags
>
> ├── teeui
>
> ├── testing
>
> ├── timezone
>
> ├── tools:aidl，hidl 可执行程序的源码路径
>
> ├── update_engine:负责 A/B 升级的核心逻辑代码。
>
> └── vold：Volume 守护进程，用来管理 Android 中存储类的热拔插事件，如设备增加，删除，修改等事件处理。

##### frameworks 目录

> ── av：audio/video， 如音视频解码器，录屏工具， 摄像头框架相关代码, 多媒体框架供上层程序调用的 java API，连接 java 和 C/C++的 jni 部分, 在framework/base/media 下
>
> │ ├── apex
>
> │ ├── camera：libcamera_client 共享库，是 camera 框架部分的 client 代码
>
> │ ├── CleanSpec.mk
>
> │ ├── cmds：包括录屏工具，stagefright 进程。
>
> │ ├── drm
>
> │ ├── include
>
> │ ├── MainlineFiles.cfg
>
> │ ├── media：提供音视频编解码的各种库和工具，后台进程如 audioserver, mediaserver(通过 binder 的进程间通信方式来完成其他进程（如音乐播放器）的请求)
>
> │ ├── MODULE_LICENSE_APACHE2
>
> │ ├── NOTICE
>
> │ ├── OWNERS
>
> │ ├── PREUPLOAD.cfg
>
> │ ├── services：多媒体相关的后台服务， 如 cameraservice,audioflinger
>
> │ └── tools
>
> ├── base：基础核心代码
>
> ├── compile：包含 libbcc,mclinker(llvm 编译器的插件) slang(Renderscript 语言的编译器)
>
> ├── ex:Android 内部使用的公共类，如联系人、小部件、io, 以及全新的相机框架 Camera2 的部分代码。
>
> ├── hardware:描述传感器，虚拟现实 VR，camera 等硬件的 HIDL 接口的文件.hal
>
> ├── layoutlib:布局相关
>
> ├── libs
>
> ├── minikin：Android 原生字体，连体字效果
>
> ├── ml：机器视觉
>
> ├── multidex：多 dex 加载器，大多数 App，解压其 apk 后，一般只有一个classes.dex 文件，采用 MultiDex 的 App 解压后可以看到有 classes.dex，classes2.dex，… classes(N).dex
>
> ├── native：c/c++本地语言编写的相关工具源码和特定硬件控制的头文件，如bugreport, dumpstate, 各种 硬件的访问权限声明. 
>
> ├── opt：一些 UI 插件和 java 后台服务，如 timezonepicker，datetimepicker，colorpicker， 网络服务，如 EthernetService， WifiService, 电话服务 telephony
>
> ├── rs：渲染脚本 rendor script
>
> ├── support.md -> ../prebuilts/sdk/current/androidx-README.md
>
> └── wilhelm：基于 Khronos 的 OpenSL ES/OpenMAX AL 的 audio/multimedia实现

其中 base 目录是中应用框架层的主要核心代码，目录结构解释如下所示

> ── Android.bp
>
> ├── Android.mk
>
> ├── apct-tests
>
> ├── apex
>
> ├── api：主要是 txt 文件， 声明了 android 应用框架层的类、属性和资源
>
> ├── ApiDocs.bp
>
> ├── CleanSpec.mk
>
> ├── cmds：开机启动的进程代码和命令（脚本），著名的 zygote 进程代码和启动动画代码就在此处。
>
> ├── config
>
> ├── core：
>
> ├ ├── java：Android 应用开发锁依赖的各种包，四大组件代码就在此，如android/app/Activity.java，android/app/Service.java，android/content/ContentProvider.java， ./android/content/BroadcastReceiver.java
>
> ├ ├── jni：服务于 android 系统 java 核心代码所需要的 jni，被编译成libandroid_runtime.so
>
> ├ ├── proto
>
> ├ ├── res:系统中所需要的各种资源，图片，字符串，尺寸，布局文件等。
>
> ├ ├──TEST_MAPPING
>
> ├ ├── tests
>
> ├ └── xsd
>
> ├── data：系统默认的铃声，字体， 根文件系统 etc 目录部分配置文件，触摸需要的 kl, kcm 文件等。
>
> ├── docs
>
> ├── drm：Digital Rights Management(数字版权管理) ，应用根据与受版权保护的内容关联的许可限制来管理 自己的内容，从而达到保护应用内容的知识产权。
>
> ├── errorprone
>
> ├── framework-jarjar-rules.txt
>
> ├── graphics：为应用提供的 android.graphics 包，提供基本的图形原语（如画点画线，设置图形上下文等） 另外和图形相关的包：android.view 图形事件机制和 android.widget 包提供开发图形用户界面的控件
>
> ├── identity
>
> ├── keystore：提供 android.security.keystore 包, 应用可以通过 KeyStore API生成密钥、使用密钥签名, keystorek 可以保护密钥材料免遭未经授权的使用
>
> ├── libs：大部分都是 c/c++代码，编译成 so， 如 Canvas 的 drawing 操作转换为 OpenGL 的操作的 libhwui.so
>
> ├── location：定位相关接口，被 App 调用的，API 包是 android.location
>
> ├── lowpan：提供 android.net.lowpan 报，提供基于 IPv6 的低速无线个域网的API, lowpan 功能与 Zigbee 类似
>
> ├── media：多媒体相关接口，被 App 调用的， 包含 MediaPlayer 代码
>
> ├── mime
>
> ├── mms
>
> ├── MODULE_LICENSE_APACHE2
>
> ├── native：部分硬件相关的 jni 库：如 sensor, net, 存储管理， 最后合并在libandroid.so 中
>
> ├── nfc-extras
>
> ├── non-updatable-api
>
> ├── NOTICE
>
> ├── obex：蓝牙文件传输 obex 协议
>
> ├── opengl：提供 android.opengl 包, 提供 android 2D/3D 图形处理各种 API
>
> ├── packages：著名的 SystemUI，彩蛋，锁屏，SettingProvider(系统设置数据提供者)
>
> ├── pathmap.mk
>
> ├── PREUPLOAD.cfg
>
> ├── proto
>
> ├── rs：提供 android.renderscript 包，RenderScript 是 Android 平台上的一种类 C 脚本语言，用于渲染绘图
>
> ├── samples
>
> ├── sax
>
> ├── services：android 开机启动的大部分后台服务，如 PMS, AMS, WMS 等
>
> ├── startop
>
> ├── StubLibraries.bp
>
> ├── telecomm
>
> ├── telephony
>
> ├── test-base
>
> ├── test-legacy
>
> ├── TEST_MAPPING
>
> ├── test-mock
>
> ├── test-runner
>
> ├── tests
>
> ├── tools：提供应用开发的各种工具和脚本：如 aapt
>
> └── wifi：提供 android.net.wifi 包，如 wifi 扫描， p2p，hotspot2 相关 API

##### libcore 目录

Libcore 提供 Android 系统中支持众多 Java 标准的库，例如 java.lang 包

> ├── Android.bp
>
> ├── AndroidTest-core-tests.xml
>
> ├── apex
>
> ├── benchmarks
>
> ├── check-ojluni-files
>
> ├── CleanSpec.mk
>
> ├── dalvik：dalvik 虚拟机相关包
>
> ├── dom：文档对象模型，用于解析 xml
>
> ├── expectations
>
> ├── harmony-tests
>
> ├── JavaLibrary.bp
>
> ├── json:json 串在 Java 中的支持
>
> ├── jsr166-tests
>
> ├── known_oj_tags.txt
>
> ├── libandroidio.map.txt
>
> ├── libart：art 运行时部分相关包
>
> ├── LICENSE
>
> ├── luni ：Java 基础包、扩展包、组织提供的类库等，luni 实际上是 lang、util、net 、 io 这 4 个 内 容 头 字 母 的 组 合 ， 通 过 文 件luni/src/module/java/module-info.java 可以导出的所有类
>
> ├── metrictests
>
> ├── mmodules
>
> ├── NativeCode.bp
>
> ├── non_openjdk_java_files.bp
>
> ├── NOTICE
>
> ├── nullability_annotated_classes.txt
>
> ├── nullability_warnings.txt
>
> ├── ojluni：Open JDK 版的 Java 包，android7.0 之后就开始用这个开源的 java api
>
> ├── openjdk_java_files.bp
>
> ├── OWNERS
>
> ├── PREUPLOAD.cfg
>
> ├── support
>
> ├── TEST_MAPPING
>
> ├── test-rules
>
> ├── tools
>
> └── xml：一种轻量级的 xml 解析工具

### uboot编译

#### uboot简介

uboot 是一段裸机代码，它的实现非常复杂，主要是初始化一些硬件，部署整个计算机系统，将内核读到内存，根据环境变量去启动内核，并向内核传递参数。它的目标就是启动内核，内核启动后它的生命也随之结束。

U-boot 源码可以从官网获取，但是有的芯片厂家没有把修改过的源码开源到官网中，所以某些芯片的 u-boot 不能直接从网站获取，一般半导体厂家都会维护一个版本的 uboot，我们在进行 uboot 的时候都是在这个基础上来进行移植的，

主要有俩点原因：

第一点就是对芯片支持的比较全面

第二点是移植起来非常简单，可以提高我们开发的速度。

RK 原厂的最新的 sdk 不提供个人下载服务，可以在网盘资料下载瑞芯微提供的 Android11 源码。

uboot 版本是 v2017.09。目前瑞芯微提供的 uboot 源码支持的功能如下所示：

> 支持 RK Android 固件启动;
>
> 支持 Android AOSP 固件启动;
>
> 支持 Linux Distro 固件启动;
>
> 支持 Rockchip miniloader 和 SPL/TPL 两种 Pre-loader 引导;
>
> 支持 LVDS、EDP、MIPI、HDMI、CVBS、RGB 等显示设备;
>
> 支持 eMMC、Nand Flash、SPI Nand flash、SPI NOR flash、SD 卡、U 盘等存储设备启动;
>
> 支持 FAT、EXT2、EXT4 文件系统;
>
> 支持 GPT、RK parameter 分区表;
>
> 支持开机 LOGO、充电动画、低电管理、电源管理;
>
> 支持 I2C、PMIC、CHARGE、FUEL GUAGE、USB、GPIO、PWM、GMAC、
>
> eMMC、NAND、Interrupt 等;
>
> 支持 Vendor storage 保存用户的数据和配置;
>
> 支持 RockUSB 和 Google Fastboot 两种 USB gadget 烧写 eMMC;
>
> 支持 Mass storage、ethernet、HID 等 USB 设备;
>
> 支持通过硬件状态动态选择 kernel DTB;

#### Uoot 初次编译

进入 u-boot 文件夹中,make clean。

输入以下命令设置 java 版本为 1.8 版本，确认 java 版本是 1.8 版本之后，才可以进行下一步编译

> source javaenv.sh
>
> java -version

输入命令配置 Android 分支

> source build/envsetup.sh
>
> lunch rk3568_r-userdebug

输入以下命令查看 uboot 默认配置文件：

> get_build_var PRODUCT_UBOOT_CONFIG

输入以下命令编译 uboot

> ./build.sh -U

可以看出编译完成之后，uboot 源码多了一些文件，u-boot.bin 就是编译出来的二进制文件。Uboot 是个裸机程序，因此需要在其前面加上头部（IVT，DCD等数据）才能在 rk3568 上执行，u-boot.img 文件就是添加头部以后的 u-boot.bin，u-boot.img 就是我们最终要烧写到开发板中的 uboot 镜像文件。

烧录到开发板（略）

#### Uboot 启动信息详解

等待镜像烧写完成，使用调试串口连接电脑，打开调试终端，设置好串口参数并且打开，最后上电开发板。日志打印如下，我们来看一下日志信息。

> //uboot 的版本号和编译的时间，可以看出现在的 uboot 版本是 2017.9 ,编译时间
>
> 是 Jun 28 2022 - 22:51:17
>
> U-Boot 2017.09(Jun 28 2022 - 22:51:17 -0700)
>
> Model: Rockchip RK3568 Evaluation Board
>
> PreSerial: 2, raw, 0xfe660000
>
> DRAM: 2 GiB //内存 2GB
>
> ...
>
> //当日志出现 Hit key to stop autoboot('CTRL+C')时
>
> //按下键盘上的 ctrl+C，会进入 uboot 的命令行模式，如果没有按下，uboot 会启
>
> 动默认的参数来启动 Linux 内核
>
> Hit key to stop autoboot('CTRL+C'): 0
>
> ...

Uboot 进入命令行模式如下所示,按下键盘上的 ctrl+C，会进入 uboot 的命令行模式。当进入命令行模式以后，左侧会出现一个“=> ”标志。

#### Uboot 命令

进入 uboot 命令行，输入“？”或者“help”,然后输入回车，可以查看当前 uboot 支持的命令。

比如使用某个命令，需要了解某命令的详细用法，可以输入“？ 命令名”来查询。比如，想要了解“mmc”命令的详细信息，则输入“? mmc”。

UBoot 的命令有很多，而且还可以自定义进行添加，下面介绍常用的几个。

##### 环境变量命令

1. 在 uboot 命令行输入“printenv”即可查看环境变量，也可以简写输入“pri”。

2. 修改环境变量涉及“setenv”和“saveenv”命令，首先要使用“setenv”命令用于设置环境变量，然后使用“saveenv”命令用于保存修改之后的环境变量。

   > setenv 环境变量名 环境变量值

   例如，设置例如设置自启动倒计时环境变量 bootdelay，设置为 5 秒开发板默认的 bootdelay 值为 0。

   > setenv bootdelay 5
   >
   > saveenv

   默认 uboot 不支持 saveenv 命令，在后续的章节中讲解如何配置。修改之后，输入“pri”命令查看 bootdelay，已经修改 bootdelay=5。如果不保存环境变量，重启后会恢复默认值。

3. 删除环境变量使用命令“setenv”，如果要删除环境变量，那么只需要给这个变量赋空值即可。命令格式如下：

   > setenv 环境变量名
   >
   > saveenv

   比如删除上面新建的 bootdelay 环境变量，使用命令“setenv bootdelay”删除环境

   变量，然后输入命令“saveenv”保存环境变量。实验完成后，重新启动开发板恢复

   环境变量使设置的环境失效，因为环境变量没有保存到 flash。

4. run 命令可以执行环境变量中的命令，这个命令一般用于用户运行自定义的环境变量。

##### MMC 设备使用命令

MMC 为 MultiMedia Card，多媒体存储卡，但后续泛指一个接口协定（一种卡式），能符合这接口的内存器都可称作 mmc 储存体。EMMC 和 SD 卡属于mmc 储存体，uboot 支持操作 mmc 命令。在 uboot 命令行，输入“mmc”可以查看跟 mmc 相关的命令

由打印可以发现，mmc 后面的参数不同，mmc 的功能就不一样，例如：

> mmc info：打印 mmc 的信息
>
> mmc list ：查看 mmc 设备
>
> mmc read addr blk# cnt：从 eMMC 读 cnt 个块数据到内存 addr 处；
>
> mmc write addr blk# cnt：把内存 addr 处的 cnt 个块数据写到 eMMC。
>
> mmc erase blk# cnt：擦除 blk#开始的 cnt 个数据块
>
> 其中，addr 为内存地址，blk#是 mmc 的块号，cnt 是设备块的个数，块的单位是512 字节。

**mmc info 命令**

mmc info 命令可以查看设备信息

> Device: sdhci@fe310000 
>
> //设备节点
>
> Manufacturer ID: 15
>
> OEM: 100
>
> Name: AJTD4
>
> Timing Interface: HS200 
>
> //速度模式
>
> Tran Speed: 200000000 
>
> //当前速度
>
> Rd Block Len: 512
>
> MMC version 5.1
>
> High Capacity: Yes
>
> Capacity: 14.6 GiB //存储容量
>
> Bus Width: 8-bit  //总线宽度
>
> Erase Group Size: 512 KiB
>
> HC WP Group Size: 8 MiB
>
> User Capacity: 14.6 GiB WRREL
>
> Boot Capacity: 4 MiB ENH
>
> RPMB Capacity: 4 MiB ENH

通过上图我们可以看出，当前的设备是 emmc 设备，也就是图中 MMC version所表示的信息为 emmc 的版本为 5.1，Capacity 所表示的信息为容量大小为14.6Gib，Bus Width 表示带宽为 8-bit，Tran Speed 表示速度为 200000000hz。

**mmc list 命令**

mmc list 查看 mmc 设备，直接输入 mmc list 即可

> dwmmc@fe2b0000: 1 (SD)
>
> dwmmc@fe2c0000: 2
>
> sdhci@fe310000: 0

**mmc rescan 命令**

mmc rescan可以扫描开发板上的emmc 设备，直接输入mmc rescan命令即可。

**mmc dev 命令**

mmc dev 命令可以切换 mmc 设备，命令格式：mmc dev mmc dev [dev] [part]，其中 dev 是要切换到的 mmc 设备号，part 可以不写，不写的话默认分区为 0。

我们找一张 FAT32 格式的 TF 卡，插到开发板上的 TF 卡座子上，TF 卡也是mmc 设备。所以也可以用 mmc 命令来操作。连接好 TF 卡以后，使用以下命令切换到 TF 卡，其中 1 为 sd 卡，0 为 emmc。

> mmc dev 1

然后使用 mmc info 命令查看是否切换成功，结果如下

> => mmc dev 1 
>
> //切换到 SD 卡，如果要切换到 emmc，则 1 改成 0
>
> switch to partitions #0, OK
>
> mmc1 is current device 
>
> => mmc info 
>
> //查看设备信息
>
> Device: dwmmc@fe2b0000
>
> Manufacturer ID: 82
>
> OEM: 4a54
>
> Name: NCard
>
> Timing Interface: Legacy
>
> Tran Speed: 52000000
>
> Rd Block Len: 512
>
> SD version 3.0
>
> High Capacity: Yes
>
> Capacity: 3.7 GiB
>
> Bus Width: 4-bit
>
> Erase Group Size: 512 Bytes

从上图我们可以看到，sd 卡的版本信息为 3.0，容量为 14,8GiB，位宽为 4-bit。

**mmc part 命令**

> mmc part 可以用来查看 mmc 设备的分区，比如我们要查看 emmc 的分区，我们先切换到 emmc 设备（如果当前已经是 emmc 设备则不用切换）

> mmc dev 0

使用命令 mmc part 查看 emmc 的分区

我们可以看出，emmc 一共有 8 个分区，类型为 EFI。

第一个分区名字为 uboot，也就是用来存放 uboot 镜像，扇区范围为0x00004000~0x00005fff。

第 二 个 分 区 名 字 为 misc ， 用 来 存 放 misc 镜 像 ， 扇 区 范 围 为0x00006000~0x00007fff。

第 三 个 分 区 的 名 字 为 boot ， 用 来 存 放 内 核 镜 像 ， 扇 区 范 围 为0x00008000~0x00017fff。

第四个分区名字为 recovery，用来 recover 镜像，扇区范围为 0x00018000 ~0x00027fff

第五个分区名字为 backup，用来放 backup 镜像，扇区范围为 0x00028000 ~0x00037fff

第六个分区名字为 rootfs，用来放 rootfs 镜像，扇区范围为 0x00038000 ~0x00c37fff

第七个分区名字为 oem，用来放 oem 镜像，扇区范围为 0x00c38000 ~ 0x00c77fff

第八个分区名字为 userdata，用来放 userdata 镜像，扇区范围为 0x00c78000 ~0x01d1efbf

**mmc read 命令**

使用 mmc read 命令可以读取 mmc 设备的数据信息

> mmc read addr blk# cnt

其中 addr 是读取到内存中的地址，blk 为要读取的块起始地址，一般为十六进制，一块是 512 字节，cnt 是要读取的块的数量。块和扇区是一个意思，在 MMC 设备中，通常叫做扇区。

例如：mmc read 0x00008000 10 10 ，表示 mmc 设备的第 16（0x10）块开始，读取 16（0x10）个块到内存的 0x00008000 地址当中去。结果如下：

> => mmc read 0x00008000 10 10
>
> MMC read: dev # 0, block # 16, count 16 ... 16 blocks read: OK
>
> =>

**mmc write 命令**

使用 mmc write 命令可以向 mmc 设备写入数据信息

> mmc write addr blk# cnt

其中 addr 是内存中的地址，blk 为起始的块地址，一般为十六进制，一块是 512字节，cnt 是要读取的块的数量。

例如：mmc write 0x00008000 10 10 ，表示 mmc 设备的第 16（0x10）块开始，将内存地址为 0x00008000 的数据写入 16（0x10）块到 mmc 设备中。结果如下：

> => mmc write 0x00008000 10 10
>
> MMC write: dev # 0, block # 16, count 16 ... 16 blocks written: OK
>
> =>

**mmc erase 命令**

如果想要擦除 MMC 设备，mmc erase 命令格式如下：

> mmc erase blk# cnt

blk 为要擦除的起始块，cnt 是要擦除的数量。注意！！！擦除 MMC 设备要谨慎

##### DRAM 读写操作

直接对 DRAM 进行读写操作，常用的命令是 md,nm,mm,mw,cp 和 cmp。

**md 命令**

md 命令用于显示内存值

> => md
>
> md - memory display
>
> Usage:
>
> md [.b, .w, .l, .q] address [# of objects] 

命令中的[.b .w .l]对应 byte、 word 和 long，也就是分别以 1 个字节、 2 个字节、 4 个字节来显示内存值。

address 是要查看的内存起始地址

[# of objects]表示要查看的数据长度，比如说想要查看的内存长度是 16（十六进制是 0x10）,如果[.b .w .l]为 byte,数据长度是 16 个字节，如果[.b .w .l]为 word , 数据长度是 16 个 word，如果[.b .w .l]为 long,数据长度是 16 个 long，也就是16*4=64 字节。

**在 uboot 中，写入的数字都是十六进制！**

比如想要查看 0x00008000 开始的 16 个字节的内存值，显示格式为.b 的话，使用如下所示命令：

>md.b0x00008000 10

![image-20240713141334008](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713141334008.png)

**nm 命令**

nm 命令用于修改指定地址的内存值。

> => nm
>
> nm - memory modify (constant address)
>
> Usage:
>
> nm [.b, .w, .l, .q] address 
>
> =>

命令中的[.b .w .l]对应 byte、 word 和 long，也就是分别以 1 个字节、 2 个字节、 4 个字节来显示内存值。

address 是要查看的内存起始地址

比如现在以.l 的格式修改 0x00008000 地址的数据为 0x12345678,则输入以下命令：

> nm.l 00008000

![image-20240713141659294](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713141659294.png)

如上图所示，b9400c01 表示地址 00008000 现在的数据，？后面有光标显示，可以输入要修改的数据，输入完成之后按下回车，然后输入“q”可以退出.

![image-20240713141723293](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713141723293.png)

**mm 命令**

mm 命令也是修改指定地址内存值的，使用 mm 修改内存值的时候地址会自增，而使用命令 nm 的话地址不会自增。

> => mm
>
> mm - memory modify (auto-incrementing address)
>
> Usage:
>
> mm [.b, .w, .l, .q] address 
>
> =>

**mw 命令**

命令 mw 是使用一个指定的数据填充一段内存

> => mw
>
> mw - memory write (fill)
>
> Usage:
>
> mw [.b, .w, .l, .q] address value [count] 
>
> =>

mw 命令以.b、 .w 和.l 来指定操作格式address 表示要填充的内存起始地址，value 为要填充的指定的数据， count 是填充的长度。

举例来说，使用.l 格式将以 00008000 为起始地址的 0x10 个内存块(0x10 *4=64 字节)填充为 0X0A0A0A0A，命令如下：

> mw.l 00008000 0A0A0A0A 10

修改完成之后可以使用 md 查看修改后的值。

**cp 命令**

cp 命令格式如下，是数据拷贝命令

> => cp
>
> cp - memory copy
>
> Usage:
>
> cp [.b, .w, .l, .q] source target count 
>
> =>

cp 命令以.b、 .w 和.l 来指定操作格式

source 为源地址，target 为目的地址，count 为拷贝的长度。

**cmp 命令**

cmp 用于比较两段内存的数据是否相等

> => cmp
>
> cmp - memory compare
>
> Usage:
>
> cmp [.b, .w, .l, .q] addr1 addr2 count 
>
> =>

cmp 命令以.b、.w 和.l 来指定操作格式，addr1 为第一段内存首地址，addr2 为第二段内存首地址，count 为要比较的长度。

##### 其它常用指令

**reset 命令**

输入 reset 命令可重启开发板

**go 命令**

go 命令可以指定跳到地址处运行，命令格式为 go addr ，其中 addr 是应用在内存中的首地址

**run 命令**

可以执行环境变量中的命令，这个命令一般用于用户运行自定义的环境变量。

### uboot源码

Uboot 源码未编译目录如下图所示：

![image-20240713143330042](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713143330042.png)

我们在分析 uboot 源码之前一定要先在 Ubuntu 中编译一下 uboot 源码，因为编译过程会生成一些文件，而生成的这些恰好是分析 Uboot 源码不可或缺的一部分。编译之后 uboot 目录如下图所示：

![image-20240713143358152](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713143358152.png)

由上图可以看出编译之后的 uboot 要比没编译之前多了好多文件，这些文件夹或者文件的含义如下表所示：

> api 与硬件无关的 API 函数
>
> arch 与架构体系
>
> board 不同板子（开发板）的定制代码
>
> cmd 命令相关代码
>
> common 通用代码
>
> configs 配置文件
>
> disk 磁盘分区相关代码
>
> doc 文档
>
> drivers 驱动代码
>
> dts 设备树
>
> examples 示例代码
>
> fs 文件系统
>
> include 头文件
>
> lib 库文件
>
> Licenses 许可证相关文件
>
> net 网络相关代码
>
> post 上电自检程序
>
> scripts 脚本文件
>
> test 测试代码
>
> tools 工具文件夹
>
> .config  配置文件，重要的文件    (编译生成的文件)
>
> .gitignore git 工具相关文件。 
>
> Uboot 自带
>
> .mailmap 邮件列表
>
> Kbuild 用于生成一些和汇编有关的文件。 
>
> Kconfig 图形配置界面描述文件。 
>
> MAINTAINERS 维护者联系方式文件。
>
> make.sh 一个 shell 脚本，帮助编译 uboot 的。
>
> Makefile 主 Makefile，重要文件！
>
> README 相当于帮助文档。
>
> System.map  系统映射文件 (编译出来的文件)
>
> u-boot  编译出来的 u-boot 文件
>
> u-boot.xxx(一系列) 生成的一些 u-boot 相关文件，包括u-boot.bin,u-boot.imx 等。

### Uboot 启动流程

本小节我们将讲解瑞芯微处理器通用引导流程，包括我们在瑞芯微平台可能使用的镜像的技术细节。

方式一：使用 upsream 的 Uoot TPL/SPL 或者瑞芯微的 U-boot，全开源代码。

方式二：使用瑞芯微 idbLoader，它由 Rockchip ddr init bin 和 Rockchip rkbin项目的 miniloader bin 组合而成;

Boot 启动阶段如下图所示：

![image-20240713143955783](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713143955783.png)

当从 eMMC / SD / U-Disk / net 启动时，它们具有不同的概念：

阶段 1 始终处于引导 ROM 中，它加载阶段 2 并可能加载阶段 3（启用SPL_BACK_TO_BROM 选项时）。

从 SPI 闪存启动是指 SPI 闪存中第 2 级和第 3 级（仅限 SPL 和 U-Boot）的固件，以及其他地方的阶段 4/5 的固件;

从 eMMC 启动是指 eMMC 中的所有固件（包括阶段 2、3、4、5）;

从 SD 卡启动是指 SD 卡中的所有固件（包括阶段 2，3，4，5）;

从 U 盘启动是指磁盘中阶段 4 和 5（不包括 SPL 和 U-Boot）的固件，

可选择仅包括阶段 5;

从 net/tftp 启动是指网络上第 4 阶段和第 5 阶段（不包括 SPL 和U-Boot）的固件;

ArmV8 架构典型引导流程如下图所示：

![image-20240713144035556](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144035556.png)

> Boot Flow 1 是典型的 Rockchip Boot Flow with Rockchip miniloader;（RK 定制的引导流程，不开源，只提供二进制文件）
>
> BootRom->idbloader.img（miniloader）->uboot.img->boot.img->rootfs
>
> Boot Flow 2 用于大多数 SoC，其中 U-Boot TPL 用于 ddr init，SPL 用于信任（ATF/OP-TEE） 加载，并运行到下一阶段;
>
> BootRom->idbloader.img（TPL 和 SPL）->uboot.img->boot.img->rootfs

> 附注 1：如果 loader1 具有多个阶段，程序将返回到 bootrom 和 bootrom 加载并运行到下一个阶段。例如。如果 loader1 是 tpl 和 spl，bootrom 将首先运行到 tpl，tpl init ddr 并返回到 bootrom，bootrom 然后加载并运行到spl。
>
> 附注 2：如果启用了 trust，loader1 需要同时加载 trust.img 和 u-boot，然后在安全模式下运行 trust.img（armv8 中的 EL3），trust.img 进行初始化并在非安全模式下运行到 U-Boot（armv8 中的 EL2）。
>
> 附注 3：对于 trust（在 trust.img 或 u-boot.itb 中），armv7 只有一个 tee.bin有或没有 ta，armv8 有 bl31.elf 和带有 bl32 的选项。
>
> 附注 4：在 boot.img 中，内容可以是 zImage 及其用于 Linux 的 dtb，并且可以选择是 grub.efi，并且可以是 AOSP boot.img，ramdisk 是选项;

**CPU 启动的时候最先执行的是固化在 cpu 上的代码**

在上图中

> Bootrom:做最开始的初始化工作
>
> Atf: 初始化中断
>
> Uboot：初始化外设。最终目的是引导内核
>
> Linux 内核：加载设备驱动
>
> Rootfs：文件系统启动

### Uboot 移植

瑞芯微官方 uboot 中默认是瑞芯微自己的开发板，我们可以直接在瑞芯微官方的开发板上修改，使得 uboot 可以完整地运行在迅为 iTOP-RK3568 开发板上。但是从学习的角度来讲，这样我们就不能学习 uboot 是如何添加新平台的。接下来我们参考瑞芯微官方的rk3568_evb开发板学习如何在uboot中添加我们的开发板或者开发平台。

#### 添加开发板默认配置文件

先在 u-boot/configs 目录下创建默认配置文件，复制 rk3568_defconfig，然后重新命名为 itop_rk3568_defconfig，命令如下所示：

> cd configs/
>
> cp rk3568_defconfig itop_rk3568_defconfig

然后修改编译脚本 make.sh，修改为如下图所示：

![image-20240713144446853](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144446853.png)

然后修改 device/rockchip/rk356x/BoardConfig.mk 文件，修改PRODUCT_UBOOT_CONFIG 的值为 itop_rk3568，如下图所示：

![image-20240713144503970](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144503970.png)

#### 添加开发板默认设备树

先在 u-boot/arch/arm/dts/目录下创建默认配置文件，复制 rk3568-evb.dts ，然后重新命名为 itop_rk3568-evb.dts，命令如下所示：

> cp rk3568-evb.dts itop_rk3568-evb.dts

然后修改 itop_rk3568_defconfig 文件，设置默认的设备树为itop_rk3568-evb.dts，如下图所示：

![image-20240713144550611](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144550611.png)

#### 其他需要修改的地方

在 Uboot 启动信息中会有“Rockchip RK3568 Evaluation Board”这一句，这也就是说板子名字为“Rockchip RK3568 Evaluation ”，要将其改为我们所用底板的名字，比如“TOPEET RK3568 Evaluation Board”，打开u-boot/arch/arm/dts/itop_rk3568-evb.dts 设备树，修改为如下图所示：

![image-20240713144615819](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144615819.png)

重新编译 uboot 源码，烧写 uboot 镜像后，Board 变成了“TOPEET RK3568Evaluation Board”，如下图所示：

![image-20240713144631123](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144631123.png)

#### 修改开机倒计时

修改 rk_android11.0_sdk_211130/u-boot/configs/itop_rk3568_defconfig 文件，找到 CONFIG_BOOTDELAY 变量，如下所示，可以看到开机倒计时是 0。

![image-20240713144657309](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144657309.png)

修改 CONFIG_BOOTDELAY 的值为 10，如下所示：

重新编译 uboot 源码，将编译好的 uboot 镜像，烧写到开发板中，然后进入uboot 模式下，输入 pri 命令，查看 bootdelay 的值，其值为 10，如下图所示：

![image-20240713144720564](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144720564.png)

我们也可以在 uboot 中设置环境变量 bootdelay，然后保存环境变量，uboot默认不支持保存环境变量 saveenv 的命令，首先我们先配置 uboot，支持 saveenv命令。

进入 uboot 目录下，输入 make menuconfig 命令

![image-20240713144743572](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144743572.png)

Uboot 配置界面，如下图所示：

![image-20240713144757882](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144757882.png)

依次进入如下路径，勾选如下所示：

> Environment --->
>
> Select the location of the environment
>
> (X) Environment in a block device

![image-20240713144837417](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144837417.png)

勾选完毕，退出配置界面，点击“Exit”。

> cp .config configs/itop_rk3568_defconfig

然后重新编译 uboot 源码（u-boot/make.sh -my_rk3568），并烧写 uboot 镜像，系统启动后进入 uboot 模式.

输入以下命令设置环境变量 bootdelay 为 15，然后保存环境变量。

![image-20240713144929998](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144929998.png)

输入“pri”查看环境变量，可以发现 bootdelay 的值被修改为 15。

![image-20240713144943838](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713144943838.png)

#### 修改默认波特率

瑞芯微源码默认设置的波特率是 150000，一般我们使用波特率是 115200。

首先我们修改 ddr.bin 的波特率。

1. 首先我们需要确认打包 bin 文件。打包的脚本是在 uboot 的 config 中指定的，如果没有指定，默认是 RK3568MINIALL.ini。我们发现默认的配置文件itop_rk3568_defconfig 里面并没有指定打包脚本，所以 RK3568MINIALL.ini 是打包脚本，打开 rkbin/RKBOOTRK3568MINIALL.ini 文件，如下所示：

   ![image-20240713145056122](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145056122.png)

   由上图可知，FlashData 是 bin/rk35/rk3568_ddr_1560MHz_v1.10.bin

2. 将 bin/rk35/rk3568_ddr_1560MHz_v1.10.bin 拷贝到 rkbin/tools 目录下，如下图所示：

   ![image-20240713145151511](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145151511.png)

3. 修改参数。只修改需要修改的参数，其他参数不要去改。如果修改串口波特率的，修改 ddrbin_param.txt 中的 uart baudrate=115200，如下图所示：

   ![image-20240713145228468](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145228468.png)

4. 执行命令修改 rk3568_ddr_1560MHz_v1.10.bin，执行以下命令：

   > ./ddrbin_tool ddrbin_param.txt rk3568_ddr_1560MHz_v1.10.bin

   ![image-20240713145256022](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145256022.png)

5. 然后将修改后的 rk3568_ddr_1560MHz_v1.10.bin 拷贝回 rkbin/bin/rk35 目录下，输入以下命令：

   > cp rk3568_ddr_1560MHz_v1.10.bin ../bin/rk35/

   ![image-20240713145323374](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145323374.png)

6. 然后修改 uboot 的波特率，修改/home/topeet/Android/rk_android11.0_sdk_211130/u-boot/configs/itop_rk3568_defco

   nfig，修改如下所示，将 150000 修改为 115200。

   ![image-20240713145346453](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145346453.png)

   重新编译uboot，生成新的loader。然后重新烧写MiniLoaderAll.bin和uboot.img。

### Uboot 参数

uboot 的修改部分完成，uboot 移植也完成了，uboot 的最终目的是启动 Linux，所以我们需要通过启动 Linux 内核来判断 uboot 移植是否成功。在启动 Linux 内核之前我们先来学习俩个重要的环境变量 bootcmd 和 bootargs

#### 环境变量 bootargs

bootargs 是 U-Boot 向 kernel 传递参数的一个重要手段,诸如传递启动存储, 设备状态等。如果在移植的时候内核传参不对，或者没有传参，内核就有可能不启动。

我们进入 uboot 模式下，输入“pri”命令，bootargs 参数如下图所示：

![image-20240713145500494](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145500494.png)

#### 环境变量 bootcmd

bootcmd 保存着 uboot 默认命令，uboot 倒计时结束以后就会执行 bootcmd中的命令。这些命令一般是用来启动 Linux 内核的，比如读取 EMMC 中的 Linux内核镜像文件和设备树文件到 DRAM 中，然后启动内核，可以在 uboot 启动以后进入命令行设置 bootcmd 环境变量的值。

#### 内核启动参数 cmdline

目前 cmdline 参数有多个来源,由 U-Boot 进行拼接、过滤重复数据之后再传给 kernel。U-Boot 阶段的 cmdline 被保存在 bootargs 环境变量中。U-Boot 最终是通过修改的 kernel DTB 里的 /chosen/bootargs 实现 cmdline 传递。

cmdline 是 uboot 引导内核启动时传递给内核的，作用是指导内核启动，内核启动阶段会去解析 cmdline,并根据 cmdline 去指导内核启动。

cmdline 格式是由很多个项目用空格隔开依次排列，每个项目中都是项目名= 项目值。整个 cmdline 会被内核启动时解析，解析成一个一个的项目名=项目值的字符串。这些字符串又会被再次解析从而影响启动过程。

当内核启动之后，cmdline 命令在启动过程中如下所示：

![image-20240713145607142](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145607142.png)

我们也可以进入文件中，输入以下命令查看 cmdline。

> su
>
> cat /proc/cmdline

![image-20240713145635009](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713145635009.png)

cmdline 参数说明

> **storagemedia=emmc**
>
> **androidboot.storagemedia=emmc**
>
> 表示存储介质使用的是 EMMC
>
> **androidboot.mode=normal**
>
> 表示安卓系统的启动方式是正常启动方式。除了正常启动方式，还有 charger
>
> （电源充电）启动模式。
>
> **androidboot.dtb_idx=0**
>
> **androidboot.dtbo_idx=0**
>
> 表示设置的是 dtb 和 dtbo 的索引值，表示在多个设备树中用第几个设备树。
>
> **androidboot.verifiedbootstate=orange**
>
> 官方解释:
>
> On Android, the boot loader must set the androidboot.verifiedbootstate parameter
>
> on the kernel command-line to indicate the boot state. It shall use the following value:
>
> green: If in LOCKED state and the key used for verification was not by the end user. 
>
> yellow: If in LOCKED state and the key used for verification was setby the end user. 
>
> orange: If in the UNLOCKED state
>
> **androidboot.serialno=67188a9846568f84**
>
> 表示安卓序列号
>
> **androidboot.wificountrycode=CN**
>
> 表示设置 wifi 的国家码为 CN
>
> **androidboot.veritymode=enforcing**
>
> 表示验证固件的完整性
>
> **androidboot.slot_suffix=** 
>
> 表示用于 OTA 升级，选择指定是从 a 启动还是 b 启动
>
> **androidboot.baseband=N/A**
>
> 基带是哪一个，rk 没有这个功能。
>
> **console=ttyFIQ0**
>
> 定义串口
>
> **androidboot.hardware=rk30board**
>
> 表示启动设备的名字
>
> **firmware_class.path=/vendor/etc/firmware**
>
> 指定驱动放置的位置，一些不开源的驱动，如 wifi、bt、gpu 等
>
> **init=/init**
>
> 祖先进程的权限和位置
>
> **rootwait ro**
>
> 用于文件系统不能立即可用的情况,例如 emmc 初始化未完成,这个时候如
>
> 果不设置 root_wait 的话,就会 mount rootfs failed,而加上这个参数的话,则可以等
>
> 待 driver 加载完成后,在从存储设备中 copy 出 rootfs,再 mount 的话,就不会提
>
> 示失败了。ro:加载 rootfs 的属性,只读/读写
>
> **loop.max_part=7**
>
> 用来设定每个 loop 的设备所能支持的分区数目
>
> **androidboot.selinux=permissive**
>
> 有三种模式：
>
> enforcing :强制模式
>
> permissive ：宽容模式，这种模式可以用来作为 selinux 的 debug 之用。
>
> disabled: 关闭 selinux
>
> **buildvariant=userdebug**
>
> **earlycon=uart8250,mmio32,0xfe660000**
>
> 在串口节点未建立之前，指定串口及其配置
>
> **androidboot.boot_devices=fe310000.sdhci,fe330000.nandc**
>
> 表示 emmc 和 nand 的基地址

#### cmdline 参数来源

cmdline 是 U-Boot 向 kernel 传递参数的一个重要手段,诸如传递启动存储, 设备状态等。目前 cmdline 参数有多个来源,由 U-Boot 进行拼接、过滤重复数据之后再传给 kernel。U-Boot 阶段的 cmdline 被保存在 bootargs 环境变量中。U-Boot 最终是通过修改的 kernel DTB 里的 /chosen/bootargs 实现 cmdline 传递。

数据来源有以下几种方式：

**parameter.txt 文件**

如果是 RK 格式的分区表,可以在 parameter.txt 里存放 cmdline 信息,例如

> CMDLINE: console=ttyFIQ0 androidboot.baseband=N/A
>
> androidboot.selinux=permissive androidboot.hardware=rk30board
>
> androidboot.console=ttyFIQ0 init=/init
>
> mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00002000@0x00004000(trust),

如果是 GPT 格式的分区表,在 parameter.txt 里存放 cmdline 信息是无效的。

**kernel dts 的 /chosen/bootargs**

例如:

> chosen {
>
> bootargs = "earlyprintk=uart8250,mmio32,0xff30000 swiotlb=1
>
> console=ttyFIQ0
>
> androidboot.baseband=N/A androidboot.veritymode=enforcing
>
> androidboot.hardware=rk30board androidboot.console=ttyFIQ0
>
> init=/init kpti=0";
>
> };

**U-Boot:根据当前运行的状态,U-Boot 会动态追加一些内容到 cmdline。**

> storagemedia=emmc androidboot.mode=emmc ......

**boot/recovery.img 固件头里的通常也会有 cmdline 字段信息。**

**安卓源码下的 device/rockchip/common/BoardConfig.mk 文件**

#### 瑞星微Uboot 特殊机制

U-Boot 的原生架构要求一块板子必须对应一份 U-Boot 设备树,并且U-Boot 设备树生成的 dtb 是打包到 U-Boot 自己的镜像中的。这样就会出现各SoC 平台上,N 块板子需要 N 份 U-Boot 镜像。

我们不难发现,其实一个 SoC 平台不同的板子之间主要是外设的差异,SoC 核心部分是一致的。RK 平台为了实现一个 SoC 平台仅需要一份 U-Boot 镜像,因此增加了 kernel DTB 机制。本质就是在较早的阶段切到 kernel DTB,用它的配置信息初始化外设。

Uboot 有自己的设备树文件，在编译的时候会自动生成相应的 DTB 文件，被添加在 u-boot.bin 末尾，原生的 U-boot 只支持使用 U-boot 自己的 DTB，RK 平台增加了 kernel DTB 机制的支持，也就是使用内核设备树去初始化外设，比如，供电 时钟 显示等等，这样做的主要目的是为了兼容外设板极差异。

> U-boot 设备树：负责初始化存储，打印串口设备
>
> Kernel 设备树：负责初始化存储，打印串口以外的设备

开发者一般不需要修改 U-boot 设备树（除非更换打印串口），Uboot 中已经默认添加了对内核设备树的支持，不需要用户再去配置，kernel DTB 的启用需要依赖 OF_LIVE，如下所示：

> config USING_KERNEL_DTB
>
> bool "Using dtb from Kernel/resource for U-Boot" 
>
> depends on RKIMG_BOOTLOADER && OF_LIVE
>
> default y
>
> help
>
> This enable support to read dtb from resource and use it for U-Boot, 
>
> the uart and emmc will still using U-Boot dtb, but other devices like
>
> regulator/pmic, display, usb will use dts node from kernel.

### Android kernel 移植

#### 单独编译内核

**方法一：**

在 Android 源码目录下执行如下命令编译 Android 内核。

> ./build.sh -CKA

编译后会在 rockdev/Image-rk3568_r 目录下生成 boot.img，boot.img 为内核镜像。boot.img 镜像里面包含了设备树镜像。所以烧写 boot.img 即可更新内核镜像。

解释如下：

> 参数-C 表示用 clang 编译器编译，如果不带 C 则使用 gcc 编译，如果需要过 GMS 认证的 google 有要求要用 clang 编译。因为 kernel 编译完后需要通过 Android 去打包成 boot.img，所以这里需要加上 A 参数，即编译 kernel 的时需要一起编译 Android 才能打包 boot.img。通过以上介绍可以知道单独编译 kernel 需要同时编译 Android，导致编译很耗时，针对这个问题，我们推荐使用方法二来单独编译内核。 

**方法二(推荐)**

此方法常用于 kernel 的开发和调试，以下的方法既编译 kernel 部分时， 同时打包成 boot.img， 这样加快了我们开发的速度；进入内核目录下， 输入以下命令：

> cd kernel
>
> make ARCH=arm64 CC=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/clang
>
> LD=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/ld.lld rockchip_defconfig
>
> android-11.config && make ARCH=arm64
>
> CC=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/clang
>
> LD=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/ld.lld
>
> BOOT_IMG=../rockdev/Image-rk3568_r/boot.img rk3568-evb1-ddr4-v10.img

为方便使用，可以将上述命令写成脚本，在 kernel 目录下创建 makekernel.sh, 

在调试的过程中直接在 kernel 目录下执行该脚本，makekernel.sh 内容为：

> \#!/bin/sh
>
> make ARCH=arm64 CC=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/clang
>
> LD=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/ld.lld rockchip_defconfig
>
> android-11.config && make ARCH=arm64
>
> CC=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/clang
>
> LD=../prebuilts/clang/host/linux-x86/clang-r383902b/bin/ld.lld
>
> BOOT_IMG=../rockdev/Image-rk3568_r/boot.img rk3568-evb1-ddr4-v10.img

我们来解释以上命令是什么意思

> 1. BOOT_IMG是指定前置的boot.img，因为boot.img镜像里面不单独只有kernel和 resource，还有其他文件，所以要指定个 boot.img 把新的 kernel 和 resource 覆盖进去，boot.img 的位置在所编译出来的 rockdev/Image-rk3568_r/目录下；
>
> 2. rk3568-evb1-ddr4-v10.img 是指定所使用的设备树 DTS；
>
> 3. 注意：如果不指定 BOOT_IMG，会导致在下载后，系统会跑进了 Recovery模式(或者引起其他启动错误)，而不是进入正常的启动流程；

打包完后，在 kernel 目录有 boot.img 镜像生成，就可以把这个 boot.img 镜像单独烧入到机器中进行调试了。

#### 内核本地化

通过本章的学习，我们将掌握将半导体厂商提供的内核源码移植到我们自己的平台上。瑞芯微提供的内核源码是在瑞芯微官方的 rk3568_evb 开发板上运行的，所以我们肯定是以 rk3568_evb 开发板作为参考，然后将内核源码移植到iTOP-RK3568 开发板上。

**添加开发板默认配置文件**

1. 输入以下命令使能编译环境：

   > source javaenv.sh
   >
   > source build/envsetup.sh
   >
   > lunch rk3568_r-userdebug

2. 首先查看默认的内核配置，我们输入以下命令

   > get_build_var PRODUCT_KERNEL_CONFIG

   ![image-20240713151015281](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151015281.png)

   由上图可知，内核默认的配置文件为 rockchip_defconfig 和 android-11.config

3. 确定俩个配置文件的绝对路径，输入以下命令查找

   > find -name rockchip_defconfig
   >
   > find -name android-11.config

   ![image-20240713151050004](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151050004.png)

   由上图可知，因为 rk3568 是 arm64 架构，所以 rockchip_defconfig 的绝对路径是kernel/arch/arm64/configs/rockchip_defconfig。但是另一个确定不了，我们可以编译 内 核 看 看 是 否 有 相 关 打 印 信 息 ， 编 译 打 印 日 志 ， 如 下 所 示 ， 所 以android-11.config 的绝对路径是 kernel/kernel/configs/android-11.config

   ![image-20240713151118008](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151118008.png)

4. 将 rockchip_defconfig 和 android-11.config 重新复制一份，分别命名为itop_rockchip_defconfig 和 itop_android-11.config，命令如下：

   > cd kernel/arch/arm64/configs/
   >
   > cp rockchip_defconfig itop_rockchip_defconfig
   >
   > cd - 
   >
   > cd kernel/kernel/configs/
   >
   > cp android-11.config itop_android-11.config

   ![image-20240713151145316](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151145316.png)

5. 我们进入 device 目录下，配置编译规则，然后查找 rockchip_defconfig，命令如下：

   > cd device
   >
   > grep "rockchip_defconfig" -rw

   ![image-20240713151229092](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151229092.png)

   由上图可知 rockchip/rk356x/BoardConfig.mk:PRODUCT_KERNEL_CONFIG ?=rockchip_defconfig 是有用信息。打开 device/rockchip/rk356x/BoardConfig.mk 文件，修改为如下图所示：

   ![image-20240713151257345](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151257345.png)

6. 然后我们进入 device 目录下，查找“android-11.config”，输入如下命令：

   ![image-20240713151317357](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151317357.png)

   由上图可知 rockchip/common/BoardConfig.mk:PRODUCT_KERNEL_CONFIG +=android-11.config 是有用信息。打开 device/rockchip/common/BoardConfig.mk 文件，修改为如下所示：

   ![image-20240713151339364](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151339364.png)

**添加开发板默认设备树**

添加适合迅为开发板的设备树文件，进入 kernel/arch/arm64/boot/dts/rockchip/目录下，复制一份 rk3568-evb1-ddr4-v10.dts，然后将其重命名为itop_rk3568-evb1-ddr4-v10.dts

![image-20240713151422152](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151422152.png)

添加设备树编译规则，输入以下命令查找 rk3568-evb1-ddr4-v10.dts

> cd device/
>
> grep “rk3568-evb1-ddr4-v10.dts” -rw

![image-20240713151454765](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151454765.png)

修改 rockchip/rk356x/BoardConfig.mk 文件，修改如下所示：

![image-20240713151507092](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151507092.png)

修改 rockchip/rk356x/rk3568_r/BoardConfig.mk 文件，

![image-20240713151518695](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240713151518695.png)

这样编译 linux 的时候就可以从 itop_rk3568-evb1-ddr4-v10.dts 编译出itop_rk3568-evb1-ddr4-v10.dtb 文件了。