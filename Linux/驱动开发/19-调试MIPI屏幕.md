# 调试MIPI屏幕

# 1. RK3566 MIPI DSI外设

**`SOC` 有一个 `MIPI DSI` 接口，相对应的就应该有一个 `MIPI DSI HOST` 控制器外设,该外设内核是符合标准 `MIPI` 协议的，此 `MIPI DSI HOST` 控制器用于连接内核和 `D-PHY` ， `DSI HOST` 控制器发出的所有并行数字信号（无论是时钟还是数据 `Lanes`）都必须经过同一个 `D-PHY` 模块转换为串行差分模拟信号后才能发送到 `PCB` 走线上。**

**RK3566 的 MIPI DSI HOST 控制器支持如下特性：**

* 兼容 MIPI 联盟标准
* 支持 DPI 接口颜色映射，支持 16/18/24 bit 色深
* 所有 DPI 接口极性可编程
* 最高支持 4 lanes 的 D-PHY 数据 lanes
* Data0 支持双线通信和 Escape模式
* 可以传输所有的 Generic 命令
* 支持 EOTP 包

**RK3566 的 D-PHY 支持如下特性：**

* V1.2 版本的 MIPI D-PHY
* 集成 PPI 接口，支持 DSI
* 最多支持 4 lanes，总共有 10Gbps 传输速率
* 支持 MIPI HS 和 LP 模式
* 支持 Skew 校准
* LP 模式下 10Mbps 传输速率

![image-20251224101609828](./assets/image-20251224101609828.png)



# 2. 原理图&设备树

## 2.1 原理图

![image-20251224102507940](./assets/image-20251224102507940.png)

![image-20251224102526164](./assets/image-20251224102526164.png)

![image-20251224103454792](./assets/image-20251224103454792.png)

![image-20251224103251655](./assets/image-20251224103251655.png)

![image-20251224103533694](./assets/image-20251224103533694.png)

**可以看出：**

1. 我们的 `MIPI DSI` 接口实际上是接到了芯片内部的 `DSI_TX0` 上面 所以设备树里应该也是 `DSI0` 节点
2. 触控芯片是 `I2C` 接口的，设备树里也需要有一个对应的 `I2C` 节点
3. 背光控制 `PWM` 是接到了 `PWM0_M1` 上，所以设备树里需要把对应的引脚设置为 `PWM` 功能，但是数据手册里该引脚默认功能是 `GPIO` ,所以还得用 `pinctrl` 修改一下引脚复用功能
4. 其余就是一些 `GPIO` 正常配置成 `GPIO` 功能即可



## 2.2 设备树

### 2.2.1 RK平台显示子系统

[Rockchip_Developer_Guide_DRM_Display_Driver_CN](../核心参考文献库/Rockchip_Developer_Guide_DRM_Display_Driver_CN.pdf)

![image-20251224145541716](./assets/image-20251224145541716.png)

**可以看出：**

1. `MIPI DSI HOST` 控制器是接在 `VOP` 后面的，所以设备树里涉及到 `DSI` 的时候肯定是需要配置使用哪个 `VP` 通道来接收 `SOC` 发送到 `MIPI DSI HOST` 的显示数据的
2. 然后 `MIPI DSI HOST` 又接到了显示面板（`Panel`）上面，所以还会有一个 `dsi_panel` 的节点，这里面配置的是和显示屏幕面板相关的内容



### 2.2.2 设备树

rk3568.dtsi

```c
dsi0: dsi@fe060000 {
		compatible = "rockchip,rk3568-mipi-dsi";
		reg = <0x0 0xfe060000 0x0 0x10000>;
		interrupts = <GIC_SPI 68 IRQ_TYPE_LEVEL_HIGH>;
		clocks = <&cru PCLK_DSITX_0>, <&cru HCLK_VO>, <&video_phy0>;//时钟来源
		clock-names = "pclk", "hclk", "hs_clk";//时钟名字
		resets = <&cru SRST_P_DSITX_0>;//复位句柄
		reset-names = "apb";//复位名字，必须是apb
		phys = <&video_phy0>;//DSI所使用的PHY的设备节点
		phy-names = "mipi_dphy";//使用的PHY的名字
		power-domains = <&power RK3568_PD_VO>;//电源域句柄
		rockchip,grf = <&grf>;//SOC的grf寄存器
		#address-cells = <1>;
		#size-cells = <0>;
		status = "disabled";

		ports {
			#address-cells = <1>;
			#size-cells = <0>;

			dsi0_in: port@0 {
				reg = <0>;
				#address-cells = <1>;
				#size-cells = <0>;

				dsi0_in_vp0: endpoint@0 {
					reg = <0>;
					remote-endpoint = <&vp0_out_dsi0>;
					status = "disabled";
				};

				dsi0_in_vp1: endpoint@1 {
					reg = <1>;
					remote-endpoint = <&vp1_out_dsi0>;
					status = "disabled";
				};
			};
		};
	};

i2c1: i2c@fe5a0000 {
		compatible = "rockchip,rk3399-i2c";
		reg = <0x0 0xfe5a0000 0x0 0x1000>;
		clocks = <&cru CLK_I2C1>, <&cru PCLK_I2C1>;
		clock-names = "i2c", "pclk";
		interrupts = <GIC_SPI 47 IRQ_TYPE_LEVEL_HIGH>;
		pinctrl-names = "default";
		pinctrl-0 = <&i2c1_xfer>;
		#address-cells = <1>;
		#size-cells = <0>;
		status = "disabled";
	};
```

