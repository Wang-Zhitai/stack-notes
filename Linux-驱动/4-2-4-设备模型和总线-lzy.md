## 4-2-4-设备模型和总线

 ### 设备模型框架

#### 设备模型介绍

字符设备驱动通常适用于相对简单的设备，对于一些更复杂的功能，比如说电源管理和热插拔事件管理，使用字符设备框架可能不够灵活和高效。为了应对更复杂的设备和功能，Linux内核提供了设备模型。设备模型允许开发人员以更高级的方式来描述硬件设备和它们之间的关系，并提供一组通用 API 和机制来处理设备的注册，热插拔事件，电源管理等。通过使用设备模型，驱动开发人员可以将更多的底层功能交给内核来处理，而不必重复实现这些基础功能。这使得驱动的编写更加高级和模块化，减少了重复工作和出错的可能性。对于一些常见的硬件设备，如 USB、i2c 和平台设备，内核已经提供了相应的设备模型和相关驱动，开发人员可以基于这些模型来编写驱动，从而更快地实现特定设备的功能，并且可以借助内核的电源管理和热插拔事件管理功能。

总之，使用设备模型可以帮助简化驱动开发过程，并提供更高级的功能和灵活性，使得驱动开发人员能够更好地适应复杂的硬件设备需求。

#### 设备模型的优点

设备模型在内核驱动中扮演着重要的角色，它提供了一种统一的方式来描述硬件设备和它们之间的关系。

以下是设备模型在内核驱动中的几个重要方面。

1 代码复用：设备模型允许多个设备复用同一个驱动。通过在设备树或总线上定义不同的设备节点，这些设备可以使用相同的驱动进行初始化和管理。这样可以减少代码的冗余，提高驱动的复用性和维护性。

2 资源的动态申请和释放：设备模型提供了一种机制来动态申请和释放设备所需的资源，如内存，中断等。驱动可以使用这些机制来管理设备所需的资源，确保在设备初始化和关闭时进行正确的资源分配和释放。

3 简化驱动编写: 设备模型提供了一组通用 API 和机制，使得驱动编写更加简化和模块化。开发人员可以使用这些 API 来注册设备，处理设备事件，进行设备的读写操作等，而无需重复实现这些通用功能。

4 热插拔机制：设备模型支持热插拔机制，能够在运行时动态添加或移除设备。当设备插入或拔出时，内核会生成相应的热插拔事件，驱动可以通过监听这些事件来执行相应的操作，如设备的初始化或释放。

5 驱动的面向对象思想：设备模型的设计借鉴了面向对象编程（OOP）的思想。每个设备都被看作是一个对象，具有自己的属性和方法，并且可以通过设备模型的机制进行继承和扩展。这种设计使得驱动的编写更加模块化和可扩展，可以更好地应对不同类型的设备和功能需求。

总之，设备模型在内核驱动中扮演着关键的角色，通过提供统一的设备描述和管理机制，简化了驱动的编写和维护过程，提高了代码的复用性和可维护性，并支持热插拔和动态资源管理等重要功能。

### kobject 和 kset

kobject 和 kset 是 Linux 内核中用于管理内核对象的基本概念。

kobject(内核对象)是内核中抽象出来的通用对象模型，用于表示内核中的各种实体。kobject是一个结构体，其中包含了一些描述该对象的属性和方法。它提供了一种统一的接口和机制，用于管理和操作内核对象。kobject 结构体在内核源码 kernel/include/linux/kobject.h 文件中。

```c
struct kobject {
  const char *name;
  struct list_headentry;
  struct kobject *parent;
  struct kset *kset;
  struct kobj_type *ktype;
  struct kernfs_node *sd; /* sysfs directory entry */
  struct kref kref;
  #ifdef CONFIG_DEBUG_KOBJECT_RELEASE
  struct delayed_work release;
  #endif
  unsigned int state_initialized:1;
  unsigned int state_in_sysfs:1;
  unsigned int state_add_uevent_sent:1;
  unsigned int state_remove_uevent_sent:1;
  unsigned int uevent_suppress:1;
  ANDROID_KABI_RESERVE(1);
  ANDROID_KABI_RESERVE(2);
  ANDROID_KABI_RESERVE(3);
  ANDROID_KABI_RESERVE(4);
};
```

这是 Linux 内核中的 struct kobject 结构体的定义。它包含了一些字段用于表示和管理内核对象。

下面是一些主要字段的解释：

> \- const char *name：表示 kobject 的名称，通常用于在/sys 目录下创建对应的目录。
>
> \- struct list_head entry：用于将 kobject 链接到父 kobject 的子对象列表中，以建立层次关系。
>
> \- struct kobject *parent：指向父 kobject，表示 kobject 的层次关系。
>
> \- struct kset *kset：指向包含该 kobject 的 kset，用于进一步组织和管理 kobject。
>
> \- struct kobj_type *ktype：指向定义 kobject 类型的 kobj_type 结构体，描述 kobject 的属性和操作。
>
> \- struct kernfs_node *sd：指向 sysfs 目录中对应的 kernfs_node，用于访问和操作 sysfs 目录项。
>
> \- struct kref kref：用于对 kobject 进行引用计数，确保在不再使用时能够正确释放资源。
>
> \- unsigned int 字段：表示一些状态标志和配置选项，例如是否已初始化、是否在 sysfs 中、是否发送了 add/remove uevent 等。

该结构体还包含了一些与特定配置相关的保留字段。这些字段共同构成了 kobject 的基本属性和关系，用于在内核中表示和管理不同类型的内核对象。通过这些字段，可以建立层次化的关系，进行资源管理和操作。每一个 kobject 都会对应系统/sys/下的一个目录，如下图所示。

![image-20240714093448579](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714093448579.png)

之前我们在学习平台总线时，查看平台总线要进入/sys/bus 目录下，bus 目录下的文件都是和总线相关的目录，比如 amba 总线，CPU 总线，platform 总线。如下图所示：

![image-20240714093517073](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714093517073.png)

kobject 表示系统/sys 下的一个目录，而目录又是有多个层次，所以对应 kobject 的树状关系如下图所示：

![image-20240714100855622](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714100855622.png)

