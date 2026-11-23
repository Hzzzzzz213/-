在 STM32 的 HAL 库开发中，重定向 `printf` 的根本原因在于：**标准 C 库的 `printf` 函数默认不知道 STM32 的硬件长什么样，它需要一个“出口”来发送字符。**

### 1. 为什么需要重定向？

*   **标准库的默认行为**：在 PC 电脑上，`printf` 默认输出到“标准输出设备”（即屏幕控制台）。但 STM32 这种单片机（裸机环境）没有屏幕，也没有操作系统接管标准输出。
*   **底层缺失**：`printf` 是一个高级函数，它内部最终会调用一个底层的“单字符输出函数”（通常是 `fputc` 或 `__io_putchar`）。在标准库里，这个底层函数是空的或者指向无效的硬件地址，所以直接调用 `printf` 什么都不会发生（或者进入 HardFault）。
*   **为了调试方便**：工程师希望像在电脑上编程一样，使用 `printf("Value: %d\r\n", val);` 这种极其方便的方式来格式化字符串、打印变量，通过串口助手在电脑上查看，从而极大地提高调试效率。

### 2. 重定向的底层原理

重定向的本质就是**“狸猫换太子”**：我们自己在代码里重新实现那个底层的字符发送函数（如 `fputc`），把原本标准的输出路径，替换成 HAL 库的串口发送函数 `HAL_UART_Transmit()`。

当程序执行 `printf("Hello");` 时，流程如下：
1.  `printf` 解析字符串，发现字符 `'H'`。
2.  `printf` 内部调用底层函数 `fputc('H', stdout)`。
3.  **（重定向发生处）** 我们重写的 `fputc` 函数被调用，它内部执行 `HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);`。
4.  字符 `'H'` 通过串口 TX 引脚发送出去。

### 3. 如何实现重定向？（常见代码示例）

根据你使用的编译器环境（Keil MDK 或 GCC/STM32CubeIDE），重定向的写法不同：

#### 方式一：Keil MDK (ARMCC/ARMCLANG)
需要重写 `fputc` 函数，并且**必须在 Keil 的魔术棒设置中勾选 `Use MicroLIB`**（微库，一个精简版的 C 库，专为单片机设计）。

```c
#include <stdio.h>
// 假设你要用 USART1 打印
extern UART_HandleTypeDef huart1; 

// 重定向 fputc 函数
int fputc(int ch, FILE *f)
{
    // 每次发送一个字节，阻塞模式（超时时间根据情况设）
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);
    return ch;
}
```

#### 方式二：GCC 编译器 (STM32CubeIDE / CLion 等)
GCC 环境下，底层调用的是 `_write` 或者 `__io_putchar`。

```c
#include <stdio.h>
extern UART_HandleTypeDef huart1;

// 方式 A：重定向 _write (更底层)
int _write(int file, char *ptr, int len)
{
    HAL_UART_Transmit(&huart1, (uint8_t *)ptr, len, HAL_MAX_DELAY);
    return len;
}

// 方式 B：重定向 __io_putchar (需要在 syscalls.c 中取消注释对应实现)
int __io_putchar(int ch)
{
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, HAL_MAX_DELAY);
    return ch;
}
```

### ⚠️ 重定向时的几个大坑（注意事项）

1.  **阻塞问题**：上面的例子使用的是 `HAL_UART_Transmit`（阻塞发送）。如果你在很高频的中断服务函数（ISR）里调用 `printf`，会导致 CPU 卡在发送上，严重影响系统实时性，甚至导致程序跑飞。
    *   **解决**：**绝对不要在中断里使用阻塞的 `printf`**。如果非要打印，可以使用**DMA+空闲中断**的异步发送方式。
2.  **宏定义关闭**：如果使用了 MicroLIB 并且发现打不出来，检查是否在编译器中开启了 Semi-hosting（半主机模式），单片机环境必须关闭它。
