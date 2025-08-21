

**开发板执行：**

scp wangzhitai@172.172.14.109:/aa/34-CMake /

nfs 80800000 172.172.14.109:/linux/uboot-zdyz/u-boot.imx

mount -t nfs -o nfsvers=3,nolock 172.172.14.109:/aa/ /mnt/

cd /mnt/rknn-toolkit2-master/rknpu2/examples/rknn_yolov5_demo/install/rknn_yolov5_demo_Linux

./rknn_yolov5_demo

insmod /mnt/my-work/00-leddriver/led-driver.ko

cd /sys/bus/platform/devices/ledctrl

sudo scp -r wangzhitai@172.172.14.108:/aa/07_plate_detection_recongnition /

sudo scp -r root@172.172.14.108:/opt/* /opt

./plate_detection_recongnition model/RK3568/yolov5n.rknn model/RK3568/best.rknn model/test.mp4

./plate_detection_recongnition model/RK3568/best.rknn model/RK3568/yolov5n.rknn model/test.mp4

**LED驱动控制：**

echo on > /sys/devices/platform/ledctrl/mode

echo off > /sys/devices/platform/ledctrl/mode

echo fastblink > /sys/devices/platform/ledctrl/mode

echo slowblink > /sys/devices/platform/ledctrl/mode

**Ubuntu执行：**

cd /aa/rknn-toolkit2-master/rknpu2/examples/rknn_yolov5_demo

rm -rf platform-tools/

./build-linux_RK3566_RK3568.sh

> ls /dev

# 一、Linux操作系统介绍

Linux 是一种开源且免费的操作系统内核，是由芬兰计算机科学家 Linus Torvalds 于 1991  年开始编写，并在其后的几年中不断完善和发展而来。Linux  最初是作为个人电脑使用的，但现在已经成为了许多服务器、移动设备、超级计算机等各种类型的硬件设备上的主要操作系统。

嵌入式Linux的应用领域非常广泛，主要的应用领域有信息家电、PDA 、机顶盒、Digital Telephone、Answering Machine、Screen Phone 、数据网络、Ethernet Switches、Router、Bridge、Hub、Remote access servers、ATM、Frame relay 、远程通信、医疗电子、交通运输计算机外设、工业控制、航空航天领域等。

**Linux的组成：uboot（引导加载程序），kernel（系统内核）， rootfs（root file system根文件系统）**

![image-20250212171804.png](./assets/image-20250212171804.png)

# 二、开发环境搭建

> 现在主流的操作系统开发方法：编译系统需要在Ubuntu环境下进行编译，所以要先装一个具有编译环境的Ubuntu虚拟机用来开发驱动

## （一）安装ubuntu虚拟机

### 1、下载VMWare Workstation Pro

1. 进入VMware官网：[VMware by Broadcom - Cloud Computing for the Enterprise](https://www.vmware.com/)

2. 搜索产品：Desktop Hypervisor

3. 点击Products找到下载网址：[https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)

4. 注册登录

5. 点击：DOWNLOAD FUSION OR WORKSTATION

6. 左侧导航栏选择：My Dashboard

7. 搜索：VMware Workstation Pro

8. 点击：Downloading and license VMware Desktop Hypervisor

![Pasted image 20250213125534.png](./assets/image-20250213125534.png)

9. 滚动到下面，点击下载 VMware Workstation Pro

![image-20250213125550.png](./assets/image-20250213125550.png)

10. 下载并安装最新版即可

![image-20250213125916.png](./assets/image-20250213125916.png)

11. 安装时选择用于个人用途，就是免费使用的

### 2、在VMware中安装Ubuntu系统

虚拟机网络需要打开桥接模式，否则没法使用SSH和SMB服务

## （二）Ubuntu设置

### 1、修改系统时区

1. 使用命令查看当前时区

```shell
timedatectl
```

2. 修改时区到Asia/Shanghai

```shell
sudo timedatectl set-timezone Asia/Shanghai
```

### 2、启用root用户

1. 设置root用户密码

```shell
sudo passwd
```

2. 然后输入ubuntu的密码，接着输入两次Unix密码（Unix密码就是想要设置的root用户登录密码）
   
3. 输入以下命令来切换到root用户

```shell
su root
```

4. 然后输入刚刚设置的Unix密码，可以看到用户名变成root说明已经启用成功了。

### 3、更换镜像源

1. 先执行以下指令备份系统默认的镜像源

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

2. 打开原文件

```shell
vi /etc/sources.list
```

3. 删除原本的所有内容，直接替换成网上搜到的阿里云镜像源

（Ubuntu18.04）
```c
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse    
```

4. 执行指令更新软件源列表

```shell
sudo apt-get update
```

### 4、安装VMware Tools

```shell
sudo apt upgrade
sudo apt install open-vm-tools-desktop -y
sudo reboot
```

### 5、安装vim

Ubuntu系统执行以下代码：

```shell
sudo apt-get install vim-gtk
```

### 6、安装Samba

1. Samba最大的用途就是可以实现Linux与windows系统共享文件夹，他可以将Linux的目录挂载成网络硬盘的形式出现在Windows的文件资源管理器里

2. 搭建SMB服务首先要保证Windows和Ubuntu网络互通，然后在Ubuntu下载安装：

```shell
sudo apt-get install samba
```

3. 给需要共享的文件夹设置权限为777，修改配置文件：

```shell
sudo vim /etc/samba/smb.conf
```

4. 将下面的代码添加到配置文件的最后面

```c
[Ubuntu]
comment = SMB File Share
path = /
public = yes
writeable = yes
valid users = root
create mask = 0777
directory mask = 0777
force user = root
force group = root
available = yes
unix charset = UTF-8
dos charset = cp936
```

5. 改完配置文件后保存，然后使用命令为SMB服务添加用户和密码

```shell
sudo smbpasswd -a root
```

6. 重启SMB服务:

```shell
sudo service smbd restart
```

7. 在 Windows 上输入“win+r”弹出运行窗口，然后输入Ubuntu 的 \\\IP 访问，然后右键Samba文件夹将他映射为网络驱动器

### 7、安装依赖

安装一些用于编译Linux系统的工具软件包：

```shell
sudo apt-get install uuid uuid-dev zlib1g-dev liblz-dev liblzo2-2 liblzo2-dev lzop \
git-core curl u-boot-tools mtd-utils android-tools-fsutils openjdk-8-jdk device-tree-compiler \
gdisk m4 libz-dev git gnupg flex bison gperf libsdl1.2-dev libesd-java libwxgtk3.0-dev \
squashfs-tools build-essential zip curl libncurses5-dev zlib1g-dev pngcrush schedtool \
libxml2 libxml2-utils xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev \
lib32ncurses5-dev lib32readline-dev gcc-multilib libswitch-perl libssl-dev unzip \
zip liblz4-tool repo git ssh make gcc libssl-dev liblz4-tool vim expect \
g++ patchelf chrpath gawk texinfo chrpath diffstat binfmt-support \
qemu-user-static live-build bison flex fakeroot cmake gcc-multilib g++-multilib \
unzip device-tree-compiler python-pip libncurses5-dev rsync subversion \
sed make binutils  build-essential  gcc  g++  wget python libncurses5 bzr cvs git mercurial \
patch gzip bzip2 perl tar cpio unzip rsync file bc wget qemu-user-static live-build -y
```

### 8、安装SSH

1. 安装SSH服务

```shell
apt-get install openssh-server
```

2. 启动SSH服务

```shell
sudo systemctl start sshd
```

3. 修改配置文件，配置文件中 “ PermitRootLogin prohibit-password ” 修改成 “ PermitRootLogin yes ” （记得取消注释），允许root用户登录

```shell
vim /etc/ssh/sshd_config
```

4. 重启SSH服务

```shell
sudo systemctl restart sshd
```

### 9、安装FTP服务

1. 开启Ubuntu下的 FTP 服务 

```shell
sudo apt-get install vsftpd
```

2. 安装完成以后使用vim命令打开/etc/vsftpd.conf

```shell
sudo vim /etc/vsftpd.conf
```

3. 打开vsftpd.conf 文件以后找到如下两行：

```c
local_enable=YES 
write_enable=YES 
//确保上面两行前面没有“#”，有的话就取消掉
```

4. 修改完vsftpd.conf保存退出，使用如下命令重启FTP服务：  

```shell
sudo /etc/init.d/vsftpd restart
```

5. Windows下载FTP工具：

https://filezilla-project.org/

6. 文件→站点管理器→新建站点→选择FTP协议→加密方式选择明文传输→输入账号密码IP（端口不用输）→字符集选择强制使用UTF-8→连接

### 10、安装TFTP服务

1. tftp 命令的作用和 nfs 命令一样，都是用于通过网络下载东西到 DRAM 中，只是 tftp 命令使用的 TFTP 协议，Ubuntu 主机作为 TFTP 服务器。因此需要在 Ubuntu 上搭建 TFTP 服务器， 
2. 需要安装 tftp-hpa 和tftpd-hpa，命令如下：

```shell
sudo apt-get install tftp-hpa tftpd-hpa 
sudo apt-get install xinetd 
```

3. 和 NFS 一样，TFTP 也需要一个文件夹来存放文件，在用户目录下新建一个目录，命令如下： 

```shell
mkdir /linux/tftpboot 
chmod 777 /linux/tftpboot 
```

4. 这样我就在我的电脑上创建了一个名为 tftpboot 的 目录( 文件夹 ) ，路径为 /linux/tftpboot。注意！我们要给 tftpboot 文件夹权限，否则的话 uboot 不能从 tftpboot 文件夹里面下载文件。 
5. 最后配置tftp，安装完成以后新建文件/etc/xinetd.d/tftp，如果没有/etc/xinetd.d 目录的话自行创建，然后在里面输入如下内容： 

```shell
server tftp 
2  { 
3 socket_type = dgram 
4 protocol = udp 
5 wait = yes 
6 user = root 
7 server = /usr/sbin/in.tftpd 
8 server_args = -s /linux/tftpboot/ 
9 disable = no 
10 per_source = 11 
11 cps = 100 2 
12 flags = IPv4 
13 } 
```

6. 完了以后启动 tftp 服务，命令如下

```shell
sudo service tftpd-hpa start 
```

7. 打开/etc/default/tftpd-hpa 文件，将其修改为如下所示内容： 

```shell
# /etc/default/tftpd-hpa 
2  
3 TFTP_USERNAME="tftp" 
4 TFTP_DIRECTORY="/linux/tftpboot"  
5 TFTP_ADDRESS=":69"                                 
6 TFTP_OPTIONS="-l -c -s" 
```

8. TFTP_DIRECTORY 就是我们上面创建的 tftp 文件夹目录，以后我们就将所有需要通过 TFTP 传输的文件都放到这个文件夹里面，并且要给予这些文件相应的权限。

9. 最后输入如下命令， 重启 tftp 服务器：

```shell
sudo service tftpd-hpa restart 
```

10. tftp 服务器已经搭建好了，接下来就是使用了。比如将要发送到开发板的 zImage 镜像文件拷贝到 tftpboot 文件夹中，并且给予 zImage 相应的权限，命令如下： 

```shell
cp zImage linux/tftpboot/ 
cd linux/tftpboot/ 
chmod 777 zImage 
```

## （三）Ubuntu安装NFS服务

>**功能：** 实现开发板（芯片）与Ubuntu互传文件
>
>**前提：** 开发板和Ubuntu在同一局域网

1. Ubuntu安装NFS服务

```shell
apt-get install nfs-kernel-server
```

2. 修改配置文件

```shell
vim /etc/exports
```

3. 在最后一行添加配置：

```c
/ *(rw,sync,no_root_squash)
    
/*
/：要共享的目录
*： 代表允许所有的网络段访问
w： 是可读写权限
sync：是资料同步写入内存和硬盘
no_root_squash：是 Ubuntu NFS 客户端分享目录使用者的权限，如果客户端使用的是root用户，那么对于该共享目录而言，该客户端就具有root权限
*/   
```

4. 重启rpcbind服务

```shell
sudo systemctl restart rpcbind
```

5. 重启NFS服务

```shell
sudo systemctl restart nfs-kernel-server
```

6. 查看是否成功挂载NFS目录（这里也可能要重启Ubuntu才会挂载目录）

```shell
showmount -e
```

7. 在开发板终端使用以下命令挂载到NFS服务器

```shell
mount -t nfs -o nfsvers=3,nolock 172.172.14.109:/linux /mnt/
```

   *172.172.14.109：* 这个是Ubuntu的IP，要设置成自己的IP，这个经常会变动
   */linux：* 这个是要挂载的目标目录
   */mnt/：* 这个是开发板的本地目录，用于挂载到目标NFS服务器

8. 挂载成功后，在开发板的/mnt/目录中就可以访问Ubuntu的/aa/目录里的所有文件了

## （四）VS Code配置包含路径

>将解压好的Linux源码下的内核路径包含到VS Code 的配置文件中，这样就能跳转到函数定义了

```json
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
		  "Z:\\aa\\rk356x_linux\\kernel\\include",
		  "Z:\\aa\\rk356x_linux\\kernel\\arch\\arm64\\include",
		  "Z:\\aa\\rk356x_linux\\kernel\\arch\\arm64\\include\\generated\\",
		  "Z:\\usr\\include",
		  "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ]
        }
    ],
    "version": 4
}
```

## （五）配置交叉编译器

### 1、使用厂家提供的SDK中的交叉编译器

1. RK3568的Linux_SDK中会附带一个交叉编译器，路径在解压后的Linux源码文件夹中：

```text
/rk356x_linux/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin
```

2. 配置环境变量，让Ubuntu可以识别到交叉编译器：

```shell
vim /etc/profile
```

3. 将下面的PATH路径添加到环境变量文件最后：

```c
export PATH=$PATH:/aa/rk356x_linux/prebuilts/gcc/linux-x86/aarch64/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin
```

4. 重新加载配置文件：

```shell
source /etc/profile
```

5. 编译一个程序（此时输入aarch再按下TAB就应该会自动补全命令了）：

```shell
aarch64-linux-gnu-gcc  main.c
```

6. 挂载好NFS后到开发板上面运行，测试成功

### 2、下载Linaro AArch64 GNU/Linux

[Linaro Releases](https://releases.linaro.org/components/toolchain/gcc-linaro/)

# 三、源码编译

## （一）RK3568源码编译

### 1、安装驱动Rockchip Driver Assistant

[Rockchip Driver Assistant for Windows (all versions)](https://androidmtk.com/download-rockchip-driver-assistant)

### 2、瑞芯微烧录工具RKDevTool

| 模式       | 工具烧录 | 介绍                                                         |
| :--------- | -------- | ------------------------------------------------------------ |
| Normal模式 | 不支持   | 此模式是正常启动的过程，各个组件依次加载，正常进入系统。系统引导 rootfs 启动，加载 rootfs，大多数的开发都是在这个模式调试的。 |
| Loader模式 | 支持     | 在 Loader 模式下，可以进行固件的烧写，升级，可以通过工具单独烧写某一个分区镜像文件，方便调试。 |
| MaskRo模式 | 支持     | Flash 在未烧录固件时，芯片会引导进入 MaskRom 模式，可以进行初次固件的烧写;开发调试过程中若遇到 uboot 无法正常启动的情况，也可以进入 MaskRom 模式烧写固件。 |

进入Loader模式的方法：

> 1. 连接烧录线到PC
> 2. 按下组合键进入烧录模式
> 3. 开机状态下在ADB命令行模式或者调试串口执行命令也可以进入烧录模式：reboot loader

### 3、源码编译

复制源码到Ubuntu中，用**普通用户**登录，使用命令解压：

```shell
sudo tar -xvf rk356x_linux_20240430.tar.xz
```

进入源码目录运行envsetup.sh

```shell
./envsetup.sh
# 选开发板,选择66
```

执行

```shell
export ARCH=arm64
```

执行

```shell
make menuconfig
# 在build options菜单下配置linux源码所在的路径
```

执行

```shell
make savedefconfig
```

运行build.sh

```shell
./build.sh
```

编译完成后会将生成的update.img文件存在rockdev中。
