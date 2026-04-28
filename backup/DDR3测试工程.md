



# 测试DDR3

- 若硬件是自己设计的pcb，为了检验fpga硬件是否正常工作，需要进行测试
  - 抓DDR初始化信号是否拉高
  - 进行一个读写测试





### 上板时钟选择

- A7芯片
  - DDR芯片时钟：400M
  - 系统时钟：200M
  - 参考时钟：使用系统时钟
- K7芯片：
  - 的DDR芯片时钟800M





# DDR3测试工程

-  创建工程，添加MIG的IP核
- MIG配置：
  - AXI4接口
  - 400M的DDR3工作时钟
  - 选择DDR3型号
  - 根据几个DDR，选择Data Width，一个就选16
  - 系统时钟200M，no Buffer
  - 参考时钟：使用系统时钟
  - 打开Internal Vref
  - Pin/Bank选择模式：Fixed Pin out
  - 复制进来xdc文件，读入，验证合理性
- 例化MIG
  - 例化进来MIG核
  - app开头的三个置零，三个悬空；`mmcm_locked`悬空
  - 复位低有效：`aresetn`改为`ui_clk_sync_rst`
- 移植ADMA工程到此工程
- 生成example
  - 右键MIG核，Open IP Example Design
  - 复制`example_top.v`中的端口信号到自己的工程，15个信号
- *例化ADMA*







### 如何批量移植IP核？

- **设置ip容器**：
  - 设置- IP，勾选`Use Core Containers For IP`
- **复制ip核文件**：
  - 打开地址：`prj/prj.srcs/sources_1/ip`，复制需要的.xcix后缀的文件到新工程的`sources_1/past_rtl/`目录下
- **复制verilog文件**：
  - 复制原工程的`.v`后缀文件到新工程的`past_rtl`目录下
- **添加文件**：
  - 在vivado中添加所有源文件，勾选`Scan and add RTL include files into project`和`Copy sources into project`



