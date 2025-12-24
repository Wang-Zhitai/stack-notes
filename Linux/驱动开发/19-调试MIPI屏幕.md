# 调试MIPI屏幕

# 1. RK3566 MIPI DSI外设

**`SOC` 有一个 `MIPI DSI` 接口，相对应的就应该有一个 `MIPI DSI HOST` 控制器外设,该外设内核是符合标准 `MIPI` 协议的，此 `MIPI DSI HOST` 用于连接内核和 `D-PHY` ，`DSI HOST` 发出的所有并行数字信号（无论是时钟还是数据 `Lanes`）都必须经过同一个 `D-PHY` 模块转换为串行差分模拟信号后才能发送到 `PCB` 走线上。**

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

1. MIPI DSI HOST 是接在 VOP 后面的，所以设备树里涉及到 DSI 的时候肯定是需要配置使用哪个 VP 通道来接收 SOC 发送到 MIPI DSI HOST的显示数据的
2. 然后 MIPI DSI HOST 又接到了显示面板（Panel）上面，所以还会有一个 dsi_panel 的节点，这里面配置的是和显示屏幕面板相关的内容



### 2.2.2 设备树