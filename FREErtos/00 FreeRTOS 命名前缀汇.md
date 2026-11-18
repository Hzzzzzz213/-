
> FreeRTOS 采用匈牙利命名法，**只是命名习惯，不是 C 语言关键字**，方便一眼看懂变量类型

表格

|前缀|全称|含义|常见例子|
|---|---|---|---|
|pd|Project Data|执行结果（pdPASS /pdFAIL），整数|`pdPASS`、`pdFALSE`|
|x|BaseType_t|有符号整数；函数带 x 一般返回 BaseType_t|`xTaskCreate`、`xQueueSend`|
|v|void|无返回值函数|`vTaskDelay`、`vTaskDelete`|
|px|pointer + x|指向结构体 / 句柄的指针|`pxTaskCode`、`pxCreatedTask`|
|pc|pointer + char|字符指针 / 字符串|`pcName`|
|pv|pointer + void|void 万能指针|`pvParameters`|
|ux|unsigned + x|无符号 BaseType_t|`uxPriority`、`uxTicksToWait`|
|us|unsigned short|无符号短整型|`usStackDepth`|
|uc|unsigned char|无符号字节（0~255）|`ucQueueNumber`|
|ul|unsigned long|无符号长整型|`ulTimeout`|
|c|char|有符号字符|`cRxChar`|
|s|short|有符号短整型|`sErrorCount`|
|l|long|有符号长整型|`lResult`|
|e|enum|枚举类型|`eTaskState`、`eErr`|
|prv|private|文件私有静态函数|`prvTaskInit()`|
|**g_**|global|全局变量（工程自定义，非官方原生）|`g_uxTickCount`|
|**s_**|static|文件静态变量（工程自定义，非官方原生）|`s_ucBufIndex`|

## 极简口诀

1. **p** 开头 → 指针
2. **u** 开头 → 无符号（不能是负数）
3. c=char，v=void，x=BaseType_t，s=short
4. **pd** 专门代表函数执行结果（成功 / 失败）
# Portable（port，移植层）

一句话：**FreeRTOS 内核（task.c/list.c）写的是通用逻辑，不绑定任何芯片；portable 就是专门用来适配不同 CPU 的代码**

## 打个比方

内核就像游戏本体，游戏逻辑写死了； portable 相当于**显卡驱动**：游戏本身不用改，换显卡，只换驱动，游戏照样跑。

- 内核：只管任务排队、判断谁优先级高（纯 C，和芯片无关）
- Portable 移植层：负责和 CPU 硬件打交道

![[FreeRTOS入门与工程实践(基于DshanMCU-103)(最新版本，推荐) 2.pdf]]