我们来分析一下上图。在 kobject 结构体中，parent 指针用于表示父 kobject，从而建立了kobject 之间的层次关系，类似于目录结构中的父目录和子目录的关系。一个 kobject 可以有一个父 kobject 和多个子 kobject，通过 parent 指针可以将它们连接起来形成一个层次化的结构，类似于目录结构中，一个目录可以有一个父目录和多个子目录，通过目录的路径可以表示目录之间的层次关系。这种层次化的关系可以方便地进行遍历，查找和管理，使得内核对象能够按照层次关系进行组织和管理。这种设计使得 kobject 的树状结构在内核中具有很高的灵活性和可扩展性。

接下来我们继续学习 kset。

kset(内核对象集合)是一种用于组织和管理一组相关 kobject 的容器。kset 是 kobject 的一种扩展，它提供了一种层次化的组织结构，可以将一组相关的 kobject 组织在一起。kset 在内核里面用 struct kset 结构体来表示，定义在 include/linux/kobject.h 头文件中

```c
struct kset {
    struct list_head list;
    spinlock_t list_lock;
    struct kobject kobj;
    const struct kset_uevent_ops *uevent_ops;
    ANDROID_KABI_RESERVE(1);
    ANDROID_KABI_RESERVE(2);
      ANDROID_KABI_RESERVE(3);
    ANDROID_KABI_RESERVE(4);
} __randomize_layout;
```

这是 Linux 内核中的 struct kset 结构体的定义。kset 是一种用于组织和管理 kobject 的集合。

下面是一些主要字段的解释：

> \- struct list_head list：用于将 kset 链接到全局 kset 链表中，以便对 kset 进行遍历和管理。
>
> \- spinlock_t list_lock：用于保护对 kset 链表的并发访问，确保线程安全性。
>
> \- struct kobject kobj：作为 kset 的 kobject 表示，用于在/sys 目录下创建对应的目录，并与kset 关联。
>
> \- const struct kset_uevent_ops *uevent_ops：指向 kset 的 uevent 操作的结构体，用于处理与 kset 相关的 uevent 事件。

该结构体还包含了一些与特定配置相关的保留字段。kset 通过包含一个 kobject 作为其成员，将 kset 本身表示为一个 kobject，并使用 kobj 来管理和操作 kset。通过 list 字段，kset 可以链接到全局 kset 链表中，以便进行全局的遍历和管理。同时，list_lock 字段用于保护对 kset 链表的并发访问。kset 还可以定义自己的 uevent 操作，用于处理与 kset 相关的 uevent 事件，例如在添加或删除 kobject 时发送相应的 uevent 通知。

这些字段共同构成了 kset 的基本属性和关系，用于在内核中组织和管理一组相关的kobject。kset 提供了一种层次化的组织结构，并与 sysfs 目录相对应，方便对 kobject 进行管理和操作。

### kset 和 kobject 的关系

在 Linux 内核中，kset 和 kobject 是相关联的两个概念，它们之间存在一种层次化的关系，

![image-20240714101113515](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714101113515.png)

1. kset 是 kobject 的一种扩展：

   kset 可以被看作是 kobject 的一种特殊形式，它扩展了 kobject并提供了一些额外的功能。kset 可以包含多个 kobject，形成一个层次化的组织结构。

2. kobject 属于一个 kset：每个 kobject 都属于一个 kset。

   kobject 结构体中的 struct kset *kset字段指向所属的 kset。这个关联关系表示了 kobject 所在的集合或组织。

通过 kset 和 kobject 之间的关系，可以实现对内核对象的层次化管理和操作。kset 提供了对 kobject 的集合管理接口，可以通过 kset 来迭代、查找、添加或删除 kobject。同时，kset 也提供了特定于集合的功能，例如在集合级别处理 uevent 事件。

总结起来，kset 和 kobject 之间的关系是：一个 kset 可以包含多个 kobject，而一个 kobject只能属于一个 kset。kset 提供了对 kobject 的集合管理和操作接口，用于组织和管理具有相似特性或关系的 kobject。这种关系使得内核能够以一种统一的方式管理和操作不同类型的内核对象。

### 创建 kobject

本章介绍了两种创建 kobject 的方法，一种是使用 kobject_create_and_add 函数，另一种是使用 kzalloc 和 kobject_init_and_add 函数，还介绍了如何释放创建的 kobject。最后，实验演示了将驱动程序加载到开发板上，并验证了创建的 kobject 是否成功。

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>

// 定义了三个kobject指针变量：mykobject01、mykobject02、mykobject03
struct kobject *mykobject01;
struct kobject *mykobject02;
struct kobject *mykobject03;

// 定义了一个kobj_type结构体变量mytype，用于描述kobject的类型。
struct kobj_type mytype;
// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;
    // 创建kobject的第一种方法
    // 创建并添加了名为"mykobject01"的kobject对象，父kobject为NULL
    mykobject01 = kobject_create_and_add("mykobject01", NULL);
    // 创建并添加了名为"mykobject02"的kobject对象，父kobject为mykobject01。
    mykobject02 = kobject_create_and_add("mykobject02", mykobject01);

    // 创建kobject的第二种方法
    // 1 使用kzalloc函数分配了一个kobject对象的内存
    mykobject03 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    // 2 初始化并添加到内核中，名为"mykobject03"。
    ret = kobject_init_and_add(mykobject03, &mytype, NULL, "%s", "mykobject03");

    return 0;
}

