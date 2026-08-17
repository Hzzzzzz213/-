![[Pasted image 20260812161258.png]]![[Pasted image 20260812161400.png]]•EXTI（Extern Interrupt）外部中断

•EXTI可以监测指定GPIO口的电平信号，当其指定的GPIO口产生电平变化时，EXTI将立即向NVIC发出中断申请，经过NVIC裁决后即可中断CPU主程序，使CPU执行EXTI对应的中断程序

•支持的触发方式：上升沿/下降沿/双边沿/软件触发

•支持的GPIO口：所有GPIO口，==但相同的Pin不能同时触发中断==

•通道数：16个GPIO_Pin，外加PVD输出、RTC闹钟、USB唤醒、以太网唤醒

触发响应方式：中断响应/事件响应
![[Pasted image 20260812161529.png]]
![[Pasted image 20260812161554.png]]
•在STM32中，AFIO主要完成两个任务：复用功能引脚重映射、中断引脚选择

![[Pasted image 20260812162230.png]]