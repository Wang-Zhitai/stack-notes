# USB总线协议

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

![10305073708]()

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

![https://www.usbzh.com/uploadimg/202202/232239921302.jpg]()

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

**这里没有SOP 开始位，是因为他只是一个上升沿或者下降沿**

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