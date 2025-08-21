## 4-2-5-pinctrl&gpio&can&usb

### pinmux 介绍

Pinmux（引脚复用）是指在系统中配置和管理引脚功能的过程。在许多现代集成电路中，单个引脚可以具有多个功能，例如作为 GPIO、UART、SPI 或 I2C 等。通过使用引脚复用功能，可以在这些不同的功能之间切换。

引脚复用通过硬件和软件的方式实现。硬件层面，芯片设计会为每个引脚提供多个功能的选择。这些功能通常由芯片厂商在芯片规格文档中定义。通过编程设置寄存器或开关，可以选择某个功能来连接引脚。这种硬件层面的配置通常是由引脚控制器（Pin Controller）或引脚复用控制器（Pin Mux Controller）负责管理。

软件层面，操作系统或设备驱动程序需要了解和配置引脚的功能。它们使用设备树（Device Tree）或设备树绑定（Device Tree Bindings）来描述和配置引脚的功能。在设备树中，可以指定引脚的复用功能，将其连接到特定的硬件接口或功能。操作系统或设备驱动程序在启动过程中解析设备树，并根据配置对引脚进行初始化和设置。

### pinctrl 子系统

Linux 中的 pinctrl 子系统（Pin Control Subsystem）是一个用于管理和配置通用输入/输出（GPIO）引脚的框架。它提供了一种标准化的方法，以在 Linux 内核中对 GPIO 引脚进行配置、分配和控制，从而适应不同的硬件平台和设备。

pinctrl 子系统也符合 Linux 内核的设备模型规范，所以 pinctrl 子系统同样可以根据设备模型规范分为设备、驱动、总线和类四个部分，本章节将从设备和驱动两个部分来引入 pinctrl 子系统。

### 使用 pinctrl 设置复用关系

pinctrl（引脚控制）用于描述和配置硬件设备上的引脚功能和连接方式。它是设备树的一部分，用于在启动过程中传递引脚配置信息给操作系统和设备驱动程序，以便正确地初始化和控制引脚。

接下来将使用三个例子对客户端要用到的属性进行讲解。

例 1：

> node **{**
>
> ​	pinctrl**-**names **=** "default"**;**
>
> ​	pinctrl**-**0 **= <&**pinctrl_hog_1**>;**
>
> }

在例 1 中，pinctrl-names 属性定义了一个状态名称：default。

pinctrl-0 属性指定了第一个状态 default 对应的引脚配置。

<&pinctrl_hog_1> 是一个引脚描述符，它引用了一个名为 pinctrl_hog_1 的引脚控制器节点。这表示在 default 状态下，设备的引脚配置将使用 pinctrl_hog_1 节点中定义的配置。

例 2：

> node **{**
>
> ​		pinctrl**-**names **=** "default"**,** "wake up"**;**
>
> ​		pinctrl**-**0 **= <&**pinctrl_hog_1**>;**
>
> ​		pinctrl**-**1 **= <&**pinctrl_hog_2**>;**
>
> **}**

在例 2 中，pinctrl-names 属性定义了两个状态名称：default 和 wake up。

pinctrl-0 属性指定了第一个状态 default 对应的引脚配置，引用了 pinctrl_hog_1 节点。

pinctrl-1 属性指定了第二个状态 wake up 对应的引脚配置，引用了 pinctrl_hog_2 节点。

这意味着设备可以处于两个不同的状态之一，每个状态分别使用不同的引脚配置。

例 3：

> node **{**
>
> ​	pinctrl**-**names **=** "default"**;**
>
> ​	pinctrl**-**0 **= <&**pinctrl_hog_1 **&**pinctrl_hog_2**>;**
>
> **}**

在这个例子中，pinctrl-names 属性仍然定义了一个状态名称：default。

pinctrl-0 属性指定了第一个状态 default 对应的引脚配置，但与之前的例子不同的是，它引用了两个引脚描述符：pinctrl_hog_1 和 pinctrl_hog_2。

这表示在 default 状态下，设备的引脚配置将使用 pinctrl_hog_1 和 pinctrl_hog_2 两个节点中定义的配置。这种方式可以将多个引脚控制器的配置组合在一起，以满足特定状态下的引脚需求。

### pinctl 服务端

服务端是设备树中定义引脚配置的部分。它包含引脚组和引脚描述符，为客户端提供引脚配置选择。服务端在设备树中定义了 pinctrl 节点，其中包含引脚组和引脚描述符的定义。

这里以瑞芯微的 RK3568 为例进行 pinctrl 服务端的讲解，瑞芯微原厂 BSP 工程师为了方便用户通过 pinctrl 设置管脚的复用关系，将包含所有复用关系的配置写在了内核目录下的“arch/arm64/boot/dts/rockchip/rk3568-pinctrl.dtsi”设备树中，具体内容如下

![image-20240715204535145](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715204535145.png)

在 pinctrl 节点中就是每个节点的复用功能，然后我们以 uart4 的引脚复用为例进行讲解，uart4 的 pinctrl 服务端内容如下

![image-20240715204549419](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715204549419.png)

其中<3 RK_PB1 4 &pcfg_pull_up>和<3 RK_PB2 4 &pcfg_pull_up>分别表示将 GPIO3 的 PB1 引脚设置为功能 4，将 GPIO3 的 PB2 也设置为功能 4，且电器属性都会设置为上拉。

最后来看客户端对 uart4 服务端的引用，具体内容在内核源码目录“arch/arm64/boot/dts/rockchip/rk3568-evb1-ddr4-v10-linux.dts”

![image-20240715204733566](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715204733566.png)

通过在客户端中引用服务端的引脚描述符，设备树可以将客户端和服务端的引脚配置关联起来。这样，在设备树被解析和处理时，操作系统和设备驱动程序可以根据客户端的需求，查找并应用适当的引脚配置。

### sysfs 文件系统控制 GPIO

GPIO 软件编程方式有多种，可以写驱动程序调用 GPIO 函数操作 GPIO，也可以直接通过操作寄存器的方式操作 GPIO，还可以通过 sysfs 方式实现对 GPIO 的控制。本章节我们来学习使用 sysfs 方式实现对 GPIO 的控制。

使用 sysfs 方式控制 gpio，首先需要底层驱动的支持，需要在 make menuconfig 图形化配置

界面中加入以下配置：

> Device Drivers
>
> ->GPIO Support
>
> ->/sys/class/gpio/xxxx

![image-20240715205459223](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715205459223.png)

sysfs 控制接口为/sys/class/gpio/export 和/sys/class/gpio/unexport。

![image-20240715205522041](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715205522041.png)

/sys/class/gpio/export 用 于 将 GPIO 控 制 从 内 核 空 间 导 出 到 用 户 空 间 。/sys/class/gpio/unexport 用于取消 GPIO 控制从内核空间到用户空间的导出。 export 和unexport，他们都是只写的。GpiochipX 代表 GPIO 控制器。

**export**：用于将指定编号的 GPIO 引脚导出。在使用 GPIO 引脚之前，需要将其导出，导出成功之后才能使用它。注意 export 文件是只写文件，不能读取，将一个指定的编号写入到 export 文件中即可将对应的 GPIO 引脚导出，以 GPIO0_PB7 为例（pin 计算值为 15）使用 export 文件进行导出(如果没有更换本章开始部分的内核设备树镜像，会导出不成功)，导出成功如下图所示：

![image-20240715205556494](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715205556494.png)

会发现在/sys/class/gpio 目录下生成了一个名为 gpio15 的文件夹（gpioX，X 表示对应的编 号），该文件夹就是导出来的 GPIO 引脚对应的文件夹，用于管理、控制该 GPIO 引脚。

**unexport**：将导出的 GPIO 引脚删除。当使用完 GPIO 引脚之后，需要将导出的引脚删除，同样该文件也是只写文件、不可读，使用 unexport 文件进行删除 GPIO0_PB7，删除成功如下图所示：

![image-20240715205623971](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715205623971.png)

可以看到之前生成的 gpio15 文件夹就会消失！

**需要注意的是，并不是所有 GPIO 引脚都可以成功导出，如果对应的 GPIO 已经被导出或者在内核中被使用了，那便无法成功导出，**导出失败如下图所示：

![image-20240715205646887](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715205646887.png)

出现上图报错的原因是该 GPIO 已经被其他 GPIO 使用，需要在内核中找到使用 GPIO 的驱动，并取消该驱动才可以正常使用 GPIO。在使用 GPIO15 时，需要取消 Linux 内核源码中LED 灯的配置，如下所示：

![image-20240715205705709](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715205705709.png)

再次使用以下命令导出 GPIO0_PB7 引脚，导出成功之后进入 gpio15 文件夹

可以看到 gpio15 文件夹下分别有 active_low、device、direction、edge、power、subsystem、uevent、value 八个文件，需要关心的文件是 active_low、direction、edge 以及 value 这四个属性文件，接下来分别介绍这四个属性文件的作用：

**direction**：配置 GPIO 引脚为输入或输出模式。该文件可读、可写，读表示查看 GPIO 当前是输入还是输出模式，写表示将 GPIO 配置为输入或输出模式；读取或写入操作可取的值为"out"（输出模式）和"in"（输入模式）。

在“/sys/class/gpio/gpio15”目录下使用 cat 命令查看 direction 输入输出模式

默认状态下的输入输出状态为“in”,由于 direction 为可读可写，可以使用以下命令将模式

配置为输出，配置完成如下

> echo out > direction
>
> cat direction

**active_low**：用于控制极性得属性文件，可读可写，默认情况下为 0，使用 cat 命令进行文件内容的查看，如下图

