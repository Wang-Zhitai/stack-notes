## 4-3-3-安卓系统开发

### ADB工具

ADB（Android Debug Bridge）是 Android SDK里的一个工具，用这个工具可以操作管理 Android 模拟器或真实的 Android 设备。主要功能有：

运行设备的 shell（命令行）

管理模拟器或设备的端口映射

计算机和设备之间上传/下载文件

将本地 apk软件安装至模拟器或 Android 设备

ADB 是一个“客户端－服务器端”程序，其中客户端主要是指PC，服务器端是Android 设备的实体机器或者虚拟机。根据PC连接Android设备的方式不同，ADB 可以分为两类：

网络 ADB：主机通过有线/无线网络（同一局域网）连接到STB设备

USB ADB：主机通过 USB 线连接到STB设备

#### **USB adb使用说明**

USB adb 使用有以下限制：

只支持 USB OTG 口 

不支持多个客户端同时使用（如 cmd 窗口，eclipse等）

只支持主机连接一个设备，不支持连接多个设备

连接步骤如下：

1. Android设备已经运行 Android 系统，设置->开发者选项->已连接到计算机 打开，usb调试开关打开。

2. PC主机只通过 USB 线连接到机器 USB otg 口，然后电脑通过如下命令与Android设备相连。

> adb shell

3. 测试是否连接成功，运行”adb devices”命令，如果显示机器的序列号，表示连接成功。

#### **网络adb使用要求**

adb早期版本只能通过USB来对设备调试，从adb v1.0.25开始，增加了对通过tcp/ip调试Android设备的功能。

如果你需要使用网络adb来调试设备，必须要满足如下条件：

1、设备上面首先要有网口，或者通过WiFi连接网络。

2、设备和研发机（PC机）已经接入局域网，并且设备设有局域网的IP地址。

3、要确保研发机和设备能够相互ping得通。

4、研发机已经安装了adb。

5、确保Android设备中adbd进程（adb的后台进程）已经运行。adbd进程将会监听端口5555来进行adb连接调试。

#### **SDK网络adb端口配置**

SDK默认未开启网络adb，需要手动在开发者选项中打开。

Setting-System-Advanced-Developer options-Open net adb

#### **网络adb使用**

本节假设设备的ip为192.168.1.5，下文将会用这个ip建立adb连接，并调试设备。

1、首先Android设备需要先启动，如果可以的话，可以确认adbd是否启动(ps命令查看)。

2、在PC机的cmd中，输入：

> adb connect 192.168.1.5:5555

如果连接成功会进行相关的提示，如果失败的话，可以先kill-server命令，然后重试连接。

> adb kill-server

3、如果连接已经建立，在研发机中，可以输入adb相关的命令进行调试了。比如adb shell，将会通过

tcp/ip连接设备上面。和USB调试是一样的。

4、调试完成之后，在研发机上面输入如下的命令断开连接：

> adb disconnect 192.168.1.5:5555

#### **手动修改网络adb端口号**

若SDK未加入adb端口号配置，或是想修改adb端口号，可通过如下方式修改：

1、首先还是正常地通过USB连接目标机，在windows cmd下执行adb shell进入。

2、设置adb监听端口：

> \#setprop service.adb.tcp.port 5555

3、通过ps命令查找adbd的pid

4、重启adbd

\#kill -9，这个pid就是上一步找到那个pid

杀死adbd之后，android的init进程后自动重启adbd。adbd重启后，发现设置了

service.adb.tcp.port，就会自动改为监听网络请求。

#### **ADB常用命令详解**

（1）查看设备情况

查看连接到计算机的 Android 设备或者模拟器：

> adb devices

返回的结果为连接至开发机的 Android 设备的序列号或是IP和端口号（Port）、状态。

（2）安装apk

将指定的 apk 文件安装到设备上：

> adb install <apk文件路径>
>
> adb install “F:\WishTV\WishTV.apk”

（3）卸载apk

完全卸载：

> adb uninstall <package>
>
> adb uninstall com.wishtv

（4）使用 rm移除 apk 文件：

> adb shell rm <filepath>
>
> adb shell rm “system/app/WishTV.apk”

示例说明：移除“system/app”目录下的“WishTV.apk”文件。

（5）进入设备和模拟器的shell

进入设备或模拟器的 shell 环境：

> adb shell

（6）从电脑上传文件到设备

用 push 命令可以把本机电脑上的任意文件或者文件夹上传到设备。本地路径一般指本机电脑；远程路

径一般指 adb 连接的单板设备。

adb push <本地路径> <远程路径> 

示例如下：

> adb push “F:\WishTV\WishTV.apk” “system/app”

示例说明：将本地“WishTV.apk”文件上传到 Android 系统的“system/app”目录下。

示例说明：将本地“WishTV.apk”文件上传到 Android 系统的“system/app”目录下。

（7）从设备下载文件到电脑

pull 命令可以把设备上的文件或者文件夹下载到本机电脑中。

