# ![[Pasted image 20260818182446.png]]EXTI 外部中断完整配置流程

整体通路：**GPIO 引脚 → AFIO 多路选择 → EXTI 边沿检测 → NVIC 中断控制器 → CPU 执行中断服务函数**

> 基于 STM32F1，标准库逻辑，一步对应代码操作

## 1、开启时钟

1. 开启目标 GPIO 的 APB2 时钟（引脚硬件要工作）
2. **开启 AFIO 复用功能 APB2 时钟**（多路选择器，必开！漏开中断无效）

> ❗EXTI、NVIC 是内核相关，**没有时钟，不用开启**

c

运行

```
RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
```

## 2、GPIO 初始化，配置输入模式

把引脚配置成输入：上拉输入 / 下拉输入 / 浮空输入。

> 外部中断引脚必须是输入模式！不能推挽输出。

c

运行

```
GPIO_InitTypeDef GPIO_InitStruct;
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IPU; //上拉输入
GPIO_Init(GPIOA, &GPIO_InitStruct);
```

## 3、AFIO 配置：选择 GPIO 端口映射到 EXTI 线

寄存器：`AFIO->EXTICR`，选择哪一组 GPIO 接到 EXTI 通道。

> 例：PA0 → EXTI0；PB2 → EXTI2。
> 
> 同一根 EXTI 线，只能选一个端口，PA0、PB0 不能同时用外部中断。

c

运行

```
GPIO_EXTILineConfig(GPIO_PortSourceGPIOA, GPIO_PinSource0);
//含义：把PA0映射到EXTI0线路
```

## 4、配置 EXTI 模块（边沿检测、使能中断）

配置结构体`EXTI_InitTypeDef`

1. EXTI_Line：选择 EXTI 线 EXTI_Line0
2. EXTI_Mode：EXTI_Mode_Interrupt 中断模式（送往 NVIC）

> 区分：`EXTI_Mode_Event`事件模式，只输出硬件脉冲，**不进 CPU 中断**

3. EXTI_Trigger：触发边沿：下降沿 / 上升沿 / 双边沿
4. EXTI_LineCmd：ENABLE，开启这条线中断输出

c

运行

```
EXTI_InitTypeDef EXTI_InitStruct;
EXTI_InitStruct.EXTI_Line = EXTI_Line0;
EXTI_InitStruct.EXTI_Mode = EXTI_Mode_Interrupt;
EXTI_InitStruct.EXTI_Trigger = EXTI_Trigger_Falling; //下降沿触发
EXTI_InitStruct.EXTI_LineCmd = ENABLE;
EXTI_Init(&EXTI_InitStruct);
```

## 5、配置 NVIC 嵌套向量中断控制器

NVIC 属于 Cortex‑M3 内核，**优先级管理，使能中断通道**。

作用：EXTI 发过来中断请求，NVIC 判断要不要交给 CPU 响应。

设置：抢占优先级、子优先级，使能中断。

c

运行

```
NVIC_InitTypeDef NVIC_InitStruct;
NVIC_InitStruct.NVIC_IRQChannel = EXTI0_IRQn;  //EXTI0中断通道
NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority = 1; //抢占优先级
NVIC_InitStruct.NVIC_IRQChannelSubPriority = 1;        //子优先级
NVIC_InitStruct.NVIC_IRQChannelCmd = ENABLE;
NVIC_Init(&NVIC_InitStruct);
```

## 6、编写中断服务函数（CPU 执行）

> 函数名固定，启动文件定义好，不能自己随便改名

- EXTI0~EXTI4：各自独立中断函数
- EXTI5~9 共用 `EXTI9_5_IRQHandler()`
- EXTI10~15 共用 `EXTI15_10_IRQHandler()`

c

运行

```
void EXTI0_IRQHandler(void)
{
    // 判断：确认是EXTI0产生中断（读PR挂起标志）
    if(EXTI_GetITStatus(EXTI_Line0) != RESET)
    {
        //============业务代码============
        //按键处理、置标志位等
        //================================
        
        //✅必须清除中断挂起标志位，写1清零，不清除会反复进中断
        EXTI_ClearITPendingBit(EXTI_Line0);
    }
}
```

# 硬件触发完整执行链路