>  cat active_low

当 active_low 等于 0 时， value 值若为 1 则引脚输出高电平，value 值若为 0 则引脚输出低电平。当 active_low 等于 1 时 ，value 值若为 0 则引脚输出高电平，value 值若为 1 则引脚输出低电平。

**edge**：控制中断的触发模式，该文件可读可写。在配置 GPIO 引脚的中断触发模式之前，需将其设置为输入模式，四种触发模式的设置如下

> 非中断引脚：**echo** "none" **>** edge
>
> 上升沿触发：**echo** "rising" **>** edge
>
> 下降沿触发：**echo** "falling" **>** edge
>
> 边沿触发： **echo** "both" **>** edge

**value:** 设置高低电平，如果我们要把这个管脚设置成高电平，我们只需要给 value 设置成 1即可，反之，则设置成 0。

### GPIO 子系统 API 函数控制GPIO

在目前的 Linux 内核主线中，GPIO（通用输入/输出）子系统存在两个版本，这里将两个版本区分为新版本和旧版本。新版本 GPIO 子系统接口是基于描述符（descriptor-based）来实现的，而旧版本的 GPIO 子系统接口是基于整数（integer-based）来实现的，在 Linux 内核中为了保持向下的兼容性，旧版本的接口在最新的内核版本中仍然得到支持，而随着时间的推移，新版本的 GPIO 子系统接口会越来越完善，最终完全取代旧版本，所以在本课程中主要讲解新版本的 GPIO 子系统接口。

新的 GPIO 子系统接口需要与与设备树（Device Tree）结合使用。使用设备树和新的 GPIO接口可以更加灵活地配置和管理系统中的 GPIO 资源，提供了更好的可扩展性和可移植性。所以如果没有设备树，就无法使用新的 GPIO 接口。

那要如何对新旧 GPIO 子系统接口进行区分呢，一个明显的区别是新的 GPIO 子系统接口使用了以 "gpiod_"作为前缀的函数命名约定，而旧的 GPIO 子系统接口使用了以"gpio_"作为前缀的函数命名约定。

新的 GPIO 子系统接口是基于描述符（descriptor-based）来实现的，由 struct gpio_desc 结构体来表示，该结构体定义在内核源码的“drivers/gpio/gpiolib.h”目录下。

```shell
struct gpio_desc {
  struct gpio_device gdev; // GPIO 设备结构体
  unsigned long flags; // 标志位，用于表示不同的属性
  /* 标志位符号对应的位号 */
  #define FLAG_REQUESTED 0 // GPIO 已请求
  #define FLAG_IS_OUT 1 // GPIO 用作输出
  #define FLAG_EXPORT 2 // 受 sysfs_lock 保护的导出标志
  #define FLAG_SYSFS 3 // 通过/sys/class/gpio/control 导出的标志
  #define FLAG_ACTIVE_LOW 6 // GPIO 值为低电平时激活
  #define FLAG_OPEN_DRAIN 7 // GPIO 为开漏类型
  #define FLAG_OPEN_SOURCE 8 // GPIO 为开源类型
  #define FLAG_USED_AS_IRQ 9 // GPIO 连接到中断请求（IRQ）
  #define FLAG_IS_HOGGED 11 // GPIO 被独占占用
  #define FLAG_TRANSITORY 12 // GPIO 在休眠或复位时可能失去值
  /* 连接标签 */
  const char *label; // GPIO 的名称
  const char *name; // GPIO 的名称
};
```

> （1）struct gpio_device gdev 是一个 struct gpio_device 类型的字段，用于表示 GPIO 设备的相关信息。struct gpio_device 结构体通常包含设备的底层硬件控制器等信息。
>
> （2）unsigned long flags:用于表示 GPIO 的标志位，以表示不同的属性。通过使用位操作，每个标志位可以单独设置或清除。
>
> （3）第 5-14 行用于表示不同标志位的符号常量，与 flags 字段中的位号相对应。例如，在 flags 字段中的第 0 位表示FLAG_REQUESTED，第 1 位表示 FLAG_IS_OUT，以此类推
>
> （4）const char *label: 这是一个指向字符串的指针，表示 GPIO 的标签或名称。
>
> （5）const char *name: 这是一个指向字符串的指针，表示 GPIO 的名称

上面需要重点关注的是然后 struct gpio_device 结构体，该结构体同样定义在内核源码的“drivers/gpio/gpiolib.h”目录下，具体内容如下所示：

```c
struct gpio_device {
  int id; // GPIO 设备 ID
  struct device *dev; // 对应的设备结构体指针
  struct cdev chrdev; // 字符设备结构体
  struct device *mockdev; // 模拟设备结构体指针
  struct module *owner; // 拥有该 GPIO 设备的内核模块指针
  struct gpio_chip *chip; // 对应的 GPIO 芯片结构体指针
   struct gpio_desc *descs; // GPIO 描述符数组指针
  int base; // GPIO 编号的起始值
  u16 ngpio; // GPIO 的数量
  const char *label; // GPIO 设备的标签
  void *data; // 与 GPIO 设备相关的数据指针
  struct list_head list; // 用于将 GPIO 设备结构体连接到链表中
#ifdef CONFIG_PINCTRL
  /*
  * 如果启用了 CONFIG_PINCTRL 选项，GPIO 控制器可以选择描述它们在 SoC 中服务的实际引脚范围。
  * 此信息将由 pinctrl 子系统用于配置相应的 GPIO 引脚。
  */
  struct list_head pin_ranges; // 描述 GPIO 控制器引脚范围的链表
#endif
};
```

该结构体是用来描述 GPIO 设备的数据结构，关于该结构体的参数介绍如下所示：

> （1）int id:整型字段，表示 GPIO 设备的 ID。每个 GPIO 设备可以有一个唯一的 ID。
>
> （2）struct device *dev:指向 struct device 的指针，表示与 GPIO 设备相关联的设备结构体。
>
> （3）struct cdev chrdev: 字符设备结构体，用于实现 GPIO 设备的字符设备接口。
>
> （4）struct device *mockdev: 指向 struct device 的指针，用于表示 GPIO 设备的模拟设备结构体。
>
> （5）struct module *owner: 指向拥有该 GPIO 设备的内核模块的指针。
>
> （6）struct gpio_chip *chip: 指向 struct gpio_chip 的指针，表示与 GPIO 设备关联的 GPIO 芯片（GPIO 控制器）结构体。
>
> （7）struct gpio_desc *descs: 指向 struct gpio_desc 数组的指针，表示与 GPIO 设备关联的 GPIO 描述符。每个 GPIO 描述符用于描述 GPIO 的属性和状态。
>
> （8）int base: 表示 GPIO 编号的起始值。
>
> （9）u16 ngpio: 16 位无符号整型字段，表示 GPIO 的数量。
>
> （10）const char *label: 指向字符串的指针，表示 GPIO 设备的标签或名称。
>
> （11）void *data: 指向与 GPIO 设备相关的数据的指针，用于存储和访问与 GPIO 设备相关的自定义数据。
>
> （12）struct list_head list: 将 GPIO 设备结构体连接到链表中的字段，用于管理多个 GPIO 设备的列表。
>
> （13）struct list_head pin_ranges (仅在启用 CONFIG_PINCTRL 选项时存在): 用于描述 GPIO 控制器引脚范围的链表。如果配置了 GPIO 控制器的引脚范围，该链表将包含描述每个范围的元素。

在上面一系列的参数中，要重点关注的是 struct gpio_chip *chip 这一结构体，表示与 GPIO 设备关联的 GPIO 芯片（GPIO 控制器）结构体，该结构体定义在内核源码目录下的“include/linux/gpio/driver.h”文件中，

```c
struct gpio_chip {
  const char *label; // GPIO 芯片标签
  struct gpio_device gpiodev; // GPIO 设备
  struct device *parent; // 父设备指针
  struct module *owner; // 拥有者模块指针
  int (*request)(struct gpio_chip *chip, unsigned offset);// 请求 GPIO
  void (*free)(struct gpio_chip *chip, unsigned offset); // 释放 GPIO
  int (*get_direction)(struct gpio_chip *chip, unsigned offset); // 获取 GPIO 方向
  int (*direction_input)(struct gpio_chip *chip, unsigned offset); // 设置 GPIO 为输入
  int (*direction_output)(struct gpio_chip *chip, unsigned offset, int value); // 设置 GPIO 为输出
  int (*get)(struct gpio_chip chip, unsigned offset); // 获取 GPIO 值
  int (*get_multiple)(struct gpio_chip chip, unsigned long *mask, unsigned long *bits); // 获取多个 GPIO 的值
  void (*set)(struct gpio_chip chip, unsigned offset, int value); // 设置 GPIO 值
  void (*set_multiple)(struct gpio_chip chip, unsigned long mask, unsigned long *bits); // 设置多个 GPIO的值
  int (*set_config)(struct gpio_chip *chip, unsigned offset, unsigned long config); // 设置 GPIO 配置
  int (*to_irq)(struct gpio_chip chip, unsigned offset); // 将 GPIO 转换为中断
  void (*dbg_show)(struct seq_file *s, struct gpio_chip chip); // 在调试信息中显示 GPIO
  int base; // GPIO 编号的基准值
  u16 ngpio; // GPIO 的数量
  const char *const *names; // GPIO 的名称数组
	.......... 
};
```