> adb pull <远程路径> <本地路径>
>
> adb pull system/app/Contacts.apk F:\

示例说明：将 Android 系统“system/app”目录下的文件或文件夹下载到本地“F:\”目录下。

（8）查看 bug报告

需要查看系统生成的所有错误消息报告，可以运行 adb bugreport指令来实现，该指令会将 Android 系

统的dumpsys、dumpstate 与 logcat 信息都显示出来。

（9）查看设备的系统信息

在 adb shell下查看设备系统信息的具体命令。

> adb shell getprop

#### 安卓命令

查看所有包

> pm list packages

启动一个程序

> am start -n  包/活动窗口
>
> aapt 命令查看，这个是安卓应用开发工具包中的一个工具。
>
> 打开设置
>
> adb shell am start -n com.android.settings/com.android.settings.Settings
>
> 启动高德地图
>
> am start -n com.autonavi.amapauto/com.autonavi.auto.remote.fill.UsbFillActivity

### 获取 root 权限

> 1. 使用RK3568的安卓源码
> 2. 源码编译不能使用root用户

#### 关闭 selinux

修改 device/rockchip/common/BoardConfig.mk 文件，要确保 BOARD_SELINUX_ENFORCING为 false。

![image-20240718210445942](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210445942.png)

#### 注释用户组权限检测

修改 system/extras/su/su.cpp 文件，注释掉如下图所示内容：

![image-20240718210517419](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210517419.png)

#### su 文件默认授予 root 权限

修改 system/core/libcutils/fs_config.cpp 文件，修改为如下图所示：

![image-20240718210538502](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210538502.png)

修改 frameworks/base/core/jni/com_android_internal_os_Zygote.cpp 文件，注释掉如下图所示内容：

![image-20240718210554051](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210554051.png)

修改 kernel/security/commoncap.c 文件，注释掉如下图所示内容：

![image-20240718210605792](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210605792.png)

修改完毕后重新编译 Android11 的源码并烧写到开发板上。

使用 adb 安装 root 检查 apk,

![image-20240718210631534](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210631534.png)

apk 安装成功后，打开 app 如下图所示，如果提示 root check pass，说明 root 检查通过！

![image-20240718210655678](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718210655678.png)

### 预安装应用功能

Android11 上的应用预安装功能，主要是指配置产品时，需要将提前准备好的第三方应用apk 放进 android 系统。在实际的研发过程中，经常需要将某个 apk 提升为系统应用。

预安装分为可卸载安装和不可卸载安装，以及卸载后恢复出厂设置后自动恢复预安装。

1 在编译完源码之后，输入以下命令，查看添加应用所需要的目录，如下图所示：

> get_build_var TARGET_DEVICE_DIR

![image-20240718212105459](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212105459.png)

2 进入到这个目录下，分别新建三个文件夹，如下图所示：

![image-20240718212121936](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212121936.png)

这三个文件夹分别为：

preinstall //存放不可卸载应用

preinstall_del_forever //存放可卸载应用

preinstall_del //存放卸载后恢复出厂设置复原应用

3 根据需求只需将 apk 放在对应文件夹即可，比如将 RootChecker.apk 设置为不可卸载应用，那么需要放进 preinstall 文件夹中。如下图所示：

![image-20240718212143676](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212143676.png)

4 重新编译 Android11 源码，编译完会在相应的目录下自动生成对应名字的文件。

/home/topeet/Android11/rk_android11.0_sdk/out/target/product/rk3568_r/obj/APPS/RootChecker_intermediates 如下图所示：

![image-20240718212203768](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212203768.png)

5 编译完源码之后，重新烧写镜像，就会发现刚刚预安装的 app，如下图所示：

![image-20240718212217483](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212217483.png)

### 设置语言和默认时区

修改文件：device/rockchip/rk356x/rk3568_r/rk3568_r.mk，添加如下内容：

```
PRODUCT_PROPERTY_OVERRIDES += \
    persist.sys.language=zh \
    persist.sys.country=CN \
    persist.sys.timezone=Asia/Shanghai
```

![image-20240718212310633](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212310633.png)

修改完，保存修改，重新编译 android 源码。

### 设置系统默认不锁屏

修改 frameworks/base/packages/SettingsProvider/res/values/defaults.xml 文件，修改为如下所示：

```
- <bool name="def_lockscreen_disabled">false</bool>
+ <bool name="def_lockscreen_disabled">true</bool>
```

![image-20240718212359662](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212359662.png)

修改完，保存修改，重新编译 android 源码。

### 设置系统默认不休眠

修改文件：

device/rockchip/rk356x/overlay/frameworks/base/packages/SettingsProvider/res/values/defaults.xml

文件，如下图所示：

```
- <integer name="def_screen_off_timeout">60000</integer>
+ <integer name="def_screen_off_timeout">0x7fffffff</integer>
```

![image-20240718212439646](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212439646.png)

修改完，保存修改，重新编译 android 源码。

### 旋转屏幕

