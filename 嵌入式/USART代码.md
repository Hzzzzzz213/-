```c
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
```
**复用 AF** 普通 GPIO 输出（Out_PP）：由 CPU 写寄存器控制引脚高低电平。 复用模式：引脚交给片内外设（USART、TIM）接管。 👉 TX 引脚要输出起始位、数据位、停止位这些串口波形，**必须让 USART 硬件来驱动引脚**，所以要开复用。

•HEX模式/十六进制模式/二进制模式：以原始数据的形式显示
•文本模式/字符模式：以原始数据编码后的形式显示
![[USART代码.png|383]]