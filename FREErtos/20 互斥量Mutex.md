
## 一、为什么需要互斥量

### 1. 互斥访问的必要性
多任务系统中，若任务A正在使用某资源且未用完，任务B也来使用，可能导致冲突。

常见场景：
- **访问外设**：如串口打印，A、B信息混杂。
- **读、修改、写操作**：如 `a = a + 8;` 内部分为读出、修改、写入三步，可能被打断。
- **非原子化访问**：修改变量、设置结构体、16位机器写32位变量等，操作过程可能被打断。
- **函数重入问题**：函数使用全局变量、静态变量、外设时，不是“可重入/线程安全”的；若正在被调用，必须阻止其他任务、中断再次调用。

这些必须被独占地使用的全局变量、函数代码等，称为**临界资源**。解决方法是访问临界资源前加锁，实现互斥。

### 2. 信号量实现互斥的不足
以信号量为例：
- 信号量初值为1；
- 任务A `take` 成功，进入临界区；
- 任务B `take` 失败，等待；
- 任务A `give`，任务B获得信号量。

前提：
- 任务B老实，不“撬门”；
- 没有其他任务乱 `give`。

问题：信号量本身不能保证“谁上锁，谁开锁”。

### 3. 互斥量的核心
- **量**：值为0、1；
- **互斥**：用于实现互斥访问；
- 核心约定：**谁上锁，就只能由谁开锁**。

注意：FreeRTOS的互斥锁在代码上并未严格实现这点——任务A获得互斥锁后，任务B竟然也可以释放互斥锁。“谁上锁、谁释放”只是约定。

---

## 二、互斥量的使用流程

1. 互斥量初始值为1；
2. 任务A想访问临界资源，先获得并占有互斥量，然后访问；
3. 任务B也想访问，先获得互斥量；若已被占有，则阻塞；
4. 任务A使用完毕，释放互斥量；
5. 任务B被唤醒、得到并占有互斥量，开始访问；
6. 任务B使用完毕，释放互斥量。

正常应保证：任务A占有互斥量期间，任务B、C等都无法释放它。但FreeRTOS未实现该限制。

---

## 三、互斥量函数

互斥量是一种特殊的二进制信号量。使用句柄表示互斥量。使用前需在 `FreeRTOSConfig.h` 中定义：

```c
#define configUSE_MUTEXES 1
```

### 1. 创建互斥量

动态分配内存：

```c
SemaphoreHandle_t xSemaphoreCreateMutex(void);
```

静态分配内存：

```c
SemaphoreHandle_t xSemaphoreCreateMutexStatic(StaticSemaphore_t *pxMutexBuffer);
```

返回值非 `NULL` 表示成功。

### 2. 删除、释放、获得

删除：

```c
void vSemaphoreDelete(SemaphoreHandle_t xSemaphore);
```

释放：

```c
BaseType_t xSemaphoreGive(SemaphoreHandle_t xSemaphore);
```

释放（ISR版本）：

```c
BaseType_t xSemaphoreGiveFromISR(
    SemaphoreHandle_t xSemaphore,
    BaseType_t *pxHigherPriorityTaskWoken
);
```

获得：

```c
BaseType_t xSemaphoreTake(
    SemaphoreHandle_t xSemaphore,
    TickType_t xTicksToWait
);
```

注意：
- 文中强调：**互斥量不能在ISR中使用**。
- 删除、give/take 等操作与一般信号量基本相同。

---

## 四、优先级继承

### 1. 优先级反转问题
低优先级任务获得锁后，可能因优先级太低而无法运行，导致高优先级任务也无法获得锁，形成“优先级反转”。

### 2. 优先级继承机制
假设：
- 任务A持有互斥锁；
- 更高优先级任务B也尝试获得该锁，但失败而阻塞；
- 则任务A临时继承任务B的优先级，以便尽快运行并释放锁；
- 任务A释放互斥锁后，恢复原优先级。

即：**互斥锁内部实现了优先级的提升与恢复。**

### 3. 示例：

在22_mutex_priority_inversion里，创建的是互斥量，代码如下：

```c
259 void car_game(void)

260 {

261      int x;

262      int i, j;

263      g_framebuffer = LCD_GetFrameBuffer(&g_xres, &g_yres, &g_bpp);

264      draw_init();

265      draw_end();

267

268      //g_xSemTicks = xSemaphoreCreateCounting(1, 1);

269      g_xSemTicks = xSemaphoreCreateMutex();

```
把第268行打开、第269行去掉，就会有优先级反转的问题。

把第268行去掉、第269行打开，就解决了优先级反转的问题。

22_mutex_priority_inversion的实验现象为：car1先运行一会；然后car2运行一会；接着car3任务启动，但是它无法获得互斥量而阻塞，并且提升了car1的优先级；于是：car1、car2交替运行（虽然car1的优先级高于car2，但是car1会使用vTaskDelay阻塞，car2就有机会运行了）；当car1运行到终点，是否了互斥量，car3就可以运行了。




## 五、本章要点总结

1. 多任务访问外设、全局变量、不可重入函数等临界资源时，需要互斥访问。
2. 信号量可实现互斥，但不能保证“谁上锁谁开锁”。
3. 互斥量是特殊二进制信号量，核心是“谁上锁，谁开锁”的约定。
4. FreeRTOS中任务B也可释放任务A持有的互斥量，代码上未严格限制。
5. 使用互斥量需配置 `configUSE_MUTEXES 1`。
6. 互斥量可动态或静态创建，操作函数与信号量类似。
7. **互斥量不能在ISR中使用。**
8. 互斥量支持优先级继承，可缓解优先级反转问题。



