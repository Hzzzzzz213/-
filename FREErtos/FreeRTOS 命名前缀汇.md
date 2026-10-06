
> FreeRTOS 采用匈牙利命名法，**只是命名习惯，不是 C 语言关键字**，方便一眼看懂变量类型

表格

| 前缀         | 全称含义           | 通俗解释                          | 例子                       |
| ---------- | -------------- | ----------------------------- | ------------------------ |
| **pd**     | Project Data   | 用来标记执行结果，代表成功 / 失败，整数         | pdPASS、pdFALSE           |
| **x**      | BaseType_t     | 有符号整数；函数名带`x`一般返回`BaseType_t` | xTaskCreate、xQueueSend   |
| **px**     | pointer + x    | `p`= 指针，指向结构体 / 复杂对象的指针       | pxTaskCode、pxCreatedTask |
| **pc**     | pointer + char | `p`指针 + `c`char，字符指针、字符串      | pcName                   |
| ==**pv**== | pointer + void | `p`指针 + `v`void，万能指针，可以指向任意类型 | pvParameters             |
| **ux**     | unsigned + x   | `u`无符号 + `x`BaseType_t，无符号长整数 | uxPriority、uxTicksToWait |
| **us**     | unsigned short | 无符号短整型，数值不能为负数                | usStackDepth             |
| **uc**     | unsigned char  | 无符号字符 / 字节，范围 0~255           | ucQueueNumber            |

## 极简口诀

1. **p** 开头 → 指针
2. **u** 开头 → 无符号（不能是负数）
3. c=char，v=void，x=BaseType_t，s=short
4. **pd** 专门代表函数执行结果（成功 / 失败）