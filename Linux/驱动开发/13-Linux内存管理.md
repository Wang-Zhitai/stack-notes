# Linux内存管理

# 1. 常用内存分配函数的异同点

## 1.1 用户空间

| API名称                            | 物理连续？ | 大小限制 | 单位 | 场景                                                         |
| ---------------------------------- | ---------- | -------- | ---- | ------------------------------------------------------------ |
| `malloc / calloc / realloc / free` | 不保证     | 堆申请   | 字节 | `calloc `初始化为0；`realloc `改变内存大小                   |
| `alloca`                           | 不保证     | 栈申请   | 字节 | 向栈申请内存                                                 |
| `mmap / munmap`                    | 不保证     |          |      | 将文件利用虚拟内存技术映射到内存中去                         |
| `brk / sbrk`                       | 不保证     |          |      | 虚拟内存到内存的映射。`sbrk(0)` 返回 `program break` 地址，`sbrk` 调整对的大小 |

## 1.2 内核空间

|            | API名称                                 | 物理连续？         | 大小限制                   | 单位                           | 场景                                                         |
| ---------- | --------------------------------------- | ------------------ | -------------------------- | ------------------------------ | ------------------------------------------------------------ |
|            | `vmalloc / vfree`                       | 虚拟连续，物理不定 | `vmalloc` 区大小限制       | 页 `vmalloc` 区域              | 可能睡眠，不能从中断上下文中调用，或其他不允许阻塞情况下调用。`VMALLOC` 区域 `vmalloc_start~vmalloc_end` 之间，`vmalloc` 比 `kmalloc` 慢，适用于分配大内存。 |
| `slab`     | `kmalloc / kcalloc / krealloc / kfree`  | 物理连续           | 64MB-4MB（随 `slab` 而变） | `2^order` 字节 `Normal` 区域   | 大小有限，不如 `vmalloc/malloc` 大。最大/小值由`KMALLOC_MIN_SIZE/KMALLOC_SHIFT_MAX`，对应64B/4MB。从 `/proc/slabinfo` 中的 `kmalloc-xxxx` 中分配，建立在`kmem_cache_create` 基础之上。 |
| `slab`     | `kmem_cache_create`                     | 物理连续           | 64MB-4MB                   | 字节大小，需对齐 `Normal` 区域 | 便于固定大小数据的频繁分配和释放，分配时从缓存池中获取地址，释放时也不一定真正释放内存。通过slab进行管理。 |
| `伙伴系统` | `__get_free_page / __get_free_pages`    | 物理连续           | 4MB（1024页）              | 页 `Normal` 区域               | `__get_free_pages` 基于 `alloc_pages`，但是限定不能使用HIGHMEM。 |
| `伙伴系统` | `alloc_page / alloc_pages / free_pages` | 物理连续           | 4MB                        | 页 `Normal / Vmalloc` 都可     | `CONFIG_FORCE_MAX_ZONEORDER` 定义了最大页面数2^11，一次能分配到的最大页面数是1024。 |



# 2. RK3566内存解析

## 2.1 查看系统整体内存状况

使用 **`cat /proc/meminfo`**

[linux内存占用分析之 meminfo - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000022518282)

```bash
root@ATK-DLRK3506:/# cat /proc/meminfo 
MemTotal:         456632 kB
MemFree:          347572 kB
MemAvailable:     382328 kB
Buffers:               0 kB
Cached:            49692 kB
SwapCached:            0 kB
Active:             7868 kB
Inactive:          67464 kB
Active(anon):        248 kB
Inactive(anon):    37484 kB
Active(file):       7620 kB
Inactive(file):    29980 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 0 kB
Writeback:             0 kB
AnonPages:         25648 kB
Mapped:            46504 kB
Shmem:             12092 kB
KReclaimable:       7932 kB
Slab:              17548 kB
SReclaimable:       7932 kB
SUnreclaim:         9616 kB
KernelStack:         944 kB
PageTables:          672 kB
SecPageTables:         0 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      228316 kB
Committed_AS:      93060 kB
VmallocTotal:     770048 kB
VmallocUsed:        7584 kB
VmallocChunk:          0 kB
Percpu:              168 kB
CmaTotal:          55296 kB
CmaAllocated:      12644 kB
CmaReleased:       42652 kB
CmaFree:               0 kB 

```



## 2.2 查看物理内存映射信息

使用 **`cat /proc/iomem`** 会打印模块实际映射的地址

