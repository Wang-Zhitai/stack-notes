
# 一、DMA数据传输方向

* DMA数据传输是双向的，可以从内存到内存，也可以从外设到内存，内存到外设，外设到外设

# 二、DMA的使用
## 1、DMA初始化结构体成员

```c
// 演示DMA初始化结构体参数及选项示例
void DMA_Init(void) {
    // 1. DMA_BufferSize参数
    // 含义：指定DMA传输的数据单元个数
    // 类型：uint16_t
    // 示例用法，假设要传输10个数据单元
    uint16_t bufferSize = 10;
	
    // 2. DMA_DIR参数
    // 含义：指定DMA传输方向
    // 类型：uint32_t
    // 选项：
    // DMA_DIR_PeripheralSRC：数据传输从外设到内存
    // DMA_DIR_PeripheralDST：数据传输从内存到外设
    uint32_t dirOption1 = DMA_DIR_PeripheralSRC;
    uint32_t dirOption2 = DMA_DIR_PeripheralDST;
	
    // 3. DMA_M2M参数
    // 含义：指定是否允许内存到内存的传输
    // 类型：uint32_t
    // 选项：
    // DMA_M2M_Enable：允许内存到内存的传输
    // DMA_M2M_Disable：禁止内存到内存的传输
    uint32_t m2mOption1 = DMA_M2M_Enable;
    uint32_t m2mOption2 = DMA_M2M_Disable;
	
    // 4. DMA_MemoryBaseAddr参数
    // 含义：指定内存的基地址，即数据传输的目的地址（外设到内存传输时）或源地址（内存到外设传输时）
    // 类型：uint32_t
    // 示例用法，假设定义了一个数组，这里取其首地址作为内存基地址
    uint32_t memoryArray[10];
    uint32_t memoryBaseAddr = (uint32_t)&memoryArray[0];
	
    // 5. DMA_MemoryDataSize参数
    // 含义：指定内存中数据单元的大小
    // 类型：uint32_t
    // 选项：
    // DMA_MemoryDataSize_Byte：数据单元大小为字节（8位）
    // DMA_MemoryDataSize_HalfWord：数据单元大小为半字（16位）
    // DMA_MemoryDataSize_Word：数据单元大小为字（32位）
    uint32_t memoryDataSizeByte = DMA_MemoryDataSize_Byte;
    uint32_t memoryDataSizeHalfWord = DMA_MemoryDataSize_HalfWord;
    uint32_t memoryDataSizeWord = DMA_MemoryDataSize_Word;
	
    // 6. DMA_MemoryInc参数
    // 含义：指定内存地址是否在每次传输后自动递增
    // 类型：uint32_t
    // 选项：
    // DMA_MemoryInc_Enable：使能内存地址递增
    // DMA_MemoryInc_Disable：禁止内存地址递增
    uint32_t memoryIncEnable = DMA_MemoryInc_Enable;
    uint32_t memoryIncDisable = DMA_MemoryInc_Disable;
	
    // 7. DMA_Mode参数
    // 含义：指定DMA传输模式
    // 类型：uint32_t
    // 选项：
    // DMA_Mode_Normal：正常模式，完成一次传输后停止，需手动重启进行下一次传输
    // DMA_Mode_Circular：循环模式，传输完成后自动重新开始下一轮传输
    uint32_t modeNormal = DMA_Mode_Normal;
    uint32_t modeCircular = DMA_Mode_Circular;
	
    // 8. DMA_PeripheralBaseAddr参数
    // 含义：指定外设的基地址，即数据传输的源地址（外设到内存传输时）或目的地址（内存到外设传输时）
    // 类型：uint32_t
    // 示例：不同外设对应不同的固定地址，这里假设一个外设地址示例（实际需按具体外设确定）
    uint32_t peripheralBaseAddrExample = 0x40000000;
	
    // 9. DMA_PeripheralDataSize参数
    // 含义：指定外设中数据单元的大小
    // 类型：uint32_t
    // 选项与DMA_MemoryDataSize相同：
    // DMA_MemoryDataSize_Byte：数据单元大小为字节（8位）
    // DMA_MemoryDataSize_HalfWord：数据单元大小为半字（16位）
    // DMA_MemoryDataSize_Word：数据单元大小为字（32位）
    uint32_t peripheralDataSizeByte = DMA_MemoryDataSize_Byte;
    uint32_t peripheralDataSizeHalfWord = DMA_MemoryDataSize_HalfWord;
    uint32_t peripheralDataSizeWord = DMA_MemoryDataSize_Word;
	
    // 10. DMA_PeripheralInc参数
    // 含义：指定外设地址是否在每次传输后自动递增
    // 类型：uint32_t
    // 选项：
    // DMA_PeripheralInc_Enable：使能外设地址递增
    // DMA_PeripheralInc_Disable：禁止外设地址递增
    uint32_t peripheralIncEnable = DMA_PeripheralInc_Enable;
    uint32_t peripheralIncDisable = DMA_PeripheralInc_Disable;
	
    // 11. DMA_Priority参数
    // 含义：指定DMA通道的优先级
    // 类型：uint32_t
    // 选项（通常有多个级别，这里示例高、中、低三个常见级别概念）：
    // DMA_Priority_High：高优先级
    // DMA_Priority_Medium：中优先级
    // DMA_Priority_Low：低优先级
    uint32_t priorityHigh = DMA_Priority_High;
    uint32_t priorityMedium = DMA_Priority_Medium;
    uint32_t priorityLow = DMA_Priority_Low;
	
    // 以下是初始化DMA结构体示例（未实际使用，仅展示如何配置结构体各参数）
    DMA_InitTypeDef DMA_InitStruct;
    DMA_InitStruct.DMA_BufferSize = bufferSize;
    DMA_InitStruct.DMA_DIR = dirOption1;
    DMA_InitStruct.DMA_M2M = m2mOption1;
    DMA_InitStruct.DMA_MemoryBaseAddr = memoryBaseAddr;
    DMA_InitStruct.DMA_MemoryDataSize = memoryDataSizeByte;
    DMA_InitStruct.DMA_MemoryInc = memoryIncEnable;
    DMA_InitStruct.DMA_Mode = modeNormal;
    DMA_InitStruct.DMA_PeripheralBaseAddr = peripheralBaseAddrExample;
    DMA_InitStruct.DMA_PeripheralDataSize = peripheralDataSizeByte;
    DMA_InitStruct.DMA_PeripheralInc = peripheralIncEnable;
    DMA_InitStruct.DMA_Priority = priorityHigh;
	
    // 初始化DMA通道（这里以DMA1_Channel1为例，实际需按具体需求确定通道）
    DMA_Init(DMA1_Channel1, &DMA_InitStruct);
}
```

