# 一、U-Boot 常用命令整理

## 1. 信息查询命令

### 1.1 `bdinfo`

- **功能**: 查看板子信息。

- **格式**: `bdinfo`

- **示例**:

  ```shell
  bdinfo
  ```

### 1.2 `printenv`

- **功能**: 输出环境变量信息。

- **格式**: `printenv`

- **示例**:

  ```shell
  printenv
  ```

### 1.3 `version`

- **功能**: 查看 U-Boot 版本号。

- **格式**: `version`

- **示例**:

  ```shell
  version
  ```

## 2. 环境变量操作命令

### 2.1 `setenv`

- **功能**: 设置或修改环境变量的值。

- **格式**: `setenv <variable> <value>`

- **示例**:

  ```shell
  setenv bootdelay 5
  ```

### 2.2 `saveenv`

- **功能**: 保存修改后的环境变量。

- **格式**: `saveenv`

- **示例**:

  ```shell
  saveenv
  ```

### 2.3 删除环境变量

- **功能**: 删除环境变量。

- **格式**: `setenv <variable>`

- **示例**:

  ```shell
  setenv author
  ```

## 3. 内存操作命令

### 3.1 `md`

- **功能**: 显示内存值。

- **格式**: `md[.b, .w, .l] address [# of objects]`

- **示例**:

  ```shell
  md.b 80000000 10
  ```

### 3.2 `nm`

- **功能**: 修改指定地址的内存值。

- **格式**: `nm [.b, .w, .l] address`

- **示例**:

  ```shell
  nm.l 80000000
  ```

### 3.3 `mm`

- **功能**: 修改指定地址内存值，地址自增。

- **格式**: `mm [.b, .w, .l] address`

- **示例**:

  ```shell
  mm.l 80000000
  ```

### 3.4 `mw`

- **功能**: 使用指定数据填充一段内存。

- **格式**: `mw [.b, .w, .l] address value [count]`

- **示例**:

  ```shell
  mw.l 80000000 0A0A0A0A 10
  ```

### 3.5 `cp`

- **功能**: 内存间数据拷贝。

- **格式**: `cp [.b, .w, .l] source target count`

- **示例**:

  ```shell
  cp.l 80000000 80000100 10
  ```

### 3.6 `cmp`

- **功能**: 比较两段内存的数据是否相等。

- **格式**: `cmp [.b, .w, .l] addr1 addr2 count`

- **示例**:

  ```shell
  cmp.l 80000000 80000100 10
  ```

## 4. 网络操作命令

### 4.1 `ping`

- **功能**: 测试网络连接。

- **格式**: `ping <IP地址>`

- **示例**:

  ```shell
  ping 192.168.1.253
  ```

### 4.2 `dhcp`

- **功能**: 从路由器获取 IP 地址。

- **格式**: `dhcp`

- **示例**:

  ```shell
  dhcp
  ```

### 4.3 `nfs`

- **功能**: 通过 NFS 下载文件到 DRAM。

- **格式**: `nfs [loadAddress] [[hostIPaddr:]bootfilename]`

- **示例**:

  ```shell
  nfs 80800000 192.168.1.253:/home/zuozhongkai/linux/nfs/zImage
  ```

### 4.4 `tftp`

- **功能**: 通过 TFTP 下载文件到 DRAM。

- **格式**: `tftpboot [loadAddress] [[hostIPaddr:]bootfilename]`

- **示例**:

  ```shell
  # 1.要先设置tftp的服务器IP（要是搭建了TFTP服务的Ubuntu主机IP才行）
  setenv serverip 172.172.14.109
  # 2.从tftp服务器的ftp文件夹下载名为zImage的文件到DRAM中
  tftp 80800000 zImage
  ```

## 5. EMMC 和 SD 卡操作命令

### 5.1 `mmc info`

- **功能**: 输出当前选中的 MMC 设备信息。

- **格式**: `mmc info`

- **示例**:

  ```shell
  mmc info
  ```

### 5.2 `mmc rescan`

- **功能**: 扫描当前开发板上所有的 MMC 设备。

- **格式**: `mmc rescan`

- **示例**:

  ```shell
  mmc rescan
  ```

### 5.3 `mmc list`

- **功能**: 列出当前有效的所有 MMC 设备。

- **格式**: `mmc list`

- **示例**:

  ```shell
  mmc list
  ```

### 5.4 `mmc dev`

