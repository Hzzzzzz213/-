```
GPIO_InitTypeDef GPIO_InitStructure;
```

- `GPIO_InitTypeDef`：**结构体类型**（类比 `int`）
- `GPIO_InitStructure`：**变量名**（类比 `a`）
- ## 通俗类比

```
int num;        // int是类型，num是普通变量
GPIO_InitTypeDef GPIO_InitStructure;
// GPIO_InitTypeDef是结构体类型，GPIO_InitStructure是结构体变量
```

## 例 1：指针传参（STM32 GPIO_Init 同款，重点掌握）

### 1. 先定义 typedef 结构体（和你截图一致）

c

运行

```
#include <stdio.h>

// 自定义结构体类型
typedef struct
{
    uint16_t pin;
    int mode;
    int speed;
} GPIO_InitTypeDef;

// 函数：形参是【结构体指针】，接收结构体地址
// 对应库函数 void GPIO_Init(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct);
void GPIO_Init(GPIO_InitTypeDef *cfg)
{
    // 指针用 -> 访问成员
    printf("引脚：%d，模式：%d，速度：%d\n", cfg->pin, cfg->mode, cfg->speed);
}

int main(void)
{
    // 定义结构体变量
    GPIO_InitTypeDef GPIO_InitStructure;
    // 给结构体成员赋值（.访问）
    GPIO_InitStructure.pin = 12;
    GPIO_InitStructure.mode = 1;
    GPIO_InitStructure.speed = 50;

    // 结构体指针传参：&取变量地址传给函数
    GPIO_Init(&GPIO_InitStructure);
    return 0;
}
```

### 对应你截图代码的关键点

c

运行

```
// 函数需要指针，所以必须加 &
GPIO_Init(GPIOB, &GPIO_InitStructure);
```

- `&GPIO_InitStructure`：取出结构体内存地址，只传 4/8 字节地址，不复制整个结构体，单片机高效。
- 函数内部用 `->` 读写成员。
##  补充

变量名可以随便改，不强制叫 `GPIO_InitStructure`，比如写成下面这样完全等价：

```
GPIO_InitTypeDef gpio_conf;
gpio_conf.GPIO_Pin = GPIO_Pin_12;
GPIO_Init(GPIOB, &gpio_conf);
```