## 2、示例代码内存到内存搬运

```c
// 定义一个全局变量transfer_num，用于记录要传输的数据单元个数，初始化为0
uint8_t transfer_num = 0; 

// num表示要传输的数据单元数，SRCAddr表示数据传输源地址，DISAddr表示数据传输目的地址
void MyDMA_Init(uint8_t num, uint32_t SRCAddr, uint32_t DISAddr) {
    // 将传入的参数num的值赋给全局变量transfer_num，以便后续在DMA配置及相关操作中使用
    transfer_num = num; 
	
    // 使能DMA1外设的时钟
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE); 
    
    // DMA初始化结构体
    DMA_InitTypeDef DMA_InitTypeDefStructure; 
    // 设置DMA传输的数据单元个数
    DMA_InitTypeDefStructure.DMA_BufferSize = transfer_num;
    // 配置DMA的传输方向为从外设作为源地址，内存作为目的地址
    DMA_InitTypeDefStructure.DMA_DIR = DMA_DIR_PeripheralSRC; 
    // 使能内存到内存（Memory to Memory）的传输模式。
    DMA_InitTypeDefStructure.DMA_M2M = DMA_M2M_Enable; 
    // 指定数据传输的目的内存基地址（这里填我们的目的地址）
    DMA_InitTypeDefStructure.DMA_MemoryBaseAddr = DISAddr; 
    // 设置内存中数据单元的大小为字节（8位），每次传输的数据单元是以字节为单位进行操作
    DMA_InitTypeDefStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte; 
    // 使能内存地址递增功能，在每次传输完一个数据单元后，内存地址会自动增加相应的偏移量
    DMA_InitTypeDefStructure.DMA_MemoryInc = DMA_MemoryInc_Enable; 
    // 设置DMA传输模式为正常模式，在这种模式下，完成一次指定BufferSize数量后停止
    DMA_InitTypeDefStructure.DMA_Mode = DMA_Mode_Normal; 
    // 指定数据传输的外设基地址，因为我们想要内存到内存，这里填我们的内存基地址就行
    DMA_InitTypeDefStructure.DMA_PeripheralBaseAddr = SRCAddr; 
    // 设置外设中数据单元的大小也为字节（8位），与内存中数据单元大小设置保持一致
    DMA_InitTypeDefStructure.DMA_PeripheralDataSize = DMA_MemoryDataSize_Byte; 
    // 使能外设地址递增功能，每次传输完一个数据单元后，外设地址会自动增加相应的偏移量
    DMA_InitTypeDefStructure.DMA_PeripheralInc = DMA_PeripheralInc_Enable; 
    // 设置DMA通道的优先级为高优先级
    DMA_InitTypeDefStructure.DMA_Priority = DMA_Priority_High; 
    // 按照上述配置初始化DMA1的通道1
    DMA_Init(DMA1_Channel1, &DMA_InitTypeDefStructure); 
}

// MyDMA_Run函数用于启动DMA传输，并等待传输完成
void MyDMA_Run(void) {
    // 先关闭DMA1的通道1，否则可能出现配置异常的情况
    DMA_Cmd(DMA1_Channel1, DISABLE); 
	
    // 设置transfer_num，也就是设置本次DMA传输还剩余需要传输的数据单元个数
    DMA_SetCurrDataCounter(DMA1_Channel1, transfer_num); 
	
    // 重新开启DMA1的通道1，开启后DMA就会按照配置开始进行数据传输操作
    DMA_Cmd(DMA1_Channel1, ENABLE); 
	
    // 检查DMA1通道1的传输完成标志位（DMA1_FLAG_TC1）是否被置位（SET）
    while (DMA_GetFlagStatus(DMA1_FLAG_TC1)!= SET); 
}

//实验：把内存中的 uint8_t DataA[4] 用DMA拷贝到 uint8_t DataB[4]
uint8_t DataA[4] = {0xA0,0xA1,0xA2,0xA3};
uint8_t DataB[4] = {0x00};

int main(void)
{
	OLED_Init();
	OLED_Clear();
	MyDMA_Init(4,(uint32_t)DataA,(uint32_t)DataB);//给定数据单元个数和目的（源）地址，并按照配置初始化DMA
	while (1)
	{
		DataA[0]++;
		DataA[1]++;
		DataA[2]++;
		DataA[3]++;	
		OLED_ShowHexNum(1,1,DataA[0],2);
		OLED_ShowHexNum(1,5,DataA[1],2);
		OLED_ShowHexNum(1,9,DataA[2],2);
		OLED_ShowHexNum(1,13,DataA[3],2);
		Delay_s(1);	
		MyDMA_Run();//启动DMA，因为是正常模式，传输完一次就停了，下次再重新启动
		OLED_ShowHexNum(2,1,DataB[0],2);
		OLED_ShowHexNum(2,5,DataB[1],2);
		OLED_ShowHexNum(2,9,DataB[2],2);
		OLED_ShowHexNum(2,13,DataB[3],2);	
		Delay_s(1);
	}
}
```