struct gpio_chip *chip 这一结构体用于描述 GPIO 芯片的属性和操作函数，可以通过函数指针调用相应的函数来请求、释放、设置、获取 GPIO 的状态和数值等操作，从而实现对 GPIO 的控制和管理，需要注意的是这个结构体中的一系列函数都不需要我们来填充，这些工作都是由芯片原厂工程师来完成的，我们只需要学会新 gpio 子系统相应 API 函数的使用即可。

#### 相关函数

**获取** **GPIO** **描述符**

struct gpio_desc *gpiod_get 是 Linux 内核中用于获取 GPIO 描述符的函数。下面是对该函数的详细介绍：

> **函数原型：**
>
> struct gpio_desc *__must_check gpiod_get(struct device *dev,const char *con_id,enum gpiod_flags flags);
>
> **头文件：**
>
> \#include <linux/gpio/consumer.h>
>
> **参数：**
>
> dev：指向设备结构体的指针，表示与 GPIO 相关联的设备。
>
> con_id：连接标识符（connection identifier），用于标识所需的 GPIO 连接。通常由设备树（Device Tree）或其他设备描述信息定义
>
> flags：GPIO 描述符的选项标志，用于指定 GPIO 的属性和操作模式。以下是一些常用的选项标志（enum gpiod_flags）：
>
> GPIOF_INPUT：将 GPIO 配置为输入模式。
>
> GPIOF_OUTPUT：将 GPIO 配置为输出模式。
>
> GPIOF_ACTIVE_LOW：指示 GPIO 的默认电平为低电平（激活低电平）。
>
> GPIOF_OPEN_DRAIN：将 GPIO 配置为开漏输出模式。
>
> GPIOF_OPEN_SOURCE：将 GPIO 配置为开源输出模式。
>
> **函数功能：**
>
> 获取与给定设备和连接标识符（con_id）相关联的 GPIO 描述符。
>
> **返回值：**
>
> 如果成功获取到 GPIO 描述符，则返回指向 struct gpio_desc 的指针；如果获取失败，则返回 NULL。

在 Linux 内核中还有另外三个同样是获取 GPIO 描述符资源的函数，三个函数内容如下所示：

> struct gpio_desc *****gpiod_get_index**(**struct device *****dev**,** const char *****con_id**,** unsigned int idx**,** enum gpiod_flags flags**);**
>
> struct gpio_desc *****gpiod_get_optional**(**struct device *****dev**,** const char *****con_id**,** enum gpiod_flags flags**);**
>
> struct gpio_desc *****gpiod_get_index_optional**(**struct device *****dev**,** const char *****con_id**,** unsigned int index**,** enum gpiod_flags flags**);**

相较于上面介绍的 gpiod_get 函数，下面的三个函数可能会多一个 index 参数和 optional的函数后缀，其中 index 表示 GPIO 的索引值，当设备树的 GPIO 属性值包含多个 GPIO 引脚描述时，使用 index 来表示每个 GPIO 引脚的唯一标识。而带 optional() 后缀的函数与不带 optional 后缀的函数在功能上是相同的，都用于获取 GPIO 描述符，两者的区别在于返回值的不同：

使用带 optional() 的函数时，如果获取失败，返回值为 NULL。

使用不带 optional 的函数时，如果获取失败，返回值是一个特殊的结构表示获取 GPIO 描述符失败。

**释放** **GPIO** **描述符**

gpiod_put() 函数是 Linux 内核中用于释放 GPIO 描述符资源的函数。下面是对该函数的详细介绍：

> **函数原型：**
>
> void gpiod_put(struct gpio_desc *desc);
>
> **头文件：**
>
> \#include <linux/gpio/consumer.h>
>
> **参数：**
>
> desc：指向要释放的 GPIO 描述符的指针。
>
> **功能：**
>
> gpiod_put() 函数用于释放之前通过 gpiod_get() 或类似函数获取的 GPIO 描述符。
>
> **返回值：**
>
> 无返回值。

#### 实验

新版本的 gpio 子系统 api 接口要与设备树结合才能使用，所以需要在设备树中将用于获取 GPIO 描述符的引脚复用为 GPIO 模式。为了让教程更贴近于实战，这里选择 RK3568 开发板背面 20Pin GPIO 座子的 1 号引脚，右边对应的丝印为 I2C3_SDA_M0，这里的丝印表示该引脚可以复用为 I2C3 的 SDA 功能，而在当前的设备树源码中这个引脚是没有任何复用的，该引脚的具体位置如下所示：

![image-20240715211133757](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715211133757.png)

在前面 pinctrl 的章节中已经学习了如何将一个管脚复用为 GPIO，首先我们需要确定该引脚的 GPIO 编号，来到 RK3568 开发板的底板原理图，找到 J39 GPIO 底座，如下图所示：

![image-20240715211156822](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715211156822.png)

可以看到 1 号管脚的网络标号为 I2C3_SDA_M0，然后打开核心板原理图，根据这个网络标号进行搜索，查找到的核心板内容如下所示：

![image-20240715211216804](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715211216804.png)

左侧为该引脚的一些复用功能，箭头指向的部分为接下来要用到的 GPIO 引脚编号 GPIO1_ A0，然后对设备树进行内容的添加，从而将该引脚复用为 GPIO 的功能。

首先根据上图中的复用功能查看设备树中是否已经对该引脚进行了复用，在确保该引脚无任何复用之后，对 rk3568-evb1-ddr4-v10.dtsi 设备树进行内容的添加，在根节点的结尾添加以下内容

```
my_gpio:gpiol_a0 {
  compatible = "mygpio";
  my-gpios = <&gpio1 RK_PA0 GPIO_ACTIVE_HIGH>;
  pinctrl-names = "default";
  pinctrl-0 = <&my_gpio_ctrl>;
};
```

compatible: 用于指定设备的兼容性字符串，与驱动程序中的值相匹配。

my-gpios: 指定了与该设备相关联的 GPIO。&gpio表示 GPIO 控制器的句柄（handle），RK_PA0 是与该 GPIO 相关的资源描述符（resource specifier），GPIO_ACTIVE_HIGH 表示 GPIO 的默认电平为高电平。

pinctrl-names 和 pinctrl-0: 用于指定引脚控制器（pinctrl）的配置。pinctrl-names 表示引脚控制器配置的名称，这里为 "default"。pinctrl-0 指定了与该配置相关联的引脚控制器句柄，这里为 &mygpio_ctrl。

然后找到 pinctrl 节点，在节点尾部添加以下内容，

```
mygpio {
  my_gpio_ctrl: my-gpio-ctrl {
  	rockchip,pins = <1 RK_PA0 RK_FUNC_GPIO &pcfg_pull_none>;
  };
};
```

在第三行的内容中，1 表示引脚索引，RK_PA0 表示资源描述符，用于标识与该引脚相关联的物理资源，表示引脚所属的功能组，RK _FUNC_GPI0 表示将引脚的功能设置为 GPIO，&pcfg_pull_none 表示引脚配置为无上下拉。

至此，关于设备树相关的修改就完成了，保存退出之后，编译内核，然后将生成的 boot.img镜像烧写到开发板上即可。

驱动代码

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/gpio/consumer.h>

struct gpio_desc *mygpio1;  // GPIO 描述符指针
struct gpio_desc *mygpio2;  // GPIO 描述符指针
int num;  // GPIO 编号

