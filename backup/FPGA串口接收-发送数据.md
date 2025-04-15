# 接收模块

### 变量

- 端口：
  - 输入：
    - `clk`
    - `reset`
    - `uart_rxd`：串口接收的数据
  - 输出：
    - `uart_rx_en`：接收完数据的标志
    - `uart_rx_data`：接收到的数据
- 参数：
  - `CLK_FREQ `：FPGA的时钟频率
  - `BAUD_RATE`：波特率，一秒传输多少bit数据
- 本地参数：
  - `BAUD_CNT_MAX `：波特计数的最大值
  - `BAUD_CNT_MAX_HALF`：波特计数的中间值
- 代码中用到的变量：
  - `uart_rxd_d0`，`uart_rxd_d1`：用于把接收的数据级联两层寄存器，防止亚稳态
  - `baud_cnt`：波特计数器，数到最大周期数时(即一个bit传输完毕)清零
  - `bit_cnt`：比特计数器，数哪一位数据。若一共1起始位、8数据位、1停止位，则范围是0-9
  - `rx_flag`：接收标志位，置1时接收数据
  - `bit_flag`：记数据标志位，表示到了波特计数的中间值，开始可以记数据







### 完整流程：

-  接收1个起始位、8个数据位、1个停止位
- `always1`将接收的数据打两拍，防止亚稳态
- `always2`检测到数据的下降沿，`rx_flag`拉高（开始接收数据）
- `always3`检测到`rx_flag==1`，`baud_cnt`持续加1，加到`BAUD_CNT_MAX `
- `always4`检测到`rx_flag==1`且`baud_cnt==BAUD_CNT_MAX`，即一位数据接收完毕，`bit_cnt`加1
- `always6`检测到`bit_cnt`在1-8之间，且`baud_cnt==BAUD_CNT_MAX_HALF`，即现在在8个数据位中且到达了中间位置(数据稳定)，此时`bit_flag`拉高，表示可以记数据
- `always7`检测到rx_flag拉高，且`baud_cnt`到达最大计数周期，且记了8个数据后，拉高`uart_rx_en`表示数据接收完毕
- `always8`检测到`bit_flag==1`，即可以记录数据时，将接收到的打两拍后的数据赋值给8位的`uart_rx_data`作为输出



**举例子：**

- 空闲：火车没来（rx_flag = 0），你啥也不干，秒表（baud_cnt）和车厢计数（bit_cnt）都是0。
- 火车头到：看到下降沿（起始位），打开笔记本（rx_flag = 1），开始计时。
- 记车厢： 
  - 每数434个时钟（baud_cnt = BAUD_CNT_MAX），一节车厢经过，bit_cnt加1。
  - 在数据车厢（bit_cnt = 1~8）的中间（baud_cnt = 217），记下车厢号码（uart_rxd_d1）到uart_rx_data。
- 火车尾到：bit_cnt = 9，最后一节车厢结束，挥旗（uart_rx_en = 1），合上笔记本（rx_flag = 0），准备下一列火车。





### 理解：

- `BAUD_CNT_MAX `是用来干什么的？

  - `CLK_FREQ `是时钟频率，如果50M，即每秒钟有50 000 000 个时钟周期（tick），每个周期是20纳秒

  - `BAUD_RATE`是波特率，如果是11 5200，即每秒传115200个bit，每个bit持续8680纳秒

  - 如果FPGA的时钟每20纳秒滴答一次，想知道一个bit传输什么时候结束，只需要计算8680纳秒需要多少个20纳秒，即8680/20= 434

  - 所以，传输一个bit的数据，需要数434个时钟周期，等数完434次滴答，就可以看下一个bit了

  - 类比：想象你在火车站，每节车厢（bit）停留8.68秒（8680微秒）。你手里的秒表（FPGA时钟）每0.02秒（20纳秒）滴答一次。为了知道一节车厢什么时候开走，你得数：8.68/0.02=434次滴答

    所以，数到434次滴答，就说明一个bit结束了，可以去看下一节车厢（下一个bit）。

- `BAUD_CNT_MAX_HALF`用来干什么？

  - 在一个bit的中间读，数据更稳定。`BAUD_CNT_MAX_HALF`是最大周期数的一半





![Image](https://github.com/user-attachments/assets/17d26fab-7d0e-4b14-8578-44ac4a3a9389)

```verilog
//uart_rx.v
`timescale 1ns / 1ps

module uart_rx #(
	parameter 		   CLK_FREQ  = 50000000,
	parameter 		   BAUD_RATE = 115200
)(
	input	           clk,
	input              reset,
	input              uart_rxd,      //串口接收RX

	output reg         uart_rx_en,    //接收数据使能
	output reg [7:0]   uart_rx_data	  //接受的数据	

    );
localparam BAUD_CNT_MAX        = CLK_FREQ  / BAUD_RATE;  //复位之前已经计算好了
localparam BAUD_CNT_MAX_HALF   = BAUD_CNT_MAX / 2;      //复位之前已经计算好了

reg  		uart_rxd_d0;
reg  		uart_rxd_d1;

reg [12:0] 	baud_cnt;
reg [3:0]  	bit_cnt;

reg         rx_flag;
reg         bit_flag;

//FPGA外部信号传入，打两拍，防止亚稳态
always @(posedge clk) begin
	uart_rxd_d0 <= uart_rxd;
    uart_rxd_d1 <= uart_rxd_d0;
end

always @(posedge clk ) begin
	if (~uart_rxd_d0 && uart_rxd_d1) 
		rx_flag <= 1'b1;
	else if (rx_flag && baud_cnt == BAUD_CNT_MAX && bit_cnt == 'd9) 
		rx_flag <= 1'b0;
end

always @(posedge clk) begin
	if (reset) 
		baud_cnt <= 13'd0;
	else if (~rx_flag) 
		baud_cnt <= 13'd0;
	else if (rx_flag && baud_cnt == BAUD_CNT_MAX)
		baud_cnt <= 13'd0;
	else if (rx_flag) 
		baud_cnt <= baud_cnt + 1'b1;
	else 
		baud_cnt <= baud_cnt;
end

always @(posedge clk ) begin
	if (reset) 
		bit_cnt <= 'd0;
	else if (~rx_flag)
		bit_cnt <= 'd0;
	else if (rx_flag && baud_cnt == BAUD_CNT_MAX)	
		bit_cnt <= bit_cnt + 1'b1;
	else 
		bit_cnt <= bit_cnt;	
end

always @(posedge clk ) begin
	if (bit_cnt >= 1 && bit_cnt <= 8 && baud_cnt == BAUD_CNT_MAX_HALF) 
		bit_flag <= 1'b1;
	else 
		bit_flag <= 1'b0;	
end

always @(posedge clk ) begin
	if (reset) 
		uart_rx_en <= 1'b0;
	else if (rx_flag && baud_cnt == BAUD_CNT_MAX && bit_cnt == 'd9) 
		uart_rx_en <= 1'b1;
	else 
		uart_rx_en <= 1'b0;	
end

//写法1
always @(posedge clk ) begin
	if (bit_flag) 
		uart_rx_data[bit_cnt - 1] <= uart_rxd_d1;
	else 
		uart_rx_data <= uart_rx_data;
end

//写法2，移位写法
/*always @(posedge clk ) begin
	if (bit_flag)
		uart_rx_data <= {uart_rxd_d1,uart_rx_data[7:1]};
	else 
		uart_rx_data <= uart_rx_data;
end*/
endmodule
```







