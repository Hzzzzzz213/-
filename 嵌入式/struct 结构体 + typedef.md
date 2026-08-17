# C 语言 struct 结构体 + typedef 完整笔记

## 一、基础概念

1. **关键字：`struct`**
    
    作用：**打包不同类型的数据**，把多个 char、int、float 等变量封装成一个整体，统一管理。
2. 原生结构体写法（不用 typedef）



```
// 定义结构体类型 struct 结构体名
struct Test{
    char x;
    int y;
    float z;
};
// 定义结构体变量
struct Test obj;
// 访问成员：变量.成员名
obj.x = 'A';
obj.y = 66;
obj.z = 1.23f;
```

缺点：每次定义变量都要写 `struct Test`，代码冗长。

## 二、typedef 更改结构体变量名

### 1. 语法模板



```
typedef struct{
    // 各种成员变量
    char x;
    int y;
    float z;
} 新类型名_t; // 后缀_t是行业规范，代表type类型
```

### 2. 示例代码（第一张截图）


```
// 用typedef给匿名结构体起别名 StructName_t
typedef struct{
    char x;
    int y;
    float z;
} StructName_t;

int main(void)
{
    // 无需写struct，直接用别名定义变量
    StructName_t c;
    StructName_t d;

    // 点号 . ：结构体普通变量访问成员
    c.x = 'A';
    c.y = 66;
    c.z = 1.23f;
    return 0;
}
```

核心好处：以后定义变量只写 `StructName_t 变量名`，简洁，嵌入式代码全部用这种格式。

## 三、结构体成员访问两种方式

### 方式 1：普通结构体变量 → `.` 点运算符



```
StructName_t c;
c.x = 'A';
c.y = 66;
c.z = 1.23;
```

### 方式 2：结构体指针变量 → `->` 箭头运算符


```
StructName_t *p = &c;
p->x = 'A';
p->y = 66;
p->z = 1.23;
```

区分口诀：

- 实体变量用 `.`
- 指针 / 地址变量用 `->`

## 四、STM32 标准库实战示例（GPIO_InitTypeDef）

### 源码对应笔记



```
// 定义GPIO初始化结构体，typedef简化类型名
typedef struct
{
    uint16_t GPIO_Pin;          // 要配置的GPIO引脚
    GPIOSpeed_TypeDef GPIO_Speed;// 引脚输出速度
    GPIOMode_TypeDef GPIO_Mode; // 引脚工作模式（输入/输出/复用等）
} GPIO_InitTypeDef; // 别名，STM32库全部以TypeDef结尾

// 使用示例
GPIO_InitTypeDef GPIO_InitStruct;
GPIO_InitStruct.GPIO_Pin = GPIO_Pin_0;
GPIO_InitStruct.GPIO_Mode = GPIO_Mode_Out_PP;
GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
```

### 工程规范说明

1. 结构体别名后缀 `_t` / `TypeDef`：约定俗成，一眼看出是自定义类型；
2. 结构体内部可以嵌套其他 typedef 类型（如上`GPIOSpeed_TypeDef`、`GPIOMode_TypeDef`）；
3. 作用：把一个外设所有配置参数打包成一个结构体，一次性传给初始化函数，代码整洁。

## 五、核心总结

1. `struct`：用来封装一组不同类型的数据；
2. `typedef`：给结构体起短别名，省去每次写`struct`；
3. 访问成员：
    
    - 实体变量：`变量.成员`
    - 结构体指针：`指针->成员`
    
4. 嵌入式开发标准模板：`typedef struct{...} XXX_TypeDef;`，STM32、单片机全部通用。