// 模块退出函数
static void mykobj_exit(void)
{
    // 释放了之前创建的kobject对象
    kobject_put(mykobject01);
    kobject_put(mykobject02);
    kobject_put(mykobject03);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
MODULE_AUTHOR("woniu"); // 模块的作者
```

### 创建 kset

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>

// 定义kobject结构体指针，用于表示第一个自定义内核对象
struct kobject *mykobject01;
// 定义kobject结构体指针，用于表示第二个自定义内核对象
struct kobject *mykobject02;
// 定义kset结构体指针，用于表示自定义内核对象的集合
struct kset *mykset;
// 定义kobj_type结构体，用于定义自定义内核对象的类型
struct kobj_type mytype;

// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;

    // 创建并添加kset，名称为"mykset"，父kobject为NULL，属性为NULL
    mykset = kset_create_and_add("mykset", NULL, NULL);

    // 为mykobject01分配内存空间，大小为struct kobject的大小，标志为GFP_KERNEL
    mykobject01 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    // 将mykset设置为mykobject01的kset属性
    mykobject01->kset = mykset;
    // 初始化并添加mykobject01，类型为mytype，父kobject为NULL，格式化字符串为"mykobject01"
    ret = kobject_init_and_add(mykobject01, &mytype, NULL, "%s", "mykobject01");

    // 为mykobject02分配内存空间，大小为struct kobject的大小，标志为GFP_KERNEL
    mykobject02 = kzalloc(sizeof(struct kobject), GFP_KERNEL);
    // 将mykset设置为mykobject02的kset属性
    mykobject02->kset = mykset;
    // 初始化并添加mykobject02，类型为mytype，父kobject为NULL，格式化字符串为"mykobject02"
    ret = kobject_init_and_add(mykobject02, &mytype, NULL, "%s", "mykobject02");

    return 0;
}

// 模块退出函数
static void mykobj_exit(void)
{
    // 释放mykobject01的引用计数
    kobject_put(mykobject01);

    // 释放mykobject02的引用计数
    kobject_put(mykobject02);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
MODULE_AUTHOR("woniu"); // 模块的作者
```

### 设备模型简介

设备模型在内核驱动中扮演着关键的角色，通过提供统一的设备描述和管理机制，简化了驱动的编写和维护过程，提高了代码的复用性和可维护性，并支持热插拔和动态资源管理等重要功能。设备模型包含以下四个概念：

> 1. 总线（Bus）：总线是设备模型中的基础组件，用于连接和传输数据的通信通道。总线可以是物理总线（如 PCI、USB）或虚拟总线（如虚拟设备总线）。总线提供了设备之间进行通信和数据传输的基本机制。
>
> 2. 设备（Device）：设备是指计算机系统中的硬件设备，例如网卡、显示器、键盘等。每个设备都有一个唯一的标识符，用于在系统中进行识别和管理。设备模型通过设备描述符来描述设备的属性和特性。
>
> 3. 驱动（Driver）：驱动是设备模型中的软件组件，用于控制和管理设备的操作。每个设备都需要相应的驱动程序来与操作系统进行交互和通信。驱动程序负责向设备发送命令、接收设备事件、进行设备配置等操作。
>
> 4. 类（Class）：类是设备模型中的逻辑组织单元，用于对具有相似功能和特性的设备进行分类和管理。类定义了一组共享相同属性和行为的设备的集合。通过设备类，可以对设备进行分组、识别和访问。

设备模型的设计目的是为了提供一种统一的方式来管理和操作系统中的各种硬件设备。通过将设备、驱动和总线等概念进行抽象和标准化，设备模型可以提供一致的接口和数据结构，简化驱动开发和设备管理，并实现设备的兼容性和可移植性。

#### 相关结构体

在 Linux 设备模型中，虚构了一条名为“platform”的总线，用来连接一些直接与 CPU 相连的设备控制器。这种设备控制器通常不符合常见的总线标准，比如 PCI 总线和 USB 总线，所以 Linux 使用 platform 总线来管理这些设备。Platform 总线允许设备控制器与设备驱动程序进行通信和交互。设备控制器在设备树中定义，并通过设备树与对应的设备驱动程序匹配。在设备模型中，Platform 总线提供了一种统一的接口和机制来注册和管理这些设备控制器。设备驱动程序可以通过注册到 Platform 总线的方式，与相应的设备控制器进行绑定和通信。设备驱动程序可以访问设备控制器的寄存器、配置设备、处理中断等操作，如下图所示：

![image-20240714101655125](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714101655125.png)

当作为嵌入式/驱动开发人员时，了解设备，驱动程序，总线和类这几个结构体的概念和关系非常重要。尽管在芯片原厂提供的 BSP 中已经实现了设备模型，但是了解这些概念可以帮助您更好地理解设备的工作原理，驱动程序的编写和设备的管理。

##### struct bus_type

bus_type 结构体是 Linux 内核中用于描述总线的数据结构，定义在 include/linux/device.h头文件中。以下是对 bus_type 结构体的一般定义。

```c
struct bus_type {
  const char *name;
  const char *dev_name;
  struct device *dev_root;
  const struct attribute_group **bus_groups;
  const struct attribute_group **dev_groups;
  const struct attribute_group **drv_groups;
  int (*match)(struct device *dev, struct device_driver *drv);
  int (*uevent)(struct device *dev, struct kobj_uevent_env *env);
  int (*probe)(struct device *dev);
  void (*sync_state)(struct device *dev);
  int (*remove)(struct device *dev);
  void (*shutdown)(struct device *dev);
  int (*online)(struct device *dev);
  int (*offline)(struct device *dev);
  int (*suspend)(struct device *dev, pm_message_t state);
  int (*resume)(struct device *dev);
  int (*num_vf)(struct device *dev);
  int (*dma_configure)(struct device *dev);
  const struct dev_pm_ops *pm;
  const struct iommu_ops *iommu_ops;
  struct subsys_private *p;
  struct lock_class_key lock_key;
  bool need_parent_lock;
  ANDROID_KABI_RESERVE(1);
  ANDROID_KABI_RESERVE(2);
  ANDROID_KABI_RESERVE(3);
  ANDROID_KABI_RESERVE(4);
};
```

以下是结构体包含的成员和其作用的简要说明

> name:总线类型的名称
>
> dev_name:总线设备名称
>
> dev_root: 总线设备的根设备
>
> bus_groups: 总线类型的属性组
>
> dev_groups: 设备的属性组
>
> drv_groups: 驱动程序的属性组

以下是一些回调函数成员

> match: 设备和驱动程序之间的匹配函数
>
> uevent:设备的事件处理函数
>
> probe: 设备的探测函数
>
> sync_state: 设备状态同步函数
>
> remove: 设备的移除函数
>
> online:设备上线函数
>
> offline：设备离线函数
>
> suspend: 设备的挂起函数
>
> resume: 设备的恢复函数
>
> num_vf: 设备的虚拟功能数目函数
>
> dma_configure:设备的 DMA 配置函数

以下是一些其他成员：

> pm：设备的电源管理操作。
>
> iommu_ops：设备的 IOMMU 操作。
>
> p：子系统私有数据。
>
> lock_key：用于锁机制的锁类别键。
>
> need_parent_lock：是否需要父级锁。

##### struct device

device 结构体是 Linux 内核中用于描述设备的数据结构，定义在 include/linux/device.h 头文件中。以下是 device 

```c
struct device {
  //设备的父设备
  struct device *parent;
  //私有指针
  struct device_private *p;
  //对应的 kobj
  struct kobject kobj;
  //设备初始化的名字
  const char *init_name;
  //设备类型
  const struct device_type *type;
  //设备所属的总线
  struct bus_type *bus;
  struct device_driver *driver;
    ........... 
  //设备所属的类
  struct class *class;
  //设备的属性组
  const struct attribute_group **groups; /* optional groups */
  ........... 
};
```

##### struct device_driver

struct device_driver 是 Linux 内 核 中 描 述 设 备 驱 动 程 序 的 数 据 结 构 ， 定 义 在include/linux/device.h 头文件中。以下是 struct device_driver 

```c
struct device_driver {
  const char *name;
  struct bus_type *bus;
  struct module *owner;
  const char *mod_name; /* used for built-in modules */
  bool suppress_bind_attrs; /* disables bind/unbind via sysfs */
  enum probe_type probe_type;
  const struct of_device_id *of_match_table;
  const struct acpi_device_id *acpi_match_table;
  int (*probe) (struct device *dev);
  void (*sync_state)(struct device *dev);
  int (*remove) (struct device *dev);
  void (*shutdown) (struct device *dev);
  int (*suspend) (struct device *dev, pm_message_t state);
  int (*resume) (struct device *dev);
  const struct attribute_group **groups;
  const struct dev_pm_ops *pm;
  void (*coredump) (struct device *dev);
   struct driver_private *p;
  ANDROID_KABI_RESERVE(1);
  ANDROID_KABI_RESERVE(2);
  ANDROID_KABI_RESERVE(3);
  ANDROID_KABI_RESERVE(4);
};
```

以下是该结构体包含的成员和其作用的简要说明。

> name:设备驱动程序的名称
> bus: 设备驱动程序所属的总线类型
> owner：拥有该驱动程序的模块。
> mod_name：用于内置模块的名称。
> suppress_bind_attrs：禁用通过 sysfs 进行绑定/解绑的属性。
> probe_type：探测类型，用于指定探测的方式。
> of_match_table：Open Firmware（OF）设备匹配表。
> acpi_match_table：ACPI 设备匹配表。
> groups：驱动程序的属性组。
> pm：电源管理操作。
> p：驱动程序的私有数据。

以下是一些回调函数成员

> probe:设备的探测函数，用于初始化和配置设备，
>
> **注意：device 和 device_driver 必须挂在同一个 bus 下。这样才可以触发 probe**
> sync_state: 设备状态同步函数
> remove:设备的移除函数
> shutdown: 设备的关机函数
> suspend: 设备的挂起函数
> resume: 设备的恢复函数
> coredump: 设备的核心转储函数

##### struct class

struct class 是 Linux 内核中描述设备类的数据结构，定义在 include/linux/device.h 头文件中。以下是 struct class 的一般定义：

```c
struct class {
  const char *name;
  struct module *owner;
  const struct attribute_group **class_groups;
  const struct attribute_group **dev_groups;
  struct kobject *dev_kobj;
  int (*dev_uevent)(struct device *dev, struct kobj_uevent_env *env);
  char *(*devnode)(struct device *dev, umode_t *mode);
  void (*class_release)(struct class *class);
  void (*dev_release)(struct device *dev);
  int (*shutdown_pre)(struct device *dev);
  const struct kobj_ns_type_operations *ns_type;
  const void *(*namespace)(struct device *dev);
  void (*get_ownership)(struct device *dev, kuid_t *uid, kgid_t *gid);
  const struct dev_pm_ops *pm;
  struct subsys_private *p;
  ANDROID_KABI_RESERVE(1);
  ANDROID_KABI_RESERVE(2);
  ANDROID_KABI_RESERVE(3);
  ANDROID_KABI_RESERVE(4);
};
```

以下是该结构体包含的成员和其作用的简要说明

> name : 设备类的名称
> owner：拥有该类的模块
> 以下是一些属性组成员
> class_groups: 类属性组，用于描述设备类的属性，在类注册到内核时，会自动在
> /sys/class/xxx_class 下创建对应的属性文件
> dev_groups: 设备属性组，用于描述设备的属性，在类注册到内核时，会自动在该类下的
> 设备目录创建对应的属性文件
> dev_kobj: 设备的内核对象

以下是一些回调函数成员

> dev_uevent :设备的事件处理函数
> devnode : 生成设备节点的函数

以下是一些其他成员

> class_release: 类资源的释放函数
> dev_release: 设备资源的释放函数
> shutdown_pre: 设备关机前的回调函数
> ns_type: 命名空间类型操作
> namespace: 命名空间函数
> get_ownership: 获取设备所有权的函数
> pm: 电源管理操作
> p: 子系统私有数据

### sysfs 文件系统（/sys  File System）

在前面的章节中，我们介绍了设备模型四个重要的组成部分：总线，设备，驱动，类。然而，要更深入地理解设备模型，我们需要进一步探究其代码层面的实现。在上一章节实验中，我们创建了 kobject。本章节我们从代码的层面分析下——为什么当创建 kobj 的时候，父节点为 NULL，会在系统根目录/sys 目录下创建呢。

sysfs 文件系统是 Linux 内核提供的一种虚拟文件系统，用于向用户空间提供内核中设备，驱动程序和其他内核对象的信息。它以一种层次结构的方式组织数据，并将这些数据表示为文件和目录，使得用户空间可以通过文件系统接口访问和操作内核对象的属性。

sysfs 提供了一种统一的接口，用于浏览和管理内核中的设备、总线、驱动程序和其他内核对象。它在 /sys 目录下挂载，用户可以通过查看和修改 /sys 目录下的文件和目录来获取和配置内核对象的信息。

#### 设备模型的基本框架

当使用 kobject 时，通常不会单独使用它，而是将其嵌入到一个数据结构中。这样做的目

的是将高级对象接入到设备模型中。比如 cdev 结构体和 platform_device 结构体，如下所示：cdev 结构体如下所示，其成员有 kobject

```c
struct cdev {
    struct kobject kobj;//内嵌到 cdev 中的 kobject
    struct module *owner;
    const struct file_operations *ops;
    struct list_head list;
    dev_t dev;
    unsigned int count;
}
```

platform_device结构体如下所示，其成员有device结构体，在device结构体中包含了kobject结构体。

```c
struct platform_device {
  const char*name;
  int id;
  bool id_auto;
  struct device dev;
  u32 num_resources;
  struct resource *resource;
  const struct platform_device_id *id_entry;
  char *driver_override; /* Driver name to force a match */
  /* MFD cell pointer */
  struct mfd_cell *mfd_cell;
  /* arch specific additions */
  struct pdev_archdata archdata;
};
```

device 结构体如下所示：

```c
struct device {
  //设备的父设备
  struct device *parent;
  //私有指针
  struct device_private *p;
  //对应的 kobj
  struct kobject kobj;
  //设备初始化的名字
  const char *init_name;
  //设备类型
  const struct device_type *type;
  //设备所属的总线
  struct bus_type *bus;
  struct device_driver *driver;
  ........... //设备所属的类
  struct class *class;
  //设备的属性组
  const struct attribute_group **groups; /* optional groups */
  ........... 
};
```

所以我们也可以把总线，设备，驱动看作是 kobject 的派生类。因为他们都是设备模型中的实体，通过继承或扩展 kobject 来实现与设备模型的集成。

在 Linux 内核中，kobject 是一个通用的基础结构，用于构建设备模型。每个 kobject 实例对应于 sys 目录下的一个目录，这个目录包含了该 kobject 相关的属性，操作和状态信息。如下图所示：

![image-20240714102816544](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714102816544.png)

因此，可以说 kobject 是设备模型的基石，通过创建对应的目录结构和属性文件， 它提供了一个统一的接口和框架，用于管理和操作设备模型中的各个实体。

### sysfs 目录层次分析

和设备模型有关的文件夹为 bus，class，devices。完整路径为如下所示：

> /sys/bus
>
> /sys/class
>
> /sys/devices

/sys/devices：该目录包含了系统中所有设备的子目录。每个设备子目录代表一个具体的设备，通过其路径层次结构和符号链接反映设备的关系和拓扑结构。每个设备子目录中包含了设备的属性、状态和其他相关信息。如下所示：

![image-20240714102956238](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714102956238.png)

/sys/bus：该目录包含了总线类型的子目录。每个子目录代表一个特定类型的总线，例如PCI、USB 等。每个总线子目录中包含与该总线相关的设备和驱动程序的信息。

![image-20240714103014962](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714103014962.png)

比如 I2C 总线下连接的设备，如下所示：

![image-20240714103030949](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714103030949.png)

/sys/class：该目录包含了设备类别的子目录。每个子目录代表一个设备类别，例如磁盘、网络接口等。每个设备类别子目录中包含了属于该类别的设备的信息。如下图所示：

![image-20240714103046127](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714103046127.png)

使用 class 进行归类的好处有以下几点：

1. 逻辑上的组织：通过将设备按照类别进行归类，可以在设备模型中建立逻辑上的组织结构。这样，相关类型的设备可以被放置在同一个类别目录下，使得设备的组织结构更加清晰和可管理。

2. 统一的接口和属性：每个设备类别目录下可以定义一组统一的接口和属性，用于描述和配置该类别下所有设备的共同特征和行为。这样，对于同一类别的设备，可以使用相同的方法和属性来操作和配置，简化了设备驱动程序的编写和维护。

3. 简化设备发现和管理：通过将设备进行分类，可以提供一种简化的设备发现和管理机
4. 制。用户和应用程序可以在类别目录中查找和识别特定类型的设备，而无需遍历整个设备模型。这样，设备的发现和访问变得更加高效和方便。
5. 扩展性和可移植性：使用`class`进行归类可以提供一种扩展性和可移植性的机制。当引入新的设备类型时，可以将其归类到现有的类别中，而无需修改现有的设备管理和驱动程序。这种扩展性和可移植性使得系统更加灵活，并且对于开发人员和设备供应商来说，更容易集成新设备。

比如应用现在要设置 gpio

如果使用类可以直接使用以下命令：

>  echo 1 > /sys/class/gpio/gpio157/value

如果不使用类，使用以下命令：

> echo 1 > /sys/devices/platform/fe770000.gpio/gpiochip4/gpio/gpio157/value

![image-20240714103203913](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240714103203913.png)

### 创建属性文件并实现读写功能

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/sysfs.h> 

// 自定义的kobject结构体，包含一个kobject对象和两个整型值
struct mykobj
{
    struct kobject kobj;
    int value1;
    int value2;
};

// 定义了mykobj结构体指针变量mykobject01
struct mykobj *mykobject01;

// 自定义的kobject释放函数
static void dynamic_kobj_release(struct kobject *kobj)
{
    struct mykobj *mykobject01 = container_of(kobj, struct mykobj, kobj);
    printk("kobject: (%p): %s\n", kobj, __func__);
    kfree(mykobject01);
}

// 自定义的attribute对象value1和value2
struct attribute value1 = {
    .name = "value1",
    .mode = 0666,
};
struct attribute value2 = {
    .name = "value2",
    .mode = 0666,
};

// 将attribute对象放入数组中
struct attribute *myattr[] = {
    &value1,
    &value2,
    NULL,
};

// 自定义的show函数，用于读取属性值
ssize_t myshow(struct kobject *kobj, struct attribute *attr, char *buf)
{
    ssize_t count;
    struct mykobj *mykobject01 = container_of(kobj, struct mykobj, kobj);
    if (strcmp(attr->name, "value1") == 0)
    {
        count = sprintf(buf, "%d\n", mykobject01->value1);
    }
    else if (strcmp(attr->name, "value2") == 0)
    {
        count = sprintf(buf, "%d\n", mykobject01->value2);
    }
    else
    {
        count = 0;
    }
    return count;
}

// 自定义的store函数，用于写入属性值
ssize_t mystore(struct kobject *kobj, struct attribute *attr, const char *buf, size_t size)
{
    struct mykobj *mykobject01 = container_of(kobj, struct mykobj, kobj);
    if (strcmp(attr->name, "value1") == 0)
    {
        sscanf(buf, "%d\n", &mykobject01->value1);
    }
    else if (strcmp(attr->name, "value2") == 0)
    {
        sscanf(buf, "%d\n", &mykobject01->value2);
    }
    return size;
}

// 自定义的sysfs_ops结构体，包含show和store函数指针
struct sysfs_ops myops = {
    .show = myshow,
    .store = mystore,
};

// 自定义的kobj_type结构体，包含释放函数、默认属性和sysfs_ops
static struct kobj_type mytype = {
    .release = dynamic_kobj_release,
    .default_attrs = myattr,
    .sysfs_ops = &myops,
};

// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;
    // 分配并初始化mykobject01
    mykobject01 = kzalloc(sizeof(struct mykobj), GFP_KERNEL);
    mykobject01->value1 = 1;
    mykobject01->value2 = 1;

    // 初始化并添加mykobject01到内核中，名为"mykobject01"
    ret = kobject_init_and_add(&mykobject01->kobj, &mytype, NULL, "%s", "mykobject01");

    return 0;
}

