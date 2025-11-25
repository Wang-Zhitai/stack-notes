
# 平台总线介绍

平台总线（Platform bus）是 Linux 内核中提供的一种虚拟总线，用于管理和组织与特定硬 件平台相关的设备和驱动。它充当了平台设备（platform device）和平台驱动（platform driver） 之间的桥梁，负责将它们进行匹配和绑定。

当系统注册一个平台设备时，平台总线会寻找与之匹配的平台驱动。它会遍历已注册的平台驱动列表，尝试与每个平台驱动进行匹配，直到找到与平台设备匹配的驱动为止。一旦找到匹配的驱动，平台总线会将平台设备与平台驱动进行绑定，使得设备可以被正确地初始化和操作。

同样地，当系统注册一个平台驱动时，平台总线会寻找与之匹配的平台设备。它会遍历已注册的平台设备列表，尝试与每个平台设备进行匹配，直到找到与平台驱动匹配的设备为止。一旦找到匹配的设备，平台总线会将平台设备与平台驱动进行绑定，使得驱动可以管理和控制 与该设备相关的操作。

设备、平台总线、驱动的关系如下图所示：

![image-20240710201921249](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240710201921249.png)

通过引入平台总线，Linux 内核提供了一种通用的机制来管理和组织与特定硬件平台相关的设备和驱动。它使得设备和驱动之间的匹配过程更加自动化和灵活，同时也提高了嵌入式系统的可移植性和可扩展性。

## 平台总线的优势

在前面的章节中，我们编写的驱动程序将驱动和设备相关的内容放在一起，但是当涉及到 多个相同类型的设备时，这种方法会引发一系列问题。举个例子，假设我们有一个硬件平台， 该硬件平台上存在了500 个模块，这些模块都使用了 LED 灯。如果我们使用杂项设备来编写驱 动，虽然相比字符设备，杂项设备的代码量较少，但我们仍旧需要编写500 份类似的代码，从 而生成相应的设备节点，以供上层应用在不同模块上控制 LED 灯。

编写500 份重复的代码会带来两个问题。首先，会造成大量重复劳动。其次，代码的重用性较差。如果我们需要将这些驱动从一个平台移植到另一个平台，就需要逐个修改驱动代码， 尽管只需修改与硬件相关的部分，但仍旧是一个很大的工作量

而在引入了平台总线模型后，这些问题就得到了很好地解决。通过使用平台总线模型，将设备驱动和平台设备进行了分离。这样一来，我们只需编写一份通用的驱动代码即可，然后针对不同的平台设备进行配置，这就大大减少了重复编写代码的工作量，并提高了驱动代码的重用性。当我们需要将驱动移植到不同的平台时，只需对硬件相关的部分进行适配即可，其他部分可以保持不变。

平台总线优势如下所示：

**（1）设备与驱动的分离：** 传统的设备驱动模型将设备和驱动代码合并在同一个文件中， 导致代码冗余和可维护性差。而平台总线模型将设备代码和驱动代码分离，设备代码放在 device.c 文件中，驱动代码放在 driver.c 文件中。这种分离使得设备和驱动的职责更加清晰，提 高了代码的可读性和可维护性。

**（2）提高代码的重用性：** 平台总线模型使得相同类型的设备可以共享相同的驱动代码。 例如，在一个硬件平台上存在多个相同类型的设备，传统的驱动模型需要为每个设备编写独立 的驱动代码。而使用平台总线模型，只需编写一个通用的驱动代码，然后为每个设备创建相应 的 device.c 文件，将设备特定的代码放在其中。这样可以减少代码的重复性，提高了代码的重 用性和可维护性。

**（3）减少重复性代码：** 在传统的设备驱动模型中，如果有多个相同类型的设备存在，就 需要为每个设备编写独立的驱动代码。而使用平台总线模型，只需编写一个通用的驱动代码， 然后为每个设备创建相应的 device.c 文件，将设备特定的代码放在其中。这样可以避免大量的重复性代码，简化了驱动开发过程。

**（4）提高可移植性：** 平台总线模型可以提高驱动的可移植性。开发者可以编写适应平台 总线的平台驱动程序，从而支持特定的外设，而无需依赖于特定的标准总线。这使得驱动可以更容易地在不同的硬件平台之间进行移植和重用。

# 注册platform设备

## **platform_device_register 函数**

platform_device_register 函数用于将 platform_device 结构体描述的平台设备注册到内核中。下面是对 platform_device_register 函数的详细介绍：

> **函数原型：**
>
> int platform_device_register(struct platform_device *pdev);
>
> **头文件：**
>
> \#include <linux/platform_device.h>
>
> **函数作用：**
>
> platform_device_register函数用于将 platform_device 结构体描述的平台设备注册到内核中，使其能够参与设备的资源分配和驱动的匹配。
>
> **参数含义：**
>
> pdev：指向 platform_device 结构体的指针，描述要注册的平台设备的信息。
>
> **返回值：**
>
> 成功：返回 0，表示设备注册成功。
>
> 失败：返回负数，表示设备注册失败，返回的负数值表示错误代码。

pdev 参数是一个指向 platform_device 结构体的指针，其中包含了描述平台设备的各种属性和信息。platform_device 结构体包含了设备名称、设备资源、设备 ID 等信息，用于描述和标识平台设备，会在接下来的小节对该结构体进行详细的介绍。

该函数在内核源码目录下的“/include/linux/platform_device.h”文件中，具体内容如下所示：

> extern int platform_device_register**(**struct platform_device ***);**

函数声明中的 extern 关键字表示该函数在其他地方定义，而不是在当前文件中实现。这样的声明通常出现在头文件中，用于告诉编译器该函数的定义存在于其他源文件中，以便在编译时能够正确引用该函数。

而 platform_device_register 实际定义在“/drivers/base/platform.c”文件中，相关定义如下所示：

```c
int platform_device_register(struct platform_device *pdev)
{
  device_initialize(&pdev->dev);
  arch_setup_pdev_archdata(pdev);
  return platform_device_add(pdev);
}
```

函数内部有三个主要的操作。

第 3 行：调用了 device_initialize 函数，用于对 pdev->dev 进行初始化。pdev->dev 是 struct platform_device 结构体中的一个成员，它表示平台设备对应的 struct device 结构体。通过调用 device_initialize 函数，对 pdev->dev 进行一些基本的初始化工作，例如设置设备的引用计数、 设备的类型等。

第 4 行：调用了 arch_setup_pdev_archdata 函数，用于根据平台设备的架构数据来设置 pdev 的架构相关数据。这个函数的具体实现可能与具体的架构相关，它主要用于在不同的架构下对 平台设备进行特定的设置。

第 5 行 ： 调 用 了 platform_device_add  函 数 ， 将 平 台 设 备 pdev 添 加 到 内 核 中 。 platform_device_add 函数会完成平台设备的添加操作，包括将设备添加到设备层级结构中、添 加设备的资源等。它会返回一个 int类型的结果，表示设备添加的结果。

platform_device_register 函数的主要作用是将 platform_device 结构体描述的平台设备注册 到内核中，包括设备的初始化、添加到 platform 总线和设备层级结构、添加设备资源等操作。 通过该函数，平台设备被注册后，就能够参与设备的资源分配和驱动的匹配过程。函数的返回 值可以用于判断设备注册是否成功。

## **platform_device_unregister 函数**

platform_device_unregister 函数用于取消注册已经注册的平台设备，即从内核中移除设备。在设备不再需要时，调用该函数可以进行设备的清理和释放操作。

> **函数原型：**
>
> void platform_device_unregister(struct platform_device *pdev);
>
> **头文件：**
>
> \#include <linux/platform_device.h>
>
> **函数作用：**
>
> platform_device_unregister 函数用于取消注册已经注册的平台设备，从内核中移除设备。
>
> 参数含义：
>
> pdev：指向要取消注册的平台设备的 platform_device 结构体指针。

该函数在内核源码目录下的“/include/linux/platform_device.h”文件中，具体内容如下

> extern int platform_device_unregister**(**struct platform_device ***);**

函数声明中的 extern 关键字表示该函数在其他地方定义，而不是在当前文件中实现。这样的声明通常出现在头文件中，用于告诉编译器该函数的定义存在于其他源文件中，以便在编译时能够正确引用该函数。

而 platform_device_unregister 实际定义在“/drivers/base/platform.c”文件中，相关定义如下所示

```c
void platform_device_unregister(struct platform_device *pdev)
{
    platform_device_del(pdev);
    platform_device_put(pdev);
}
```

函数内部有两个主要的操作：

第 3 行：调用了 platform_device_del 函数，用于将设备从 platform 总线的设备列表中移除。 它会将设备从设备层级结构中移除，停止设备的资源分配和驱动的匹配。

第 4 行：这一步调用了 platform_device_put 函数，用于减少对设备的引用计数。这个函数 会检查设备的引用计数，如果引用计数减为零，则会释放设备结构体和相关资源。通过减少引 用计数，可以确保设备在不再被使用时能够被释放。

platform_device_unregister 函数的作用是取消注册已经注册的平台设备，从内核中移除设 备 。 它 先 调 用 platform_device_del 函 数 将 设 备 从 设 备 层 级 结 构 中 移 除 ， 然 后 调 用 platform_device_put 函数减少设备的引用计数，确保设备在不再被使用时能够被释放。

## **platform_device 结构体**

platform_device 结构体是用于描述平台设备的数据结构。它包含了平台设备的各种属性和信息，用于在内核中表示和管理平台设备。该结构体定义在内核的“/include/linux/platform_device.h”文件中，具体内容如下所示：

```c
struct platform_device {
    const char *name; // 设备的名称，用于唯一标识设备
    int id; // 设备的 ID，可以用于区分同一种设备的不同实例
    bool id_auto; // 表示设备的 ID 是否自动生成
    struct device dev; // 表示平台设备对应的 struct device 结构体，用于设备的基本管理和操作
    u32 num_resources; // 设备资源的数量
    struct resource *resource; // 指向设备资源的指针
    const struct platform_device_id *id_entry; // 指向设备的 ID 表项的指针，用于匹配设备和驱动
    char *driver_override; // 强制设备与指定驱动匹配的驱动名称
    /* MFD cell pointer */
    struct mfd_cell *mfd_cell; // 指向多功能设备（MFD）单元的指针，用于多功能设备的描述
    /* arch specific additions */
    struct pdev_archdata archdata; // 用于存储特定于架构的设备数据
};
```

下面对于几个重要的参数和结构体进行讲解

const char *name：设备的名称，用于唯一标识设备。必须提供一个唯一的名称，以便内 核能够正确识别和管理该设备。

int id：设备的 ID ，可以用于区分同一种设备的不同实例。这个参数是可选的，如果不需 要使用 ID 进行区分，可以将其设置为-1，

struct device dev：表示平台设备对应的 struct device 结构体，用于设备的基本管理和操作。 必须为该参数提供一个有效的 struct device 对象，该结构体的 release 方法必须要实现，否则 在编译的时候会报错。

u32 num_resources：设备资源的数量。如果设备具有资源（如内存区域、中断等），则需 要提供资源的数量。

struct resource *resource：指向设备资源的指针。如果设备具有资源，需要提供一个指向 资源数组的指针，会在下个小节对该结构体进行详细的讲解。

## **resource 结构体**

struct resource 结构体用于描述系统中的设备资源，包括内存区域、I/O 端口、中断等，该结构体定义在内核的“/include/linux/ioport.h”文件中，具体内容如下所示：

```c
struct resource {
    resource_size_t start; /* 资源的起始 */
    resource_size_t end; /* 资源的结束 */
    const char *name; /* 资源的名称 */
    unsigned long flags; /* 资源的标志位 */
    unsigned long desc; /* 资源的描述信息 */
    struct resource *parent; /* 指向父资源的指针 */
    struct resource *sibling; /* 指向同级兄弟资源的指针 */
    struct resource *child; /* 指向子资源的指针 */
    /* 以下宏定义用于保留未使用的字段 */
    ANDROID_KABI_RESERVE(1);
    ANDROID_KABI_RESERVE(2);
    ANDROID_KABI_RESERVE(3);
    ANDROID_KABI_RESERVE(4);
};
```

其中最重要的是前四个参数，每个参数的具体介绍如下所示：

（1）resource_size_t start：资源的起始地址。它表示资源的起始位置或者起始寄存器的地 址。

（2）resource_size_t end：资源的结束地址。它表示资源的结束位置或者结束寄存器的地 址。

（3）const char *name：资源的名称。它是一个字符串，用于标识和描述资源。

（4）unsigned long flags：资源的标志位。它包含了一些特定的标志，用于表示资源的属 性或者特征。例如，可以用标志位来指示资源的可用性、共享性、缓存属性等。flags 参数的 具体取值和含义可以根据系统和驱动的需求进行定义和解释，但通常情况下，它用于表示资源 的属性、特征或配置选项。下面是一些常见的标志位及其可能的含义：