- **功能**: 切换当前 MMC 设备。

- **格式**: `mmc dev [dev] [part]`

- **示例**:

  ```shell
  mmc dev 0
  ```

### 5.5 `mmc read`

- **功能**: 读取 MMC 中的数据。

- **格式**: `mmc read addr blk# cnt`

- **示例**:

  ```shell
  mmc read 80800000 600 10
  ```

### 5.6 `mmc write`

- **功能**: 向 MMC 设备写入数据。addr 是要写入 MMC 中的数据在 DRAM 中的起始地址，blk 是要写入 MMC 的块起始地址(十六进制)，cnt 是要写入的块大小，一个块为 512 字节。
- **格式**: `mmc write addr blk cnt`

- **示例**:

  ```shell
  mmc write 80800000 2 2E6
  ```

### 5.7 `mmc erase`

- **功能**: 擦除 MMC 设备的指定块。

- **格式**: `mmc erase blk# cnt`

- **示例**:

  ```shell
  mmc erase 600 10
  ```

### 5.8 `mmc part`

- **功能**: 查看当前选中的 EMMC 的分区情况

- **格式**: `mmc part`

- **示例**:

  ```shell
  mmc part
  ```

  

## 6. FAT 格式文件系统操作命令

### 6.1 `fatinfo`

- **功能**: 查询指定 MMC 设备分区的文件系统信息。

- **格式**: `fatinfo <interface> [<dev[:part]>]`

- **示例**:

  ```shell
  fatinfo mmc 1:1
  ```

### 6.2 `fatls`

- **功能**: 查询 FAT 格式设备的目录和文件信息。

- **格式**: `fatls <interface> [<dev[:part]>] [directory]`

- **示例**:

  ```shell
  fatls mmc 1:1
  ```

### 6.3 `fatload`

- **功能**: 将指定的文件读取到 DRAM 中。

- **格式**: `fatload <interface> [<dev[:part]> [<addr> [<filename> [bytes [pos]]]]]`

- **示例**:

  ```shell
  fatload mmc 1:1 80800000 zImage
  ```

### 6.4 `fatwrite`

- **功能**: 将 DRAM 中的数据写入到 MMC 设备中。

- **格式**: `fatwrite <interface> <dev[:part]> <addr> <filename> <bytes>`

- **示例**:

  ```shell
  fatwrite mmc 1:1 80800000 zImage 6788f8
  ```

## 7. EXT 格式文件系统操作命令

### 7.1 `ext4ls`

- **功能**: 查询 EXT4 格式设备的目录和文件信息。

- **格式**: `ext4ls <interface> <dev[:part]> [directory]`

- **示例**:

  ```shell
  ext4ls mmc 1:2
  ```

## 8. NAND 操作命令

### 8.1 `nand info`

- **功能**: 打印 NAND Flash 信息。

- **格式**: `nand info`

- **示例**:

  ```shell
  nand info
  ```

### 8.2 `nand erase`

- **功能**: 擦除 NAND Flash。

- **格式**: `nand erase[.spread] [clean] off size`

- **示例**:

  ```shell
  nand erase 600 10
  ```

### 8.3 `nand write`

- **功能**: 向 NAND 指定地址写入指定的数据。

- **格式**: `nand write addr off size`

- **示例**:

  ```shell
  nand write 80800000 600 10
  ```

### 8.4 `nand read`

- **功能**: 从 NAND 中的指定地址读取指定大小的数据到 DRAM 中。

- **格式**: `nand read addr off size`

- **示例**:

  ```shell
  nand read 80800000 600 10
  ```

## 9. BOOT 操作命令

### 9.1 `bootz`

- **功能**: 启动 zImage 镜像文件。addr 是Linux 镜像文件在 DRAM 中的位置，initrd 是 initrd 文件在DRAM 中的地址，如果不使用 initrd 的话使用‘-’代替即可，fdt 就是设备树文件在DRAM中的地址。

- **格式**: `bootz [addr [initrd[:size]] [fdt]]`

- **示例**:

  ```shell
  bootz 80800000 - 83000000
  ```

### 9.2 `bootm`

- **功能**: 启动 uImage 镜像文件。

- **格式**: `bootm [addr [initrd[:size]] [fdt]]`

- **示例**:

  ```shell
  bootm 80800000 - 83000000
  ```

### 9.3 `boot`

- **功能**: 读取环境变量 bootcmd 来启动 Linux 系统。