// 模块的退出函数
static void mykobj_exit(void)
{
    // 释放mykobject01
    kobject_put(&mykobject01->kobj);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
MODULE_AUTHOR("topeet"); // 模块的作者

MODULE_LICENSE("GPL");   // 模块使用的许可证
MODULE_AUTHOR("woniu"); // 模块的作者
```

### 优化属性文件读写函数

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/sysfs.h>

// 自定义的kobject结构体，包含一个kobject对象和两个整型值
struct mykobj
{
    struct kobject kobj;
    int value1;
    int value2;
};

// 定义了mykobj结构体指针变量mykobject01
struct mykobj *mykobject01;

// 自定义的show函数，用于读取属性值
ssize_t show_myvalue1(struct kobject *kobj, struct kobj_attribute *attr, char *buf)
{
    ssize_t count;
    count = sprintf(buf, "show_myvalue1\n");
    return count;
};

// 自定义的store函数，用于写入属性值
ssize_t store_myvalue1(struct kobject *kobj, struct kobj_attribute *attr, const char *buf, size_t count)
{
    printk("buf is %s\n", buf);
    return count;
};

// 自定义的show函数，用于读取属性值
ssize_t show_myvalue2(struct kobject *kobj, struct kobj_attribute *attr, char *buf)
{
    ssize_t count;
    count = sprintf(buf, "show_myvalue2\n");
    return count;
};

// 自定义的store函数，用于写入属性值
ssize_t store_myvalue2(struct kobject *kobj, struct kobj_attribute *attr, const char *buf, size_t count)
{
    printk("buf is %s\n", buf);
    return count;
};

// 定义attribute对象value1和value2
struct kobj_attribute value1 = __ATTR(value1, 0664, show_myvalue1, store_myvalue1);
struct kobj_attribute value2 = __ATTR(value2, 0664, show_myvalue2, store_myvalue2);

// 自定义的kobject释放函数
static void dynamic_kobj_release(struct kobject *kobj)
{
    struct mykobj *mykobject01 = container_of(kobj, struct mykobj, kobj);
    printk("kobject: (%p): %s\n", kobj, __func__);
    kfree(mykobject01);
}

// 将attribute对象放入数组中
struct attribute *myattr[] = {
    &value1.attr,
    &value2.attr,
    NULL,
};

// 自定义的show函数，用于读取属性值
ssize_t myshow(struct kobject *kobj, struct attribute *attr, char *buf)
{
    ssize_t count;
    struct kobj_attribute *kobj_attr = container_of(attr, struct kobj_attribute, attr);
    count = kobj_attr->show(kobj, kobj_attr, buf);
    return count;
}

// 自定义的store函数，用于写入属性值
ssize_t mystore(struct kobject *kobj, struct attribute *attr, const char *buf, size_t size)
{
    struct kobj_attribute *kobj_attr = container_of(attr, struct kobj_attribute, attr);
    return kobj_attr->store(kobj, kobj_attr, buf, size);
}

// 自定义的sysfs_ops结构体，包含show和store函数指针
struct sysfs_ops myops = {
    .show = myshow,
    .store = mystore,
};

// 自定义的kobj_type结构体，包含释放函数、默认属性和sysfs_ops
static struct kobj_type mytype = {
    .release = dynamic_kobj_release,
    .default_attrs = myattr,
    .sysfs_ops = &myops,
};

// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;

    // 分配并初始化mykobject01
    mykobject01 = kzalloc(sizeof(struct mykobj), GFP_KERNEL);
    mykobject01->value1 = 1;
    mykobject01->value2 = 1;

    // 初始化并添加mykobject01到内核中，名为"mykobject01"
    ret = kobject_init_and_add(&mykobject01->kobj, &mytype, NULL, "%s", "mykobject01");

    return 0;
}

// 模块的退出函数
static void mykobj_exit(void)
{
    // 释放mykobject01
    kobject_put(&mykobject01->kobj);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出/
MODULE_LICENSE("GPL");    // 模块使用的许可证
MODULE_AUTHOR("woniu");  // 模块的作者
```

### 创建属性文件并实现读写功能 2

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/sysfs.h>

// 定义了mykobj结构体指针变量mykobject01
struct kobject *mykobject01;

// 自定义的show函数，用于读取属性值
ssize_t show_myvalue1(struct kobject *kobj, struct kobj_attribute *attr, char *buf)
{
    ssize_t count;
    count = sprintf(buf, "show_myvalue1\n");
    return count;
};

// 自定义的store函数，用于写入属性值
ssize_t store_myvalue1(struct kobject *kobj, struct kobj_attribute *attr, const char *buf, size_t count)
{
    printk("buf is %s\n", buf);
    return count;
};

// 自定义的show函数，用于读取属性值
ssize_t show_myvalue2(struct kobject *kobj, struct kobj_attribute *attr, char *buf)
{
    ssize_t count;
    count = sprintf(buf, "show_myvalue2\n");
    return count;
};

// 自定义的store函数，用于写入属性值
ssize_t store_myvalue2(struct kobject *kobj, struct kobj_attribute *attr, const char *buf, size_t count)
{
    printk("buf is %s\n", buf);
    return count;
};

// 定义attribute对象value1和value2
struct kobj_attribute value1 = __ATTR(value1, 0664, show_myvalue1, store_myvalue1);
struct kobj_attribute value2 = __ATTR(value2, 0664, show_myvalue2, store_myvalue2);

// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;

    // // 分配并初始化mykobject01
    // mykobject01 = kzalloc(sizeof(struct mykobj), GFP_KERNEL);
    // mykobject01->value1 = 1;
    // mykobject01->value2 = 1;
    // // 初始化并添加mykobject01到内核中，名为"mykobject01"
    // ret = kobject_init_and_add(&mykobject01->kobj, &mytype, NULL, "%s", "mykobject01");

    mykobject01 = kobject_create_and_add("mykobject01", NULL);
    ret = sysfs_create_file(mykobject01, &value1.attr);
    ret = sysfs_create_file(mykobject01, &value2.attr);
    return ret;
}