rk3566-lubancat-dsi0-ebf410125_1080p.dtsi

```c
#include "rk3568-lubancat-dsi0-ebf410125_1080p.dtsi"

&route_dsi0 {
	status = "okay";
	connect = <&vp0_out_dsi0>;
};

&dsi0_in_vp0 {
	status = "okay";//启用VP0
};

&dsi0_in_vp1 {
	status = "disabled";
};

&dsi0_panel {
	reset-gpios = <&gpio3 RK_PA4 GPIO_ACTIVE_LOW>; //reset引脚一般是低电平有效，驱动最后会把reset引脚拉高，enable引脚一般是高电平有效，驱动最后会把enable引脚拉高
};

&gt911 {
	interrupt-parent = <&gpio3>;
	interrupts = <RK_PA1 IRQ_TYPE_LEVEL_LOW>;
	reset-gpios = <&gpio3 RK_PA2 GPIO_ACTIVE_LOW>;
	irq-gpios = <&gpio3 RK_PA1 GPIO_ACTIVE_HIGH>;
};

```

rk3568-lubancat-dsi0-ebf410125_1080p.dtsi

```c
&route_dsi0 {
	status = "okay";//路由DSI0状态
	connect = <&vp1_out_dsi0>;//连接到DSI0输入虚拟通道1
};

&video_phy0 {
	status = "okay";
};

&dsi0_in_vp0 {
	status = "disabled";
};
// RK3566有两个VOP，这里是用来链接VOP的，可以选择通道1或者通道0，我们选的是通道1
&dsi0_in_vp1 {
	status = "okay";//虚拟通道状态
};

&dsi0 {
	status = "okay";//DSI0控制器状态
	power-supply = <&mipi_dsi0_power>;//panel的电压要求，可选，regulator配置

	dsi0_panel: panel@0 {
		compatible = "simple-panel-dsi";
		reg = <0>;
		backlight = <&backlight>;//关联背光节点
		reset-gpios = <&gpio0 RK_PC5 GPIO_ACTIVE_LOW>;//复位引脚

		enable-delay-ms = <35>;// panel 开始接收视频数据到显示出第一帧画面所用的时间
		prepare-delay-ms = <6>;//panel 准备就绪并开始接受视频数据的时间
		reset-delay-ms = <0>;//
		init-delay-ms = <20>;
		unprepare-delay-ms = <0>;
		disable-delay-ms = <20>;

		size,width = <74>;
		size,height = <133>;

		dsi,flags = <(MIPI_DSI_MODE_VIDEO | MIPI_DSI_MODE_VIDEO_BURST | MIPI_DSI_MODE_LPM | MIPI_DSI_MODE_EOT_PACKET)>;
		dsi,format = <MIPI_DSI_FMT_RGB888>;
		dsi,lanes  = <4>;

		panel-init-sequence = [
			39 00 04 B9 FF 83 99
			15 00 02 D2 77
			39 00 10 B1 02 04 74 94 01 32 33 11 11 AB 4D 56 73 02 02
			39 00 10 B2 00 80 80 AE 05 07 5A 11 00 00 10 1E 70 03 D4	
			15 00 02 36 02
			39 00 2D B4 00 FF 02 C0 02 C0 00 00 08 00 04 06 00 32 04 0A 08 21 03 01 00 0F B8 8B 02 C0 02 C0 00 00 08 00 04 06 00 32 04 0A 08 01 00 0F B8 01		
			39 05 22 D3 00 00 00 00 00 00 06 00 00 10 04 00 04 00 00 00 00 00 00 00 00 00 00 01 00 05 05 07 00 00 00 05 40						
			39 05 21 D5 18 18 19 19 18 18 21 20 01 00 07 06 05 04 03 02 18 18 18 18 18 18 2F 2F 30 30 31 31 18 18 18 18		
			39 05 21 D6 18 18 19 19 40 40 20 21 06 07 00 01 02 03 04 05 40 40 40 40 40 40 2F 2F 30 30 31 31 40 40 40 40			
			39 00 11 D8 A2 AA 02 A0 A2 A8 02 A0 B0 00 00 00 B0 00 00 00			
			15 00 02 BD 01			
			39 00 11 D8 B0 00 00 00 B0 00 00 00 E2 AA 03 F0 E2 AA 03 F0			
			15 00 02 BD 02			
			39 00 09 D8 E2 AA 03 F0 E2 AA 03 F0			
			15 00 02 BD 00			
			39 00 03 B6 8D 8D
			39 05 37 E0 00 0E 19 13 2E 39 48 44 4D 57 5F 66 6C 76 7F 85 8A 95 9A A4 9B AB B0 5C 58 64 77 00 0E 19 13 2E 39 48 44 4D 57 5F 66 6C 76 7F 85 8A 95 9A A4 9B AB B0 5C 58 64 77
			05 C8 01 11 
			05 C8 01 29 
		];

		panel-exit-sequence = [
			05 78 01 28
			05 00 01 10
		];

		disp_timings0: display-timings {
			native-mode = <&dsi0_timing0>;
			dsi0_timing0: timing0 {
				clock-frequency = <131376000>;
				hactive = <1080>;
				vactive = <1920>;
				hsync-len = <10>;
				hback-porch = <20>;
				hfront-porch = <10>;
				vsync-len = <5>;
				vback-porch = <20>;
				vfront-porch = <10>;
				hsync-active = <0>;
				vsync-active = <0>;
				de-active = <0>;
				pixelclk-active = <0>;
			};
		};

		ports {
			#address-cells = <1>;
			#size-cells = <0>;
			port@0 {
				reg = <0>;
				panel_in_dsi: endpoint {
					remote-endpoint = <&dsi_out_panel>;
				};
			};
		};
	};

	ports {
		#address-cells = <1>;
		#size-cells = <0>;

		port@1 {
			reg = <1>;
			dsi_out_panel: endpoint {
				remote-endpoint = <&panel_in_dsi>;
			};
		};
	};
};

&i2c1 {
	status = "okay";
	clock-frequency = <100000>;

	gt911: gt911@5d {
		status = "okay";
		compatible = "goodix,gt911";
		reg = <0x5d>;
		interrupt-parent = <&gpio0>;
		interrupts = <RK_PB5 IRQ_TYPE_LEVEL_LOW>;
		reset-gpios = <&gpio0 RK_PB6 GPIO_ACTIVE_LOW>;
		irq-gpios = <&gpio0 RK_PB5 GPIO_ACTIVE_HIGH>;
		touchscreen-inverted-x = <1>;
        touchscreen-inverted-y = <1>;
	};
};
```

