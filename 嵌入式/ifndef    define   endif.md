# C 语言头文件保护宏：#ifndef / #define / #endif 完整用法

## 一、三条指令分别含义

1. `#define ABC`
    
    宏定义，作用：给名字`ABC`做标记，编译器会记住 “`ABC`已被定义过”。
2. `#ifndef __XX_H__`
    
    `#ifndef` = if not defined，翻译：**如果 `__XX_H__` 这个名字还没有被定义**，才执行下面代码。
3. `#endif`
    
    预编译分支结束标记，和`#ifndef`成对出现，相当于预编译代码块的结束括号。

---

## 二、完整标准模板（.h 头文件固定写法）

c

运行

```
#ifndef __XX_H__      // 第一步：判断是否没定义__XX_H__
#define __XX_H__      // 第二步：如果没定义，立刻定义它

// 这里放头文件真正内容：函数声明、结构体、宏、引脚定义等
void Key_Scan(void);
#define KEY_PIN PA0

#endif                // 第三步：结束判断分支
```

### 执行逻辑（防重复包含核心原理）

1. **第一次引入这个头文件**
    
    程序里第一次`#include "xx.h"`：
    
    - `__XX_H__`从未定义 → `#ifndef`条件成立
    - 执行`#define __XX_H__`，打上标记
    - 加载中间所有函数、定义代码
    - 走到`#endif`，结束
    
2. **第二次重复引入同一个头文件**
    
    别的文件再次`#include "xx.h"`：
    
    - `__XX_H__`已经被第一次定义过 → `#ifndef`条件不成立
    - 直接跳过中间全部代码，直到`#endif`
    - 不会重复加载，避免「重复定义报错」
    

---

## 三、为什么必须这么写？

C 语言允许文件互相`#include`，很容易出现**头文件被重复包含**：

例：`main.c`包含`key.h`，`led.h`也包含`key.h`，编译时`key.h`内容会被加载两次，报重定义错误。

这套三段式宏就是**头文件保护卫士**，保证任意头文件只会被编译一次。

## 四、命名规范说明 `__XX_H__`

- 命名规则：双下划线开头、文件名大写、`_H_`结尾
    
    例：文件`key.h` → `__KEY_H__`；文件`uart.h` → `__UART_H__`
- 前后双下划线是约定，避免和你自己写的变量、宏重名

## 五、易错点提醒

1. 三句必须成对出现：`#ifndef`开头，末尾必须加`#endif`，缺一不可；
2. `#define`要紧跟在`#ifndef`下一行，不能写在文件末尾；
3. 这套代码只写在`.h`头文件，`.c`源文件不需要；
4. 区分近似指令：
    
    - `#ifdef` = if defined（如果已经定义）
    - `#ifndef` = if not defined（如果未定义，头文件保护专用）
    

## 六、最简实操示例（按键头文件 key.h）

c

运行

```
#ifndef __KEY_H__
#define __KEY_H__

#define KEY_IO PA0
void Key_Init(void);
uint8_t Key_Read(void);

#endif
```

在`main.c`、`led.c`随便多少次`#include "key.h"`都不会报错。
# #ifdef / #endif 预编译指令完整讲解

## 1. 各行代码含义

c

运行

```
#ifdef AAA
bfoahdaiu
#endif
```

- `#ifdef AAA`
    
    `#ifdef` = if defined，翻译：**如果宏 `AAA` 已经被 `#define` 定义过**，则编译中间的代码。
- `bfoahdaiu`
    
    条件代码段：只有满足`AAA已定义`，这行代码才会参与编译；未定义则直接忽略。
- `#endif`
    
    条件分支结束标记，必须和`#ifdef`成对使用，代表预编译判断的结束。

---

## 2. 两种运行情况

### 情况 1：代码上方写了 `#define AAA`

c

运行

```
#define AAA
#ifdef AAA
bfoahdaiu   // 这行会保留，参与编译
#endif
```

编译器识别到`AAA`存在，中间代码正常生效。

### 情况 2：没有 `#define AAA`

c

运行

```
// #define AAA  这行注释/删掉
#ifdef AAA
bfoahdaiu   // 整段直接被编译器删除，不参与编译
#endif
```