1. GPIO 引脚电平发生跳变
2. AFIO 选通通路，信号送入 EXTI 对应通道
3. EXTI 检测边沿，如果和配置触发条件匹配
4. EXTI 硬件置位 **PR 挂起寄存器**，向 NVIC 发送中断请求
5. NVIC 判断中断优先级，如果允许响应中断
6. CPU 暂停当前 main 主循环，保存寄存器现场，跳转到**中断服务函数**
7. 在中断服务函数执行业务逻辑，软件清除 PR 挂起标志
8. 中断返回，恢复现场，继续运行 main 函数
# EXTI 外部中断完整处理流程（STM32F1，结合框图）

整体数据流：

**GPIO 引脚电平变化 → AFIO 选择通路 → EXTI 边沿检测 → 产生中断请求 → NVIC 裁决 → CPU 响应中断 → 执行中断服务函数 → 软件清除中断挂起标志**

## 一、初始化阶段（程序启动只执行 1 次）

1. **开启时钟**
    
    - 开启对应 GPIO 时钟（APB2），配置引脚为输入模式（上拉 / 下拉 / 浮空输入）
    - 开启 **AFIO 时钟 (APB2)**，AFIO 负责多路开关选择哪个端口接到 EXTI 线
    
    > EXTI、NVIC 是内核相关，**不需要开时钟**
    
2. **AFIO 配置（EXTICR 寄存器）**
    
    设置 EXTICRx 寄存器：选择对应 GPIO 端口连接到 EXTI 线。
    
    例：PA0 → 将 EXTI0 通道选择 GPIOA。
    
    > 注意：EXTI0 通道只能选 PA0/PB0/PC0 其中一个，不能多个同时用。
    
3. **EXTI 模块配置**
    
    1. 设置触发边沿：上升沿、下降沿、双边沿触发
    2. 开启中断屏蔽位：使能这条 EXTI 线的**中断请求输出**（送给 NVIC）
    
    > 区分：中断输出给 NVIC；事件输出（框图 20 号线）给其他外设，不打扰 CPU。
    
4. **NVIC 内核中断配置**
    
    配置中断抢占优先级、子优先级，**使能该中断通道**。
    
    NVIC 是 Cortex‑M 内核部件，决定 CPU 要不要响应这个中断。
    

---

## 二、硬件触发流程（硬件自动完成，引脚电平变化时）

1. GPIO 引脚电平发生跳变（比如按键按下，电平由高变低）
2. 电平信号送到 AFIO 多路选择器，AFIO 根据 EXTICR 选择的端口，把信号送到对应的 EXTI 通道
3. **EXTI 边沿检测电路**检测电平变化是否匹配设定的触发边沿
    
    - 如果符合触发条件：
        
        ① EXTI 把**挂起寄存器 PR 对应位置 1**（标记：有中断待处理）
        
        ② 向 NVIC 发送中断请求信号
    
4. NVIC 收到请求，对比优先级：
    
    - 如果当前 CPU 允许响应这个中断：向 CPU 发信号，打断当前主程序，进入中断。
    - 如果优先级不够：等待，挂起，直到 CPU 可以处理。
    

## 三、软件处理（中断服务函数，CPU 执行）

1. CPU 保存主程序现场，跳转到对应的中断服务函数。

> 注意：EXTI5~9 共用`EXTI9_5_IRQHandler`；EXTI10‑15 共用`EXTI15_10_IRQHandler`，进入函数后必须读 PR 寄存器判断到底是哪根线触发。

2. 在中断函数里面写业务逻辑（例如：按键计数、标志位置 1）
3. **必须手动清除 EXTI‑PR 挂起标志位**（写 1 清零）

> ⚠重要：不清除标志，退出中断以后会立刻再次进中断，无限重复触发。

4. 中断返回，恢复现场，回到被打断的 main 主循环继续运行。

## 四、举实例：PA0 按键外部中断完整流程

1. 初始化：开启 GPIOA、AFIO 时钟，PA0 上拉输入；AFIO 选择 PA0 映射 EXTI0；EXTI 下降沿触发；NVIC 打开 EXTI0 中断。
2. 按键按下 → PA0 电平由高→低
3. AFIO 选通 PA0 信号送入 EXTI0
4. EXTI 检测到下降沿，PR 寄存器 bit0 置 1，向 NVIC 提交中断请求
5. NVIC 裁决通过，CPU 暂停 main 函数，跳转到`EXTI0_IRQHandler()`
6. 中断服务函数：置按键标志，`EXTI->PR |= EXTI_PR_PR0;`清除挂起标志
7. 退出中断，回到 main 循环。