- **格式**: `boot`

- **示例**:

  ```shell
  boot
  ```

## 10. 其他常用命令

### 10.1 `reset`

- **功能**: 复位重启。

- **格式**: `reset`

- **示例**:

  ```shell
  reset
  ```

### 10.2 `go`

- **功能**: 跳到指定的地址处执行应用。

- **格式**: `go addr [arg ...]`

- **示例**:

  ```shell
  go 87800000
  ```

### 10.3 `run`

- **功能**: 运行环境变量中定义的命令。

- **格式**: `run <variable>`

- **示例**:

  ```shell
  run mybootemmc
  ```

### 10.4 `mtest`

- **功能**: 简单的内存读写测试。

- **格式**: `mtest [start [end [pattern [iterations]]]`

- **示例**:

  ```shell
  mtest 80000000 80001000
  ```

# 二、U-Boot 命令使用

## 1. 修改网络信息，实现在U-Boot内上网

### 1.1 想要上网需要配置的变量

```shell
# 网口的MAC地址，是一个48bit的数，这个在同一网段内要独一无二
ethaddr
# 网口的IP地址，用DHCP直接自动获取就行
ipaddr
```

### 1.2 执行指令修改变量

```shell
# 1.执行dhcp自动获取IP
dhcp 
# 2.修改MAC地址
setenv ethaddr b8:ae:1d:01:00:00 
# 3.保存变量
saveenv
```

### 1.3 测试网络

```shell
# ping一下主机，出现 host172.172.14.109 is alive，说明网络连通
ping 172.172.14.109
```

## 2. 在U-Boot内更新U-Boot

### 2.1 操作步骤

```shell
# 1.切换到EMMC设备的分区0（存放U-Boot的位置）
mmc dev 1 0
# 2.从服务器对应的ftp文件夹内下载U-Boot镜像到DRAM中（也可以使用NFS，都一样的）
tftp 80800000 u-boot.imx
# 3.从DRAM中烧写U-Boot到EMMC的第二个块，一共烧写32E个块
mmc write 80800000 2 32E
# 4.给EMMC配置分区
mmc partconf 1 1 0 0
# 5.重启U-Boot
reset
```

### 2.2 为什么烧写到EMMC分区0的第二个块？

在 i.MX 系列处理器中，BootROM 会从存储设备（如 eMMC 或 SD 卡）的特定位置加载 U-Boot 镜像。这个位置通常是由处理器的启动流程和存储设备的布局决定的，分区0是存放U-Boot镜像的位置。

- **Block 0 和 Block 1 的用途：**
  - **Block 0** 通常保留给 MBR（主引导记录）或分区表。
  - **Block 1** 可能用于存储额外的引导信息（如 IVT 和 DCD 表）。
  - **Block 2** 是 U-Boot 镜像的实际起始位置。BootROM 会从 Block 2 开始读取 U-Boot 镜像。

### 2.3 **为什么烧写 `32E` 的大小？**

- **U-Boot 镜像的大小：**
  `u-boot.imx` 是 U-Boot 镜像的二进制文件，通常包含以下部分：

  - **IVT（Image Vector Table）**：用于 BootROM 识别镜像。
  - **DCD（Device Configuration Data）**：用于初始化 DDR 和其他外设。
  - **U-Boot 代码**：实际的 U-Boot 程序。

  这些部分的总大小决定了需要写入的块数。`32E` 是根据 `u-boot.imx` 的实际大小计算得出的，确保整个镜像被完整写入。

- **为什么是 814 个块？**
  如果 `u-boot.imx` 的大小是 407 KB 左右，而 `每个块大小通常是512字节` ，那么需要写入的块数为：

  (407×1024)/512=814块

  因此，`32E` 是十六进制表示的 814，确保整个镜像被写入。

### 2.4 EMMC设备分区结构

如果 EMMC 里面烧写了 Linux 系统的话，EMMC 是有 3 个分区的，第 0 个分区存放 uboot，第 1 个分区存放 Linux 镜像文件和设备树，第 2 个分区存放根文件系统。

**各个分区的文件系统格式：**

* 分区 0 格式未知，因为分区 0 存放的 uboot，并且分区0 没有格式化，所以文件系统格式未知。
* 分区 1 的格式为 fat，分区 1 用于存放 linux 内核镜像和设备树。
* 分区 2 的格式为 ext4，用于存放 Linux 的根文件系统(rootfs)。
