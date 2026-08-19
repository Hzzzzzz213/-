![[Pasted image 20260809175513.png]]
# GPIO 模式全称 + 宏名 + 中文翻译

> STM32F1 标准库，`stm32f10x_gpio.h`

1. **GPIO_Mode_IN_FLOATING**

- 全称：`GPIO_Mode_Input_Floating`
- 翻译：**浮空输入模式**

> IN = Input 输入；FLOATING = 浮空、悬浮。内部不上拉不下拉，电平随外部信号；悬空时电平不确定。

2. **GPIO_Mode_IPU**

- 全称：`GPIO_Mode_Input_Pull‑Up`
- 翻译：**上拉输入模式**

> I=Input 输入，PU=Pull‑Up 上拉。内部接上拉电阻，引脚悬空默认高电平。CountSensor、按键常用。

3. **GPIO_Mode_IPD**

- 全称：`GPIO_Mode_Input_Pull‑Down`
- 翻译：**下拉输入模式**

> I=Input 输入，PD=Pull‑Down 下拉。内部接下拉电阻，引脚悬空默认低电平。

4. **GPIO_Mode_AIN**

- 全称：`GPIO_Mode_Analog_IN`
- 翻译：**模拟输入模式**

> AIN = Analog Input。关闭数字输入缓冲，直接给 ADC 模拟采集使用。

5. **GPIO_Mode_Out_OD**

- 全称：`GPIO_Mode_Output_Open‑Drain`
- 翻译：**通用开漏输出模式**

> Out=Output 输出；OD=Open‑Drain 开漏。输出高为高阻态，输出低接 GND；需要外部上拉电阻。用于 I2C、电平转换。

6. **GPIO_Mode_Out_PP**

- 全称：`GPIO_Mode_Output_Push‑Pull`
- 翻译：**通用推挽输出模式**

> Out=Output 输出；PP=Push‑Pull 推挽。高电平接 VDD，低电平接 VSS；驱动能力强，LED 点灯最常用。

7. **GPIO_Mode_AF_OD**

- 全称：`GPIO_Mode_Alternate_Function_Open‑Drain`
- 翻译：**复用功能开漏输出模式**

> AF=Alternate Function 复用功能；OD=Open‑Drain 开漏。引脚由片上外设接管，不是 CPU 直接控制，例：硬件 I2C_SCL/SDA。

8. **GPIO_Mode_AF_PP**

- 全称：`GPIO_Mode_Alternate_Function_Push‑Pull`
- 翻译：**复用功能推挽输出模式**

> AF=Alternate Function 复用功能；PP=Push‑Pull 推挽。外设接管引脚；串口 TX、SPI SCK/MOSI 使用。

## 缩写速记小笔记

表格

|宏缩写|英文缩写含义|
|---|---|
|IN|Input 输入|
|PU|Pull‑Up 上拉|
|PD|Pull‑Down 下拉|
|AIN|Analog Input 模拟输入|
|Out|Output 输出|
|OD|Open‑Drain 开漏|
|PP|Push‑Pull 推挽|
|AF|Alternate Function 复用功能|

### 补充记忆

- **IN_xxx**：4 个输入模式；
- **Out_xxx**：CPU 直接控制输出；
- **AF_xxx**：片上外设接管引脚（复用功能）。