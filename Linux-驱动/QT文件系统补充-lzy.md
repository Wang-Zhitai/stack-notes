## 补充

### **命令提示符**

```
PS1=“command list”
　　命令列表有很多参数如下：
　　\! 显示该命令的历史记录编号。
　　\# 显示当前命令的命令编号。
　　\$ 显示$符作为提示符，如果用户是root的话，则显示#号。
　　\\ 显示反斜杠。
　　\d 显示当前日期。
　　\h 显示主机名。
　　\n 打印新行。
　　\nnn 显示nnn的八进制值。
　　\s 显示当前运行的shell的名字。
　　\t 显示当前时间。
　　\u 显示当前用户的用户名。
　　\W 显示当前工作目录的名字。
　　\w 显示当前工作目录的路径
　　我们查看我们发行版linux中的PS1.
　　$echo echo $PS1
　　[\u@\h \w] \$scp -r root@192.168.106.171:/aa/07_plate_detection_recongnition/install / 
```

### **QT移植**

1. 编译一个可以运行的文件系统，并解压到某个（system）目录中

2. 编译QT源码和触摸驱动

3. 把字体文件存入编译好的QT中：/opt/qt5.15/lib

4. 把编译好的QT和触摸驱动复制到自己的文件系统中

   ```
   cp /opt/* 自己的sytem目录/opt/
   ```

5. 自己写一个QT程序

6. 把QT程序复制到自己的文件系统中

7. 在自己的文件系统中的QT程序的目录下执行编译好的QT目录下的bin目录下的qmake

8. 再执行make

9. 制文件镜像并烧录

10. 在开发板上运行自己的QT程序。

**网络配置**

1. 在buildroot中选择dhcpcd，可以实现自动获取ip地址上网

   >  Target packages  ---> 
   >
   >  Networking applications  ---> 
   >
   >  [*] dhcpcd 

