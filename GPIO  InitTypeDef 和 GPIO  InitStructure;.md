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

##  补充

变量名可以随便改，不强制叫 `GPIO_InitStructure`，比如写成下面这样完全等价：

```
GPIO_InitTypeDef gpio_conf;
gpio_conf.GPIO_Pin = GPIO_Pin_12;
GPIO_Init(GPIOB, &gpio_conf);
```