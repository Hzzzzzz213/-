•SPI（Serial Peripheral Interface）是由Motorola公司开发的一种通用数据总线

•四根通信线：SCK（Serial Clock）、MOSI（Master Output Slave Input）、MISO（Master Input Slave Output）、SS（Slave Select）

•同步，全双工

支持总线挂载多设备（一主多从）

| 引脚                | 全称                                         | 方向（主机 ↔ 从机） | 作用                                                                  |
| ----------------- | ------------------------------------------ | ----------- | ------------------------------------------------------------------- |
| **SCK**           | Serial Clock 串行时钟                          | 主机 → 所有从机   | 同步时钟，由主机产生，控制数据移位速度，所有挂在这条 SPI 总线上的从机共用这根时钟线                        |
| **MOSI**          | Master Output Slave Input<br><br>主机输出，从机输入 | 主机 → 从机     | 主机往外发数据，送到从机的数据接收引脚；总线共享                                            |
| **MISO**          | Master Input Slave Output<br><br>主机输入，从机输出 | 从机 → 主机     | 从机返回数据给主机；**未选中的从机 MISO 引脚为高阻态**，不会干扰总线                             |
| **SS / CS / NSS** | Slave Select 从机选择                          | 主机 → 单个从机   | 片选信号，**每个从设备独立一根 SS 线**，不是共享总线！<br><br>一般低电平有效：拉低 = 选中该从机；拉高 = 取消选中 |

![[SPI.png]]
收发数据

### 时钟上升沿

**移位 / 输出（shift/change）**：把下一位数据放到 MOSI/MISO 线上
![[SPI-2.png]]
![[SPI-3.png]]
### 下降沿

**采样（sample）**：读取 MOSI/MISO 线上的电平，存进移位寄存器
![[SPI-4.png]]

##                起始条件                                            终止条件
![[SPI-6.png]]





![[SPI-8.png]]

高阻态：**MISO 是从机输出引脚，SS 还没拉低、从机没被选中时，从机不能驱动 MISO 总线，必须进入高阻（Hi-Z）**，避免和其他设备打架。