2. 修改自己的文件系统etc/profile

   ```
   export PATH="/bin:/sbin:/usr/bin:/usr/sbin"
   
   if [ "$PS1" ]; then
           if [ "`id -u`" -eq 0 ]; then
                   export PS1='[\u@\h \w]# '
           else
                   export PS1='[\u@\h \w]$ '
           fi
   fi
   
   export EDITOR='/bin/vi'
   
   # Source configuration files from /etc/profile.d
   for i in /etc/profile.d/*.sh ; do
           if [ -r "$i" ]; then
                   . $i
           fi
   done
   unset i
   export USER LOGNAME
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

### Buildroot中编译QT

1. 在编译中选中QT，编译，烧录到开发板

2. 写一个QT程序

3. 在ubuntu中buildroot的目录中找qmake

4. 在QT的项目目录下执行上一步找到的qmake

5. 再make

6. 执行自己的QT程序

   ```shell
   ./xxx -platform linuxfb
   ```

7. 把字体文件存在以下位置

   > /usr/share/fonts

#### 编译瑞星微方官buildroot文件系统

1. 配置

   > make menuconfig
   >
   > make busybox-menuconfig

2. 保存配置

   >  make savedefconfig

3. 编译文件系统

   > ./build.sh buildroot

### 使用瑞星微官方Buildroot系统

1. 烧录官方Buildroot系统

2. 取消默认的QT程序

   > vi /etc/init.d/S50launcher

   ![image-20240725153107918](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240725153107918.png)

3. 旋转桌面

   > echo "output:all:rotate270" > /tmp/.weston_drm.conf

### 在QT中控制硬件

1. 通过设备模型向应用层暴露的接口，使用system发送指令来控制。

   > system("echo 1 > /sys/bus/platform/device/leds/leds/work/brightness");

2. 通过调用c编译的方式控制，比如open一个设备节点，然后write或read

3. 调用QT里面的函数来实现

### V4L2

```
//查看所有的摄像头设备
v4l2-ctl --list-devices
//查看摄像头支持的格式
v4l2-ctl --list-formats-ext -d /dev/video0
//开启摄像头预览
gst-launch-1.0 v4l2src device=/dev/video9 !video/x-raw, format=NV12, width=640, height=480,framerate=30/1 ! waylandsink
//拍照
gst-launch-1.0 v4l2src device=/dev/video0 num-buffers=1 !video/x-raw,format=NV12,width=640,height=480 ! mppjpegenc !filesink location=pic.jpg
//录视频 H264格式
gst-launch-1.0 filesrc location=5695_h264.mp4 !qtdemux !queue !h264parse !mppvideodec !waylandsink
```

### 使用4G网络

1. 插sim卡

   > 可能需要接一个卡套

2. 接三根天线

3. 停用eth0

   > ifconfig eth0 down
   >
   > ifconfig eth1 down

4. 在根目录下后台运行移远的程序

   > ./quectel-CM &

5. 测试

   > ping www.baidu.com

### 使用wifi

1. 禁用eth0和eth1

2. 修改wifi的热点名称和连接密码

   > vi /rockchip_test/wifi/wifi_sta_connect.sh
   >
   > vi /data/wifi_configure.txt
   
   ![image-20240725172713287](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240725172713287.png)

3. 执行脚本

   > ./wifi_sta_connect.sh

4. 测试

   > ping www.baidu.com

### GPS模块

1. 连接硬件

   ![image-20240726092819519](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240726092819519.png)

   ![image-20240726093922014](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240726093922014.png)

   使用串口9

   ![image-20240726092833865](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240726092833865.png)

   使用串口4

   ![image-20240726100109042](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240726100109042.png)

2. 运行测试程序

   > 交叉编译器编译程序：
   >
   > 
   >
   > 运行：
   >
   > ./a.out /dev/ttyS4
   >
   > ./a.out /dev/ttyS9

### 使用adc

> cat /sys/devices/platform/fe720000.saradc/iio:device0/in_voltage7_raw

### 安卓中运行QT程序

下载gradle失败

修改

> C:\Qt\6.7.2\android_arm64_v8a\src\3rdparty\gradle\gradle\wrapper\gradle-wrapper

> distributionBase=GRADLE_USER_HOME
> distributionPath=wrapper/dists
> distributionUrl=https\://mirrors.cloud.tencent.com/gradle/gradle-8.3-bin.zip
> networkTimeout=10000
> validateDistributionUrl=true
> zipStoreBase=GRADLE_USER_HOME
> zipStorePath=wrapper/dists

### 安装vnc

```csharp
sudo apt install ubuntu-desktop
sudo apt-get install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal 
sudo apt-get install x11vnc
sudo apt-get install lightdm
选lightdm
设置密码
x11vnc -storepasswd
sudo vim /lib/systemd/system/x11vnc.service
```

```shell
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target
 
[Service]
Type=simple
ExecStart=/usr/bin/x11vnc  -display :0  -auth /home/richard/.Xauthority -forever -loop -noxdamage -repeat -rfbauth /home/richard/.vnc/passwd -rfbport 5900 -shared
 
[Install]
WantedBy=multi-user.target
```

```shell
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc.service
sudo startx
```

### 创建 QT 程序桌面快捷方式

使用命令“cd /usr/share/applications”进入到 applications 目录

![image-20241226105130259](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20241226105130259.png)

Qlauncher 桌面程序会读取以.desktop 为结尾的文件内容， 并根据相应的内容进行 QT 桌面应用的添加。 使用以下命令对 qplayer.desktop 文件内容进行查看， 具体内容如下所示：

> vi qplayer.desktop

![image-20241226105233305](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20241226105233305.png)

2 到 5 行参数解释如下：

> Name： 在默认桌面显示的名称
> Exec： 运行 QT 程序所要用的命令（使用的是绝对路径）
> Icon： 应用的图标存放路径
> Type： QT 应用程序类型， 一般都为 Application 类型