鲁班猫默认配置的时钟是 (1080+10+20+10)\*(1920+5+20+10)\*60 = 131376000 ，但是这么配置是错误的，时钟配置错误会影响到图像帧率，实际的时钟配置需要考虑的因素：

1. 色深是几位的，以及额外开销
2. MIPI DSI 是双边沿采集

实际计算公式：

![image-20251225191924853](./assets/image-20251225191924853.png)

所以应该是：

(1080+10+20+10)\*(1920+5+20+10)\*60*24/4/2 = 394128000 也就是 394MHz

# 3. 背光

屏幕背光 `backlight` 是通过一个 `PWM` 输出引脚来控制的，这个引脚需要具备 `PWM` 输出功能，结合上面的原理图可以看出，`LCD_BL_PWM` 是连接在了 `CPIO_C7_d` 上面，需要把设备树里这个引脚对应的功能复用成数据手册里描述的 `PWM_M0_M1` 功能，然后来控制屏幕的亮度：

![image-20251223173316333](./assets/image-20251223173316333.png)



# 4. 复位引脚

![image-20251225111554710](./assets/image-20251225111554710.png)



# 5. 配置 DSI 速率

# 6. 配置 MIPI 屏幕的 lanes 数

# 7. MIPI 屏幕初始化序列

# 8.屏幕参数调试

# 9. 调试 MIPI 屏幕步骤

## 9.1 和 SOC 以及屏幕原厂合作

* 不同的 `SOC` 的显示硬件设计也不一样，像 `RK` 就是有个 `VOP` ，别的 `SOC` 也会有不同的设计，需要根据芯片原厂的设计文档，才能知道设备树是怎么配的
* 不同的屏幕，他的屏幕参数，初始化序列等等也不一样，如果是大规模供货的屏幕，网上可能会有资料，没有资料的话就只能找屏幕原厂寻找帮助了



## 9.2 使用逻辑分析仪帮助

使用支持 `MIPI DSI LP` 的逻辑分析仪，`MIPI DSI` 协议会通过 `Data0` 这一对差分对来完成初始化序列的发送，而且是在 `LP` 模式下，速率并不高，初始化序列是 `MIPI` 屏幕驱动的重点，一般能正常发送初始化序列，大概率是可以正常显示的



## 9.3  MIPI 屏幕三大核心步骤

1. **屏幕背光： **一定最先搞定背光，背光不亮屏幕是不会显示任何东西的
2. **初始化序列：** 初始化序列正常，`MIPI` 屏幕才能正常显示
3. **屏幕的 `DPI` 参数：** 也就是 `HBP 、 HFP 、 VBP 、 VFP` 等这些参数



## 9.4 对 MIPI DSI  外设进行读写



## 9.5 判断 MIPI DSI 外设是否正常工作

## 9.6 如何支持 DCS 背光

## 9.7 使能 EOTP（EoT packet） 特性

## 9.8 发送 packet

## 9.9 如何使能非连续时钟

## 9.10 调试 D-PHY