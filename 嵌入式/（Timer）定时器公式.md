# PSC 和 ARR 参数==怎么==算（结合你图片的公式）

> 公式： $$CK_CNT_OV=\frac{CK_PSC}{((\boldsymbol{PSC+1}))\times((\boldsymbol{ARR+1}))}$$

- $CK_PSC$：定时器输入时钟，STM32F1的TIM2，**72 MHz = 72000000 Hz**
- $PSC$：代码写的预分频寄存器值，实际分频系数是 $\boldsymbol{(\boldsymbol{PSC+1})}$
- $ARR$：代码写的自动重装载寄存器值，实际计数次数 $\boldsymbol{(\boldsymbol{ARR+1})}$
- $CK_CNT_OV$：溢出频率（每秒溢出多少次）
- 溢出周期 $T=\frac{1}{CK_CNT_OV}$，单位秒

## 你的代码

```
TIM_Prescaler = 7200 - 1;   //PSC=7199，实际分频系数 =7200
TIM_Period    = 10000 - 1;  //ARR=9999，实际计数次数=10000
```

### 第一步：算计数器时钟 CK_CNT

$$CK_CNT=\frac{CK_PSC}{(\boldsymbol{PSC+1})}=\frac{72000000}{7200}=10000\ \text{Hz}$$ 👉 计数器CNT每**1/10000 s = 0.1 ms**加1。

### 第二步：算溢出频率、溢出周期

$$CK_CNT_OV=\frac{CK_CNT}{(\boldsymbol{ARR+1})}=\frac{10000}{10000}=1\ \text{Hz}$$ 频率1Hz → 周期 $\boldsymbol{T=1\ \text{s}}$。

> 含义：计数器从0数到9999，刚好1秒，发生一次更新溢出事件。

---

## 🎯需求反过来推PSC、ARR（实际写代码的思考方式）

> 需求举例：想要**500ms（0.5秒）定时中断**，TIM2时钟72MHz 目标周期 $T=0.5\ \text{s}$，溢出频率 $f_{ov}=1/T=2\ \text{Hz}$

$$f_{ov}= \frac{72000000}{((\boldsymbol{PSC+1}))\times((\boldsymbol{ARR+1}))}=2$$ $$((\boldsymbol{PSC+1}))\times((\boldsymbol{ARR+1}))= \frac{72000000}{2}=36,000,000$$

⚠️寄存器硬件限制：

- PSC寄存器：**0~65535（16位）**
- ARR寄存器：**0~65535（16位）** 也就是 $(\boldsymbol{PSC+1}) ≤65536$；$(\boldsymbol{ARR+1}) ≤65536$。

### 选参数思路

1. 先选PSC，把计数器时钟降到合适档位 比如选 `\(\boldsymbol{PSC+1}\) =7200`，PSC=7199；计数器时钟10000Hz $$(\boldsymbol{ARR+1})=\frac{72000000}{((\boldsymbol{PSC+1}))\times f_{ov}}=\frac{72000000}{7200\times2}=5000$$ 得到：

- PSC = 7200‑1
- ARR = 5000‑1

```
TIM_TimeBaseInitStructure.TIM_Prescaler = 7200-1;
TIM_TimeBaseInitStructure.TIM_Period    = 5000-1;
```

> 校验：$T=\frac{7200\times5000}{72000000}=0.5\ \text{s}$ ✔

### 为什么代码都要写 `xxx‑1`？

> 寄存器里面存的是**N‑1**。 想分7200倍，寄存器填7199；想数10000次溢出，寄存器填9999。 公式永远用 `\(\boldsymbol{PSC+1}\)`、`\(\boldsymbol{ARR+1}\)` 去运算，写代码赋值的时候再减1。

### 2. 直观换算口诀（方便以后速算）

在嵌入式定时器配置中，你只需要记住这个换算链条：
### 1Hz是1秒

- **1 kHz**（1000 Hz） = 周期 **1 ms**（0.001秒）
    
- **10 kHz**（10000 Hz） = 周期 **0.1 ms**（100 μs） ← **你当前的状态**
    
- **100 kHz**（100000 Hz） = 周期 **0.01 ms**（10 μs）
    
- **1 MHz**（1000000 Hz） = 周期 **0.001 ms**（1 μs）

