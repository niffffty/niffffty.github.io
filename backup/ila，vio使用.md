# ila使用

 功能：当程序下载到板子之后， 可以抓取FPGA代码中的信号，并将信号的波形在VIVADO界面以时序图的方 式呈现。 

### 法一：ip核添加ila

- 添加lia的ip核
  - 配置第一页修改Number of Probes，采样端口（输入的信号数）；Sample Data Depth，采样深度。
  - 第二页配置信号的位宽，与输入的信号位宽相同
- 例化ip核
  - 在源文件中找到ila_led.veo中的函数
- 生成bit流，烧录
- 弹出波形窗，使用与仿真类似







### 法二：setbug debug添加ila

- 在要测试的信号定义前边添加Language Templates中的mark的括号前缀

```verilog
//led.v
module led(
	input wire clk,
    input wire reset_n,
    (* MARK_DEBUG="ture")  output reg [3:0] led
)
```

- 综合
- 打开综合菜单下的set up debug，配置
- 生成bit流













# vio

 功能：①监测FPGA内部信号；②驱动FPGA内部信号。 



- 添加vio的ip核
- 配置：第一页输入信号/输出信号的数量；第2/3页为输入/输出信号的位宽
- 例化vio
- 生成bit流，烧录
- 在弹出的hw_vios中添加需要的信号，可以更改对应value值





















