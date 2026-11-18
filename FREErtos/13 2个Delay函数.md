**reeRTOS 工程，一律优先使用 vTaskDelay；不要用 mdelay 忙等待。**

- mdelay：占着 CPU 死等（裸机用）
- vTaskDelay：任务阻塞，让出 CPU（RTOS 专用）


### 两个 Delay 函数：

- `vTaskDelay`：至少等待指定个数的 Tick Interrupt 才能变为就绪状态
- `xTaskDelayUntil`：等待到指定的绝对时刻，才能变为就绪态。

|项目|vTaskDelay|xTaskDelayUntil|
|---|---|---|
|类型|相对延时|绝对周期延时|
|计时起点|调用函数的瞬间|上一次唤醒的时刻|
|周期稳定性|不稳定，业务耗时会叠加漂移|稳定，固定周期执行|
|使用场景|简单延时，不要求严格周期|需要固定周期任务（周期性采集、串口定时发送）|
|返回值|void，无返回|BaseType_t，|


```c
void vTaskDelay( const TickType_t xTicksToDelay );
/* xTicksToDelay: 等待多少个Tick */



/* pxPreviousWakeTime: 上一次被唤醒的时间
 * xTimeIncrement: 要阻塞到(pxPreviousWakeTime + xTimeIncrement)
 * 单位都是Tick Count */
BaseType_t xTaskDelayUntil( TickType_t * const pxPreviousWakeTime, const TickType_t xTimeIncrement );
```