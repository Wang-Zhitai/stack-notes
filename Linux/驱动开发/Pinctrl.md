Pinctrl文件中：

```c
	// LED控制引脚
	gpio4c5:gpio4c5{
		rockchip,pins =
			<4 RK_PC5 0 &pcfg_pull_none>;
	};
```

DTS文件中：

```c
// LED
	ledctrl:ledctrl{
		compatible = "ledctrl";
		ledcrtl-gpios = <&gpio4 RK_PC5 GPIO_ACTIVE_LOW>;
 		pinctrl-names = "default";
  		pinctrl-0 = <&gpio4c5>;
	};
```

# 一、Linux Pinctrl 子系统简介

在许多SOC内部都包含有IOMUX（输入输出多路复用器），我们可以配置该IOMUX修改一个或者一组引脚的功能和电气特性。在软件方面，Linux内核提供了Pinctrl子系统，目的是为了统一对各SOC厂商的PIN脚进行管理。

1. Linux Pinctrl子系统提供的功能

   （1）管理系统中所有的可以控制的pin，在系统初始化的时候，枚举所有可以控制的pin，并标识这些pin。

   （2）管理这些pin的复用（Multiplexing）。对于SOC而言，其引脚除了配置成普通的GPIO之外，若干个引脚还可以组成一个pin group，行程特定的功能。pin control subsystem要管理所有的pin group。

   （3）配置这些pin的特性。例如使能或关闭引脚上的pull-up、pull-down电阻，配置引脚的driver strength。

3.pinctrl相关概念
普通driver调用pin control subsystem的主要目标有两个：

    （1）设定该设备的功能复用。
    
    （2）设定该device对应的pin的电气特性。

设定设备的功能复用需要了解两个概念，一个是function，另外一个是pin group。function是功能抽象，对应一个HW逻辑block，例如SPI0.虽然给定了具体的function name，我们并不能确定其使用的pins的情况。例如为了设计灵活，芯片内部的SPI0的功能引出到pin group{C6，C7，C8，C9}，也可能引出的另外一个pin group{C22.C23，C24，C25}，但毫无疑问，这两个pin group不能同时active，毕竟芯片内部的SPI0的逻辑功能电路只有一个，因此只有给出function selector以及function的pin group selector才能进行function mux 的设定。

此外，由于电源管理的要求，某个device可能处于某个电源管理状态，例如idle或者sleep，这时候，属于device的所有pin就会需要处于另外的状态。

综合上述的需求，就定义了pin control state的概念，也就是说设备可能处于非常多的状态中的一个，device driver可以切换设备处于的状态。为了方便管理pin control state 就有了pin control state holder的概念，用来管理一个设备的所有的pin control状态。

综上所述，普通的driver调用pin control subsystem 的接口就是只有三个步骤：

（1）驱动加载或是运行时，获取pin control state holder的句柄

（2）设定pin control的状态

（3）驱动卸载或是退出时，释放pin control state holder的句柄

