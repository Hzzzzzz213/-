# AFIO（Alternate‑Function I/O）复用功能 I/O 控制器

> **只存在于 STM32F1 系列**；F4/F7/H7/L4 没有独立 AFIO 外设，改用 GPIOx_AFR 寄存器配置复用功能。
> 
> 总线：**APB2 外设**，必须开启 AFIO 时钟才能操作它的寄存器。

## 三大核心功能（F1）

1. **引脚重映射 Remap**：把外设从默认引脚切换到备用引脚（USART1、SPI1、TIM 等）
2. **EXTI 外部中断引脚选择**：AFIO_EXTICR 寄存器，选择 PAx/PBx/PCx… 接到 EXTI 中断线
3. **调试端口配置**：关闭 JTAG，释放 PB3/PB4/PA15 用作普通 GPIO

### ⚠️关键误区

- 使用**默认复用引脚**（比如 USART1 默认 PA9 PA10）：**不需要开启 AFIO 时钟**
- 做**重映射 / EXTI 外部中断 / 修改 JTAG 调试口**：**必须开启 AFIO 时钟**

## 重要寄存器

1. **AFIO‑MAPR**：重映射 + 调试端口控制寄存器，实现引脚重映射、关闭 JTAG/SWDJTAG
2. **AFIO‑EXTICR1~EXTICR4**：外部中断选择寄存器，每 4 位选择一个 GPIO 端口连接对应 EXTI 线
3. **AFIO‑EVCR**：事件输出控制寄存器（较少用）

## 标准库示例代码

### 1）开启 AFIO 时钟

c

运行

```
// APB2总线下，操作AFIO寄存器前必须打开
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
```

### 2）USART1 重映射到 PB6 (TX) PB7 (RX)

c

运行

```
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB|RCC_APB2Periph_USART1, ENABLE);

//开启重映射
GPIO_PinRemapConfig(GPIO_Remap_USART1, ENABLE);

GPIO_InitTypeDef GPIO_InitStruct;
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_6;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AF_PP;  //复用推挽输出
GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOB, &GPIO_InitStruct);

GPIO_InitStruct.GPIO_Pin = GPIO_Pin_7;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IN_FLOATING; //浮空输入
GPIO_Init(GPIOB, &GPIO_InitStruct);
```

### 3）EXTI 外部中断，把 PB0 接到 EXTI0

c

运行

```
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB | RCC_APB2Periph_AFIO, ENABLE);

// AFIO配置：选择PB0作为EXTI0的信号源
GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource0);
```

## 寄存器直接操作（寄存器版本）

c

运行

```
// 开启AFIO时钟
RCC->APB2ENR |= RCC_APB2ENR_AFIOEN;

// EXTI0选择PB0：修改EXTICR1寄存器
AFIO->EXTICR[0] &= ~(0xF << 0);
AFIO->EXTICR[0] |= (0x01 << 0);  //Port‑B
```

## F1 vs F4 差异

表格

|项目|STM32F1（有 AFIO）|STM32F4/F7/H7（无 AFIO 外设）|
|---|---|---|
|复用控制|独立 AFIO 外设|每个 GPIO 自带 AFR 寄存器|
|重映射|AFIO_MAPR 全局重映射|每个引脚独立配置 AF0‑AF15|
|EXTI 引脚选择|AFIO‑EXTICR|依然保留 AFIO_EXTICR 寄存器|
|时钟|重映射、EXTI 要开 AFIO 时钟|不需要 AFIO 时钟，开 GPIO 时钟即可|

## 高频踩坑点

1. 写外部中断，忘记开启 AFIO 时钟，中断完全不触发。
2. 使用默认外设引脚，错误开启 AFIO 时钟（无害但多余）。
3. 重映射后忘记把引脚模式设置为**复用模式 GPIO_Mode_AF_PP**，外设无输出。
4. JTAG 调试占用 PB3 PB4 PA15，如果要用作普通 IO，需要在 AFIO‑MAPR 关闭 JTAG 功能。

