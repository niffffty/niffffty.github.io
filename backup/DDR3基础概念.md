# DDR3概念

### 特点

- DDR3：第三代双倍速率同步动态随机存储器
- 特点：
  - 掉电无法保存数据，需要周期性刷新
  - 时钟上升沿/下降沿都传输数据
  - 突发传输，突发长度Burst Length一般为8
  - 驱动复杂，使用官方给的`MIG IP`核，读写`MIG`即间接读取`DDR3`
- 存储：
  - 存储到DDR3：先指定一个Bank地址，再指定行地址，再指定列地址
- DDR3的容量
  - 位宽：
    - bank位宽：表示DDR3内部bank数量的对数形式。如果bank位宽为3，则有2的三次方=8个bank
    -   行位宽：表示每个Bank中行地址的位数，用于寻址行的数量。如果行位宽为14，意味着有2的14次方=16384行
  - 数据位宽：表示每次读写操作的数据宽度，即芯片的I/O宽度。如果数据位宽是16，则每次访问传输16位数据
  - 地址大小：表示芯片的寻址能力，即可以访问的存储单元总数
  - 容量 = 地址大小*数据位宽，即：容量=bank位宽 * 行位宽 * 列位宽 * 存储单元容量 
  - 例：本次实验使用的 DDR3 芯片是南亚的 NT5CC128M16IP-DI，NT5CC128M16IP-DI 的 bank位宽为 3，行位宽为 14，列位宽为 10，所以它的地址大小等于 2^3*2^14*2^10（即 2^27=128M），数据位宽为 16bit，所以容量大小为 128M*16bit，也就是 256MByte。 





### 命名

- 以镁光公司的DDR3举例：MT41J 64M16 -125
  - **J**：封装
  - **64M16**：从数据手册中可以通过此参数找到bank、行、列数量
  - **M16**：传输数据的位宽为16
  - **-125**：速度等级，表示支持的最大时钟频率。tCK为1.25ns，支持的最大时钟频率为：1/1.25*10^3=800M
  - 带宽：最大时钟频率 * 位宽 = 800M * 16 * 2 = 25600 Mbit/s ，即25.6G





# MIG核 时钟架构

- MIG核
  - `sys_clk`：输入，**系统时钟**，频率无具体要求，来源于晶振或PLL
  - `ref_clk`：输入，**参考时钟**，频率建议200M，来源于晶振或PLL
  - `输出时钟`：输出，**DDR3的工作时钟**，频率取决于fpga时钟和DDR3的最大时钟频率
  - `ui_clk`：输出，**用户时钟**，MIG核给用户的时钟（频率一般是DDR3工作时钟的一半或四分之一，若DDR3大于400M，则必须四分之一）



# MIG核配置

### Memory Interface Generator

- 第二页：

  - AXI4 interface：Native与AXI4，是否使用AXI4接口

- Controller Options：

  - Clock Period：工作时钟频率，A7选400M，K7选800M
  - PHY to Controller Clock Ratio：1:2或1:4
  - Memory Part：型号，例程选MT41J128M16XX-125

  - Data Width：DDR3芯片总位宽，如果挂了2个16位的DDR3，则选32
  - Ordering：一般使用Strict模式

- AXI Parameter：

  - Data Width：位宽
  - Arbitration Scheme：读写优先级。MIG核与DDR3不能同时读写，但AXI4是有读写模式的，用来决定读/写优先

- Memory Options：

  - Input Clock Period：系统时钟，一般选200M

- FPGA Options：

  - System Clock：根据来源(晶振/差分/PLL)来选择
  - Reference Clock：同上
  - System Reset Polarity：复位信号的电平，例程选低有效

- IO Planning Options：

  - PIn/Bank Selection Mode ：不上板选New Desigh；上板选Fixed Pin Out





# MIG核端口信号

- 存储接口（DDR3芯片相关信号）：
  - DDR3开头的：MIG核与DDR3的接线
  - init_calib_complete：**初始化完成信号**，拉高表示完成，可读写
- 应用接口（用户端）：
  - ui_clk：MIG核给用户端的时钟
  - ui_clk_sync_rst：MIG核给用户端的复位信号
  - mmcm_locked：一般用不到，一般悬空
  - app开头的：用来做校准刷新，例化时三个置零，三个悬空
- 从接口写地址接口（AXI4）：
  - aresetn：AXI4从机端口的复位信号
  - s开头的：AXI4从机的信号
- 系统时钟：
  - sys_clk_i：系统时钟，给MIG核的，一般200M
  - sys_rst：**低有效**，给MIG核的复位信号





### 利用example+modelsim仿真