//平台设备初始化函数
// 平台设备初始化函数
static int my_platform_probe(struct platform_device *dev)
{
    printk("This is my_platform_probe\n");

    // 获取可选的GPIO描述符
    mygpio1 = gpiod_get_optional(&dev->dev, "my", 0);
    if (mygpio1 == NULL) {
        printk("gpiod_get_optional error\n");
        return -1;
    }
    num = desc_to_gpio(mygpio1);
    printk("num is %d\n", num);

    // 释放GPIO描述符
    gpiod_put(mygpio1);

    // 获取指定索引的GPIO描述符
    mygpio2 = gpiod_get_index(&dev->dev, "my", 0, 0);
    if (IS_ERR(mygpio2)) {
        printk("gpiod_get_index error\n");
        return -2;
    }
    num = desc_to_gpio(mygpio2);
    printk("num is %d\n", num);

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
	{.compatible="mygpio"},
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
	gpiod_put(mygpio2);
    platform_driver_unregister(&my_platform_driver);

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

#### GPIO 操作函数

**获取** **GPIO** **的方向**

> （1）函数原型：
> int gpiod_get_direction(struct gpio_desc *desc);
> （2）头文件：
> #include <linux/gpio/consumer.h>
> （3）参数：
> desc：指向 GPIO 描述符的指针。
> （4）函数功能：
> gpiod_get_direction 函数用于获取 GPIO 的方向，即判断 GPIO 是输入还是输出。
> （5）返回值：
> 返回值为整型，表示 GPIO 的方向。如果成功获取到 GPIO 方向，返回值为 GPIO_LINE_DIRECTION_IN（0）表示输入，或 GPIO_LINE_DIRECTION_OUT（1）表示输出。如果获取失败，返回值为负数，表示错误码。

该函数的作用是获取给定 GPIO 描述符所代表的 GPIO 的方向。通过该函数，可以确定 GPIO是配置为输入还是输出。返回值可以用于进一步判断和处理 GPIO 的方向相关逻辑。

**配置** **GPIO** **的方向**

> （1）函数原型：
> int gpiod_direction_input(struct gpio_desc *desc);
> int gpiod_direction_output(struct gpio_desc *desc, int value);
> （2）头文件：
> #include <linux/gpio/consumer.h>
> （3）参数：
> desc：指向 GPIO 描述符的指针。
> value（仅适用于 gpiod_direction_output）：初始输出值，可以是 0 或 1。
> （4）函数功能：
> gpiod_direction_input 函数用于配置 GPIO 的方向为输入。
> gpiod_direction_output 函数用于配置 GPIO 的方向为输出，并可指定初始输出值。
> （5）返回值：
>
> 返回值为整型，表示配置 GPIO 方向的结果。
>
> 如果成功配置 GPIO 方向，返回值为 0。
>
> 如果配置失败，返回值为负数，表示错误码。

这两个函数用于配置 GPIO 的方向。gpiod_direction_input 将给定的 GPIO 描述符所代表的GPIO 配置为输入模式。而 gpiod_direction_output 将 GPIO 配置为输出模式，并可以指定初始输出值。

**读取** **GPIO** **的电平状态**

> （1）函数原型：
> int gpiod_get_value(const struct gpio_desc *desc);
> （2）头文件：
> #include <linux/gpio/consumer.h>
> （3）参数：
> desc：指向 GPIO 描述符的指针。
> （4）函数功能：
> gpiod_get_value 函数用于读取 GPIO 的电平状态。
> （5）返回值：
> 返回值为整型，表示 GPIO 的电平状态。
> 如果成功读取到 GPIO 的电平状态，返回值为 0 或 1，分别表示低电平和高电平。
> 如果读取失败，返回值为负数，表示错误码。

该函数用于读取给定 GPIO 描述符所代表的 GPIO 的电平状态。通过调用该函数，可以获取 GPIO 当前的电平状态，以便进一步处理和判断 GPIO 的状态。

**设置** **GPIO** **的电平状态**

> （1）函数原型：
> void gpiod_set_value(struct gpio_desc *desc, int value);
> （2）头文件：
> #include <linux/gpio/consumer.h>
> （3）参数：
> desc：指向 GPIO 描述符的指针。
> value：要设置的 GPIO 的电平状态，可以是 0 或 1。
> （4）函数功能：
> gpiod_set_value 函数用于设置 GPIO 的电平状态。
> （5）返回值：无（void）
> 该函数用于设置给定 GPIO 描述符所代表的 GPIO 的电平状态。通过调用该函数，您可以将 GPIO设置为特定的电平状态，以便控制外部设备或执行其他相关操作

value 参数表示要设置的 GPIO 的电平状态，可以是 0 或 1。当 value 为 0 时，表示设置GPIO 为低电平；当 value 为 1 时，表示设置 GPIO 为高电平。该函数没有返回值，因为它只是执行设置操作而不需要返回任何结果。在使用该函数之前，需要确保 GPIO 已经被正确地配置为输出模式。

**将** **GPIO** **描述符转换为中断编号**

> （1）函数原型：
> int gpiod_to_irq(const struct gpio_desc *desc);
> （2）头文件：
> #include <linux/gpio/consumer.h>
> （3）参数：
> desc：指向 GPIO 描述符的指针。
> （4）函数功能：
> gpiod_to_irq 函数用于将 GPIO 描述符转换为中断号。
> （5）返回值：
> 返回值为整型，表示中断号。
> 如果成功将 GPIO 描述符转换为中断号，返回值为大于等于 0 的中断号。
> 如果转换失败，返回值为负数，表示错误码。

该函数用于将给定 GPIO 描述符所代表的 GPIO 转换为对应的中断号。

#### 实验

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/mod_devicetable.h>
#include <linux/gpio/consumer.h>

struct gpio_desc *mygpio1;  // GPIO 描述符指针
struct gpio_desc *mygpio2;  // GPIO 描述符指针
int num;  // GPIO 编号

//平台设备初始化函数
static int my_platform_probe(struct platform_device *dev)
{
    printk("This is mydriver_probe\n");

    mygpio1 = gpiod_get_optional(&dev->dev, "my", 0);
    if (mygpio1 == NULL) {
        printk("gpiod_get_optional error\n");
        return -1;
    }
    num = desc_to_gpio(mygpio1);
    printk("num is %d\n", num);

    gpiod_put(mygpio1);

    mygpio2 = gpiod_get_index(&dev->dev, "my", 0, 0);
    if (IS_ERR(mygpio2)) {
        printk("gpiod_get_index error\n");
        return -2;
    }
    num = desc_to_gpio(mygpio2);
    printk("num is %d\n", num);

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
	{.compatible="mygpio"},
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
	gpiod_put(mygpio2);
    platform_driver_unregister(&my_platform_driver);

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");
}

module_init(my_platform_driver_init);
module_exit(my_platform_driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("woniu");
```

### I2C子系统

可以将整个 I2C 子系统用下面的框图来描述：

![image-20240715212345960](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715212345960.png)

可以将上面这一 I2C 子系统划分为三个层次，分别为用户空间、内核空间和硬件层，内核空间就包括 I2C 设备驱动层、I2C 核心层和 I2C 适配器驱动层，而本章的主要内容就是介绍 I2C子系统框架中的内核空间。

#### I2C 设备驱动层

I2C 设备驱动层的主要作用为编写驱动程序,使 I2C 外设能够正常工作，然后创建了对应的设备节点，提供了标准化的接口,使得上层应用程序能够方便地与 I2C 设备进行交互。

具体来说,I2C 设备驱动层包含以下几个关键部分:

（1）i2c_client 

·代表一个连接到 I2C 总线上的从设备

·包含从设备的地址、所属的 I2C 适配器等信息

（2）/dev/i2X 设备节点

·为上层应用程序提供设备访问的接口

·通过打开/读写/控制设备节点,应用程序可以与 I2C 设备进行交互

·内核 I2C 子系统负责将应用程序的操作转发到对应的 i2c_driver

（3）i2c_driver 

·实现了具体 I2C 从设备的驱动程序

·负责设备的初始化、读写、配置等操作

·通过 i2c_client 与设备进行交互

·向上层提供设备访问的标准化接口

（4）I2C 总线子系统

·管理整个 I2C 总线,包括注册/注销 I2C 适配器和从设备

·协调 i2c_client 和 i2c_driver 之间的交互

·为上层提供统一的 I2C 访问接口

#### I2C 适配器驱动层

I2C 适配器驱动层是 I2C 子系统的另一个重要组成部分,它负责实现具体的 I2C 硬件控制器

的驱动程序。I2C 适配器驱动程序的作用如下所示：

·提供标准化的 I2C 传输接口,供 I2C 核心层调用

·实现 I2C 总线协议的时序控制和数据收发

·管理 I2C 总线上的从设备

·处理 I2C 总线错误和异常情况

#### I2C 核心层

I2C 核心层位于 I2C 设备驱动层和 I2C 适配器驱动层中间，起到了承上启下的作用，负责 I2C设备驱动层和 I2C 适配器驱动层之间数据的传递，I2C 核心层的主要函数为 i2c_master_send、i2c_master_recv 和 i2c_transfer，其中 i2c_master_send 和 i2c_master_recv 函数，是 I2C 核心层提供的基本读写接口。

i2c_master_send 用于向 I2C 从设备发送数据,i2c_master_recv 用于从从设备接收数据。

它们分别接受如下参数:

（1）struct i2c_client *client: 指向目标 I2C 从设备的指针

（2）const char *buf/char *buf: 数据缓冲区

（3）int count: 要发送/接收的字节数

这两个函数负责生成符合 I2C 协议的时序和数据帧,并通过对应的 I2C 适配器驱动程序进行实际的总线操作

而 i2c_transfer 函数是一个更加综合的 I2C 传输函数，i2c_master_send 和 i2c_master_recv函数实际上便是调用的 i2c_transfer，在后面的实验中，我们并不会直接调用 i2c_master_send和 i2c_master_recv 函数进行数据的收发，而是通过 i2c_transfer 函数独立编写 I2C 数据收发函数，从而真正理解 I2C 数据收发的细节，i2c_transfer 函数它接受如下参数:

（1）struct i2c_adapter *adap: 指向目标 I2C 适配器的指针

（2）struct i2c_msg *msgs: 指向一个 I2C 消息数组的指针

（3）int num: 消息数组中的消息数量

最后提出一个问题，既然已经在 I2C 设备驱动层中创建了对应的设备节点，根据前面在字符设备中讲解的知识，有了驱动程序就可以直接对 I2C 具体硬件进行操作了，但是在 I2C 子系统并不是这样实现的，而是添加了 I2C 核心层和 I2C 适配器驱动层，那为什么要这样设计呢？

最主要的原因是通过驱动分层可以解决多个应用同时访问一个 I2C 设备冲突的问题，除此之外通过这种模块化设计,可以提高了代码的复用性和可维护性，使得 I2C 核心层和设备驱动程序可以独立开发和升级，I2C 适配器驱动程序也可以针对不同的硬件平台进行优化。

#### FT5X06驱动案例分析

.....

### USB总线协议

USB是由世界著名计算机和通信公司等共同推出的新一代接口标准，全称为Universal Serial Bus（通用串行总线，USB是低位先行），是一种快速、灵活的总线接口。它是为了解决日益增加的PC外设与有限的主板插槽和端口之间的矛盾而制定的一种串行通信标准。

USB一般分为USB低速，USB全速，USB高速和USB超高速,其分别对应于USB1.0,USB1.1,USB2.0和USB3.0,而USB3.0又分了GEN1,GEN2等。最新一代是USB4，传输速度为40Gbit/s，三段式电压5V/12V/20V，最大供电100W ，新型Type C接口允许正反盲插。

![USB的版本区别和发展历程](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/20210508223837906638.png)

#### PD

电力供应规范- USB Power Delivery（USB PD），设计上兼容现有的USB 2.0和USB 3.0线材和连接器，新的USB  3.1 Type-C连接器也通用。USB PD支持更高的电压和电流，以满足不同的应用装置，同时也相容现有的USB Battery  Charging 1.2充电规格。USB  PD为埠对埠的架构，USB和电力沟通讯号分开，电力的供应是透过主机端和装置端的VBus通讯协定来沟通，如果装置支持，则可依组态（Profiles）的电压和电流，提供更高的瓦数供应。USB  PD依装置不同分成5个组态（Profiles），皆需要使用新的可侦测线材，才能提供大於1.5A或5V的电力。组态1针对手机，为5V/2A（10W），基本的电力输出，已比USB 3.0的5V/0.9A（4.5W）还多；  组态2为5V/2A或12V/1.5A（最大18W），可为平板和笔电充电；组态3为5V/2A或12V/3A（可提供较大的笔电最大36W的电力）；组态4为最大为20V/3A（60W）（Micro-A/B连接器不支持），，组态5为最大为20V/5A（100W）的电力供应，（[TYPE-A](https://www.usbzh.com/article/detail-903.html)/B连接器不支持）,也就是说只有type-c才能完全支持最大100W电力供应。USB 3.1供电的提升，以及支持充电的应用将会让USB 3.1有望大一统手机、平板、电脑传输介面。

![电力供应特性](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1610521958786.png)

![USB PD 快速充电标准历史版本更新](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/113514779665.png)

常见充电协议

![：PD3.0的PPS（programmable power supply）功能](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/161110927513.png)

#### USB接口类型

![img](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1.jpg)

> USB接口的三大类型：
>
> TYPE
>
> Mini
>
> Micro

##### TYPE类型

TYPE类型常见于PC机，按其接口形状的不同又分为三大类。
分别为TYPE-A,TYPE-B,TYPE-C三种类型，其中Type-C现在是主流。

>  TYPE-A也叫USB-A,TYPE-B也叫USB-B,TYPE-C也叫USB-C接口。

![image-20240715221808967](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715221808967.png)

##### Mini类型

![image-20240715221834466](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715221834466.png)

##### Micro类型

![image-20240715221855962](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715221855962.png)

##### 使用情况

![USB插头对各USB协议规范的支持](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/20210508225120565838.png)

#### TYPE-A接口

![USB-A 公头](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1617349311762.png)

![usb接口接线图](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1617349131426.png)

> （1）USB的全速信号的比特率定为12Mbps；
> （2）低速信号传送的模式定为1.5Mbps；
> （3）USB高速为480Mbps;

电缆中包括VBUS、GND二条线，向设备提供电源。VBUS使用+5V电源。USB对电缆长度的要求很宽，最长可为几米。通过选择合适的导线长度以匹配指定的IR drop和其它一些特性，如设备能源预算和电缆适应度。为了保证足够的输入电压和终端阻抗。重要的终端设备应位于电缆的尾部。在每个端口都可检测终端是否连接或分离，并区分出高速，或低速设备。

| 引脚 | 名称 | 电缆颜色 | 描述           |
| ---- | ---- | -------- | -------------- |
| 1    | VBUS | Red      | +5 V，电源     |
| 2    | D−   | White    | Data −，数据线 |
| 3    | D+   | Green    | Data +，数据线 |
| 4    | GND  | Black    | Ground，接地   |

##### 兼容3.x

![image-20240715222353060](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/image-20240715222353060.png)

| 引脚 | A型连接器  | B型连接器  | 线缆颜色 | 描述                |
| ---- | ---------- | ---------- | -------- | ------------------- |
| 1    | VBUS       | 红色       | 红色     | 供电                |
| 2    | D-         | D-         | 白色     | 2.0数据差分对       |
| 3    | D+         | D+         | 绿色     | 2.0数据差分对       |
| 4    | GND        | GND        | 黑色     | 电源地              |
| 5    | StdA_SSRX− | StdB_SSTX− | 蓝色     | 高速数据差分对-接收 |
| 6    | StdA_SSRX+ | StdB_SSTX+ | 黄色     | 高速数据差分对-接收 |
| 7    | GND_DRAIN  | N/A        | N/A      | 信号地              |
| 8    | StdA_SSTX− | StdB_SSRX− | 紫色     | 高速数据差分对-发送 |
| 9    | StdA_SSTX+ | StdB_SSRX+ | 橙色     | 高速数据差分对-发送 |

#### TYPE-C接口

插座

![TYPE-C母头](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/20210508230603660879.png)

插头

![TYPE-C公头](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/20210508230654145762.png)

![Type C接线定义](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/typec-pin-define-connect.png)

![10305073708](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/10305073708.png)

#### USB的连接模型 

 USB是一种主从结构。主机叫做Host，从机叫做Device（也叫做设备），集线器也被当作一种特殊的设备处理,用于扩展USB端口（扩展的USB端口可以增加USB总线上物理设备的连接）。

USB的数据交换只能发生在主机和设备之间，主机和主机，设备和设备之间不能互连。为了在物理上区分主机和设备，使用了不同的插头和插座，可以参见USB硬件接口分类一节。USB总线上所有的数据传输都由主机主动发起，而设备只是被动的负责应答。例如，在读数据时，USB主机先发出读命令，设备收到该命令后，才返回数据。在USB OTG中，一个设备可以在从机和主机之间切换，这样就可以实现设备与设备之间的连接，大大增加了USB的使用范围。但这时依然没有脱离这种主从关系，两个设备之间必然有一个作为主机，另一个作为从机。USB OTG增加了一种MINI USB接头，比普通的4线USB多了一个ID表识线，用来表明它是主机还是设备，这个以后会讲到。

USB的拓扑结构为金字塔型。由一个USB主控制器出发，下面接USB集线器，USB集线器将一个USB口扩展为多个USB口，多个USB口又可以通过集线器为更多个接口。但USB协议中对集线器的层数是有限制的，USB1.1规定最多为5层，USB2.0规定最多为7层。

![USB层级结构](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1613722985418.png)

#### USB2.0物理特性

USB设备支持即插即用，所以对于USB主机端，一个重要的特性就是USB设备的动态连接检测。
USB主机端支持设备的连接状态的检测，是需要USB设备的配合的。USB主机端与USB设备端相互配合，实现了USB设备的连接状态检测。

![USB设备连接图](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1616513442305.png)

USB主机端的D+和D-均有一个15K的下拉电阻，而设备端根据USB设备分为两大类:USB低速和USB[全速](https://www.usbzh.com/article/detail-851.html)[高速](https://www.usbzh.com/article/detail-852.html)，分别在其设备的D-或D+上拉一个1.5K的电阻。

    主机端D+和D-均有一个15K的下拉电阻。
    低速设备端D-上拉一个1.5K的电阻
    高速/全速设备端D+上拉一个1.5K的电阻

一个USB设备连接到主机后，大概分为以下几个阶段：

![USB2.0设备的连接状态检测](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/105708750537.png)

- 默认态，USB主机端VBus为高电平。
- USB设备连接到USB主机。
- USB设备端的VBus从低电平变为高电压(>=4.01V)
- USB设备端VBus检测到高电平
- USB设备端低速挂接D-上的1.5K上拉电阻，全速或高速设备接接D+上的1.5K上拉电阻。
- USB主机端检测到D+/或D-端的电压变高（2.0V以上）
- USB主机端根据D+/D-线缆上的电压变化识别USB2.0低速设备或全速设备（高速设备）连接上来
- USB主机对检测到的设备发送复位信号，进行设备复位。

##### J态、K态、SE0,SE1

在USB 2.0协议中经常会看到以下术语：Chirp K、KJ序列、SE0。这里的状态是根据低速、全速/高速下设备D+和D-上不同的电平信号来决定的。

| 信号 | 转换状态          | 状态                                                       | Low speed (D+ pull-up) | Low speed (D− pull-up) | Full speed (D+ pull-up) | Full speed (D+ pull-up) |
| ---- | ----------------- | ---------------------------------------------------------- | ---------------------- | ---------------------- | ----------------------- | ----------------------- |
|      |                   |                                                            | D+                     | D−                     | D+                      | D−                      |
| J    | 空闲线路状态相同  | 这是出现在传输线转换期间。或者，它正在等待一个新的数据包。 | 0                      | 1                      | 1                       | 0                       |
| K    | J相反状态         | 这是出现在传输线转换期间。                                 | 1                      | 0                      | 0                       | 1                       |
| SE0  | Single-ended zero | D+和D− 为低。指示一个包的结束或USB设备和移除               | 0                      | 0                      | 0                       | 0                       |
| SE1  | Single-ended one  | 非法的状态，不应该出现。标示为错误                         | 1                      | 1                      | 1                       | 1                       |

从J到K或者从K到J，信号翻转，说明发送的是信号0；
从J到J或者从K到K，信号保持不变，说明发送的是信号1。这就是差分信号0/1的发送。

- 差分信号1： D+ > VOH (2.8V)and D- < VOL(0.3V)
- 差分信号0： D- > VOH and D+ < VOL
- 复位信号（Reset）： D+ and D- < VOL for >= 10ms
- IDLE状态： J状态
- 恢复信号（Resume）： K 状态
- SOP：从IDLE状态切换到K状态
- EOP：持续2位时间的SE0信号，后跟随1位时间的J状态
- SYNC： 3个重复的KJ状态切换，后跟随2位时间的K状态

高速设备的J和K相反。

低速下： D+为“0”，D-为“1”是为“J”状态，“K”状态相反；
全速/高速下：D+为“1”，D-为“0”是为“J”状态，“K”状态相反；

![JK状态](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/23492654110.png)

![全速设备的传输示例](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/2021050910304965374.png)

##### 空闲状态

低速下空闲状态为“J”状态；
全速下空闲状态为“K”状态；
高速下空闲状态为“SE0”状态；

#### USB硬件编码格式NRZI

我们知道USB3.0以前采用的是两根数据线D+和D-所对应的数据传输，采用的是数据编码方式是NRZI（**Non-Return-to-Zero Inverted，翻转不归零编码**），而USB3.0以后采用的是8/10bit编码。

USB使用NRZI编码方式：**当数据为0时，电平翻转；数据为1时，电平不翻转**。为了防止出现过长时间电平不变化现象，在发送数据时采用位填充处理。具体过程如下：**当遇见连续6个高电平时，就强制插入一个0**。经过位填充后的数据由串行接口引擎（SIE）将数据串行化和NRZI编码后，发送到USB的差分数据线上。接收端完成的过程和发送端刚好相反。

##### 同步时钟编码

USB 的数据是串行发送的，就像 UART、I2C、SPI 等等，连续的01 信号只通过一根数据线发送给接受者。

但是因为发送者和接收者运行的频率不一样，信号的同步就是个问题，比如，接受者接收到了一个持续一段时间的低电平，由于没有时钟采用，所以无法得知这究竟是代表了多少个低电平0。

一个解决办法，就是在传输数据信号的同时，附加一个时钟信号，用来同步两端的传输，接受者在时钟信号的辅助下对数据信号采样，就可以正确解析出发送的数据了，比如 I2C 就是这样做的，SDA 来传输数据，SCL 来传输同步时钟：

![i2c-hd-encode](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/i2c-hd-encode.png)



虽然这样解决了问题，但是却需要附加一根时钟信号线来传输时钟。有没有不需要附加的时钟信号，也能保持两端的同步呢？
有的，这就是 RZ 编码（Return-to-zero Code），也叫做归零编码。

##### NRZ编码（Non-return-to-zero Code）

RZ 编码（Return-to-zero Code），也叫做归零编码。在 RZ 编码中，正电平代表逻辑 1，负电平代表逻辑 0，并且，每传输完一位数据，信号返回到零电平，也就是说，信号线上会出现 3 种电平：正电平、负电平、零电平：

![1610552904953](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1610552904953.png)

从图上就可以看出来，因为每位传输之后都要归零，所以接受者只要在信号归零后采样即可，这样就不在需要单独的时钟信号。实际上， RZ 编码就是相当于把时钟信号用归零编码在了数据之内。这样的信号也叫做自同步（self-clocking）信号。

这样虽然省了时钟数据线，但是还是有缺点的，因为在 RZ 编码中，大部分的数据带宽，都用来传输“归零”而浪费掉了。

##### NRZ编码（Non-Return-to-Zero Inverted Code）

我们去掉这个归零步骤，NRZ 编码（Non-return-to-zero Code）就出现了，和 RZ 的区别就是 NRZ 是不需要归零的：

![1610552921072](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1610552921072.png)

这样，浪费的带宽又回来了，不过又丧失宝贵的自同步特性了，貌似我们又回到了原点，其实这个问题也是可以解决的，不过待会儿再讲，先看看什么是 NRZI：

##### NRZI 编码（Non-Return-to-Zero Inverted Code）

NRZI 编码（Non-Return-to-Zero Inverted Code）和 NRZ 的区别就是 NRZI 用信号的翻转代表一个逻辑，信号保持不变代表另外一个逻辑。

USB 传输的编码就是 NRZI 格式，在 USB 中，电平翻转代表逻辑 0，电平不变代表逻辑1：

![1610552934466](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1610552934466.png)

![1610553336619](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1610553336619.png)

翻转的信号本身可以作为一种通知机制，而且可以看到，即使把 NRZI 的波形完全翻转，所代表的数据序列还是一样的，对于像 USB 这种通过差分线来传输的信号尤其方便~

现在再回到那个同步问题：

的确，NRZ 和 NRZI 都没有自同步特性，但是可以用一些特殊的技巧解决。

比如，先发送一个同步头，内容是 0101010 的方波，让接受者通过这个同步头计算出发送者的频率，然后再用这个频率来采样之后的数据信号，就可以了。

##### USB使用NRZI编码同步原理

在 USB 中，每个 USB 数据包，最开始都有个同步域（SYNC），这个域固定为 0000 0001，这个域通过 NRZI 编码之后，就是一串方波（复习下前面：NRZI 遇 0 翻转遇 1 不变），接受者可以用这个 SYNC 域来同步之后的数据信号。

此外，因为**在 USB 的 NRZI 编码下，逻辑 0 会造成电平翻转**，所以接受者在接受数据的同时，根据接收到的翻转信号不断调整同步频率，保证数据传输正确。

但是，这样还是会有一个问题，就是虽然接受者可以主动和发送者的频率匹配，但是两者之间总会有误差。

假如数据信号是 1000 个逻辑 1，经过 USB 的 NRZI 编码之后，就是很长一段没有变化的电平，在这种情况下，即使接受者的频率和发送者相差千分之一，就会造成把数据采样成 1001 个或者 999 个 1了。

##### USB中用Bit-Stuffing来同步时钟信号

USB 对这个问题的解决办法，就是强制插 0，也就是传说中的 bit-stuffing，如果要传输的数据中有 7 个连续的 1，发送前就会在第 6 个 1 后面强制插入一个 0，让发送的信号强制出现翻转，从而强制接受者进行频率调整。
接受者只要删除 6 个连续 1 之后的 0，就可以恢复原始的数据了。

> 很多人会说如果传输了  11111101，会不会接受者把0勿删，答案是不会的，接收端收到6个1后，紧接着肯定会收到0（如果第7位为1则插入0，如果传输的第7位为0则接受者接收到的也为0）。这个0是否删除需要查看0的下一位，下一位如果1则发送端是真的发送了连续的7个1，之前收到的0的确是发送端插入进入的，需要删除；如果下一位是0，则接受端只发送了6个1，之前收到的0为接收端发送的数据，不能删除。

**Bit-Stuffing实例**

在使用力科的抓包调试工具时，有一个很好的功能是可以显示波形。如对于如下的的[OUT](https://www.usbzh.com/article/detail-449.html)令牌包，其因为PID和地址出现连续的1，故插入了Bit-Stuffing。

![Bit-Stuffing](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/100810691179.png)

**不显示Bit-Stuffing**

![不显示Bit-Stuffin](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/100836481465.png)

**显示BIT-stuffing**

![显示BIT-stuffing](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/100916968142.png)

#### USB2.0时钟频率与NRZI编码 

USB的编码方式NRZI（非归零反向编码），提到这个是为了说明信号的时钟周期（等效时钟，并没有专门的时钟线）到底是多少，因为这个才是我们硬件工程师关注的。

对于差分对的传输线:

```
传输速率或带宽（Mbps）= 时钟频率（MHz）* 位宽 * 通道数 * 每时钟传输数据组数（cycle）
```

所以USB高速的计算公式为:

```
480Mbps=240MHz *1  *1 *2
```

每个时钟周期传送2次数据（这跟编码方式有关USB为NRZI），也即当传输速率为480Mbps时，对应的时钟频率为240MHz，而且这个240MHz的时钟频率还是USB芯片里面晶振经过倍频得到的，实际USB晶振有12MHz,24MHz,48MHz等.所以现在大家知道，频率和传输速率完全两个概念，对于差分讯号的传输线.

NRZI编码，如下图，实际的波形如下图的NRZI，红色的方框可以看作是信号的一个完整周期。一个完整的周期传输了2个bit，上升沿，下降沿各传输一个bit。

![NRZI编码](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/142251665825.png)

所以，USB2.0高速模式数据bit率是480Mb/s，USB等效时钟是240Mhz，并不是480Mhz。

#### USB2.0通信时序

![https://www.usbzh.com/uploadimg/202202/232239921302.jpg](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/232239921302.jpg)

##### USB通讯原理 

USB是轮询总线，USB主机与设备之间的数据交换都是由主机发起的，设备端只能被动的响应。USB数据传入或传出 USB 设备中的端点。

USB 主机中的客户端将数据存储在缓冲区中，USB主机没有端点的概念。

USB Host 和外围 USB Device 有不同的层，如下图所示。各层之间的连接是每个水平层之间的逻辑主机-设备接口。在逻辑连接之间使用USB Pipes传输数据。

![USB 主机客户端和USB设备端点之间的逻辑连接](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/114319284578.png)

**USB通讯过程**

一次完整的通信分为三个过程：请求过程（令牌包）、数据过程（数据包）和状态过程（握手包），没有数据要传输时，跳过数据过程。
通信过程包含以下三种情况：

![USB通讯过程](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/143742437276.png)

主机发送令牌包（Token）开始请求过程，如果请求中声明有数据要传输则有数据过程，最后由数据接收方（有数据过程）或从机（无数据过程）发起状态过程，结束本次通信。
与USB全速设备通信时，主机将每秒等分为1000个帧（Frame）。主机在每帧开始时，向所有从机广播一个帧起始令牌包（Start Of Frame，SOF包）。它的作用有两个：一是通知所有从机，主机的USB总线正常工作；二是从机以此同步主机的时序。
  与USB高速设备通信时，主机将帧进一步等分为8个微帧（Microframe），每个微帧占125μ \muμs。在同一帧内，8个微帧的帧号都等于当前SOF包的帧号。

**管道PIPE**

管道分为两种类型：

消息管道具有已定义的 USB 格式并受主机控制。消息管道允许数据双向流动并且仅支持控制传输。
流管道没有定义的 USB 格式，可以由主机或设备控制。数据流具有预定义的方向，即IN或OUT。流管道支持中断传输、同步传输和批量传输。
当 USB 设备连接到 USB 总线并由 USB 主机配置时，大多数管道就会存在。管道源自主机客户端内的数据缓冲区，并在 USB 设备内的端点处终止。

**传输**

传输（数据流类型）可以由一个或多个事务组成。管道仅支持以下传输类型之一：

控制传输通常用于设置 USB 设备。他们总是使用 IN/OUT 端点 0。
中断传输可用于定期发送数据的地方，例如状态更新。
同步传输传输实时数据，例如音频和视频。它们有保证的固定带宽，但没有错误检测。
批量传输可用于在时间不重要的情况下发送数据，例如发送到打印机。

**事务**

数据在所谓的事务中传输。通常，它们由三个数据包组成：

令牌包是定义事务类型和方向、设备地址和端点的标头。
数据以数据包的形式传输。
交易的最终状态是握手包中的确认。

![传输模型](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/114624865637.png)

在事务中，数据从 USB 主机传输到 USB 设备，反之亦然。传输方向在从 USB 主机发送的令牌包中指定。然后，源发送一个数据包或指示它没有数据要传输。一般情况下，目的地会以握手包进行响应，指示传输是否成功。

![数据包模型](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/114654276362.png)

**数据包**

数据包可以被认为是数据传输的最小元素。每个数据包以当前传输速率传输整数个字节。数据包以同步模式开始，随后是数据包的数据字节，并以数据包结束 (EOP) 信号结束。所有 USB 数据包模式都先传输最低有效位。数据包前后，总线处于空闲状态。

![帧开始 (SOF) 数据包](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/114723154027.png)

一个特殊的数据包是将 USB 总线分成时间段的帧起始数据包 (SOF)。每个管道在每个帧中分配一个时隙。Start-of-Frame 数据包在全速链路上每 1ms 发送一次。在高速下，1ms 帧被分成 8 个微帧，每个微帧 125μs。Start-of-Frame 数据包在每个微帧的开头使用相同的帧号发送。帧号每 1ms 递增一次。

**端点**

端点可以描述为数据源或接收器，并且仅存在于 USB 设备中。存储在端点的数据可以从 USB 主机接收或等待发送到 USB 主机。端点可以配置为支持USB 规范中定义的四种传输类型（控制传输、中断传输、同步传输和批量传输）。在硬件限制范围内，端点可以使用 USB 中间件进行配置（例如，将端点限制为某种传输类型）。

端点充当一种缓冲区。例如，USB 主机的客户端可以向端点 1 发送数据。来自 USB 主机的数据将被发送到OUT 端点 1. 微控制器上的程序将在准备好后立即读取数据。返回数据必须写入IN Endpoint 1，因为程序无法自由访问 USB 总线（USB 总线由 USB 主机控制）。IN Endpoint 1 中的数据一直保留在那里，直到主机向 Endpoint 1 发送 IN 数据包请求数据。

这些规则适用于所有微控制器设备：

一个设备最多可以有16 个 OUT和16 个 IN端点。
每个端点只能有一个 传输 方向。
端点 0仅用于控制传输，不能分配给任何其他功能。

> 端点的总数和每个端点的数据大小由底层硬件定义。
>
> OUT总是指从主机指向设备的方向。
> IN总是指指向主机的方向。

##### USB2.0包Packet的组成

USB包由SOP,SYNC,Packet内容和EOP组成.

![USB包](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/000436275782.png)

**SOP起始包**

起始包SOP（Start Of Packet），通过将D +和D-线从空闲状态驱动到相反的逻辑电平（K状态），由始发端口发信号通知分组的开始（SOP）。 此开关级别表示SYNC字段的第一位。 当重新传输到小于±5 ns时，集线器必须限制SOP第一位宽度的变化。 通过将通过集线器的标称数据延迟与集线器的输出使能延迟相匹配，可以最小化失真。

![SOP包](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/235738272732.png)

**SYNC同步域**

同步域(SYNC),这个域固定为 0000 0001,这个域通过 NRZI 编码之后,就是一串方波(NRZI 遇 0 翻转遇 1 不变),接受者可以用这个 SYNC 域来同步之后的数据信号。

> 因为在 USB 的 NRZI 编码下,逻辑 0 会造成电平翻转,所以接受者在接受数据的同时,根据接收到的翻转信号不断调整同步频率,保证数据传输正确。但是,这样还是会有一个问题,就是虽然接受者可以主动和发送者的频率匹配,但是两者之间总会有误差。假如数据信号是 1000 个逻辑 1,经过 USB的 NRZI 编码之后,就是很长一段没有变化的电平,在这种情况下,即使接受者的频率和发送者相差千分之一,就会造成把数据采样成 1001 个或者 999 个1 了。
> USB 对这个问题的解决办法,就是强制插 0,也就是传说中的 bit-stuffing,如果要传输的数据中有 7 个连续的 1,发送前就会在第 6 个 1 后面强制插入一个 0,让发送的信号强制出现翻转,从而强制接受者进行频率调整。接受者只要删除 6 个连续 1 之后的 0,就可以恢复原始的数据了.

**EOP 结束包（End of Packet）**

全速或低速设备的结束包：SE0状态用于发信号通知分组结束（EOP）。 通过将D +和D-驱动到SE0状态两位时间，然后将线路驱动到J状态一位时间来发信号通知EOP。 从SE0到J状态的转换定义了接收器处的分组的结束。 J状态被置位一个位时间，然后D +和D-输出驱动器都处于高阻态。 总线终端电阻将总线保持在空闲状态。
SE0的意思是D+和D-都表示为低电平。

![全速或低速设备的结束包](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/000337146274.png)

高速设备的EOP:在高速信号中，以从EOP之前的最后一个符号到相反符号的转换开始。这个相反的符号是EOP模式中的第一个符号对于SOF以外的高速数据包。故意生成位填充错误以指示EOP。需要接收器将任何位错误解释为EOP。发送的EOP定界符必须是没有位填充的NRZ字节01111111。例如，如果EOP字段之前的最后一个符号是J，则这将导致EOP为KKKKKKKK。对于高速SOF，传输的EOP分隔符需要5个NRZ字节而不需要填充比特，由01111111 11111111 11111111 11111111 11111111组成。因此，如果EOP字段之前的最后一位是J，这将导致线路上有40个K状态，线路必须返回到高速空闲状态。额外的EOP长度对接收器没有意义，它用于断开检测。

**Packet内容**

包内容属于协议层，包括的内容大概为PID(包ID),地址，帧号，数据和CRC校验。

![Packet内容](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/000501823761.png)

**PID：**
根据PID的内容，包可分为四大类：
Packet分四大类：

- 命令 (Token) Packet
- 帧首 (Start of Frame) Packet
- 数据 (Data) Packet
- 握手 (Handshake) Packet

**地址：**
地址包括设备地址和端点地址。其中设备地址占1字节，端点地址占4位。

![地址](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/000835103762.png)

**帧号：**

- 总共11位。主机每发出一个帧
- 帧号都会自加1
- 当帧号达到7FFH时，将归零重新开始计数
- 仅在每个帧的帧首传输一次SOF包

**数据域**
为USB传输的数据。对于不同的USB传输类型，数据域的数据长度从0到1024字节不等。

![数据域](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/001457298017.png)

**CRC校验域：** 
CRC校验域分为令牌校验域和数据校验域

- 令牌Token CRC校验域
  计算地址域和帧号域的CRC：G(X) = X5 + X2 + 1
- Data CRC
  计算数据域数据的CRC： G(X) = X16 + X15 + X2 + 1

##### 包标识PID

USB协议定义的包格式PID由8位组成，低4位是类型字段，高4位为低四位的反码。

| PID  | 含义          | 说明                                                        |
| ---- | ------------- | ----------------------------------------------------------- |
| 0001 | 令牌OUT       | 主机发送数据到USB设备                                       |
| 1001 | 令牌IN        | 主机接收从USB设备发出的数据                                 |
| 0101 | 令牌SOF       | 此时作为一个帧或者小帧的开始信息                            |
| 1101 | 令牌SETUP     | 主机向USB设备发送配置信息                                   |
| 0010 | 握手ACK       | 数据正确接收                                                |
| 1010 | 握手NAK       | 数据未正确接收                                              |
| 1110 | 握手STALL     | 使用的端点被挂起                                            |
| 0110 | 握手NYET      | 接收方没有响应                                              |
| 0011 | 数据DATA0     | 数据包偶数包                                                |
| 1011 | 数据DATA1     | 数据为奇数据包                                              |
| 0111 | 数据DATA2     | 此为作为一个高速同步事务的专用数据包                        |
| 1111 | 数据MDATA     | 此时作为一个SPLIT事务的专用数据包。                         |
| 1100 | PRE令牌包     | 低速数据的先导包                                            |
| 1100 | 特殊用途ERR   | SPLIT事务中表示出现错误                                     |
| 1000 | 特殊用途SPLIT | 高速主使用事该SPLIT事务解决从高速模式到低速和全速模式的转换 |
| 0100 | 特殊用途PING  | 仅用于高速模式下主机使用该事务判断设备是否可以接收数据      |

##### USB2.0 令牌包  		

| PID  | 含义                                                         | 说明                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 0001 | 令牌[OUT](https://www.usbzh.com/article/detail-449.html)     | 主机发送数据到USB设备                                        |
| 1001 | 令牌IN                                                       | 主机接收从USB设备发出的数据                                  |
| 0101 | 令牌[SOF](https://www.usbzh.com/article/detail-451.html)     | 此时作为一个帧或者小帧的开始信息                             |
| 1101 | 令牌[SETUP](https://www.usbzh.com/article/detail-446.html)   | 主机向USB设备发送配置信息                                    |
| 1000 | 特殊用途[SPLIT](https://www.usbzh.com/article/detail-702.html) | [高速](https://www.usbzh.com/article/detail-852.html)主使用事该[SPLIT](https://www.usbzh.com/article/detail-702.html)[事务](https://www.usbzh.com/article/detail-691.html)解决从[高速](https://www.usbzh.com/article/detail-852.html)模式到低速和[全速](https://www.usbzh.com/article/detail-851.html)模式的转换 |
| 0100 | 特殊用途[PING](https://www.usbzh.com/article/detail-454.html) | 仅用于[高速](https://www.usbzh.com/article/detail-852.html)模式下主机使用该[事务](https://www.usbzh.com/article/detail-691.html)判断设备是否可以接收数据 |
| 1100 | PRE令牌包                                                    | 低速数据的先导包                                             |

##### USB2.0 数据包 

USB主机发出的包在USB总线上广播，所有在USB总线上的设备需要根据自己的设备地址对由USB主机广播的令牌包进行过滤。如果该令牌包的地址与其自身地址不匹配，USB设备默认不处理即忽略该令牌包。

USB包的目标地址只有7位，所以一条US总线上最多可以挂接127个USB设备（地址0用于设备在枚举过程中），包中的目标端点地址占4位，故USB最大可以支持16个双向端即点其32个端点。

| PID  | 含义      | 说明                                                         |
| ---- | --------- | ------------------------------------------------------------ |
| 0011 | 数据DATA0 | 数据包偶数包                                                 |
| 1011 | 数据DATA1 | 数据为奇数据包                                               |
| 0111 | 数据DATA2 | 此为作为一个[高速](https://www.usbzh.com/article/detail-852.html)同步[事务](https://www.usbzh.com/article/detail-691.html)的专用数据包 |
| 1111 | 数据MDATA | 此时作为一个[SPLIT](https://www.usbzh.com/article/detail-702.html)[事务](https://www.usbzh.com/article/detail-691.html)的专用数据包。 |

数据包中并没有其传输的目的地址和端点信息，所以数据包必须紧跟在令牌包之后。
数据包是用于传输数据的，由8位的包标识PID，数据字段和16位的循环冗余校验字段CRC组成。

![数据包格式](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1599989508942.png)

- PID字段：用于指明不同的数据包类型。支持 4 种数据包，分别为： DATA0 、 DATAI 、DATA2 和MDATA。 在介绍的数据触发机制中，使用 DATA0 和 DATA1。
  - [SPLIT](https://www.usbzh.com/article/detail-702.html) 令牌[事务](https://www.usbzh.com/article/detail-691.html)处理则使用DATA0，DATA1和MDATA。
  - 对于[高速](https://www.usbzh.com/article/detail-852.html) USB 同步数据传输，一般需要使用全部。
- 数据字段：其中包含了传输的数据。其数据的大小根据数据传输类吧和用户需要而定。
  - 根据 USB 协议的规定，对于低速 USB 数据传输， 最大长度为8字节
  - 对于[全速](https://www.usbzh.com/article/detail-851.html)USB 数据传输，其最大长度为 1023 字节；
  - 对于[高速](https://www.usbzh.com/article/detail-852.html) USB 数据传输，数据最大长度为 1024 。
- CRC 字段：这里使用 16 位的循环冗余校验来对数据字段进行保护。

![数据包格式CRC](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/172510138279.png)

![数据MDATA](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/205656841635.png)

##### USB2.0 握手包  		

握手包跟随在令牌包或者数据包之后，组成一个完整的[事务](https://www.usbzh.com/article/detail-691.html)，是对一次[事务](https://www.usbzh.com/article/detail-691.html)完成的确认。USB主机或者USB设备会根据[事务](https://www.usbzh.com/article/detail-691.html)的完成状态返回相应的握手包。

握手包由8位的PID构成，用于数据传输的末位报告本次数据传输的状态。握手包之后使是整个事务处理的结束信号EOP.

![握手包](https://woniumd.oss-cn-hangzhou.aliyuncs.com/aiot/luozhaoyong/1599989892253.png)

握手包根据PID的不同，可分为：

| PID  | 含义                                                         | 说明                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 0010 | 握手[ACK](https://www.usbzh.com/article/detail-452.html)     | 数据正确接收                                                 |
| 1010 | 握手[NAK](https://www.usbzh.com/article/detail-453.html)     | 数据未正确接收                                               |
| 1110 | 握手[STALL](https://www.usbzh.com/article/detail-698.html)   | 使用的端点被挂起                                             |
| 0110 | 握手[NYET](https://www.usbzh.com/article/detail-699.html)    | 接收方没有响应                                               |
| 1100 | 特殊用途[ERR](https://www.usbzh.com/article/detail-700.html) | [SPLIT](https://www.usbzh.com/article/detail-702.html)事务中表示出现错误 |

USB规范定义了以下几个响应：
1 . ACK 握手包

当 USB 数据传输的接收方正确接收到数据包时，接收方将返回 ACK 握手包。 ACK 握手包表征了一次正确的数据传输，之后才可以进行下一次事务处理。
2 . NAK 握手包

NAK 握手包一般由 USB 功能设备发出。对于IN数据传输，表示 USB 设备没有计划向 USB 主机发送数据；对于 OUT 数据传输，表示 USB 设备无法接收 USB 主机发送的数据。
3 . STALL 握手包

STALL 握手包一般由 USB 功能设备发送，表示该 USB 功能设备不支持这个请求，或者无法发送和接收数据。 STALL 握手包分为以下两种情况。

    协议 STA LL 握手包：在控制传输中使用。协议 STALL 握手包表明了该 USB 功能设备不支持这个请求协议。
    功能 STALL 握手包：表明该 USB 功能设备的相应端点已经停止，而无法完成发送数据或者接收数据的操作。 

4 . NYET 握手包

在 SPLIT 令牌包事务处理中，如果 USB 集线器无法正常处理 SPLIT 请求，则 USB 集线器向 USB 主机返回 NYET 握手包。 NYET 握手包一般只发生在高速数据传输过程中。
在USB2.0高速设备OUT事务中使用，它表示设备本次数据成功接收，但是没有足够的空间来接收下一次数据。主机在下一次输出数据时，将先使用PING命令牌包来探测设备是否有足够的空间接收数据，一面不必要的带宽浪费。
5 . ERR 握手包

ERR 握手包用于表示总线数据传输发生错误，其一般发生在高速数据传输过程中。
其功能是由USB集线器HUB向主机报告挂载其上的低速或全速设备错误。




# 点亮GPIO1_B0

```c
#include <linux/module.h>

#include <linux/platform_device.h>

#include <linux/mod_devicetable.h>

#include <linux/gpio/consumer.h>

#include <linux/delay.h>

#include <linux/of.h>

#include <linux/of_irq.h>

#include <linux/gpio.h>

  

struct gpio_desc *mygpio; // GPIO 描述符指针

int num;                  // GPIO 编号

struct device_node *mydevice_node;

struct irq_data *my_irq_data;

int irq;

  

// 平台设备初始化函数

static int my_platform_probe(struct platform_device *dev)

{

    printk("进入绑定函数\n");

  

    mygpio = gpiod_get_optional(&dev->dev, "irq_led_test", 0);

    if (mygpio == NULL)

    {

        printk("gpiod_get_optional error\n");

        return -1;

    }

    num = desc_to_gpio(mygpio);

    printk("GPIO引脚号：%d\n", num);

    num = gpiod_get_direction(mygpio);

    printk("GPIO方向：%d\n", num);

    gpiod_direction_output(mygpio, 0);

    ssleep(3);

    gpiod_direction_output(mygpio, 1);

    num = gpiod_get_direction(mygpio);

    printk("GPIO方向：%d\n", num);

  

    // 查找设备节点

    mydevice_node = of_find_node_by_name(NULL, "woniu__test_irq");

  

    // 解析和映射中断

    irq = irq_of_parse_and_map(mydevice_node, 0);

    printk("中断号：%d\n", irq);

  

    // 获取中断数据结构

    my_irq_data = irq_get_irq_data(irq);

  

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

  

const struct of_device_id of_match_table_id[] = {

    {.compatible = "irq_led_test"},

    {.compatible = "my__irq"},

};

  

// 定义平台驱动结构体

static struct platform_driver my_platform_driver = {

    .probe = my_platform_probe,

    .remove = my_platform_remove,

    .driver = {

        .name = "irq_led_test",

        .owner = THIS_MODULE,

        .of_match_table = of_match_table_id,

    },

};

  

// 模块初始化函数

static int __init my_platform_driver_init(void)

{

    int ret;

  

    // 注册平台驱动

    ret = platform_driver_register(&my_platform_driver);

    if (ret)

    {

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

    gpiod_put(mygpio);

    platform_driver_unregister(&my_platform_driver);

  

    printk(KERN_INFO "my_platform_driver: Platform driver exited\n");

}

  

module_init(my_platform_driver_init);

module_exit(my_platform_driver_exit);

  

MODULE_LICENSE("GPL");

MODULE_AUTHOR("woniu");
```