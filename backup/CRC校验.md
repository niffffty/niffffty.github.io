# CRC校验

- 校验：发送数据时，带着校验一起发送；在接收端检测校验是否一致
- 特点：
  - 基于模2除法（异或）
  - 若模型为`CRC-8 x8+x2+x+1`，则展开为：x8+x7+x6+...+x0，取系数，则为1000 0011 1
- CRC校验步骤：
  - 把G（X）换成二进制
  - 将待校验数据转换成二进制后，在低位补0（模型是几，补几个0）
  - 进行模2除法，待校验数据当被除数，G(x)的表达式当除数
  - 得到模2除法的余数，就是CRC校验的值



# Verilog实现CRC校验

### 端口

- 输入：
  - `clk`
  - `reset`
  - `crc_din_vld`：CRC输入有效
  - `crc_din`：待校验数据
  - `crc_done`：校验成功
- 输出：
  - `crc_dout`：CRC输出待校验的值





### 代码思路

- 将待校验的数据，进行两位两位校验。校验完8位后，`crc_done`拉高，清空`crc_dout`



### 利用工具生成CRC步骤

- 选择一个CRC参数模型
- 在CRC校验的计算网站，判断初始值、输入/输出是否反转
- 在生成CRC verilog代码的网站上改写代码
  - 复制生成的列表代码
  - 替换代码：
    - 将`d`替换为`crc_din`
    - 将`c`替换为`crc_dout`
    - 将`newcrc`替换成`crc_dout`







![Image](https://github.com/user-attachments/assets/2b87b3ba-08d5-40d8-8611-ce28f84a89dd)



```verilog
`timescale 1ns / 1ps

module crc8_d8(
	input  wire			  clk,
	input  wire           reset,

	input  wire 		  crc_din_vld, //CRC输入有效
	input  wire [7:0]     crc_din,    //CRC输入待校验的数据
	output reg  [7:0]     crc_dout,   //CRC输出呆校验的值 
	input  wire           crc_done    //CRC校验完成 
    );

/*//写法一
always @(posedge clk) begin
    if (reset) 
       crc_dout <= 0;    //根据网站的初始值 
    else if (crc_done) 
       crc_dout <= 0;  //根据网站的初始值
    else if (crc_din_vld)begin
    	crc_dout[0] = crc_din[7] ^ crc_din[6] ^ crc_din[0] ^ crc_dout[0] ^ crc_dout[6] ^ crc_dout[7];
    	crc_dout[1] = crc_din[6] ^ crc_din[1] ^ crc_din[0] ^ crc_dout[0] ^ crc_dout[1] ^ crc_dout[6];
    	crc_dout[2] = crc_din[6] ^ crc_din[2] ^ crc_din[1] ^ crc_din[0] ^ crc_dout[0] ^ crc_dout[1] ^ crc_dout[2] ^ crc_dout[6];
    	crc_dout[3] = crc_din[7] ^ crc_din[3] ^ crc_din[2] ^ crc_din[1] ^ crc_dout[1] ^ crc_dout[2] ^ crc_dout[3] ^ crc_dout[7];
    	crc_dout[4] = crc_din[4] ^ crc_din[3] ^ crc_din[2] ^ crc_dout[2] ^ crc_dout[3] ^ crc_dout[4];
    	crc_dout[5] = crc_din[5] ^ crc_din[4] ^ crc_din[3] ^ crc_dout[3] ^ crc_dout[4] ^ crc_dout[5];
    	crc_dout[6] = crc_din[6] ^ crc_din[5] ^ crc_din[4] ^ crc_dout[4] ^ crc_dout[5] ^ crc_dout[6];
    	crc_dout[7] = crc_din[7] ^ crc_din[6] ^ crc_din[5] ^ crc_dout[5] ^ crc_dout[6] ^ crc_dout[7];    	
    end      
end*/

//写法2

wire [7:0] crc_data;

assign crc_data[0] = crc_din[7] ^ crc_din[6] ^ crc_din[0] ^ crc_dout[0] ^ crc_dout[6] ^ crc_dout[7]; 
assign crc_data[1] = crc_din[6] ^ crc_din[1] ^ crc_din[0] ^ crc_dout[0] ^ crc_dout[1] ^ crc_dout[6];
assign crc_data[2] = crc_din[6] ^ crc_din[2] ^ crc_din[1] ^ crc_din[0] ^ crc_dout[0] ^ crc_dout[1] ^ crc_dout[2] ^ crc_dout[6];
assign crc_data[3] = crc_din[7] ^ crc_din[3] ^ crc_din[2] ^ crc_din[1] ^ crc_dout[1] ^ crc_dout[2] ^ crc_dout[3] ^ crc_dout[7];
assign crc_data[4] = crc_din[4] ^ crc_din[3] ^ crc_din[2] ^ crc_dout[2] ^ crc_dout[3] ^ crc_dout[4];
assign crc_data[5] = crc_din[5] ^ crc_din[4] ^ crc_din[3] ^ crc_dout[3] ^ crc_dout[4] ^ crc_dout[5];
assign crc_data[6] = crc_din[6] ^ crc_din[5] ^ crc_din[4] ^ crc_dout[4] ^ crc_dout[5] ^ crc_dout[6];
assign crc_data[7] = crc_din[7] ^ crc_din[6] ^ crc_din[5] ^ crc_dout[5] ^ crc_dout[6] ^ crc_dout[7]; 

always @(posedge clk) begin
    if (reset) 
       crc_dout <= 0;    //根据网站的初始值 
    else if (crc_done) 
       crc_dout <= 0;  //根据网站的初始值
    else if (crc_din_vld)begin
	   crc_dout <= crc_data ; 	
    end      
end

endmodule

```

```verilog
//tb.v
`timescale 1ns / 1ps
module tb(

    );
	reg clk;
	reg reset;

	reg 		crc_din_vld;
	reg [7:0]   crc_din;
	reg         crc_done;
	wire [7:0]  crc_dout;

	initial begin
		clk = 0;
		forever #(10) 
		clk = ~clk;
	end

	initial begin
		reset = 1;
		#(200) 
		reset = 0;
	end

reg [7:0] cnt;

always @(posedge clk) begin
    if (reset) 
       cnt <= 0;  
    else if (cnt == 9) 
       cnt <= 0;
    else 
       cnt <= cnt + 1;    
end

always @(posedge clk) begin
    if (reset) begin
    	crc_din_vld <= 0;
    	crc_done    <= 0;
    end 
    else if (cnt == 1) 
     	crc_din_vld <= 1;      
    else if (cnt == 3)  
        crc_din_vld <= 0; 
    else if (cnt == 5)
        crc_din_vld <= 1;
    else if (cnt == 8) begin
        crc_din_vld <= 0; 
        crc_done    <= 1;  	
    end
    else begin
        crc_din_vld <= crc_din_vld; 
        crc_done    <= 0;     	
    end
end

always @(posedge clk) begin
    if (reset) 
       crc_din <= 0;  
    else if (crc_din_vld) 
       crc_din <= crc_din + 1;  
end

	crc8_d8 crc8_d8
		(
			.clk         (clk),
			.reset       (reset),
			.crc_din_vld (crc_din_vld),
			.crc_din     (crc_din),
			.crc_dout    (crc_dout),
			.crc_done    (crc_done)
		);

endmodule

```













# 工具

- CRC计算网站： [[CRC（循环冗余校验）在线计算_ip33.com](http://www.ip33.com/crc.html)](http://www.ip33.com/crc.html) 
- 生成CRC校验的verilog代码网站：https://jobs.keysight.com/location-belgium