// 模块的退出函数
static void mykobj_exit(void)
{
    // 释放mykobject01
    kobject_put(mykobject01);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出/
MODULE_LICENSE("GPL");    // 模块使用的许可证
MODULE_AUTHOR("woniu");  // 模块的作者
```

### 创建多个属性文件的简便方法

**sysfs_create_group 函数**

sysfs_create_group 函数，用于在 sysfs 中创建一个组(group)。组是一组相关的属性文件的集合，可以将它们放在同一个目录下提供更好的组织性和可读性。

> int sysfs_create_group**(**struct kobject *****kobj**,** const struct attribute_group *****grp**);**

此函数有俩个参数，分别为如下所示：

> kobj: 指向包含目标组的 kobject 的指针。
>
> grp: 指向 attribute_group 结构体的指针，该结构体定义了组中的属性文件。

attribute_group 结构体定义如下：

```c
struct attribute_group {
  const char *name;
  const struct attribute **attrs;
  mode_t (*is_visible)(struct kobject *kobj, struct attribute *attr, int index);
};
```

结构体中包含如下字段：

> name: 组的名称，将用作 sysfs 目录的名称。
>
> attrs: 指向属性文件数组的指针
>
> is_visible : 可选的回调函数，用于决定每个属性文件是否可见。如果为 NULL，则所有属性文件都可见。

属性文件数组是一个以 struct attribute *类型为元素的数组，以 NULL 结束。每个 structattribute 结构体表示一个属性文件，可以使用&运算符将属性对象（如 struct kobject_attribute）的.attr 字段传递给属性文件数组。

下面展示使用 sysfs_create_group 创建一个组并添加属性文件

```c
struct attribute *attrs[] = {
	&attr1.attr, 
    &attr2.attr, 
    NULL, 
};
const struct attribute_group attr_group = {
	.name = "my_group", 
    .attrs = attrs, 
};
sysfs_create_group(kobj, &attr_group);
```

在上述示例中，我们创建了一个名叫“my_group”的组，并将属性文件 attr1 和 attr2 添加到该组中，然后使用 sys_create_group 将该组添加到指定的 kobject kobj 中。

实验代码

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/sysfs.h>

// 定义了mykobj结构体指针变量mykobject01
struct kobject *mykobject01;

// 自定义的show函数，用于读取属性值
ssize_t show_myvalue1(struct kobject *kobj, struct kobj_attribute *attr, char *buf)
{
    ssize_t count;
    count = sprintf(buf, "show_myvalue1\n");
    return count;
};

// 自定义的store函数，用于写入属性值
ssize_t store_myvalue1(struct kobject *kobj, struct kobj_attribute *attr, const char *buf, size_t count)
{
    printk("buf is %s\n", buf);
    return count;
};

// 自定义的show函数，用于读取属性值
ssize_t show_myvalue2(struct kobject *kobj, struct kobj_attribute *attr, char *buf)
{
    ssize_t count;
    count = sprintf(buf, "show_myvalue2\n");
    return count;
};

// 自定义的store函数，用于写入属性值
ssize_t store_myvalue2(struct kobject *kobj, struct kobj_attribute *attr, const char *buf, size_t count)
{
    printk("buf is %s\n", buf);
    return count;
};

// 定义attribute对象value1和value2
struct kobj_attribute value1 = __ATTR(value1, 0664, show_myvalue1, store_myvalue1);
struct kobj_attribute value2 = __ATTR(value2, 0664, show_myvalue2, store_myvalue2);

struct attribute *attr[] = {
    &value1.attr,
    &value2.attr,
    NULL,
};

const struct attribute_group my_attr_group = {
    .name = "myattr",
    .attrs = attr,
};

// 模块的初始化函数
static int mykobj_init(void)
{
    int ret;

    // 创建并添加kobject "mykobject01"
    mykobject01 = kobject_create_and_add("mykobject01", NULL);

    // 在kobject "mykobject01" 中创建属性组
    ret = sysfs_create_group(mykobject01, &my_attr_group);

    return ret;
}

// 模块的退出函数
static void mykobj_exit(void)
{
    // 释放kobject "mykobject01"
    kobject_put(mykobject01);
}

module_init(mykobj_init); // 指定模块的初始化函数
module_exit(mykobj_exit); // 指定模块的退出函数
MODULE_LICENSE("GPL");    // 模块使用的许可证
MODULE_AUTHOR("woniu");  // 模块的作者
```

### 注册总线

我们进入开发板的/sys/bus 目录下，/sys/bus 是 Linux 系统中的一个目录，用于表示总线（bus）子系统的根目录。如果我们自己注册一个总线，会在此目录下显示

![image-20240715134312299](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715134312299.png)

bus_register 是一个函数，用于将一个自定义总线注册到 Linux 内核中。

函数原型如下：

> int bus_register**(**struct bus_type *****bus**);**

参数 bus 是一个指向 struct bus_type 结构体的指针，表示要注册的自定义总线。

函数返回一个整数值，表示注册操作的结果。通常情况下，返回值为 0 表示注册成功，负值表示注册失败。

bus_unregister 是一个函数，用于取消注册一个已经注册的自定义总线。

函数原型如下：

> void bus_unregister**(**struct bus_type *****bus**);**

参数 bus 是一个指向 struct bus_type 结构体的指针，表示要取消注册的自定义总线。该函数没有返回值。

代码

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/device.h>

int mybus_match(struct device *dev, struct device_driver *drv)
{
    return (strcmp(dev_name(dev), drv->name) == 0);
};

int mybus_probe(struct device *dev)
{
    struct device_driver *drv = dev->driver;
    if (drv->probe)
        drv->probe(dev);
    return 0;
};

struct bus_type mybus = {
    .name = "mybus",
    .match = mybus_match,
    .probe = mybus_probe,
};

// 模块的初始化函数
static int bus_init(void)
{
    int ret;
    ret = bus_register(&mybus);
    return 0;
}

// 模块退出函数
static void bus_exit(void)
{
    bus_unregister(&mybus);
}

module_init(bus_init); // 指定模块的初始化函数
module_exit(bus_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
MODULE_AUTHOR("woniu"); // 模块的作者
```

### 在总线目录下创建属性文件

bus_create_file() 函数用于在总线的 sysfs 目录下创建一个属性文件。

> int bus_create_file**(**struct bus_type *****bus**,** const struct attribute *****attr**);**

> 参数说明：
>
> bus：指向总线类型结构体 struct bus_type 的指针，表示要创建属性文件的总线。
>
> attr：指向属性 struct attribute 的指针，表示要创建的属性文件的属性。
>
> 返回值：成功时返回 0，否则返回负数错误代码。

在调用 bus_create_file()函数之前，需要先定义好属性结构体 struct attribute，并将其相关字段填充好。通常，属性结构体会包含以下字段：

> .name：属性的名称。
>
> .mode：属性的访问权限。

示例代码

```c
struct bus_attribute mybus_attr = {
	.attr = {
		.name = "value", 
        .mode = 0664, 
      },
  .show = mybus_show, 
};
ret = bus_create_file(&mybus, &mydevice.kobj, &mybus_attr.attr);
```

### 在总线下注册设备

总线

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/device.h>
#include <linux/sysfs.h>

int mybus_match(struct device *dev, struct device_driver *drv)
{
    // 检查设备名称和驱动程序名称是否匹配
    return (strcmp(dev_name(dev), drv->name) == 0);
};

int mybus_probe(struct device *dev)
{
    struct device_driver *drv = dev->driver;
    if (drv->probe)
        drv->probe(dev);
    return 0;
};

struct bus_type mybus = {
    .name = "mybus",                 // 总线的名称
    .match = mybus_match,            // 设备和驱动程序匹配的回调函数
    .probe = mybus_probe,            // 设备探测的回调函数
};
EXPORT_SYMBOL_GPL(mybus);             // 导出总线符号

ssize_t mybus_show(struct bus_type *bus, char *buf)
{
    // 在 sysfs 中显示总线的值
    return sprintf(buf, "%s\n", "mybus_show");
};

struct bus_attribute mybus_attr = {
    .attr = {
        .name = "value",             // 属性的名称
        .mode = 0664,                // 属性的访问权限
    },
    .show = mybus_show,               // 属性的 show 回调函数
};

// 模块的初始化函数
static int bus_init(void)
{
    int ret;
    ret = bus_register(&mybus);       // 注册总线
    ret = bus_create_file(&mybus, &mybus_attr);  // 在 sysfs 中创建属性文件

    return 0;
}

// 模块退出函数
static void bus_exit(void)
{
    bus_remove_file(&mybus, &mybus_attr);  // 从 sysfs 中移除属性文件
    bus_unregister(&mybus);                // 取消注册总线
}

module_init(bus_init);                    // 指定模块的初始化函数
module_exit(bus_exit);                    // 指定模块的退出函数

MODULE_LICENSE("GPL");                    // 模块使用的许可证
MODULE_AUTHOR("woniu");                  // 模块的作者
```

设备

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/device.h>
#include <linux/sysfs.h>

extern struct bus_type mybus;

void myrelease(struct device *dev)
{
    printk("This is myrelease\n");
};

struct device mydevice = {
    .init_name = "mydevice",      // 设备的初始化名称
    .bus = &mybus,                // 所属总线
    .release = myrelease,         // 设备的释放回调函数
    .devt = ((255 << 20 | 0)),    // 设备号
};

// 模块的初始化函数
static int device_init(void)
{
    int ret;
    ret = device_register(&mydevice);  // 注册设备

    return 0;
}

// 模块退出函数
static void device_exit(void)
{
    device_unregister(&mydevice);      // 取消注册设备
}

module_init(device_init);              // 指定模块的初始化函数
module_exit(device_exit);              // 指定模块的退出函数

MODULE_LICENSE("GPL");                 // 模块使用的许可证
MODULE_AUTHOR("woniu");               // 模块的作者
```

### 在总线下注册驱动

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/slab.h>
#include <linux/configfs.h>
#include <linux/kernel.h>
#include <linux/kobject.h>
#include <linux/device.h>
#include <linux/sysfs.h>

extern struct bus_type mybus;

int mydriver_remove(struct device *dev){
    printk("This is mydriver_remove\n");
    return 0;
};

int mydriver_probe(struct device *dev){
    printk("This is mydriver_probe\n");
    return 0;
};


struct device_driver mydriver = {
    .name = "mydevice",
    .bus = &mybus,
    .probe = mydriver_probe,
    .remove = mydriver_remove,
    
};

// 模块的初始化函数
static int mydriver_init(void)
{
    int ret;
    ret = driver_register(&mydriver);
    
    return ret;
}

// 模块退出函数
static void mydriver_exit(void)
{
  
    driver_unregister(&mydriver);
}

module_init(mydriver_init); // 指定模块的初始化函数
module_exit(mydriver_exit); // 指定模块的退出函数

MODULE_LICENSE("GPL");   // 模块使用的许可证
MODULE_AUTHOR("woniu"); // 模块的作者
```

### 实例分析
