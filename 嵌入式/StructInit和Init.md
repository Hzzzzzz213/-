列如
==TIM_TimeBaseStructInit==
给 `TIM_TimeBaseInitTypeDef` 结构体**填充一套默认参数**，防止结构体里面是随机垃圾值。

 如果不调用这个函数，局部结构体变量内存是乱的，没赋值的成员会是随机数，定时器工作异常。
```
TIM_TimeBaseInitStruct->TIM_Prescaler = 0;          //预分频PSC = 0
TIM_TimeBaseInitStruct->TIM_CounterMode = TIM_CounterMode_Up; //向上计数
TIM_TimeBaseInitStruct->TIM_Period = 0xFFFF;        //ARR自动重装载 最大值65535
TIM_TimeBaseInitStruct->TIM_ClockDivision = TIM_CKD_DIV1; //时钟不分频
TIM_TimeBaseInitStruct->TIM_RepetitionCounter = 0;  //高级定时器重复计数器，通用定时器无效
```

==TIM_TimeBaseInit()：==
把结构体里面的参数，真正写入定时器硬件寄存器，配置硬件。


流程
如果你**不调用 ==`TIM_TimeBaseStructInit(&table)`**==，只手动改其中 1‑2 个成员：

```
table.TIM_Prescaler = 7199;
// 其他成员：TIM_Period、CounterMode…全是随机乱数
TIM_TimeBaseInit(TIM2, &table);
```

 ==`TIM_TimeBaseInit()`== **会把结构体所有成员一股脑全部写到硬件寄存器**，那些随机垃圾值也写进寄存器 → 定时器乱套。

 如果你先执行 ==`TIM_TimeBaseStructInit(&table);`==
> 函数把结构体**全部成员填安全默认值**，之后你只改你需要的几项。 剩下没改的成员，存的是**合法默认值**，再交给`TIM_TimeBaseInit()`全部写入寄存器，硬件就正常。