```bash
root@ATK-DLRK3506:/# cat /proc/iomem
00000000-1fffffff : System RAM
00108000-0074ffff : Kernel code
0077c000-007f7e77 : Kernel data
ff000000-ff003fff : dma-controller@ff000000
ff000000-ff003fff : ff000000.dma-controller dma-controller@ff000000
ff008000-ff00bfff : dma-controller@ff008000
ff008000-ff00bfff : ff008000.dma-controller dma-controller@ff008000
ff040000-ff040fff : ff040000.i2c i2c@ff040000
ff050000-ff050fff : ff050000.i2c i2c@ff050000
ff060000-ff060fff : ff060000.i2c i2c@ff060000
ff0c0000-ff0c001f : serial
ff120000-ff120fff : ff120000.spi spi@ff120000
ff1c0000-ff1c01ff : ff1c0000.gpio gpio@ff1c0000
ff1d0000-ff1d01ff : ff1d0000.gpio gpio@ff1d0000
ff1e0000-ff1e01ff : ff1e0000.gpio gpio@ff1e0000
ff2b0000-ff2b7fff : ff2b0000.usb2-phy usb2-phy@ff2b0000
ff310000-ff310fff : ff310000.sai sai@ff310000
ff480000-ff483fff : ff480000.mmc mmc@ff480000
ff488000-ff48bfff : ff488000.spi spi@ff488000
ff4a8000-ff4a8fff : ff4a8000.sai sai@ff4a8000
ff4c8000-ff4c9fff : ff4c8000.ethernet ethernet@ff4c8000
ff4d0000-ff4d1fff : ff4d0000.ethernet ethernet@ff4d0000
ff4e8000-ff4effff : ff4e8000.adc adc@ff4e8000
ff4f0000-ff4f3fff : ff4f0000.otp otp@ff4f0000
ff4f8000-ff4f8fff : ff4f8000.audio-codec audio-codec@ff4f8000
ff600000-ff6001ff : ff600000.vop regs
ff600a00-ff600dff : ff600000.vop gamma_lut
ff640000-ff64ffff : ff640000.dsi dsi@ff640000
ff650000-ff6503ff : ff650000.tsadc tsadc@ff650000
ff670000-ff67ffff : ff670000.phy phy
ff710000-ff7101ff : ff710000.rng rng@ff710000
ff740000-ff77ffff : ff740000.usb usb@ff740000
ff780000-ff7bffff : ff780000.usb usb@ff780000
ff870000-ff8701ff : ff870000.gpio gpio@ff870000
ff940000-ff9401ff : ff940000.gpio gpio@ff940000
```



## 2.3 /proc目录下文件详解

### 2.3.1 进程相关目录

```c
/proc/[pid]/          // 每个运行进程的目录，pid为进程ID
├── cmdline          // 进程启动命令（完整命令行，\0分隔）
├── cwd -> /dir      // 链接到进程当前工作目录
├── environ          // 进程环境变量（\0分隔）
├── exe -> /bin/cmd  // 链接到进程的可执行文件
├── fd/              // 目录：进程打开的文件描述符
│   ├── 0 -> /dev/pts/0    // 标准输入
│   ├── 1 -> /dev/pts/0    // 标准输出
│   └── 2 -> /dev/pts/0    // 标准错误
├── fdinfo/          // 文件描述符详细信息
├── maps             // 进程内存映射区域（地址空间布局）
├── mem              // 进程内存内容（需root权限访问）
├── mountinfo        // 进程挂载点信息
├── mounts           // 进程挂载的文件系统列表
├── net/             // 进程网络命名空间信息
├── ns/              // 进程命名空间链接
├── root -> /        // 链接到进程根目录
├── stat             // 进程状态信息（数字格式）
├── statm            // 进程内存状态（页为单位）
├── status           // 进程状态信息（可读格式）
└── task/[tid]/      // 进程内每个线程的目录
```

### 2.3.2 系统信息文件

```c
/proc/
├── cpuinfo          // CPU详细信息（型号、核心数、频率等）
├── meminfo          // 系统内存使用情况（如你提供的示例）
├── version          // 内核版本和gcc版本信息
├── uptime           // 系统运行时间（秒）和空闲时间
├── loadavg          // 系统平均负载（1、5、15分钟）
├── stat             // 系统统计信息（CPU使用率、进程数等）
├── swaps            // 交换分区/文件信息
├── partitions       // 磁盘分区信息
├── devices          // 已注册的设备列表
├── filesystems      // 内核支持的文件系统类型
├── modules          // 已加载的内核模块列表
├── locks            // 当前文件锁信息
├── slabinfo         // slab分配器统计信息（内核对象缓存）
├── vmstat           // 虚拟内存统计信息
├── zoneinfo         // 内存区域（zone）详细信息
├── pagetypeinfo     // 页面类型信息
└── buddyinfo        // 伙伴系统内存碎片信息
```

### 2.3.3 内核参数和配置