本章节我们来讲解 Android11 如何旋转屏幕。屏幕在启动的过程中，首先会显示 uboot logo 和内核 logo，然后显示 Android 启动 logo，最后显示 Android 启动桌面。如果屏幕是触摸屏幕的话，还需要旋转触摸。所以我们的修改方向就很明确了。

1 旋转 uboot logo 和内核 logo

2 旋转 Android 桌面

3 旋转触摸

**旋转 uboot logo 和内核 logo**

修改设备树 rk_android11.0_sdk/kernel/arch/arm64/boot/dts/rockchip/topeet_rk3568_lcds.dtsi文件。

如果配套的屏幕是 MIPI7 寸屏幕，修改如下所示

![image-20240718212642394](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212642394.png)

其中：

logo,rotate = <1>;代表顺时针旋转 90 度，

logo,rotate = <2>;代表顺时针旋转 180 度，

logo,rotate = <3>;代表顺时针旋转 270 度，

logo,rotate = <0>;代表顺时针旋转 360 度。

### 旋转 Android 系统

屏幕旋转包括俩个部分：Android 显示旋转和 Recovery 显示旋转

在开发的过程中，我们使用的屏幕可能是物理横屏或者物理竖屏，如果我们想要物理横屏显示为竖屏，物理竖屏显示为横屏时，也就是系统显示方向需要旋转 90 度

修改 Android11 源码的 rk_android11.0_sdk/device/rockchip/common/BoardConfig.mk 文件，修改主屏显示的方向，角度可根据显示需求，自定义修改 0/90/180/270 0：横屏，90：竖屏，180：反向横屏，270：反向竖屏。作者想要将物理竖屏修改为横屏显示，所以旋转角度为 90度。修改如下所示：

```
SF_PRIMARY_DISPLAY_ORIENTATION := 90
```

![image-20240718212753314](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212753314.png)

修改 recovery 显示旋转，修改 rk_android11.0_sdk/device/rockchip/common/BoardConfig.mk文件，如下所示：

```
TARGET_RECOVERY_DEFAULT_ROTATION ?= ROTATION_RIGHT
```

![image-20240718212815223](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718212815223.png)

修改完，保存退出，重新编译 android 源码，烧写镜像后发现系统启动以后桌面是横屏显示，但是触摸还是竖屏

### 旋转触摸

触摸芯片是 ft5x06 的屏幕，修改rk_android11.0_sdk/kernel/arch/arm64/boot/dts/rockchip/topeet_rk3568_lcds.dtsi 设备树中的触摸节点，增加最后两行的代码，如下所示：

```
ft5x06:ft5x06@38 {
    status = "disabled";
    compatible = "edt,edt-ft5306";
    reg = <0x38>;
    touch-gpio = <&gpio0 RK_PB5 IRQ_TYPE_EDGE_RISING>;
    interrupt-parent = <&gpio0>;
    interrupts = <RK_PB5 IRQ_TYPE_LEVEL_LOW>;
    reset-gpios = <&gpio0 RK_PB6 GPIO_ACTIVE_LOW>;
    touchscreen-size-x = <800>;
    touchscreen-size-y = <1280>;
    touch_type = <1>;
    touchscreen-inverted-x;
    touchscreen-swapped-x-y;
};
```

### 修改 uboot logo

系统默认 uboot logo，如下图所示：

![image-20240718213037438](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718213037438.png)

如果想要替换这个 logo,首先要制作一个新的 logo.bmp，图片属性和默认的 logo.bmp 要一样，width，height 都为偶数，否则会出现颠倒异常。

将制作好的 logo 替换 android11 源码 rk_android11.0_sdk/kernel/logo.bmp 下的 logo.bmp 即可。

### 修改 kernel logo

系统默认内核 logo，如下图所示：

![image-20240718213118894](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718213118894.png)

如果想要替换这个 logo,首先要制作一个新的 logo_kernel.bmp，图片属性和默认的logo_kernel.bmp 要一样，width，height 都为偶数，否则会出现颠倒异常。

将制作好的 logo 替换 android11 源码 rk_android11.0_sdk/kernel/logo_kernel.bmp 下的logo_kernel.bmp 即可。

### 修改设备信息

**修改型号**

进入 Android11 源码目录下，输入以下命令打开文件：

```
vi build/tools/buildinfo.sh
```

在最下方添加如下代码：

```
echo "ro.product.model="MTK6737_64_bsp"" 
echo "ro.product.brand=$PRODUCT_BRAND" 
echo "ro.product.name="MTK6737_64_bsp"" 
echo "ro.product.device="MTK6737_64_bsp""
```

![image-20240718213401097](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718213401097.png)

**修改版本号**

输入以下命令，打开文件：

```
vi build/core/Makefile
```

将以下代码注释，并添加自己需要显示的版本号：

![image-20240718213449239](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240718213449239.png)

添加完成后，编译并烧写到开发板，点开设置->关于手机，即可看到修改过后的型号与版本号。