```
BaseType_t xTaskCreate(...); // 函数声明
int a;                        // 变量定义
```

## 拆解语法

1. `BaseType_t` 和 `int` 都属于**类型说明符**
2. 后面跟着名字：
    - `xTaskCreate`：**函数名**，后面带括号参数，代表这是函数
    - `a`：**变量名**，没有括号，代表普通变量

### 语法模板对比

- `类型 函数名(参数列表);` → 函数声明 `BaseType_t xTaskCreate( ... );`
- `类型 变量名;` → 变量定义 `int a;`