```c
/proc/sys/           // 内核参数，可动态调整（sysctl命令操作）
├── kernel/
│   ├── hostname        // 主机名
│   ├── osrelease       // 内核发行版本
│   ├── ostype          // 操作系统类型
│   ├── pid_max         // 最大PID值
│   ├── shmall          // 共享内存总页数
│   └── sysrq           // SysRq键控制
├── vm/
│   ├── swappiness      // 交换倾向性（0-100）
│   ├── dirty_ratio     // 开始写回脏页的系统内存百分比
│   ├── drop_caches     // 清空缓存（1-页缓存，2-目录项，3-全部）
│   └── overcommit_memory  // 内存超配策略
├── net/
│   ├── ipv4/
│   │   ├── ip_forward      // IP转发开关
│   │   ├── tcp_syncookies  // SYN cookies防护
│   │   └── icmp_echo_ignore_all  // 忽略ping请求
│   └── core/
│       ├── somaxconn        // 最大连接队列长度
│       └── rmem_max         // 接收缓冲区最大值
└── fs/
    ├── file-max           // 系统最大文件句柄数
    ├── file-nr            // 已分配和使用的文件句柄数
    └── inode-state        // inode状态统计
```

### 2.3.4 网络信息

```c
/proc/net/
├── dev              // 网络设备统计信息（接收/发送字节包数）
├── tcp              // TCP套接字列表（本地/远程地址、状态等）
├── udp              // UDP套接字列表
├── unix             // UNIX域套接字列表
├── netstat          // 网络统计信息（SNMP计数器）
├── sockstat         // 套接字统计信息
├── ip_conntrack     // 连接跟踪表（如有）
├── arp              // ARP缓存表
├── route            // 路由表
├── dev_mcast        // 设备组播信息
└── snmp             // SNMP代理统计信息
```

### 2.3.5 中断和系统状态

```c
/proc/
├── interrupts       // 中断请求（IRQ）统计（每个CPU和每个设备）
├── iomem            // I/O内存映射（物理地址分配）
├── ioports          // I/O端口范围
├── softirqs         // 软中断统计
├── timer_list       // 定时器列表
├── kallsyms         // 内核符号表（函数/变量地址）
├── kmsg             // 内核消息（替代dmesg）
└── kpageflags       // 物理页标志位
```

### 2.3.6 设备树相关

```c
// 如果存在设备树（ARM/嵌入式系统用设备树，X86系统用ACPI），目录结构如下：
/proc/device-tree/
├── compatible                    // 设备兼容性字符串
├── model                         // 设备型号
├── name                          // 设备名称
├── #address-cells                // 地址单元数量
├── #size-cells                   // 大小单元数量
├── cpus/                         // CPU信息
│   ├── cpu@0
│   └── cpu@1
├── memory@0                      // 内存信息
├── chosen/                       // 引导参数
│   ├── bootargs                  // 内核启动参数
│   └── stdout-path               // 标准输出路径
├── soc/                          // 片上系统
│   ├── serial@1000               // 串口
│   ├── i2c@2000                  // I2C控制器
│   └── interrupt-controller@3000 // 中断控制器
└── aliases/                      // 设备别名
```



# 3. 参考文献

[一步一图带你深入理解 Linux 虚拟内存管理](https://mp.weixin.qq.com/s/uWadcBxEgctnrgyu32T8sQ)

[一步一图带你深入理解 Linux 物理内存管理](https://mp.weixin.qq.com/s/Cn-oX0W5DrI2PivaWLDpPw)

[linux内核 - 标签 - bin的技术小屋 - 博客园](https://www.cnblogs.com/binlovetech/tag/linux内核/)

[从内核源码看 slab 内存池的创建初始化流程 - bin的技术小屋 - 博客园](https://www.cnblogs.com/binlovetech/p/17308859.html)

[细节拉满，80 张图带你一步一步推演 slab 内存池的设计与实现 - bin的技术小屋 - 博客园](https://www.cnblogs.com/binlovetech/p/17288990.html)

[一步一图带你深入理解 Linux 物理内存管理 - bin的技术小屋 - 博客园](https://www.cnblogs.com/binlovetech/p/16914715.html)

[一步一图带你深入理解 Linux 虚拟内存管理 - bin的技术小屋 - 博客园](https://www.cnblogs.com/binlovetech/p/16824522.html)

[Linux内存管理 - 随笔分类 - LoyenWang - 博客园](https://www.cnblogs.com/LoyenWang/category/1533753.html)

[Linux内存管理 - 随笔分类 - ArnoldLu - 博客园](https://www.cnblogs.com/arnoldlu/category/1132616.html?page=1)

[Linux 内存管理 - 简书](https://www.jianshu.com/p/fb345b94501f)

[万字长文，别再说你不懂Linux内存管理了，30 张图给你安排的明明白白-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1775509)

[/proc 目录下文件详解 - 赵SIR - 博客园](https://www.cnblogs.com/zhaobin-diray/p/10936220.html)

[SLUB的引入及举例说明-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1622993)

[SLUB结构体创建及创建slab分析-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1622988)

[SLUB分配一个object的流程分析-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1622989)

[Android后台杀死系列之三：LowMemoryKiller原理（4.3-6.0） - Android开发技术 - SegmentFault 思否](https://segmentfault.com/a/1190000008110561)

[Android LowMemoryKiller原理分析 - Gityuan博客 | 袁辉辉的技术博客](https://gityuan.com/2016/09/17/android-lowmemorykiller/)