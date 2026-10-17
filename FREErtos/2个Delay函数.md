**reeRTOS 工程，一律优先使用 vTaskDelay；不要用 mdelay 忙等待。**

- mdelay：占着 CPU 死等（裸机用）
- vTaskDelay：任务阻塞，让出 CPU（RTOS 专用）