1. **资源类型相关标志位：**

   > IORESOURCE_IO：表示资源是 I**/**O 端口资源。
   >
   > IORESOURCE_MEM：表示资源是内存资源。
   >
   > IORESOURCE_REG：表示资源是寄存器偏移量。
   >
   > IORESOURCE_IRQ：表示资源是中断资源。
   >
   > IORESOURCE_DMA：表示资源是 DMA（直接内存访问）资源。

2. **资源属性和特征相关标志位：**

   > IORESOURCE_PREFETCH：表示资源是无副作用的预取资源。
   >
   > IORESOURCE_READONLY：表示资源是只读的。
   >
   > IORESOURCE_CACHEABLE：表示资源支持缓存。
   >
   > IORESOURCE_RANGELENGTH：表示资源的范围长度。
   >
   > IORESOURCE_SHADOWABLE：表示资源可以被影子资源替代。
   >
   > IORESOURCE_SIZEALIGN：表示资源的大小表示对齐。
   >
   > IORESOURCE_STARTALIGN：表示起始字段是对齐的。
   >
   > IORESOURCE_MEM_64：表示资源是 64 位内存资源。
   >
   > IORESOURCE_WINDOW：表示资源由桥接器转发。
   >
   > IORESOURCE_MUXED：表示资源是软件复用的。
   >
   > IORESOURCE_SYSRAM：表示资源是系统 RAM（修饰符）。

3. **其他状态和控制标志位：**

   > IORESOURCE_EXCLUSIVE：表示用户空间无法映射此资源。
   >
   > IORESOURCE_DISABLED：表示资源当前被禁用。
   >
   > IORESOURCE_UNSET：表示尚未分配地址给资源。
   >
   > IORESOURCE_AUTO：表示地址由系统自动分配。
   >
   > IORESOURCE_BUSY：表示驱动程序将此资源标记为繁忙。

## 实验代码

本实验将注册一个名为 "my_platform_device" 的平台设备，当注册平台设备时，该驱动程序提供了两个资源：一个内存资源和一个中断资源。这些资源被定义在名为 my_resources 的结构体数组中,具体内容如下：

**内存资源：**

起始地址：MEM_START_ADDR（0xFDD60000）

结束地址：MEM_END_ADDR（0xFDD60004）

标记：IORESOURCE_MEM

**中断资源：**

中断资源号：IRQ_NUMBER（101）

标记：IORESOURCE_IRQ

platform_device.c 代码如下所示：

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/ioport.h>

#define MEM_START_ADDR 0xFDD60000
#define MEM_END_ADDR   0xFDD60004
#define IRQ_NUMBER     101

static struct resource my_resources[] = {
    {
        .start = MEM_START_ADDR,    // 内存资源起始地址
        .end = MEM_END_ADDR,        // 内存资源结束地址
        .flags = IORESOURCE_MEM,    // 标记为内存资源
    },
    {
        .start = IRQ_NUMBER,        // 中断资源号
        .end = IRQ_NUMBER,          // 中断资源号
        .flags = IORESOURCE_IRQ,    // 标记为中断资源
    },
};

static void my_platform_device_release(struct device *dev)
{
    // 释放资源的回调函数
}

static struct platform_device my_platform_device = {
    .name = "my_platform_device",                  // 设备名称
    .id = -1,                                      // 设备ID
    .num_resources = ARRAY_SIZE(my_resources),     // 资源数量
    .resource = my_resources,                      // 资源数组
    .dev.release = my_platform_device_release,     // 释放资源的回调函数
};

static int __init my_platform_device_init(void)
{
    int ret;

    ret = platform_device_register(&my_platform_device);   // 注册平台设备
    if (ret) {
        printk(KERN_ERR "Failed to register platform device\n");
        return ret;
    }

    printk(KERN_INFO "Platform device registered\n");
    return 0;
}

static void __exit my_platform_device_exit(void)
{
    platform_device_unregister(&my_platform_device);   // 注销平台设备
    printk(KERN_INFO "Platform device unregistered\n");
}

module_init(my_platform_device_init);
module_exit(my_platform_device_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

# 注册 platform 驱动

## platform_driver_register 函数

platform_driver_register 函数用于在 Linux 内核中注册一个平台驱动程序。下面是对该函数的详细介绍：

> **函数原型：**
>
> int platform_driver_register(struct platform_driver *driver);
>
> **头文件：**
>
> \#include <linux/platform_device.h>
>
> **函数作用：**
>
> platform_driver_register 函数用于将一个平台驱动程序注册到内核中。通过注册平台驱动程序，内核可以识别并与特定的平台设备进行匹配，并在需要时调用相应的回调函数。
>
> **参数含义：**
>
> driver：指向 struct platform_driver 结构体的指针，描述了要注册的平台驱动程序的属性和回调函数（会在下面的小节对该结构体进行详细的讲解）。
>
> **返回值**：
>
> 返回一个整数值，表示函数的执行状态。如果注册成功，返回 0；如果注册失败，返回一个负数错误码。

该函数在内核源码目录下的“/include/linux/platform_device.h”文件中，具体内容如下所示：

> \#define platform_driver_register(drv) \ 
>
> __platform_driver_register(drv, THIS_MODULE)
>
> extern int __platform_driver_register**(**struct platform_driver ***,** struct module ***);**

这个宏用于简化平台驱动程序的注册过程。它将实际的注册函数 __platform_driver_register 与当前模块（驱动程序）关联起来。宏的参数 drv 是一个指向 struct platform_driver 结构体的指针，描述了要注册的平台驱动程序的属性和回调函数。THIS_MODULE 是一个宏，用于获取当前模块的指针。

而__platform_driver_register 实际定义在“/drivers/base/platform.c”文件中，相关定义如下所示：

```c
int __platform_driver_register(struct platform_driver *drv, struct module *owner)
{
    drv->driver.owner = owner; // 将平台驱动程序的所有权设置为当前模块
    drv->driver.bus = &platform_bus_type; // 将平台驱动程序的总线类型设置为平台总线
    drv->driver.probe = platform_drv_probe; // 设置平台驱动程序的探测函数
    drv->driver.remove = platform_drv_remove; // 设置平台驱动程序的移除函数
    drv->driver.shutdown = platform_drv_shutdown;// 设置平台驱动程序的关机函数
    return driver_register(&drv->driver); // 将平台驱动程序注册到内核
}
```

第 3 行：将指向当前模块的指针 owner 赋值给平台驱动程序的owner 成员。这样做是为 了将当前模块与平台驱动程序关联起来，以确保模块的生命周期和驱动程序的注册和注销相关 联。

第4 行：将指向平台总线类型的指针 &platform_bus_type 赋值给平台驱动程序的 bus 成员。 这样做是为了指定该驱动程序所属的总线类型为平台总线，以便内核能够将平台设备与正确的 驱动程序进行匹配。

第 5 行：将指向平台驱动程序探测函数 platform_drv_probe 的指针赋值给平台驱动程序 的 probe 成员。这样做是为了指定当内核发现与驱动程序匹配的平台设备时，要调用的驱动程 序探测函数。

第 6 行：将指向平台驱动程序移除函数 platform_drv_remove 的指针赋值给平台驱动程序 的 remove 成员。这样做是为了指定当内核需要从系统中移除与驱动程序匹配的平台设备时， 要调用的驱动程序移除函数。

第 7 行 = platform_drv_shutdown;：将指向平台驱动程序关机函数 platform_drv_shutdow n 的指针赋值给平台驱动程序的 shutdown 成员。这样做是为了指定当系统关机时，要调用的 驱动程序关机函数。

第 9 行：调用 driver_register 函数，将平台驱动程序的 driver 成员注册到内核中。该函数 负责将驱动程序注册到相应的总线上，并在注册成功时返回 0，注册失败时返回一个负数错误 码。

通过这些操作，  platform_driver_register 函数将平台驱动程序与内核关联起来，并确保 内核能够正确识别和调用驱动程序的各种回调函数，以实现与平台设备的交互和管理。函数的 返回值表示注册过程的执行状态，以便在需要时进行错误处理。

## platform_device_unregister 函数

platform_device_unregister 函数用于取消注册已经注册的平台设备，即从内核中移除设备。在设备不再需要时，调用该函数可以进行设备的清理和释放操作。

> **函数原型：**
>
> void platform_device_unregister(struct platform_device *pdev);
>
> **头文件：**
>
> \#include <linux/platform_device.h>
>
> **函数作用：**
>
> platform_device_unregister 函数用于从内核中注销平台设备。通过调用该函数，可以将指
>
> 定的平台设备从系统中移除。
>
> **参数含义**：
>
> pdev：指向要注销的平台设备的指针。
>
> **返回值：**
>
> 无返回值。

该函数在内核源码目录下的“/include/linux/platform_device.h”文件中，具体内容如下

> extern void platform_driver_unregister**(**struct platform_driver ***);**

函数声明中的 extern 关键字表示该函数在其他地方定义，而不是在当前文件中实现。这样的声明通常出现在头文件中，用于告诉编译器该函数的定义存在于其他源文件中，以便在编译时能够正确引用该函数。

而 platform_driver_unregister 实际定义在“/drivers/base/platform.c”文件中，相关定义如下所示：

```c
void platform_driver_unregister(struct platform_driver *drv)
{
		driver_unregister(&drv->driver);
}
```

该 函 数 又 调 用 了 driver_unregister 函 数 进 行 嵌 套 ， 追 踪 之 后 找 到 定 义 在“/drivers/base/driver.c”目录下的 driver_unregister 函数，具体内容如下

```c
void driver_unregister(struct device_driver *drv)
{
    // 检查传入的设备驱动程序指针和 p 成员是否有效
    if (!drv || !drv->p) {
        WARN(1, "Unexpected driver unregister!\n");
        return;
		}
  	driver_remove_groups(drv, drv->groups); // 移除与设备驱动程序关联的属性组
		bus_remove_driver(drv); // 从总线中移除设备驱动程序
}
```

函数内部有三个主要的操作：

第 4-7 行：检查传入的设备驱动程序指针 drv 是否为空，或者驱动程序的 p 成员是否为空。 如果其中任何一个条件为真，表示传入的参数无效，会发出警告并返回。

第 9 行：调用 driver_remove_groups 函数，用于从内核中移除与设备驱动程序关联的属性 组。drv->groups 是指向属性组的指针，指定了要移除的属性组列表。

第 10 行：调用 bus_remove_driver 函数，用于从总线中移除设备驱动程序。该函数会执行 以下操作：

（1）从总线驱动程序列表中移除指定的设备驱动程序。

（2）调用与设备驱动程序关联的 remove 回调函数（如果有定义）。

（3）释放设备驱动程序所占用的资源和内存。

（4）最终销毁设备驱动程序的数据结构。

通过调用 driver_unregister 函数，可以正确地注销设备驱动程序，并在注销过程中进行必要的清理工作。这样可以避免资源泄漏和其他问题。在调用该函数后，应避免继续使用已注销的设备驱动程序指针，因为该驱动程序已不再存在于内核中。

## platform_driver 结构体

platform_driver 结构体是 Linux 内核中用于编写平台设备驱动程序的重要数据结构。它提供了与平台设备驱动相关的函数和数据成员，以便与平台设备进行交互和管理。该结构体定义在内核的“/include/linux/platform_device.h”文件中，具体内容如下所示：

```c
struct platform_driver {
    int (*probe)(struct platform_device *); /* 平台设备的探测函数指针 */
    int (*remove)(struct platform_device *); /* 平台设备的移除函数指针 */
    void (*shutdown)(struct platform_device *);/* 平台设备的关闭函数指针 */
    int (*suspend)(struct platform_device *, pm_message_t state);/* 平台设备的挂起函数指针 */
    int (*resume)(struct platform_device *);/* 平台设备的恢复函数指针 */
    struct device_driver driver;/* 设备驱动程序的通用数据 */
    const struct platform_device_id *id_table;/* 平台设备与驱动程序的关联关系表 */
    bool prevent_deferred_probe; /* 是否阻止延迟探测 */
};
```

probe：平台设备的探测函数指针。当系统检测到一个平台设备与该驱动程序匹配时，该函数将被调用以初始化和配置设备。

remove：平台设备的移除函数指针。当平台设备从系统中移除时，该函数将被调用以执行 清理和释放资源的操作。

shutdown：平台设备的关闭函数指针。当系统关闭时，该函数将被调用以执行与平台设备 相关的关闭操作。

suspend：平台设备的挂起函数指针。当系统进入挂起状态时，该函数将被调用以执行与 平台设备相关的挂起操作。

resume：平台设备的恢复函数指针。当系统从挂起状态恢复时，该函数将被调用以执行与 平台设备相关的恢复操作。

driver：包含了与设备驱动程序相关的通用数据，它是 struct device_driver 类型的实例。其 中包括驱动程序的名称、总线类型、模块拥有者、属性组数组指针等信息，该结构体的 name  参数需要与上个章节的 platform_device 的.name 参数相同才能匹配成功，从而进入 probe 函数。

id_table：指向 struct platform_device_id 结构体数组的指针，用于匹配平台设备和驱动程 序之间的关联关系。通过该关联关系，可以确定哪个平台设备与该驱动程序匹配，和.driver.name 起到相同的作用，但是优先级高于.driver.name。

prevent_deferred_probe：一个布尔值，用于确定是否阻止延迟探测。如果设置为 true，则 延迟探测将被禁用。

使用 struct platform_driver 结构体，开发人员可以定义平台设备驱动程序，并将其注册到 内核中。当系统检测到与该驱动程序匹配的平台设备时，内核将调用相应的函数来执行设备的 初始化、配置、操作和管理。驱动程序可以利用提供的函数指针和通用数据与平台设备进行交 互，并提供必要的功能和服务。

需要注意的是，struct platform_driver 结构体继承了 struct device_driver 结构体，因此可以 直接访问 struct device_driver 中定义的成员。这使得平台驱动程序可以利用通用的驱动程序机 制，并与其他类型的设备驱动程序共享代码和功能。

## 实验代码

```c
#include <linux/module.h>
#include <linux/platform_device.h>

// 平台设备的探测函数
static int my_platform_probe(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_probe: Probing platform device\n");

    // 添加设备特定的操作
    // ...

    return 0;
}

// 平台设备的移除函数
static int my_platform_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");

    // 清理设备特定的操作
    // ...

    return 0;
}

// 定义平台驱动结构体
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
    },
};

// 模块初始化函数
static int __init my_platform_driver_init(void)
{
    int ret;

    // 注册平台驱动
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        printk(KERN_ERR "Failed to register platform driver\n");
        return ret;
    }

    printk(KERN_INFO "my_platform_driver: Platform driver initialized\n");

    return 0;
}

// 模块退出函数
static void __exit my_platform_driver_exit(void)
{
    // 注销平台驱动
    platform_driver_unregister(&my_platform_driver);

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

# probe 函数

在上面的两个章节中分别注册了 platform 设备和 platform 驱动，匹配成功之后会进入在注册 platform 驱动程序中编写的 probe 函数，在上个章节只是为了验证是否匹配成功，所以只是在 probe 中加入了一句相关打印，而驱动是要控制硬件的，但是平台总线模型对硬件的描述写在了 platform_device.c 中,platform 设备和 platform 驱动匹配成功之后，那我们如何在驱动platform_driver.c 的 probe 函数中，得到 platform_device.c 中编写的硬件资源呢。

## **方法** **1**：**直接访问** **platform_device** **结构体的资源数组**

在上一章节的讲解中提到：struct platform_driver 结构体继承了 struct device_driver 结构体，因此可以直接访问 struct device_driver 中定义的成员。实例代码如下所示：

```c
if (pdev->num_resources >= 2) {
    struct resource *res_mem = &pdev->resource[0];
    struct resource *res_irq = &pdev->resource[1];
    // 使用获取到的硬件资源进行处理
    printk("Method 1: Memory Resource: start = 0x%llx, end = 0x%llx\n", res_mem->start, res_mem->end);
    printk("Method 1: IRQ Resource: number = %lld\n", res_irq->start);
}
```

在 这种 方法 中， 直接 访 问 platform_device 结 构体 的资 源数 组来 获 取硬 件资 源。pdev->resource 是一个资源数组，其中存储了设备的硬件资源信息。通过访问数组的不同索引，可以获取到特定的资源。

在这个示例中，假设资源数组的第一个元素是内存资源，第二个元素是中断资源。所以我们将第一个元素的指针赋值给 res_mem，第二个元素的指针赋值给 res_irq。

## **方法** **2**：使用 **platform_get_resource()** **获取硬件资源**

platform_get_resource()函数用于获取设备的资源信息。它的声明位于<linux/platform_device.h>头文件中，与平台设备（platform_device）相关。

> **函数原型：**
>
> struct resource *platform_get_resource(struct platform_device *pdev,unsigned int type, unsigned int num);
>
> **参数说明：**
>
> pdev：指向要获取资源的平台设备（platform_device）结构体的指针。
>
> type：指定资源的类型，可以是以下值之一：
>
> IORESOURCE_MEM：表示内存资源。
>
> IORESOURCE_IO：表示 I/O 资源。
>
> IORESOURCE_IRQ：表示中断资源。
>
> 其他资源类型的宏定义可在<linux/ioport.h>和<linux/irq.h>头文件中找到。
>
> num：指定要获取的资源的索引。在一个设备中可能存在多个相同类型的资源，通过索引可以选择获取特定的资源。
>
> **返回值：**
>
> 如果成功获取资源，则返回指向资源（struct resource）的指针。
>
> 如果获取资源失败，或者指定的资源不存在，则返回 NULL。

platform_get_resource()函数用于从平台设备的资源数组中获取指定类型和索引的资源。在平台设备的资源数组中，每个元素都是一个 struct resource 结构体，描述了一个资源的信息，如起始地址、结束地址、中断号等。

示例用法：

```c
struct platform_device *pdev;
struct resource *res;
res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
if (!res) {
// 处理获取内存资源失败的情况
}
// 使用获取到的内存资源进行处理
unsigned long start = res->start;
unsigned long end = res->end;
```

在上述示例中，首先通过 platform_get_resource()函数获取平台设备的第一个内存资源（索引为 0）。如果获取资源失败（返回 NULL），则可以根据实际情况进行错误处理。如果获取资源成功，则可以使用返回的资源指针来访问资源的信息，如起始地址和结束地址。

通过 platform_get_resource()函数，可以方便地在驱动程序中获取平台设备的资源信息，并根据这些信息进行后续的操作和配置。

## 实验代码

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/ioport.h>

static int my_platform_driver_probe(struct platform_device *pdev)
{
    struct resource *res_mem, *res_irq;

    // 方法1：直接访问 platform_device 结构体的资源数组
    if (pdev->num_resources >= 2) {
        struct resource *res_mem = &pdev->resource[0];
        struct resource *res_irq = &pdev->resource[1];

        // 使用获取到的硬件资源进行处理
        printk("Method 1: Memory Resource: start = 0x%llx, end = 0x%llx\n",
                res_mem->start, res_mem->end);
        printk("Method 1: IRQ Resource: number = %lld\n", res_irq->start);
    }

    // 方法2：使用 platform_get_resource() 获取硬件资源
    res_mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res_mem) {
        dev_err(&pdev->dev, "Failed to get memory resource\n");
        return -ENODEV;
    }

    res_irq = platform_get_resource(pdev, IORESOURCE_IRQ, 0);
    if (!res_irq) {
        dev_err(&pdev->dev, "Failed to get IRQ resource\n");
        return -ENODEV;
    }

    // 使用获取到的硬件资源进行处理
    printk("Method 2: Memory Resource: start = 0x%llx, end = 0x%llx\n",
            res_mem->start, res_mem->end);
    printk("Method 2: IRQ Resource: number = %lld\n", res_irq->start);

    return 0;
}

