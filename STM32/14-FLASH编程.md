# 一、如何保护自己的程序防止被读出来

1. **禁用SWD引脚：** 只能解决使用ST-LINK读FLASH的问题，解决不了使用串口bootloader读FLASH的问题
2. **开启读保护：** **开启读保护**后就只能运行程序不能读写FLASH了（使用Flymcu软件可以通过串口烧录FLASH，连接前记得把自己的BOOT启动引脚改成使用串口bootloader启动），**取消读保护**只能通过FlyMCU把FLASH整个内容全部擦除，数据就全部丢失了！！！
3. **使用硬件安全策略：** TAMPER（BKP备份寄存器中的侵入检测功能）



# 二、FLASH编程注意事项

* STM32的FLASH写入**只支持每次写入一个半字**（也就是16位，两个字节）
* STM32F10xxx内嵌的闪存存储器可以用于**在线编程(ICP)** 或**在程序中编程(IAP)** 烧写。
	* **在线编程(In-Circuit Programming – ICP)** 方式用于更新闪存存储器的全部内容，它通过JTAG、SWD协议或系统加载程序(Bootloader)下载用户应用程序到微控制器中。 ICP是一种快速有效的编程方法，消除了封装和管座的困扰。
	* **在程序中编程(In-Application Programming – IAP)** 可以使用微控制器支持的任一种通信接口(如I/O端口、 USB、 CAN、 UART、 I2C、 SPI等)下载程序或数据到存储器中。 IAP允许用户在程序运行时重新烧写闪存存储器中的内容。然而， IAP要求至少有一部分程序已经使用ICP烧到闪存存储器中。
* **闪存接口是在AHB协议**上实现了对指令和数据的访问，它通过对存储器的预取缓存，加快了存储器的访问；闪存接口还实现了在所有工作电压下对闪存编程和擦除所需的逻辑电路，这里还包括访问和写入保护以及选择字节的控制。
* 使用**ST-LINK Utility**或者**FlyMCU**可以读写FLASH中的所有数据、开启读保护、擦除整个FLASH等功能，前者需要STLINK的支持，后者需要串口的支持（都需要相关的硬件工具才行！）
* 每**一次最多写一页**，超过了就会折返回来从头开始占位，会把页首的内容给挤掉



# 二、代码示例

```c
#include "stm32f10x.h"                  // Device header
#include "MyFLASH.h"

//读取指定地址：一个字节的内容
uint8_t MyFLASH_ReadByte(uint32_t addr)
{
	return *((__IO uint8_t *)addr); // volatile 关键字用于修饰可能被未知因素更改的变量，可以避免编译器优化导致的错误或不稳定的访问。	
}

//读取指定地址：半字（2个字节）的内容
uint16_t MyFLASH_HalfWord(uint32_t addr)
{	
	return *((__IO uint16_t *)addr); // volatile 关键字用于修饰可能被未知因素更改的变量，可以避免编译器优化导致的错误或不稳定的访问。	
}


//读取指定地址：一个字（4个字节）的内容
uint32_t MyFLASH_Word(uint32_t addr)
{
	return *((__IO uint32_t *)addr); // volatile 关键字用于修饰可能被未知因素更改的变量，可以避免编译器优化导致的错误或不稳定的访问。	
}

//写入半字，STM32的内置FLASH只支持一次写入半字，不能一次写入1个字节
void MyFLASH_WriteHalfWorld(uint32_t addr,uint16_t halfworld)
{
	//1.解锁
	FLASH_Unlock();
	//2.FLASH编程
	FLASH_ProgramHalfWord(addr,halfworld);
	//3.上锁
	FLASH_Lock();
}

//擦除指定页（一页是1KB的大小）
void MyFlash_ErasePage(uint32_t addr)
{
	//1.解锁
	FLASH_Unlock();
    //2.擦除
	FLASH_ErasePage(addr);
	//3.上锁
	FLASH_Lock();
}
```