static int my_platform_driver_remove(struct platform_device *pdev)
{
    // 设备移除操作
    return 0;
}

static struct platform_driver my_platform_driver = {
    .driver = {
        .name = "my_platform_device", // 与 platform_device.c 中的设备名称匹配
        .owner = THIS_MODULE,
    },
    .probe = my_platform_driver_probe,
    .remove = my_platform_driver_remove,
};

static int __init my_platform_driver_init(void)
{
    int ret;

    ret = platform_driver_register(&my_platform_driver); // 注册平台驱动
    if (ret) {
        printk("Failed to register platform driver\n");
        return ret;
    }

    printk("Platform driver registered\n");
    return 0;
}

static void __exit my_platform_driver_exit(void)
{
    platform_driver_unregister(&my_platform_driver); // 注销平台驱动
    printk("Platform driver unregistered\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

# 平台总线点灯实验

驱动

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/ioport.h>
#include <linux/kdev_t.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>
#include <linux/io.h>

struct device_test{

    dev_t dev_num;  //设备号
     int major ;  //主设备号
    int minor ;  //次设备号
    struct cdev cdev_test; // cdev
    struct class *class;   //类
    struct device *device; //设备
    char kbuf[32];
    unsigned int *vir_gpio_dr;
};

struct  device_test dev1;


/*打开设备函数*/
static int cdev_test_open(struct inode *inode, struct file *file)
{
    file->private_data=&dev1;//设置私有数据
    printk("This is cdev_test_open\r\n");

    return 0;
}

/*向设备写入数据函数*/
static ssize_t cdev_test_write(struct file *file, const char __user *buf, size_t size, loff_t *off)
{
     struct device_test *test_dev=(struct device_test *)file->private_data;

    if (copy_from_user(test_dev->kbuf, buf, size) != 0) // copy_from_user:用户空间向内核空间传数据
    {
        printk("copy_from_user error\r\n");
        return -1;
    }
    if(test_dev->kbuf[0]==1){   //如果应用层传入的数据是1，则打开灯
            *(test_dev->vir_gpio_dr) = 0x8000c040;   //设置数据寄存器的地址
              printk("test_dev->kbuf [0]  is %d\n",test_dev->kbuf[0]);  //打印传入的数据
    }
    else if(test_dev->kbuf[0]==0)  //如果应用层传入的数据是0，则关闭灯
    {
            *(test_dev->vir_gpio_dr) = 0x80004040; //设置数据寄存器的地址
            printk("test_dev->kbuf [0]  is %d\n",test_dev->kbuf[0]); //打印传入的数据
    }
    return 0;
}

/**从设备读取数据*/
static ssize_t cdev_test_read(struct file *file, char __user *buf, size_t size, loff_t *off)
{

    struct device_test *test_dev=(struct device_test *)file->private_data;

    if (copy_to_user(buf, test_dev->kbuf, strlen( test_dev->kbuf)) != 0) // copy_to_user:内核空间向用户空间传数据
    {
        printk("copy_to_user error\r\n");
        return -1;
    }

    printk("This is cdev_test_read\r\n");
    return 0;
}

static int cdev_test_release(struct inode *inode, struct file *file)
{
    printk("This is cdev_test_release\r\n");
    return 0;
}

/*设备操作函数*/
struct file_operations cdev_test_fops = {
    .owner = THIS_MODULE, //将owner字段指向本模块，可以避免在模块的操作正在被使用时卸载该模块
    .open = cdev_test_open, //将open字段指向chrdev_open(...)函数
    .read = cdev_test_read, //将open字段指向chrdev_read(...)函数
    .write = cdev_test_write, //将open字段指向chrdev_write(...)函数
    .release = cdev_test_release, //将open字段指向chrdev_release(...)函数
};

static int my_platform_driver_probe(struct platform_device *pdev)
{
    struct resource *res_mem;
	int ret;
	res_mem = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!res_mem) {
        dev_err(&pdev->dev, "Failed to get memory resource\n");
        return -ENODEV;
    }

	/*注册字符设备驱动*/
    /*1 创建设备号*/
    ret = alloc_chrdev_region(&dev1.dev_num, 0, 1, "alloc_name"); //动态分配设备号
    if (ret < 0)
    {
       goto err_chrdev;
    }
    printk("alloc_chrdev_region is ok\n");

    dev1.major = MAJOR(dev1.dev_num); //获取主设备号
   dev1.minor = MINOR(dev1.dev_num); //获取次设备号

    printk("major is %d \r\n", dev1.major); //打印主设备号
    printk("minor is %d \r\n", dev1.minor); //打印次设备号
     /*2 初始化cdev*/
    dev1.cdev_test.owner = THIS_MODULE;
    cdev_init(&dev1.cdev_test, &cdev_test_fops);

    /*3 添加一个cdev,完成字符设备注册到内核*/
   ret =  cdev_add(&dev1.cdev_test, dev1.dev_num, 1);
    if(ret<0)
    {
        goto  err_chr_add;
    }
    /*4 创建类*/
  dev1. class = class_create(THIS_MODULE, "test");
    if(IS_ERR(dev1.class))
    {
        ret=PTR_ERR(dev1.class);
        goto err_class_create;
    }
    /*5  创建设备*/
  dev1.device = device_create(dev1.class, NULL, dev1.dev_num, NULL, "test");
    if(IS_ERR(dev1.device))
    {
        ret=PTR_ERR(dev1.device);
        goto err_device_create;
    }
    dev1.vir_gpio_dr=ioremap(res_mem->start,4);  //将物理地址转化为虚拟地址
    if(IS_ERR(dev1.vir_gpio_dr))
    {
        ret=PTR_ERR(dev1.vir_gpio_dr);  //PTR_ERR()来返回错误代码
        goto err_ioremap;
    }


return 0;

err_ioremap:
        iounmap(dev1.vir_gpio_dr);

 err_device_create:
        class_destroy(dev1.class);                 //删除类

err_class_create:
       cdev_del(&dev1.cdev_test);                 //删除cdev

err_chr_add:
        unregister_chrdev_region(dev1.dev_num, 1); //注销设备号

err_chrdev:
        return ret;
}

static int my_platform_driver_remove(struct platform_device *pdev)
{
    // 设备移除操作
    return 0;
}

static struct platform_driver my_platform_driver = {
    .driver = {
        .name = "my_platform_device", // 与 platform_device.c 中的设备名称匹配
        .owner = THIS_MODULE,
    },
    .probe = my_platform_driver_probe,
    .remove = my_platform_driver_remove,
};

static int __init my_platform_driver_init(void)
{
    int ret;

    ret = platform_driver_register(&my_platform_driver); // 注册平台驱动
    if (ret) {
        printk("Failed to register platform driver\n");
        return ret;
    }

    printk("Platform driver registered\n");
    return 0;
}

static void __exit my_platform_driver_exit(void)
{
        /*注销字符设备*/
    unregister_chrdev_region(dev1.dev_num, 1); //注销设备号
    cdev_del(&dev1.cdev_test);                 //删除cdev
    device_destroy(dev1.class, dev1.dev_num);       //删除设备
    class_destroy(dev1.class);                 //删除类
	platform_driver_unregister(&my_platform_driver); // 注销平台驱动
    printk("Platform driver unregistered\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

应用

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

int main(int argc, char *argv[])  
{
    int fd;
    char buf[32] = {0};   
    fd = open("/dev/test", O_RDWR);  //打开led驱动
    if (fd < 0)
    {
        perror("open error \n");
        return fd;
    }
    buf[0] =atoi(argv[1]);    // atoi()将字符串转为整型，这里将第一个参数转化为整型后，存放在 buf[0]中
    write(fd,buf,sizeof(buf));  //向/dev/test文件写入数据
    close(fd);     //关闭文件
    return 0;
}
```

# 设备树

设备树（Device Tree） 是一种硬件描述机制， 用于在嵌入式系统和操作系统中描述硬件设备的特性、 连接关系和配置信息。 它提供了一种与平台无关的方式来描述硬件， 使得内核与硬件之间的耦合度降低， 提高了系统的可移植性和可维护性。
在上一篇平台总线内容的学习中， 我们使用 platform_device 结构体来对硬件设备进行描述， 这是一种传统的平台总线设备描述方式。 每个 platform_device 结构表示一个特定的硬件设备， 并通过注册到平台总线上来使得内核能够与该设备进行通信和交互。 该结构包含设备的名称、 资源（如内存地址、 中断号等） 、 设备驱动程序等信息。
然而， 随着时间的推移， Linux 内核中的 ARM 部分存在着大量的平台相关配置代码， 这些代码通常是杂乱而重复的， 导致了维护的困难和工作量的增加。 在 2011 年 3 月 17 日， Linux的创始人 Linus Torvalds 在 ARM Linux 邮件列表中发表了一封帖子， 他表达了对 ARM 架构配置方式的不满， 并宣称"Gaah. Guys, this whole ARM thing is a f*cking pain in the ass"。 这引起了广泛的讨论和反思。 ARM 社区中的开发者们开始认识到， 传统的平台相关配置方式已经变得不可持续， 需要一种更加先进和可扩展的方法来解决这个问题。
为了应对这一挑战， ARM 社区开始探索新的硬件描述机制， 并逐渐形成了设备树的概念。设备树提供了一种更加灵活和可移植的描述硬件的机制， 将设备的描述信息转移到设备树中。设备树使用一种结构化的数据格式， 通过描述设备节点、 属性和连接关系等信息， 使得硬件的描述与具体的平台无关， 同时允许多个平台共享相同的设备树描述。

设备树的引入为 ARM 架构上的 Linux 内核带来了革命性的变化。它提供了一种统一的硬件描述方式， 使得不同芯片和板级的支持更加简单和灵活。 此外， 设备树还提供了硬件配置的可视化和可读性， 方便开发者理解和调试硬件。随着时间的推移， 设备树逐渐成为了嵌入式系统和 Linux 内核中描述硬件的标准方式。 它不仅在 ARM 架构上得到了广泛应用， 也被扩展到其他架构和平台上。

## 设备树基础知识

当描述设备树（Device Tree） 时， 通常会涉及到以下几个关键术语： DTS、 DTSI、 DTB 和DTC。 下面来对每个术语进行介绍。

DTS（ Device Tree Source） ： DTS 是设备树的源文件， 采用一种类似于文本的语法来描述硬件设备的结构、 属性和连接关系。 DTS 文件以.dts 为扩展名， 通常由开发人员编写。 它是人类可读的形式， 用于描述设备树的层次结构和属性信息。
DTSI（ Device Tree Source Include） ： DTSI 文件是设备树源文件的包含文件。 它扩展了 DTS文件的功能， 用于定义可重用的设备树片段。 DTSI 文件以.dtsi 为扩展名， 可以在多个 DTS 文件中包含和共享。 通过使用 DTSI， 可以提高设备树的可重用性和可维护性（ 和 C 语言中头文件的作用相同） 。
DTB（Device Tree Blob） ： DTB 是设备树的二进制表示形式。 DTB 文件是通过将 DTS 或 DTSI文件编译而成的二进制文件， 以.dtb 为扩展名。 DTB 文件包含了设备树的结构、 属性和连接信息， 被操作系统加载和解析。 在运行时， 操作系统使用 DTB 文件来动态识别和管理硬件设备。
DTC（ Device Tree Compiler） ： DTC 是设备树的编译器。 它是一个命令行工具， 用于将 DTS和 DTSI 文件编译成 DTB 文件。 DTC 将文本格式的设备树源代码转换为二进制的设备树表示形式， 以便操作系统能够加载和解析。 DTC 是设备树开发中一个重要的工具。

DTS、 DTSI、 DTB 和 DTC 之间的关系：

1. 开发人员使用文本编辑器编写 DTS 和 DTSI 文件， 描述硬件设备的层次结构、 属性和连接关系。
2. DTSI 文件可以在多个 DTS 文件中包含和共享， 以提高设备树的可重用性和可维护性。
3. 使用 DTC 编译器， 开发人员将 DTS 和 DTSI 文件编译成二进制的 DTB 文件， 如下图

![image-20240711145616844](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711145616844.png)

4. 操作系统在启动过程中加载和解析 DTB 文件， 以识别和管理硬件设备。

设备树文件存放路径：

**ARM 体系结构**

ARM 体系结构下的设备树源文件通常存放在 arch/arm/boot/dts/目录中。 该目录是设备树源文件的根目录。 如下图

![image-20240711145724594](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711145724594.png)

**ARM64 体系结构**

设备树源文件路径： ARM64 体系结构下的设备树源文件通常存放在 **arch/arm64/boot/dts/**目录及其子目录中。 该目录也是设备树源文件的根目录， 并包含了针对不同 ARM64 平台和设备的子目录， 如下图

![image-20240711145815759](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711145815759.png)

子目录结构： 在 ARM64 的子目录中， 同样会按照硬件平台、 设备类型或制造商进行组织和分类。 这些子目录的命名可能与特定芯片厂商（如 Qualcomm、 NVIDIA、 Samsung） 有关，由 于 我 们 本 手 册 使 用 的 soc 是 瑞 芯 微 的 rk3568 ， 所 以 匹 配 的 设 备 树 目 录 为arch/arm64/boot/dts/rockchip。 每个子目录中可能包含多个设备树文件， 用于描述不同的硬件配置和设备类型， 这里以 rockchip 目录内容如下图

![image-20240711145843518](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711145843518.png)

## 设备树的编译

设备树的编译是将设备树源文件（如上述的.dts 文件）转换为二进制的设备树表示形式（.dtb文件） 的过程。 编译器通常被称为 DTC（Device Tree Compiler） 。
在 Linux 内核源码中， DTC（ Device Tree Compiler） 的源代码和相关工具通常存放在scripts/dtc/目录中， 如下图

![image-20240711145933372](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711145933372.png)

在编译完源码之后 dtc 设备树编译器会默认生成， 如果没有生成相应的 dtc 可执行文件，可以查看在内核默认配置文件中 CONFIG_DTC 是否使能。

**设备树的编译：**

在 Linux 环境中， 可以使用以下命令将设备树源文件编译为二进制设备树文件

> dtc -I dts -O dtb -o output.dtb input.dts

其中， `input.dts`是输入的设备树源文件， `output.dtb`是编译后的二进制设备树文件。
编译器会验证设备树源文件的语法和语义， 生成与硬件描述相对应的设备树表示形式。

**设备树的反编译：**

设备树的反编译是将二进制设备树文件转换回设备树源文件的过程， 以便进行查看、 编辑或修改。 反编译器通常也是 DTC。

在 Linux 环境中， 可以使用以下命令将二进制设备树文件反编译为设备树源文件：

> dtc -I dtb -O dts -o output.dts input.dtb

其中， input.dtb 是输入的二进制设备树文件， output.dts 是反编译后的设备树源文件。
反编译器会将二进制设备树文件解析并还原为文本形式的设备树源文件， 使其可读性更好。

## 示例

下面来进行一下实际的设备树编译和反编译的演示， 首先创建一个名为 test.dts 的设备树文件， 文件内容如下所示

```
/dts-v1/;
/ {

};
```

这个设备树很简单， 只包含了根节点/， 而根节点中没有任何子节点或属性。 这个示例并没有描述任何具体的硬件设备或连接关系， 它只是一个最基本的设备树框架， 在本小节只是为了测试设备树的编译和反编译。
然后使用以下命令进行设备树的编译， 编译完成如下图

> /home/topeet/Linux/linux_sdk/kernel/scripts/dtc/dtc -I dts -O dtb -o test.dtb test.dts

![image-20240711150328687](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711150328687.png)

可以看到 test.dtb 就生成了， 然后继续使用以下命令对 test.dtb 进行反编译， 反编译完成如下图

> /home/topeet/Linux/linux_sdk/kernel/scripts/dtc/dtc -I dtb -O dts -o 1.dts test.dtb

![image-20240711150407052](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711150407052.png)

# 设备树语法

## **根节点**

设备树使用一种层次结构，其中的根节点（Root Node）是整个设备树的起始点和顶层节点。根节点由一个以/开头的标识符来表示，然后使用{}来包含根节点所在的内容，一个最简单的根节点示例如下所示：

```
/dts-v1/; // 设备树版本信息
/ {
    // 根节点开始
    /*
    在这里可以添加注释，描述根节点的属性和配置
    */
};
```

其中第一行的设备树中的版本信息行 dts-v1 是可选的，可以根据需要选择是否保留。这行注释通常用于指定设备树的语法版本。如果您不需要在设备树中指定版本信息，可以将其删除。

## **子节点**

设备树中的子节点是根节点的直接子项，用于描述具体的硬件设备或设备集合。子节点采用以下特定的格式来表示:

> [label:] node-name[@unit-address] {
> 		[properties definitions]
> 		[child nodes]
> };

以下是对这些部分的详细介绍：

1. 节点标签（Label）（可选）：节点标签是一个可选的标识符，用于在设备树中引用 该节点。标签允许其他节点直接引用此节点，以便在设备树中建立引用关系。

2. 节点名称（Node Name）：节点名称是一个字符串，用于唯一标识该节点在设备树 中的位置。节点名称通常是硬件设备的名称，但必须在设备树中是唯一的。

3. 单元地址（Unit Address）（可选）：单元地址用于标识设备的实例。它可以是一个 整数、一个十六进制值或一个字符串，具体取决于设备的要求。单元地址的目的是区分相同类 型的设备的不同实例，例如在下图（55-6）中名为 cpu 的节点通过它们的单元地址值 0 和 1 来 区分，名称为 ethernet 的节点通过其单元地址值 fe002000 和 fe003000 来区分。

![image-20240711203128588](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711203128588.png)

4. 属性定义（Properties Definitions）：属性定义是一组键值对，用于描述设备的配置 和特性。属性可以根据设备的需求进行定义，例如寄存器地址、中断号、时钟频率等，关于这 些属性会在后面的小节中进行讲解

5. 子节点（Child Nodes）：子节点是当前节点的子项，用于进一步描述硬件设备的子 组件或配置。子节点可以包含自己的属性定义和更深层次的子节点，形成设备树的层次结构。

## **reg 属性**

reg 属性用于在设备树中指定设备的寄存器地址和大小，提供了与设备树中的物理设备之 间的寄存器映射关系。

reg 属性可以在设备节点中有单个值格式和列表值格式这两种常见格式，接下来将对这两 种格式进行介绍：

1. 单个值格式如下所示：

> reg=<address	size>

这种格式适用于描述单个寄存器的情况。其中，address 是设备的起始寄存器地址，可以 是一个整数或十六进制值。size 表示寄存器的大小，即占用的字节数。

例如，假设有一个设备节点 my_device，使用单个值格式的 reg 属性来描述一个 4 字节寄 存器的地址和大小，可以这样定义：

my_device {
    compatible="vendor,device"; 
    reg=<0x10000 x4>;
    //  其他属性和子节点的定义
}


在这个示例中，`my_device` 设备节点的 `reg`  属性值为 `<0x1000 0x4>` ，表示从地址 `0x1000` 开始的 4 字节寄存器区域。

2. 列表值格式如下所示：

> reg=<address1	size1	address2	size2	...>;

当设备具有多个寄存器区域时，可以使用列表值格式的 reg 属性来描述每个寄存器区域的 地址和大小。通过这种方式，可以指定多个寄存器的位置和大小，以描述设备的完整寄存器映 射。

例如，考虑一个设备节点 my_device ，它具有两个寄存器区域，分别是 8 字节和 4 字节 大小的寄存器。可以使用列表值格式的 reg 属性来描述这种情况：

```
my_device {
		compatible="vendor,device";     
		reg=<0x10000 x8 0x2000 0x4>;
		//  其他属性和子节点的定义
};
```

在这个示例中，my_device 设备节点的 reg 属性值为 <0x1000 0x8 0x2000 0x4>，表示设备有 两个寄存器区域。第一个寄存器区域从地址 0x1000 开始，大小为 8 字节；第二个寄存器区域 从地址 0x2000 开始，大小为 4 字节。

通过使用reg 属性，设备树可以提供有关设备寄存器布局和寄存器访问方式的信息。这对 于操作系统的设备驱动程序很重要，因为它们需要了解设备的寄存器映射以正确地与设备进行 交互和配置。

## **#address-cells 和 #size-cells 属性**

\#address-cells 和 #size-cells 属性用于指定在上个小节中要设置的设备树中地址单元和大小 单元的位数。它们提供了设备树解析所需的元数据，以正确解释设备的地址和大小信息。下面 对两个属性分别进行介绍：

1. **#address-cells** 属性

\#address-cells 属性是一个位于设备树根节点的特殊属性，它指定了设备树中地址单元的位数。地址单元是设备树中用于表示设备地址的单个单位。它通常是一个整数，可以是十进制或 十六进制值。

\#address-cells 属性的值告诉解析设备树的软件在解释设备地址时应该使用多少位来表示 一个地址单元。

默认情况下，#address-cells 的值为 2，表示使用两个单元来表示一个设备地址。这意味着 设备的地址将由两个整数（每个整数使用指定位数的位）组成。

例如，对于一个使用两个 32 位（4 字节）整数表示地址的设备，可以在设备树的根节点中设置 #address-cells 属性为 <2>。

2. **#size-cells** 属性

\#size-cells 属性也是一个位于设备树根节点的特殊属性，它指定了设备树中大小单元的位 数。大小单元是设备树中用于表示设备大小的单个单位。它通常是一个整数，可以是十进制或 十六进制值。

\#size-cells 属性的值告诉解析设备树的软件在解释设备大小时应该使用多少位来表示一个 大小单元。

默认情况下，#size-cells 的值为 1 ，表示使用一个单元来表示一个设备的大小。这意味着 设备的大小将由一个整数（使用指定位数的位）表示。

例如，对于一个使用一个 32 位（4 字节）整数表示大小的设备，可以在设备树的根节点 中设置 #size-cells 属性为 <1>。

这两个属性的存在是为了使设备树能够灵活地描述各种设备的地址和大小表示方式。通过 在设备树的根节点中设置适当的 #address-cells 和 #size-cells 值，设备树解析软件能够正确地解 释设备节点中的地址和大小信息。

以下是两个个示例，展示了根节点中 #address-cells 和 #size-cells 属性的使用：

```
node1 {
    #address-cells = <1>;
    #size-cells = <1>;
    node1-child {
        reg = <0x02200000 0x4000>;
        // 其他属性和子节点的定义
    };
};
```

在这个示例中，node1-child 节点的 reg 属性使用了 <0x02200000 0x4000> 表示地址和大小。由于 #address-cells 的值为 <1>，表示使用一个单元来表示地址。#size-cells 的值也为 <1>，表示使用一个单元来表示大小。

解释后的地址和大小值如下：

地址部分：0x02200000 被解释为一个地址单元，地址为 0x02200000。

大小部分：0x4000 被解释为一个大小单元，大小为 0x4000。

```
node1 {
    #address-cells = <2>;
    #size-cells = <0>;
    node1-child {
        reg = <0x0000 0x0001>;
        // 其他属性和子节点的定义
    };
};
```

在这个示例中，node1-child 节点的 reg 属性使用了 <0x0000 0x0001> 表示地址。由于 #address-cells 的值为 <2>，表示使用两个单元来表示地址。#size-cells 的值为 <0>，表示不使用单元来表示大小。

解释后的地址值如下：

地址部分：0x0000 0x0001 被解释为两个地址单元，其中第一个地址单元为 0x0000，第二个地址单元为 0x0001。

这种使用 #address-cells 和 #size-cells 属性的方式使得设备树可以适应不同设备的寄存器映射和大小表示方式，并确保设备树解析软件能够正确解释设备的地址和大小信息。

#### model 属性

在设备树中，model 属性用于描述设备的型号或者名称。它通常作为设备节点的一个属性，用来提供关于设备的标识信息。model 属性是可选的，但在实际应用中经常被使用。

model 属性的值是一个字符串，可以是设备的型号、名称、或者其他标识符，用来识别设备。该值通常由设备的厂商定义，并且在设备树中使用。

以下是一个示例，展示了如何在设备树中使用 model 属性：

```
my_device {
    compatible = "vendor,device";
    model = "My Device XYZ";
    // 其他属性和子节点的定义
};
```

在这个示例中，my_device 节点具有 model 属性，其值为 "My Device XYZ"。这个值描述了设备的型号或名称为 "My Device XYZ"。

model 属性通常用于标识和区分不同的设备，特别是当设备节点的 compatible 属性相同或相似时。通过使用不同的 model 属性值，可以更加准确地确定所使用的设备类型。

#### status 属性

在设备树中，status 属性用于描述设备或节点的状态。它是设备树中常见的属性之一，用于表示设备或节点的可用性或操作状态。

status 属性的值可以是以下几种：

> "okay"：表示设备或节点正常工作，可用。
>
> "disabled"：表示设备或节点被禁用，不可用。
>
> "reserved"：表示设备或节点已被保留，暂时不可用。
>
> "fail"：表示设备或节点初始化或操作失败，不可用。

以下是一个示例，展示了如何在设备树中使用 status 属性：

```
my_device {
    compatible = "vendor,device";
    status = "okay";
    // 其他属性和子节点的定义
};
```

在这个示例中，my_device 节点具有 status 属性，其值为 "okay"。这表示设备处于正常工作状态，可用。

通过使用 status 属性，设备树可以动态地控制设备的启用和禁用状态。这对于在系统启动过程中选择性地启用或禁用设备，或者在运行时根据特定条件调整设备状态非常有用。

#### compatible 属性

在设备树中，compatible 属性用于描述设备的兼容性信息。它是设备树中重要的属性之一，用于识别设备节点与驱动程序之间的匹配关系。

compatible 属性的值是一个字符串或字符串列表，用于指定设备节点与相应的驱动程序或设备描述符兼容的规则。通常，compatible 属性的值由设备的厂商定义，并且在设备树中使用。

以下是一些常见的 compatible 属性值的示例：

1. 单个字符串值：例如 "vendor,device"，用于指定设备节点与特定厂商的特定设备兼容。

2. 字符串列表：例如 ["vendor,device1", "vendor,device2"]，用于指定设备节点与多个设备兼容，通常用于设备节点具有多种变体或配置。
3. 通配符匹配：例如 "vendor,*"，用于指定设备节点与特定厂商的所有设备兼容，不考虑具体的设备标识。

以下是一个示例，展示了如何在设备树中使用 compatible 属性：

```
my_device {
    compatible = "vendor,device";
    // 其他属性和子节点的定义
};
```

在这个示例中，my_device 节点具有 compatible 属性，其值为 "vendor,device"。这个值用于标识设备节点与特定厂商的特定设备兼容。

compatible 属性也可以具有多个匹配值，用于指定设备节点与多个设备或驱动程序的兼容性规则。这种情况下，compatible 属性的值是一个字符串列表，每个字符串表示一个匹配值。

以下是一个示例，展示了具有多个匹配值的 compatible 属性的用法：

```
my_device {
    compatible = ["vendor,device1", "vendor,device2"];
    // 其他属性和子节点的定义
};
```

在 这 个 示 例 中 ， my_device 节 点 具 有 compatible 属 性 ， 其 值 为 ["vendor,device1", "vendor,device2"]。这表示设备节点与厂商的 device1 和 device2 兼容。

通过使用 compatible 属性，设备树可以提供设备和驱动程序之间的匹配信息。当设备树被操作系统或设备管理软件解析时，会根据设备节点的 compatible 属性值来选择适合的驱动程序进行设备的初始化和配置。

#### aliases 节点

aliases 节点是一个特殊的节点，用于定义设备别名。该节点位于设备树的根部，并具有节点路径 /aliases。

aliases 节点是一个容器节点，包含一组属性，每个属性都代表一个设备别名。每个属性的名称是别名的标识符，而属性的值是被引用设备节点的路径或设备树中其他节点的路径。

以下是一个示例，演示了如何在设备树中使用 aliases 节点：

```
aliases {
    mmc0 = &sdmmc0;
    mmc1 = &sdmmc1;
    mmc2 = &sdhci;
    serial0 = "/simple@fe000000/seria1@11c500";
};
```

在给定的例子中，有四个别名的定义：

1. mmc0 别名与设备树中的 sdmmc0 节点相关联。通过使用别名 mmc0，其他设备节点 或客户端程序可以更方便地引用sdmmc0 节点，而不必直接使用其完整路径。

2. mmc1 别名与设备树中的 sdmmc1 节点相关联。通过使用别名 mmc1，其他设备节点 或客户端程序可以更方便地引用sdmmc1 节点，而不必直接使用其完整路径。

3. mmc2 别名与设备树中的 sdhci 节点相关联。通过使用别名 mmc2 ，其他设备节点或 客户端程序可以更方便地引用sdhci 节点，而不必直接使用其完整路径。

4. serial0 别名与设备树中的路径 /simple@fe000000/seria1@11c500 相关联。通过使用 别名 serial0，其他设备节点或客户端程序可以更方便地引用该路径，而不必记住整个路径字符 串。

在别名的定义中，& 符号用于引用设备树中的节点。别名的目的是提供可读性更高的名称， 使设备树更易于理解和维护。通过使用别名，可以简化设备节点之间的关联，并减少重复输入 设备节点的路径。

客户端程序可以使用别名属性名称来引用完整的设备路径或部分路径。当客户端程序将别 名字符串视为设备路径时，应检测并使用别名。这样，设备树的使用者可以更方便地引用设备 节点，而不必记住复杂的路径结构。

需要注意的是，aliases 节点中定义的别名只在设备树内部可见，不能在设备树之外引用。它们主要用于设备树的内部组织和引用，以提高可读性和可维护性。

#### chosen 节点

chosen 节点是设备树中的一个特殊节点，用于传递和存储系统引导和配置的相关信息。 它位于设备树的根部，并具有路径/chosen。

chosen 节点通常包含以下子节点和属性：

1. bootargs：用于存储引导内核时传递的命令行参数。它可以包含诸如内核参数、设备 树参数等信息。在引导过程中，操作系统或引导加载程序可以读取该属性来获取启动参数。

2. stdout-path：用于指定用于标准输出的设备路径。在引导过程中，操作系统可以使 用该属性来确定将控制台输出发送到哪个设备，例如串口或显示屏。

3. firmware-name：用于指定系统固件的名称。它可以用于标识所使用的引导加载程序 或固件的类型和版本。

4. linux,initrd-start 和 linux,initrd-end：这些属性用于指定 Linux 内核初始化 RAM 磁盘 （initrd）的起始地址和结束地址。这些信息在引导过程中被引导加载程序使用，以将 initrd 加 载到内存中供内核使用。

5. 其他自定义属性：chosen 节点还可以包含其他自定义属性，用于存储特定于系统引 导和配置的信息。这些属性的具体含义和用法取决于设备树的使用和上下文。

关于 chosen 节点的实际例子如下所示：

```
chosen {
		bootargs = " root=/dev/nfsrwnfsroot=192.168.1.1console=ttyS0, 115200"; 
};
```

在 这 个 示 例 中 ， chosen 节 点 具 有 一 个 属 性 bootargs ， 其 值 为 "root=/dev/nfs rw nfsroot=[192.168.1.1](192.168.1.1) console=ttyS0,115200"。

通过这些命令行参数，操作系统或引导加载程序可以配置内核在引导过程中正确地加载 NFS 根文件系统，并将控制台输出发送到指定的串口设备。

通过使用 chosen 节点，系统引导过程中的相关信息可以方便地传递给操作系统或引导加 载程序。这样，系统引导和配置的各个组件可以共享和访问这些信息，从而实现更灵活和可配 置的系统引导流程。chosen 节点提供了一种通用的机制，使得不同的设备树和引导系统可以 在传递信息方面保持一致性，并且可以根据具体需求扩展和自定义。

#### device_type 节点

在设备树中，device_type 节点是用于描述设备类型的节点。它通常作为设备节点的一个 属性存在。device_type 属性的值是一个字符串，用于标识设备的类型。

device_type 节点的存在有助于操作系统或其他软件识别和处理设备。它提供了设备的基 本分类信息，使得驱动程序、设备树解析器或其他系统组件能够根据设备的类型执行相应的操 作。

常见的设备类型包括但不限于：

1. cpu：表示中央处理器。

2. memory：表示内存设备。

3. display：表示显示设备，如液晶显示屏。

4. serial：表示串行通信设备，如串口。

5. ethernet：表示以太网设备。

6. usb：表示通用串行总线设备。
7. i2c：表示使用 I2C (Inter-Integrated Circuit)  总线通信的设备。
8. spi：表示使用 SPI (Serial Peripheral Interface) 总线通信的设备。
9. gpio：表示通用输入/输出设备。
10. pwm：表示脉宽调制设备。

这些只是一些常见的设备类型示例，实际上，设备类型可以根据具体的硬件和设备树的使 用情况进行自定义和扩展。根据设备类型，操作系统或其他软件可以加载适当的驱动程序、配 置设备资源、建立设备之间的连接等。

####  自定义属性

设备树中的自定义属性是用户根据特定需求添加的属性。这些属性可以用于提供额外的信 息、配置参数或元数据，以满足设备或系统的特定要求。

在设备树中添加自定义属性时，可以在设备节点或其他适当的节点下定义新的属性。自定 义属性可以是整数、字符串、布尔值或其他数据类型。它们的命名应遵循设备树的命名约定， 并且应该与已有的属性名称避免冲突。

例如可以在设备树中自定义一个管脚标号的属性 pinnum，添加好的设备树源码如下所示：

```
my_device {
    compatible = "my_device";
    pinnum = <0 1 2 3 4>;
};
```

在上述示例中，my_device 是一个自定义设备节点，并添加了一个自定义属性 pinnum。该属性的值 <0 1 2 3 4> 是一个整数数组，表示管脚的标号（PIN number）。

通过这样定义 pinnum 属性，您可以在设备树中为特定设备指定管教标号，以便操作系统、驱动程序或其他软件组件使用。这可以用于在设备初始化或配置过程中对特定管教进行操作或控制。

### dtb 展开成 device_node

#### dtb 展开流程

![image-20240711205423647](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240711205423647.png)

接下来将会根据上图对 deb 的展开流程进行详细的讲解：

**（1）设备树源文件编写**：根据之前的章节中讲解的设备树的基本语法和相关知识编写符 合规范的设备树。

**（2）设备树编译：**设备树源文件经过设备树编译器（dtc）进行编译，生成设备树二进制 文件（ .dtb）。设备树编译器会检查源文件的语法和语义，并将其转换为二进制格式，以便内 核能够解析和使用。

**（3）boot.img 镜像生成**：boot.img 是一个包含内核镜像、设备树二进制文件和其他一些 资源文件的镜像文件（ 目前只是适用于瑞芯微的soc 上，其他厂商的 soc 需要具体问题具体分 析）。在生成 boot.img 时，通常会将内核镜像、设备树二进制文件和其他一些资源文件打包 在一起。这个过程可以使用特定的工具或脚本完成。

**（4）U-Boot** **加载：**U-Boot（Universal Bootloader）是一种常用的开源引导加载程序，用于 引导嵌入式系统。在系统启动过程中，U-Boot 会将 boot.img 中的内核和设备树的二进制文件 加载到系统内存的特定地址。

**（5）内核初始化：**U-Boot 将内核和设备树的二进制文件加载到系统内存的特定地址后， 控制权会转交给内核。在内核初始化的过程中，会解析设备树二进制文件，将其展开为内核可 以识别的数据结构，以便内核能够正确地初始化和管理硬件资源。

**（6）** **设备树展开：** 设备树展开是指将设备树二进制文件解析成内核中的设备节点 （device_node）的过程。内核会读取设备树二进制文件的内容，并根据设备树的描述信息， 构建设备树数据结构，例如设备节点、中断控制器、寄存器、时钟等。这些设备树数据结构将 在内核运行时用于管理和配置硬件资源。

而本章节要讲解的重点就在上面的第 6 步“设备树的展开 ”，最终设备树二进制文件会被 解析成 device_node ，device_node 结构体定义在内核源码的“/include/linux/of.h ”文件中，具 体内容如下所示：

```c
struct device_node {
    const char*name;                                   //  设备节点的名称
    const char *type;                                     //  设备节点的类型
    phandlephandle;                                         //  设备节点的句柄
    constchar*full_name;                        //  设备节点的完整名称
    struct fwnode_handle fwnode;           //  设备节点的固件节点句柄

    struct property*properties;      //设备节点的属性列表 
  	struct property*deadprops;    //已删除的属性列表  					
  	struct device _node * parent;         //父设备节点指针
    structdevice_node*child;        //子设备节点指针  
  	struct device _node*sibling;    //  兄弟设备节点指针 
#ifdefined(CONFIG_OF_KOBJ)
    struct kobject kobj;        //内核对象（用于 sysfs） 
#endif
    unsignedlong_flags;                    //  设备节点的标志位
    void*data;                             //  与设备节点相关的数据指针
#ifdefined(CONFIG_SPARC)
    const char *path_component_name;//  设备节点的路径组件名称
    unsignedintunique_id;                      //  设备节点的唯一标识
    struct of_irq_controller * irq_trans;//  设备节点的中断控制器
#endif
};
```

然后对该结构体的重要参数进行讲解：

（1）name：name 字段表示设备节点的名称。设备节点的名称是在设备树中唯一标识该节 点的字符串。它通常用于在设备树中引用设备节点。

（2）type：type 字段表示设备节点的类型。设备节点的类型提供了关于设备节点功能和所属设备类别的信息。它可以用于识别设备节点的用途和特性。

（3）properties：properties 字段是指向设备节点属性列表的指针。设备节点的属性包含了 与设备节点相关联的配置和参数信息。属性以键值对的形式存在，可以提供设备的特定属性、 寄存器地址、中断信息等。property 字段同样定义在内核源码的“/include/linux/of.h ”文件中， 具体内容如下所示：

```c
struct property {
    char*name;                                            //  属性的名称
    int length;                                              //  属性值的长度（字节数）
    void*value;                                          //  属性值的指针
    struct property*next;                    //  下一个属性节点指针
#if defined(CONFIG_OF_DYNAMIC) ||defined(CONFIG_SPARC)
    unsignedlong_flags;                      //  属性的标志位
#endif
#if defined(CONFIG_OF_PROMTREE)
    unsignedint unique_id;                  //  属性的唯一标识
#endif
#if defined(CONFIG_OF_KOBJ)
    structbin_attributeattr;           //   内核对象二进制属性
#endif
};
```

（4）parent（父设备节点）： parent 字段指向当前设备节点的父设备节点。设备树中的设 备节点按照层次结构组织，每个设备节点都有一个直接上级（父设备节点）。通过 parent字 段，可以在设备树中向上遍历设备节点的父节点。

（5）child（子设备节点）： child 字段指向当前设备节点的第一个子设备节点。一个设备 节点可以拥有多个子设备节点，在设备树中表示它们是当前设备节点的直接下级。通过 child 字段获取到第一个子设备节点后，可以使用sibling 字段遍历它们的兄弟节点，从而遍历所有的 子节点。

（6）sibling（兄弟设备节点）：sibling 字段指向当前设备节点在设备树中的下一个兄弟设 备节点。兄弟设备节点是指在设备树中处于同一级别的设备节点，它们共享相同的父设备节点。 通过 sibling 字段，可以在同级设备节点之间进行遍历，获取它们的信息或进行操作。

至此，关于 device_node 的结构体讲解就完成了，虽然我们现在知道了，dtb 文件最终会展 开成 device_node 这一可以让内核识别的格式，那更具体的实现流程是怎样的呢，让我们进入 下一小节的学习吧。

### device_node 转换成 platform_device

#### 转换规则

在之前学习的平台总线模型中，device 部分是用 platform_device 结构体来描述硬件资源 的，所以内核最终会将内核认识的 device_node 树转换 platform_ device ，但是并不是所有的 device_node 都会被转换成 platform_ device，只有满足要求的才会转换成 platform_ device,转换 成 platform_device 的节点可以在/sys/bus/platform/devices 下查看，那 device_node 节点要满足 什么要求才会被转换成 platform_device 呢?

**根据规则 1 ，首先遍历根节点下包含 compatible 属性的子节点，对于每个子节点，创建 一个对应的 platform_device。**

**根据规则 2，遍历包含 compatible 属性为 "simple-bus" 、"simple-mfd" 或 "isa"  的节点以 及它们的子节点。如果子节点包含 compatible 属性值则会创建一个对应的 platform_device。父节点为"simple-bus" 、"simple-mfd" 或 "isa"，子节点才会被转成平台设备（根节点可以认为是一个simple-bus）**

**根据规则 3，检查节点的 compatible 属性是否包含 "arm" 或 "primecell"。如果是，则不 将该节点转换为 platform_device ，而是将其识别为 AMBA 设备。**

接下来将通过几个设备树示例对上述规则进行实践。

#### **举例** **1**：

```
/dts-v1/;

/ {
    model="Thisismydevicetree!";
    #address-cells= <1>; 
    #size-cells= <1>;

    chosen {
    		bootargs = " root=/dev/nfsrwnfsroot=192.168.1.1console=ttyS0,115200";
    };

    cpu1: cpu@1 {
        device_type="cpu";
        compatible="arm,cortex-a35","arm,armv8"; 
        reg=<0x00x1>;
    };

    aliases {
    		led1 = "/ gpio@22020101"; 
    };

    node1 {
    		#address-cells= <1>; 
    		#size-cells= <1>;
        gpio@22020102 {
            reg=<0x202201020x40>;
        }; 
    };
    node2 {
        node1-child {
        	pinnum=<01234>;
    	}; 
    };
    gpio@22020101 {
        compatible="led";
        reg=<0x202201010x40>; status = "okay";
    };
};
```

在上面的设备树中，总共有 chosen、cpu1: cpu@1、aliases、node1、node2、gpio@22020101 这六个节点，其中前五个节点都没有 compatible 属性，所以并不会被转换为 platform_device， 而最后一个 gpio@22020101 节点符合规则一，在根节点下，且有 compatible 属性，所以最后 会转换为 platform_device。

#### **举例** **2**：

```
/dts-v1/;
/ {
		model = "This is my devicetree!";
		#address-cells = <1>;
		#size-cells = <1>;
		chosen {
				bootargs = "root=/dev/nfs rw nfsroot=192.168.1.1 console=ttyS0, 115200";
		};
		cpu1: cpu@1 {
				device_type = "cpu";
				compatible = "arm,cortex-a35", "arm,armv8";
				reg = <0x0 0x1>;
		};
		aliases {
				led1 = "/gpio@22020101";
		};
		node1 {
				#address-cells = <1>;
				#size-cells = <1>;
				compatible = "simple-bus";
				gpio@22020102 {
						reg = <0x20220102 0x40>;
				};
		};
		node2 {
				node1-child {
						pinnum = <01234>;
				};
		};
    	gpio@22020101 {
            compatible = "led";
            reg = <0x20220101 0x40>;
            status = "okay";
		};
};
```

相较于示例 1 的设备树，这里在 node1 节点中添加了 compatible 属性，但是这个 compatible 属性值为 simple-bus，我们需要继续看他的子节点，子节点 gpio@22020102 并没有 compatible 属性值，所以这里的 node1 节点不会被转换。

#### **举例** **3**：

```
/dts-v1/;
/ {
    model = "This is my devicetree!";
    #address-cells = <1>;
    #size-cells = <1>;
    chosen {
    		bootargs = "root=/dev/nfs rw nfsroot=192.168.1.1 console=ttyS0, 115200";
    };
    cpu1: cpu@1 {
        device_type = "cpu";
        compatible = "arm,cortex-a35", "arm,armv8";
        reg = <0x0 0x1>;
    };
    aliases {
    		led1 = "/gpio@22020101";
    };
    node1 {
        #address-cells = <1>;
        #size-cells = <1>;
        compatible = "simple-bus";
        gpio@22020102 {
            compatible = "gpio";
            reg = <0x20220102 0x40>;
        };
    };
    node2 {
        node1-child {
        		pinnum = <01234>;
        };
    };
    gpio@22020101 {
        compatible = "led";
        reg = <0x20220101 0x40>;
        status = "okay";
    };
};
```

相较于示例 2 的设备树，这里在 node1 节点的子节点 gpio@22020102 中添加了 compatible 属性，node1 节点的 compatible 属性值为 simple-bus，然后需要继续看他的子节点，子节点 gpio@22020102 的 compatible 属性值为 gpio，所以这里的 gpio@22020102 节点会被转换成platform_device。

#### **举例** **4**：

```
/dts-v1/;
/ {
    model = "This is my devicetree!";
    #address-cells = <1>;
    #size-cells = <1>;
    chosen {
    		bootargs = "root=/dev/nfs rw nfsroot=192.168.1.1 console=ttyS0,115200";
    };
    cpul: cpu@1 {
        device_type = "cpu";
        compatible = "arm,cortex-a35", "arm,armv8";
        reg = <0x0 0x1>;
        amba {
            compatible = "simple-bus";
            #address-cells = <2>;
            #size-cells = <2>;
            ranges;
            dmac_peri: dma-controller@ff250000 {
                compatible = "arm,p1330", "arm,primecell";
                reg = <0x0 0xff250000 0x0 0x4000>;
                interrupts = <GIC_SPI 2 IRQ_TYPE_LEVEL_HIGH>, <GIC_SPI 3 IRQ_TYPE_LEVEL_HIGH>;
                #dma-cells = <1>;
                arm,pl330-broken-no-flushp;
                arm,p1330-periph-burst;
                clocks = <&cru ACLK DMAC_PERI>;
                clock-names = "apb_pclk";
            };
            dmac_bus: dma-controller@ff600000 {
                compatible = "arm,p1330", "arm,primecell";
                reg = <0x0 0xff600000 0x0 0x4000>;
                interrupts = <GIC_SPI 0 IRQ_TYPE_LEVEL_HIGH>, <GIC_SPI 1 IRQ_TYPE_LEVEL_HIGH>;
                #dma-cells = <1>;
                arm,pl330-broken-no-flushp;
                arm,pl330-periph-burst;
                clocks = <&cru ACLK_DMAC_BUS>;
                clock-names = "apb_pclk";
            };
        };
    };
};
```

amba 节点的 compatible 值为 simple-bus，不会被转换为 platform_device，而是作为父节点用于组织其他设备，所以需要来查看他的子节点。

dmac_peri: dma-controller@ff250000 节点: 该节点的 compatible 属性包含 "arm,p1330" 和 "arm,primecell"，根据规则 3，该节点不会被转换为 platform_device，而是被识别为 AMBA设备。

dmac_bus: dma-controller@ff600000 节点: 该节点的 compatible 属性包含 "arm,p1330" 和"arm,primecell"，根据规则 3，该节点不会被转换为 platform_device，而是被识别为 AMBA 设备。

### 设备树下 platform_device 和 platform_driver 匹配

#### of_match_table

在前面平台总线相关章节的学习中，了解到只有 platform_device 结构体中的 name 属性与platform_driver 结构体中嵌套的 driver 结构体 name 属性或者 id_table 相同才能加载 probe 初始化函数。

而为了使设备树能够与驱动程序进行匹配，需要在 platform_driver 驱动程序中添加 driver结构体的 of_match_table 属性。这个属性是一个指向 const struct of_device_id 结构的指针，用于描述设备树节点和驱动程序之间的匹配规则。of_device_id 结构体定义在内核源码的“/include/linux/mod_devicetable.h”文件中，具体内容如下所示：

```c
struct of_device_id {
    char name[32];
    char type[32];
    char compatible[128];
    const void *data;
};
```

struct of_device_id 结构体通常作为一个数组在驱动程序中定义，用于描述设备树节点和驱动程序之间的匹配规则。数组的最后一个元素必须是一个空的结构体，以标记数组的结束。

以下是一个示例，展示了如何在驱动程序中使用 struct of_device_id 进行设备树匹配：

```c
static const struct of_device_id my_driver_match[] = {
  { 
    .compatible = "vendor,device-1" 
  }, 
  { 
    .compatible = "vendor,device-2" 
  }, 
  { 
  }, 
};
```

在上述示例中，my_driver_match 是一个 struct of_device_id 结构体数组。每个数组元素都包含了一个 compatible 字段，用于指定设备树节点的兼容性字符串。驱动程序将根据这些兼容性字符串与设备树中的节点进行匹配。

#### 实验

本次实验的要求使用设备树描述下面的内存资源：

**内存资源：**

起始地址：0xFDD60000

结束地址：0xFDD60004

然后编写对应的 platform_driver 驱动程序，要求跟上述内存资源所创建的节点进行匹配，从而验证 上一小节讲解的 of_match_table 属性。

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
// 平台设备的初始化函数
static int my_platform_probe(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_probe: Probing platform device\n");

    // 添加设备特定的操作
    // ...

    return 0;
}

// 平台设备的移除函数
static int my_platform_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");

    // 清理设备特定的操作
    // ...

    return 0;
}


const struct of_device_id of_match_table_id[]  = {
	{.compatible="my devicetree"},
    {},
};

// 定义平台驱动结构体
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
		.of_match_table =  of_match_table_id,
    },
};

// 模块初始化函数
static int __init my_platform_driver_init(void)
{
    int ret;

    // 注册平台驱动
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        printk(KERN_ERR "Failed to register platform driver\n");
        return ret;
    }

    printk(KERN_INFO "my_platform_driver: Platform driver initialized\n");

    return 0;
}

// 模块退出函数
static void __exit my_platform_driver_exit(void)
{
    // 注销平台驱动
    platform_driver_unregister(&my_platform_driver);

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

dts

```
// SPDX-License-Identifier: (GPL-2.0+ OR MIT)
/*
 * Copyright (c) 2020 Rockchip Electronics Co., Ltd.
 *
 */

#include "rk3568-evb1-ddr4-v10.dtsi"
#include "rk3568-linux.dtsi"

#include <dt-bindings/display/rockchip_vop.h>
/{
    topeet{
        #address-cells = <1>;
        #size-cells = <1>;
        compatible = "simple-bus";

        myLed{
            compatible = "my devicetree";
            reg = <0xFDD60000 0x00000004>;
        };
    };
};

&vp0 {
	cursor-win-id = <ROCKCHIP_VOP2_CLUSTER0>;
};

&vp1 {
	cursor-win-id = <ROCKCHIP_VOP2_CLUSTER1>;
};


&uart7 { 
		status ="okay"; 
		pinctrl-name = "default"; 
		pinctrl-0 = <&uart7m1_xfer>; 
};
&uart4 { 
	    status = "okay"; 
		pinctrl-names = "default"; 
		pinctrl-0 = <&uart4m1_xfer>; 
};
&uart9 { 
	    status = "okay"; 
		pinctrl-names = "default"; 
		pinctrl-0 = <&uart9m1_xfer>; 
};
&can1 {
        status = "okay";
		compatible = "rockchip,canfd-1.0";
        assigned-clocks = <&cru CLK_CAN1>;
        assigned-clock-rates = <150000000>;  //If can bitrate lower than 3M,the clock-rates should set 100M,else set 200M.
        pinctrl-names = "default";
        pinctrl-0 = <&can1m1_pins>;
};
```

### of 操作-获取节点

在 Linux 内核源码中提供了一系列的 of 操作函数来帮助我们获取到设备树中编写的属性，在内核中以 device_node 结构体来对设备树进行描述，所以 of 操作函数实际上就是获取device_node 结构体，所以接下来我们学习的 of 操作函数的返回值都是 device_node 结构体。

#### of_find_node_by_name 函数

of_find_node_by_name 是 Linux 内核中用于通过节点名称查找设备树节点的函数。下面是对 of_find_node_by_name 函数的详细介绍：

> **函数原型：**
>
> struct device_node *of_find_node_by_name(struct device_node *from, const char *nam
>
> e);
>
> **头文件：**
>
> \#include <linux/of.h>
>
> **函数作用：**
>
> 该函数通过指定的节点名称在设备树中进行查找，返回匹配的节点的 struct device_nod
>
> e 指针。
>
> **参数含义：**
>
> from：指定起始节点，表示从哪个节点开始查找。如果 from 参数为 NULL，则从设备树的根节点开始查找。
>
> name：要查找的节点名称。
>
> **返回值：**
>
> 如果找到匹配的节点，则返回对应的 struct device_node 指针。
>
> 如果未找到匹配的节点，则返回 NULL。

#### of_find_node_by_path 函数

of_find_node_by_path 是 Linux 内核中用于通过节点路径查找设备树节点的函数。下面是对 of_find_node_by_path 函数的详细介绍：

> **函数原型：**
>
> struct device_node *of_find_node_by_path(const char *path);
>
> **头文件：**
>
> \#include <linux/of.h>
>
> **函数作用：**
>
> 该函数根据节点路径在设备树中进行查找，返回匹配的节点的 struct device_node 指针。
>
> **参数含义：**
>
> path：节点的路径，以斜杠分隔的字符串表示。路径格式为设备树节点的绝对路径，例如 /topeet/myLed。
>
> **返回值：**
>
> 如果找到匹配的节点，则返回对应的 struct device_node 指针。
>
> 如果未找到匹配的节点，则返回 NULL。

of_find_node_by_path 函数通过节点路径在设备树中进行查找。路径是设备树节点从根节点到目标节点的完整路径。可以通过指定正确的路径来准确地访问设备树中的特定节点。

使用 of_find_node_by_path 函数时，可以直接传递节点的完整路径作为 path 参数，函数会在设备树中查找匹配的节点。这对于已知节点路径的情况非常有用。

#### of_get_parent 函数

在 Linux 内核中，of_get_parent 函数用于获取设备树节点的父节点。下面是对 of_get_parent函数的详细介绍：

> **函数原型：**
>
> struct device_node *of_get_parent(const struct device_node *node);
>
> **头文件：**
>
> \#include <linux/of.h>
>
> **函数作用：**
>
> 该函数接收一个指向设备树节点的指针 node，并返回该节点的父节点的指针。
>
> **参数含义：**
>
> node：要获取父节点的设备树节点指针。
>
> **返回值：**
>
> 如果找到匹配的节点，则返回对应的 struct device_node 指针。
>
> 如果未找到匹配的节点，则返回 NULL。

使用 of_get_parent 函数时，可以将特定的设备树节点作为参数传递给函数，然后它将返回该节点的父节点。这对于在设备树中导航和访问节点之间的层次关系非常有用。

父节点在设备树中表示了节点之间的层次结构关系。通过获取父节点，你可以访问上一级节点的属性和配置信息，从而更好地理解设备树中的节点之间的关系。

#### of_get_next_child 函数

在 Linux 内核中，of_get_next_child 函数用于获取设备树节点的下一个子节点。下面是对of_get_next_child 函数的详细介绍：

> **函数原型：**
>
> struct device_node *of_get_next_child(const struct device_node *node, struct device_nod
>
> e *prev);
>
> **头文件：**
>
> \#include <linux/of.h>
>
> **函数作用：**
>
> 该函数接收两个参数：node 是当前节点，prev 是上一个子节点。它返回下一个子节点的指针。
>
> **参数含义：**
>
> node：当前节点，用于指定要获取子节点的起始节点。
>
> prev：上一个子节点，用于指定从哪个子节点开始获取下一个子节点。如果为 NULL，则从起始节点的第一个子节点开始。
>
> **返回值：**
>
> 如果找到匹配的节点，则返回对应的 struct device_node 指针。
>
> 如果未找到匹配的节点，则返回 NULL。

使用 of_get_next_child 函数时，可以传递当前节点以及上一个子节点作为参数。函数将从上一个子节点的下一个节点开始，查找并返回下一个子节点。

设备树中的子节点表示了节点之间的层次关系。通过获取子节点，你可以遍历和访问当前节点的所有子节点，以便进一步处理它们的属性和配置信息。

#### of_ find_ compatible_ node 函数

当设备树中存在多个设备节点，需要根据设备的兼容性字符串进行匹配时，可以使用 of_find_compatible_node 函数。该函数用于在设备树中查找与指定兼容性字符串匹配的节点。

> **函数原型：**
>
> struct device_node *of_find_compatible_node(struct device_node *from, const char *type, const char *compatible);
>
> **头文件：**
>
> \#include <linux/of.h>
>
> **函数作用：**
>
> 在设备树中查找与指定兼容性字符串匹配的节点。
>
> **参数含义：**
>
> from：指定起始节点，表示从哪个节点开始查找。如果 from 参数为 NULL，则从设备树的根节点开始查找。
>
> type：要匹配的设备类型字符串，通常是 compatible 属性中的一部分。
>
> compatible：要匹配的兼容性字符串，通常是设备树节点的 compatible 属性中的值。
>
> **返回值：**
>
> 如果找到匹配的节点，则返回对应的 struct device_node 指针。
>
> 如果未找到匹配的节点，则返回 NULL。

使用 of_find_compatible_node 函数时，可以指定起始节点和需要匹配的设备类型字符串以及兼容性字符串。函数会从起始节点开始遍历设备树，查找与指定兼容性字符串匹配的节点，并返回匹配节点的指针。

#### of_ find_matching_node_ and_ match 函数

在 Linux 内核中，of_ find matching node_ and_ match 函数用于根据给定的 of_device_id 匹配表在设备树中查找匹配的节点。

> **函数原型：**
>
> struct device_node *of_find_matching_node_and_match(struct device_node *from,const struct of_device_id *matches, const struct of_device_id **match);
>
> **头文件：**
>
> \#include <linux/of.h>
>
> **函数作用：**
>
> 根据给定的 of_device_id 匹配表在设备树中查找匹配的节点。
>
> **参数含义：**
>
> from：表示从哪个节点开始搜索。通常将上一次调用该函数返回的节点作为参数传递给 from，以便从上一次的下一个节点开始搜索。如果要从设备树的根节点开始搜索，可以将 from参数设置为 NULL。
>
> matches：指向一个 of_device_id 类型的匹配表，该表包含要搜索的匹配项。
>
> match：用于输出匹配到的 of_device_id 条目的指针。
>
> **返回值：**
>
> 如果找到匹配的节点，则返回对应的 struct device_node 指针。
>
> 如果未找到匹配的节点，则返回 NULL

of_find_matching_node_and_match 函数在设备树中遍历节点，对每个节点使用__of_match_node 函数进行匹配。如果找到匹配的节点，将返回该节点的指针，并将 match 指针更新为匹配到的 of_device_id 条目，函数会自动增加匹配节点的引用计数。

以下是使用 of_find_matching_node_and_match 函数的示例代码：

```c
#include <linux/of.h>
static const struct of_device_id my_match_table[] = {
		{ .compatible = "vendor,device" }, 
  	{ /* sentinel */ }
};
const struct of_device_id *match;
struct device_node *np;
// 从根节点开始查找匹配的节点
np = of_find_matching_node_and_match(NULL, my_match_table, &match);
```

在上述示例中，我们定义了一个 of_device_id 匹配表 my_match_table，其中包含了一个兼容性字符串为"vendor,device"的匹配项。然后，我们使用 of_find_matching_node_and_match 函数从根节点开始查找匹配的节点。

#### 实验代码

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/of.h>

struct device_node *mydevice_node;      
const struct of_device_id *mynode_match;
struct of_device_id mynode_of_match[] = {
	{.compatible="my devicetree"},
	{},
};

// 平台设备的初始化函数
static int my_platform_probe(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_probe: Probing platform device\n");

    // 通过节点名称查找设备树节点
    mydevice_node = of_find_node_by_name(NULL, "myLed");
	printk("mydevice node is %s\n", mydevice_node->name);
    
	// 通过节点路径查找设备树节点
    mydevice_node = of_find_node_by_path("/topeet/myLed");
    printk("mydevice node is %s\n", mydevice_node->name);
        
    // 获取父节点
    mydevice_node = of_get_parent(mydevice_node);
    printk("myled's parent node is %s\n", mydevice_node->name);
            
    // 获取子节点
    mydevice_node = of_get_next_child(mydevice_node, NULL);
    printk("myled's sibling node is %s\n", mydevice_node->name);

	// 使用compatible值查找节点
	mydevice_node=of_find_compatible_node(NULL ,NULL, "my devicetree");
	printk("mydevice node is %s\n" , mydevice_node->name);
	
	//根据给定的of_device_id匹配表在设备树中查找匹配的节点
	mydevice_node=of_find_matching_node_and_match(NULL , mynode_of_match, &mynode_match);
	printk("mydevice node is %s\n" ,mydevice_node->name);
	return 0;
}

// 平台设备的移除函数
static int my_platform_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");

    // 清理设备特定的操作
    // ...

    return 0;
}


const struct of_device_id of_match_table_id[]  = {
	{.compatible="my devicetree"},
};

// 定义平台驱动结构体
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
		.of_match_table =  of_match_table_id,
    },
};

// 模块初始化函数
static int __init my_platform_driver_init(void)
{
    int ret;

    // 注册平台驱动
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        printk(KERN_ERR "Failed to register platform driver\n");
        return ret;
    }

    printk(KERN_INFO "my_platform_driver: Platform driver initialized\n");

    return 0;
}

// 模块退出函数
static void __exit my_platform_driver_exit(void)
{
    // 注销平台驱动
    platform_driver_unregister(&my_platform_driver);
    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

### 获取属性

#### of_find_property 函数

of_find_property 函数用于在设备树中查找节点 下具有指定名称的属性。 如果找到了该属性， 可以通过返回的属性结构体指针进行进一步的操作， 比如获取属性值、 属性长度等。

> 函数原型:
> struct property *of_find_property(const struct device_node *np, const char *name, int * lenp)
> 头文件：
> #include <linux/of.h>
> 函数作用:
> 该函数用于在节点 np 下查找指定名称 name 的属性。
> 函数参数:
> np: 要查找的节点。
> name: 要查找的属性的属性名。
> lenp: 一个指向整数的指针， 用于接收属性值的字节数。
> 返回值:
> 如果成功找到了指定名称的属性， 则返回对应的属性结构体指针 struct property *； 如果未找到， 则返回 NULL。

#### of_property_count_elems_of_size 函数

该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性中元素的数量。 调用该函数可以用于获取设备树属性中某个属性的元素数量， 比如一个字符串列表的元素数量或一个整数数组的元素数量等。

>函数原型:
>int of_property_count_elems_of_size(const struct device_node *np, const char *propname, int elem_size)
>头文件：
>#include <linux/of.h>
>函数作用:
>
>该函数用于获取属性中指定元素的数量。
>函数参数和返回值:
>np: 设备节点。
>propname: 需要获取元素数量的属性名。
>elem_size: 单个元素的尺寸。
>返回值:
>如果成功获取了指定属性中元素的数量， 则返回该数量； 如果未找到属性或属性中没有元素， 则返回 0

#### of_property_read_u32_index 函数

该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性在给定索引位置处的u32 类型的数据值。
这个函数通常用于从设备树属性中读取特定索引位置的整数值。 通过指定属性名和索引，可以获取属性中指定位置的具体数值。

> 函数原型:
> int of_property_read_u32_index(const struct device_node *np, const char *propname, u32 index, u32 *out_value)
> 头文件：
> #include <linux/of.h>
> 函数作用:
> 该函数用于从指定属性中获取指定索引位置的 u32 类型的数据值。
> 函数参数:
> np: 设备节点。
> propname: 要读取的属性名。
> index: 要读取的属性值在属性中的索引， 索引从 0 开始。
> out_value: 用于存储读取到的值的指针。
> 返回值:
> 如果成功读取到了指定属性指定索引位置的 u32 类型的数据值， 则返回 0； 如果未找到属性或读取失败， 则返回相应的错误码。

#### of_property_read_u64_index 函数

该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性在给定索引位置处的u64 类型的数据值。
这个函数通常用于从设备树属性中读取特定索引位置的 64 位整数值。 通过指定属性名和索引， 可以获取属性中指定位置的具体数值。

> 函数原型:
> static inline int of_property_read_u64_index(const struct device_node *np, const char *propname, u32 index, u64 *out_value)
> 头文件：
> #include <linux/of.h>
> 函数作用:
> 该函数用于从指定属性中获取指定索引位置的 u64 类型的数据值。
> 函数参数和返回值:
> np: 设备节点。
> propname: 要读取的属性名。
> index: 要读取的属性值在属性中的索引， 索引从 0 开始。
> out_value: 用于存储读取到的值的指针。
> 返回值:
> 如果成功读取到了指定属性指定索引位置的 u64 类型的数据值， 则返回 0； 如果未找到属性或读取失败， 则返回相应的错误码。

#### of_property_read_variable_u32_array 函数

该函数用于从设备树中读取指定属性名的变长数组。 通过提供设备节点、 属性名和输出数组的指针， 可以将设备树中的数组数据读取到指定的内存区域中。 同时， 还需要指定数组的最小大小和最大大小， 以确保读取到的数组符合预期的大小范围。

> 函数原型：
> int of_property_read_variable_u32_array(const struct device_node *np, const char *propn
> ame, u32 *out_values, size_t SZ_min, size_t SZ_max)
> 函数作用:
> 从指定属性中读取变长的 u32 数组。
> 函数参数和返回值:
> np: 设备节点。
> propname: 要读取的属性名。
> out_values: 用于存储读取到的 u8 数组的指针。
> SZ_min: 数组的最小大小。
> SZ_max: 数组的最大大小。
> 返回值：
> 如果成功读取到了指定属性的 u8 数组， 则返回数组的大小。 如果未找到属性或读取失败，则返回相应的错误码。

上面介绍的函数用于从指定属性中读取变长的 u32 数组， 下面是另外三个读取其他数组大小的函数：
这里给出了四个函数， 用于从设备树中读取数组类型的属性值：
从指定属性中读取变长的 u8 数组：

> nt of_property_read_variable_u8_array(const struct device_node *np, const char *propname, u8 *out_values,
> size_t SZ_min, size_t SZ_max)

从指定属性中读取变长的 u16 数组：

> int of_property_read_variable_u16_array(const struct device_node *np, const char *propname, u16* out_values, size_t SZ_min, size_t SZ_max)

从指定属性中读取变长的 u64 数组：

> int of_property_read_variable_u64_array(const struct device_node *np, const char *propname, u64 *out_values, size_t SZ_min, size_t SZ_max)

#### of_property_read_string 函数

该函数在设备树中的设备节点下查找指定名称的属性， 并获取该属性的字符串值， 最后返回读取到的字符串的指针， 通常用于从设备树属性中读取字符串值。 通过指定属性名， 可以获取属性中的字符串数据。

> 函数原型:
> static inline int of_property_read_string(const struct device_node *np, const char *propname, const char **out_string)
> 头文件：
> #include <linux/of.h>
> 函数作用:
> 该函数用于从指定属性中读取字符串。
> 函数参数:
> np: 设备节点。
> propname: 要读取的属性名。
> out_string: 用于存储读取到的字符串的指针。
> 返回值:
> 如果成功读取到了指定属性的字符串， 则返回 0； 如果未找到属性或读取失败， 则返回相应的错误码。

#### 实验代码

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/of.h>

//用于存储找到的节点的信息
const struct of_device_id * my_node_id;

int len;
int pin;
const char * str;

struct of_device_id mynode_of_match[] = {
	{.compatible="woniu_dev;woniu_led"},
	{},
};

//probe函数会在设备和驱动匹配成功时执行
static int my_probe(struct platform_device * pdev){
	//使用of函数获取设备树的中的节点
	//of_find_node_by_name通过节点名称查找节点
	struct device_node * node = of_find_node_by_name(NULL,"woniu-dev");
	if (node == NULL)
	{
		printk("can't find node\n");
        return -1;
	}
	printk("node name: %s\n", node->name);
	//of_find_node_by_path通过节点路径来查找节点
	node = of_find_node_by_path("/woniu-devs/woniu-dev@00001234");
    if (node == NULL)
    {
        printk("can't find node222\n");
        return -1;
    }
    printk("node name: %s\n", node->name);
	//of_get_parent获取某个节点的父节点
	struct device_node * parent = of_get_parent(node);
    if (parent == NULL)
    {
        printk("can't get parent\n");
        return -1;
    }
    printk("parent node name: %s\n", parent->name);
	//of_find_matching_node_and_match通过of_device_id一次查找多个节点
	struct device_node * nodes = of_find_matching_node_and_match(NULL, mynode_of_match, &my_node_id);
	if (nodes == NULL)
    {
        printk("can't find nodes by of_device_id\n");
        return -1;
    }
	printk("nodes name: %s\n", nodes->name);
	//=======获取属性==========
	struct property *prop = of_find_property(node, "pin", &len);
	if(prop== NULL){
        printk("get property failed\n");
		return -1;
    }
	/**
	 * 大端序：把低位存在高地址，符合人类的阅读习惯
	 * 小端序：把低位存在低址址，符合计算机的处理方式
	 */
	//把小端序转为大端序
	int value =be32_to_cpu(*(int *)prop->value);
	printk("pin......: %d, len:%d\n", value,len);
	//of_property_read_u32_index 用于获取u32类型的属性
	of_property_read_u32_index(node, "pin", 0, &pin);
	printk("pin2: %d \n", pin);
	//of_property_read_string 用于获取char * 类型的属性
    of_property_read_string(node, "compatible", &str);
    printk("compatible: %s \n", str);

    return 0;
}

//remove函数会在卸载驱动或设备时执行
static int my_remove(struct platform_device * pdev){
	printk("remove...\n");
	return 0;
}

struct of_device_id my_device_id[] = {
	{
		.compatible = "woniu_dev;woniu_led",
	},
	{},
};

struct platform_driver my_driver = {
	.probe = my_probe,
    .remove = my_remove,
    .driver = {
		.name = "woniu_dev",
		.owner = THIS_MODULE,
		.of_match_table = my_device_id,
	},
};

static int __init woniu_init(void){
	int ret;
	ret = platform_driver_register(&my_driver);
	if(ret!=0){
        printk("platform_driver_register failed\n");
        return ret;
    }
    return 0;
}


static void __exit woniu_exit(void){
	printk("woniu is not nb>_<\n");
	platform_driver_unregister(&my_driver);
}

//注册驱动的入口函数
module_init(woniu_init);
//注册驱动的出口函数
module_exit(woniu_exit);
//同意开源协议
MODULE_LICENSE("GPL v2");
//声明作者
MODULE_AUTHOR("woniu");
```

### 地址映射

在上个章节中讲解了使用 of 操作函数来获取设备树的属性，由于设备树在系统启动的时候都会转化为 platform 设备，那我们能不能直接在驱动中使用在 53.1 小节中讲解的platform_get_resource 函数直接获取 platform_device 资源呢？

```c
static int my_platform_probe(struct platform_device *pdev)
{
  printk(KERN_INFO "my_platform_probe: Probing platform device\n");
  // 获取平台设备的资源
  myresources = platform_get_resource(pdev, IORESOURCE_MEM, 0);
  if (myresources == NULL) {
    // 如果获取资源失败，打印 value_compatible 的值
    printk("platform_get_resource is error\n");
  }
  printk("reg valus is %llx\n" , myresources->start);
  return 0;
}
```

编译成模块之后，放到开发板上进行加载，打印信息如下

![image-20240714094806790](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714094806790.png)

可以看到使用 platform_get_resource 函数获取 reg 资源的函数失败了。

在设备树中添加 ranges 属性：

![image-20240714095014129](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714095014129.png)

然后重新编译内核，将编译生成的 boot.img 烧写进开发板之后重新加载上面编写的驱动程序，可以看到之前获取失败的打印就消失了，而且成功打印出了 reg 属性的第一个值，如下图

![image-20240714095040567](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714095040567.png)

#### **ranges 属性作用**

ranges 属性是一种用于描述设备之间地址映射关系的属性。它在设备树（Device Tree）中使用，用于描述子设备地址空间如何映射到父设备地址空间。设备树是一种硬件描述语言，用于描述嵌入式系统中的硬件组件和它们之间的连接关系。

设备树中的每个设备节点都可以具有 ranges 属性，其中包含了地址映射的信息。下面是一个常见的格式：

> ranges = <child-bus-address parent-bus-address length>;

或者

> ranges;

然后对上述格式中每个部分进行解释：

**child-bus-address**：子设备地址空间的起始地址。它指定了子设备在父设备地址空间中的位置。具体的字长由 ranges 所在节点的 #address-cells 属性决定。

parent-bus-address：父设备地址空间的起始地址。它指定了父设备中用于映射子设备的地址范围。具体的字长由 ranges 的父节点的 #address-cells 属性决定。

length：映射的大小。它指定了子设备地址空间在父设备地址空间中的长度。具体的字长由 ranges 的父节点的 #size-cells 属性决定。

当 ranges 属性的值为空时，表示子设备地址空间和父设备地址空间具有完全相同的映射，即 1:1 映射。这通常用于描述内存区域，其中子设备和父设备具有相同的地址范围。

当 ranges 属性的值不为空时，按照指定的映射规则将子设备地址空间映射到父设备地址空间。具体的映射规则取决于设备树的结构和设备的特定要求。

然后以下面的设备树为例进行 ranges 属性的讲解，设备树内容如下:

```
/dts-v1/;
/ {
	compatible = "acme,coyotes-revenge";
  #address-cells = <1>;
  #size-cells = <1>;
  .... external-bus {
    #address-cells = <2>;
    #size-cells = <1>;
    ranges = <0 0 0x10100000 0x10000
    1 0 0x10160000 0x10000
    2 0 0x30000000 0x30000000>;
    // Chipselect 1, Ethernet
    // Chipselect 2, i2c controller
    // Chipselect 3, NOR Flash
```

这里以 ranges 的第一个属性值为例进行具体解释如下：

在 external-bus 节点中#address-cells 属性值为 2 表示 child-bus-address 由两个值表示，也就是 0 和 0，父节点的 #address-cells 属性值和#size-cells 属性值为 1，表示 parent-bus-address和 length 都由一个表示，也就是 0x10100000 和 0x10000，该 ranges 值表示将子地址空间（0x0-0xFFFF）映射到父地址空间 0x10100000 - 0x1010FFFF，这里的例子为带参数 ranges 属性映射，不带参数的 ranges 属性为 1：1 映射，较为简单，这里不再进行举例。

在嵌入式系统中，不同的设备可能连接到相同的总线或总线控制器上，它们需要在物理地址空间中进行正确的映射，以便进行数据交换和通信。例如，一个设备可能通过总线连接到主处理器或其他设备，而这些设备的物理地址范围可能不同。ranges 属性就是用来描述这种地址映射关系的。

#### 设备分类

##### **内存映射型设备**

内存映射型设备是指可以通过内存地址进行直接访问的设备。这类设备在物理地址空间中的一部分被映射到系统的内存地址空间中，使得 CPU 可以通过读写内存地址的方式与设备进行通信和控制。

特点：

（1）直接访问：内存映射型设备可以被 CPU 直接访问，类似于访问内存中的数据。这种直接访问方式提供了高速的数据传输和低延迟的设备操作。

（2）内存映射：设备的寄存器、缓冲区等资源被映射到系统的内存地址空间中，使用读写内存的方式与设备进行通信。

（3）读写操作：CPU 可以通过读取和写入映射的内存地址来与设备进行数据交换和控制操作。

在设备树中，内存映射型设备的设备树举例如下所示：

```
/dts-v1/;
/ {
    #address-cells = <1>;
    #size-cells = <1>;
    ranges;
    serial@101f0000 {
        compatible = "arm,pl011";
        reg = <0x101f0000 0x1000>;
    };
    gpio@101f3000 {
        compatible = "arm,pl061";
        reg = <0x101f3000 0x1000
        0x101f4000 0x10>;
    };
    spi@10115000 {
        compatible = "arm,pl022";
        reg = <0x10115000 0x1000>;
    };
};
```

第 5 行的 ranges 属性表示该设备树中会进行 1：1 的地址范围映射。



##### **非内存映射型设备**

非内存映射型设备是指不能通过内存地址直接访问的设备。这类设备可能采用其他方式与CPU 进行通信，例如通过 I/O 端口、专用总线或特定的通信协议。

特点：

（1）非内存访问：非内存映射型设备不能像内存映射型设备那样直接通过内存地址进行访问。它们可能使用独立的 I/O 端口或专用总线进行通信。

（2）特定接口：设备通常使用特定的接口和协议与 CPU 进行通信和控制，例如 SPI、I2C、UART 等。

（3）驱动程序：非内存映射型设备通常需要特定的设备驱动程序来实现与 CPU 的通信和控制。

在设备树中，非内存映射型设备的设备树举例如下所示：

```
/dts-v1/;
/ {
    compatible = "acme,coyotes-revenge";
    #address-cells = <1>;
    #size-cells = <1>;
    .... 
    external-bus {
        #address-cells = <2>;
        #size-cells = <1>;
        ranges = <0 0 0x10100000 0x10000
        		  1 0 0x10160000 0x10000
        		  2 0 0x30000000 0x30000000>;
        // Chipselect 1, Ethernet
        // Chipselect 2, i2c controller
        // Chipselect 3, NOR Flash
        ethernet@0,0 {
            compatible = "smc,smc91c111";
            reg = <0 0 0x1000>;
        };
        i2c@1,0 {
            compatible = "acme,a1234-i2c-bus";
            #address-cells = <1>;
            #size-cells = <0>;
            reg = <1 0 0x1000>;
            rtc@58 {
                compatible = "maxim,ds1338";
                reg = <0x58>;
            };
        };
    };
};
```

#### 映射地址计算

接下来以上面列举的非内存映射型设备的设备树中的 ethernet@0 节点为例，计算该网卡设备的映射地址。

首先，找到 ethernet@0 所在的节点，并查看其 reg 属性。在给定的设备树片段中，ethernet@0 的 reg 属性为<0 0 0x1000>。在根节点中，#address-cells 的值为 1，表示地址由一个单元格组成。

接下来，根据 ranges 属性进行地址映射计算。在 external-bus 节点的 ranges 属性中，有三个映射条目：

第一个映射条目为“0 0 0x10100000 0x10000”，表示外部总线的地址范围为 0x10100000到 0x1010FFFF。该映射条目的第一个值为 0，表示与 external-bus 节点的第一个子节点（ethernet@0,0）相关联。

第二个映射条目：“1 0 0x10160000 0x10000”，表示外部总线的地址范围为 0x10160000到 0x1016FFFF。该映射条目的第一个值为 1，表示与 external-bus 节点的第二个子节点（i2c@1,0）相关联。

第三个映射条目：“2 0 0x30000000 0x30000000”，表示外部总线的地址范围为 0x30000000到 0x5FFFFFFF。该映射条目的第一个值为 2，表示与 external-bus 节点的第三个子节点相关联。

由于ethernet@0与external-bus的第一个子节点相关联，并且它的reg属性为<0 0 0x1000>，我们可以进行以下计算：

ethernet@0 的物理起始地址 = 外部总线地址起始值=0x10100000

ethernet@0 的物理结束地址 = 外部总线地址起始值 + （ethernet@0 的 reg 属性的第二个值-1）

= 0x10100000 + 0xFFF 

= 0x10100FFF

因此，ethernet@0 的物理地址范围为 0x10100000 - 0x10100FFF，大家可以根据同样的方法计算 i2c@1 的物理地址。

### 获取中断资源

#### 中断配置

> myirq {
> 	compatible = "my_devicetree_irq";
> 	interrupt-parent = <&gpio3>;
> 	interrupts = <RK_PA5 IRQ_TYPE_LEVEL_LOW>;
> };

#### irq_of_parse_and_map 函数

该函数的主要功能是解析设备节点的"interrupts"属性，并将对应的中断号映射到系统的中断号。"interrupts"属性通常以一种特定的格式表示，可以包含一个或多个中断号。通过提供索引号，可以获取对应的中断号。

> **函数原型：**
>
> unsigned int irq_of_parse_and_map(struct device_node *dev, int index);
>
> **头文件：**
>
> \#include <linux/of_irq.h>
>
> **函数作用：**
>
> 从设备节点的"interrupts"属性中解析和映射对应的中断号
>
> **参数说明：**
>
> dev：设备节点，表示要解析的设备节点。
>
> index：索引号，表示从"interrupts"属性中获取第几个中断号。
>
> **返回值：**
>
> 是一个无符号整数，表示成功解析和映射的中断号。

#### irq_get_trigger_type 函数

该函数的主要功能是从给定的中断数据结构中提取中断触发类型。中断触发类型描述了中断信号的触发条件，例如边沿触发（edge-triggered）或电平触发（level-triggered）等。

> **函数原型：**
>
> u32 irqd_get_trigger_type(struct irq_data *d);
>
> **头文件：**
>
> \#include <linux/irq.h>
>
> **函数作用：**
>
> 从中断数据结构（irq_data）中获取对应的中断触发类型。
>
> **参数说明：**
>
> d：中断数据结构（irq_data），表示要获取中断触发类型的中断。
>
> **返回值：**
>
> 是一个无符号 32 位整数，表示成功获取的中断触发类型。

#### irq_get_irq_data 函数

函数 irq_get_irq_data 的作用是根据中断号获取对应的中断数据结构（irq_data）。

> **函数原型：**
>
> struct irq_data *irq_get_irq_data(unsigned int irq);
>
> **头文件：**
>
> \#include <linux/irq.h>
>
> **函数作用：**
>
> 根据中断号获取对应的中断数据结构。
>
> **参数说明：**
>
> irq：中断号，表示要获取中断数据结构的中断号。
>
> **返回值：**
>
> 指向 irq_data 结构体的指针，表示成功获取的中断数据结构。

#### of_irq_get 函数

该函数的主要功能是从给定的设备节点的"interrupts"属性中解析并获取对应的中断号。"interrupts"属性通常以一种特定的格式表示，可以包含一个或多个中断号。通过提供索引号，可以获取对应的中断号

> **函数原型：**
>
> int of_irq_get(struct device_node *dev, int index);
>
> **头文件**：
>
> \#include <linux/of_irq.h>
>
> **函数作用：**
>
> 是从设备节点的"interrupts"属性中获取对应的中断号。
>
> **参数说明：**
>
> dev：设备节点，表示要获取中断号的设备节点。
>
> index：索引号，表示从"interrupts"属性中获取第几个中断号。
>
> **返回值：**
>
> 是一个整数，表示成功获取的中断号。

#### platform_get_irq 函数

platform_get_irq 函数的主要功能是根据给定的平台设备和索引号获取对应的中断号。平台设备是指与特定硬件平台相关的设备。在某些情况下，平台设备可能具有多个中断号，通过提供索引号，可以获取对应的中断号。

> 函数原型：
>
> int platform_get_irq(struct platform_device *dev, unsigned int num);
>
> 函数作用：
>
> 根据平台设备和索引号获取对应的中断号。
>
> 头文件：
>
> linux/platform_device.h
>
> 参数说明：
>
> dev：平台设备，表示要获取中断号的平台设备。
>
> num：索引号，表示从中获取第几个中断号。
>
> 返回值：
>
> 是一个整数，表示成功获取的中断号。

#### 实验

设备树

```
myirq {
    compatible = "my_devicetree_irq";
    interrupt-parent = <&gpio3>;
    interrupts = <RK_PA5 IRQ_TYPE_LEVEL_LOW>;
};
```

驱动

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/of.h>
#include <linux/of_irq.h>
#include <linux/gpio.h>
int num;
int irq;
struct irq_data *my_irq_data;
struct device_node *mydevice_node;
u32 trigger_type;

// 平台设备的初始化函数
static int my_platform_probe(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_probe: Probing platform device\n");

    // 查找设备节点
    mydevice_node = of_find_node_by_name(NULL, "myirq");
    
    // 解析和映射中断
    irq = irq_of_parse_and_map(mydevice_node, 0);
    printk("irq is %d\n", irq);
    
    // 获取中断数据结构
    my_irq_data = irq_get_irq_data(irq);
    // 获取中断触发类型
    trigger_type = irqd_get_trigger_type(my_irq_data);
    printk("trigger type is 0x%x\n", trigger_type);
    
    // 将GPIO转换为中断号
    irq = gpio_to_irq(101);
    printk("irq is %d\n", irq);
    
    // 从设备节点获取中断号
    irq = of_irq_get(mydevice_node, 0);
    printk("irq is %d\n", irq);
    
    // 获取平台设备的中断号
    irq = platform_get_irq(pdev, 0);
    printk("irq is %d\n", irq);
    return 0;
}

// 平台设备的移除函数
static int my_platform_remove(struct platform_device *pdev)
{
    printk(KERN_INFO "my_platform_remove: Removing platform device\n");

    // 清理设备特定的操作
    // ...

    return 0;
}


const struct of_device_id of_match_table_id[]  = {
	{.compatible="my_devicetree_irq"},
};

// 定义平台驱动结构体
static struct platform_driver my_platform_driver = {
    .probe = my_platform_probe,
    .remove = my_platform_remove,
    .driver = {
        .name = "my_platform_device",
        .owner = THIS_MODULE,
		.of_match_table =  of_match_table_id,
    },
};

// 模块初始化函数
static int __init my_platform_driver_init(void)
{
    int ret;

    // 注册平台驱动
    ret = platform_driver_register(&my_platform_driver);
    if (ret) {
        printk(KERN_ERR "Failed to register platform driver\n");
        return ret;
    }

    printk(KERN_INFO "my_platform_driver: Platform driver initialized\n");

    return 0;
}

// 模块退出函数
static void __exit my_platform_driver_exit(void)
{
    // 注销平台驱动
    platform_driver_unregister(&my_platform_driver);

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

### bindings 文档

我们已经介绍了许多设备树编写相关的知识，当然上面我们讲解的都是标准属性，但当我们遇到非标准属性或无法理解的属性时，要如何处理呢？这时候就不得不提到 bindings 文档了。

Documentation/devicetree/bindings 目录是 Linux 内核源码中的一个重要目录，用于存储设备树（Device Tree）的 bindings 文档。设备树是一种描述硬件平台和设备配置的数据结构，它以一种可移植和独立于具体硬件的方式描述了设备的属性、寄存器配置、中断信息等。

bindings 目录中的文档提供了有关设备树的各种设备和驱动程序的详细说明和用法示例。这些文档对于开发人员来说非常重要，因为它们提供了在设备树中描述硬件和配置驱动程序所需的属性和约定。bindings 目录截图如下（图 70-1）所示：

![image-20240714100530251](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714100530251.png)

接下来对 Documentation/devicetree/bindings 目录的一些常见子目录和其内容的概述：

> arm：包含与 ARM 体系结构相关的设备和驱动程序的 bindings 文档。
>
> clock：包含与时钟设备和时钟控制器相关的 bindings 文档。
>
> dma：包含与直接内存访问（DMA）控制器和设备相关的 bindings 文档。
>
> gpio：包含与通用输入输出（GPIO）控制器和设备相关的 bindings 文档。
>
> i2c：包含与 I2C 总线和设备相关的 bindings 文档。
>
> interrupt-controller：包含与中断控制器相关的 bindings 文档。
>
> media：包含与多媒体设备和驱动程序相关的 bindings 文档。
>
> mfd：包含与多功能设备（MFD）子系统和设备相关的 bindings 文档。
>
> networking：包含与网络设备和驱动程序相关的 bindings 文档。
>
> power：包含与电源管理子系统和设备相关的 bindings 文档。
>
> spi：包含与 SPI 总线和设备相关的 bindings 文档。
>
> usb：包含与 USB 控制器和设备相关的 bindings 文档。
>
> video：包含与视频设备和驱动程序相关的 bindings 文档。

每个子目录中的文档通常以.txt 或.yaml 的扩展名保存，使用文本或 YAML 格式编写。这些文档提供了有关设备树中属性的详细说明、属性的语法、可选值和用法示例。它们还描述了设备树的约定和最佳实践，以帮助开发人员正确地配置和描述硬件设备和驱动程序。
