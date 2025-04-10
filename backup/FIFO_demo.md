# fifo的ip核

- 特性：
  - 存数据，先入先出
  - size：深度/宽度
  - 同步FIFO与异步FIFO
- 时序
  - 写时序
    - wr_en(写使能)和din(读数据)对齐
  - 读时序
    - 读数据与读使能对齐
    - 读数据比读使能延迟一个clk



### 使用

- 找到FIFO generator

- 配置：

  - 第一页Fifo Implementation：
    - Common是同步，Independent异步
    - BRAM：数据大
    - DRAM：数据小
  - 第二页Read Mode：
    - Standard FIFO：读数据比读使能延迟一个clk
    - First W F T：对齐
  - 宽度/深度：默认18/128
  - Reset pin：复位引脚（此次demo不用）
  - 第四页勾选Data Count

- 例化ip核：从veo里复制

- ip核内容：

  - 端口信号：
    - full:为高时，fifo满了
    - empty：为高时，fifo没数据
      - 往fifo写数据时，过几个clk，才拉高
      - 读完数据时，立马拉高
    - data_count:表示FIFO里大约有多少数据

  

  

  

  

  

### 状态机

- 两个条件：
  - 状态机的跳转
  - 状态机的输出
- demo：
  - 目标：往fifo里面写入128个数据，等待一会，然后读出来128个数据
  - 流程：IDLE--写--等待--读--等待--写。。。
- 状态机的写法：
  - 一段状态机
  - 两段状态机
  - 三段状态机(最规范)







### 读写fifo的demo

![Image](https://github.com/user-attachments/assets/e1c8a707-f56f-42cf-96a0-c933e2ab2b34)

- 使用二段状态机：

  - 第一段：描述状态机的跳转
  - 第二段：描述状态机的输出

- 三段状态机：

  - 次态给现态
  - 跳转
  - 输出

  ```verilog
  //top.v
  //往FIFO写128个数据，等待10个单位，读128个数据，等待10个单位
  //再写128个数据，循环
  
  `timescale 1ns / 1ps
  
  module top(
  	input wire 		clk,
  	input wire      reset_n
      );
  
  /*------------------------------------------*\
                   状态机定义
  \*------------------------------------------*/
  reg [3:0]   cur_status; //现态
  reg [3:0]   nex_status; //次态
  
  localparam  IDEL   = 4'b0000;
  localparam  WRITE  = 4'b0001;
  localparam  WAIT1  = 4'b0010;
  localparam  READ   = 4'b0100;
  localparam  WAIT2  = 4'b1000;
  
  /*------------------------------------------*\
                   FIFO端口信号定义
  \*------------------------------------------*/
  reg			wr_en;
  reg	[17:0]	din;
  reg			rd_en;
  wire[17:0]  dout;
  wire        full;
  wire        empty;
  
  /*------------------------------------------*\
                   计数器定义
  \*------------------------------------------*/
  reg [7:0]  wr_cnt;
  reg [7:0]  rd_cnt;
  reg [3:0]  delay_cnt;
  
  /*------------------------------------------*\
           三段状态机第一段：次态打一拍给现态
  \*------------------------------------------*/
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		cur_status <= IDEL;
  	end
  	else begin
  		cur_status <= nex_status;
  	end
  end
  
  /*------------------------------------------*\
         三段状态机第二段：组合逻辑描述状态跳转
  \*------------------------------------------*/
  always @(*) begin
  	if (~reset_n) begin
  		nex_status <= IDEL;
  	end
  	else begin
  		case(cur_status)
  			IDEL : begin
  				nex_status <= WRITE;
  			end
  			WRITE : begin
  				if(wr_cnt == 'd127 && wr_en)
  					nex_status <= WAIT1;
  				else 
  					nex_status <= cur_status;
  			end
  			WAIT1 : begin
  				if(delay_cnt == 'd9)
  					nex_status <= READ;
  				else 
  					nex_status <= cur_status;				
  			end
  			READ :begin
  				if(rd_cnt == 'd127 && rd_en)
  					nex_status <= WAIT2;
  				else 
  					nex_status <= cur_status;				
  			end
  			WAIT2 :begin
  				if(delay_cnt == 'd9)
  					nex_status <= WRITE;
  				else 
  					nex_status <= cur_status;				
  			end
  			default : nex_status <= cur_status;
  		endcase
  	end
  end
  
  /*------------------------------------------*\
         三段状态机第三段：设计状态机输出
  \*------------------------------------------*/
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		wr_en <= 1'b0;
  	end
  	else if (cur_status == WRITE && wr_cnt == 'd127 && wr_en) begin
  		wr_en <= 1'b0;
  	end
  	else if (cur_status == WRITE) begin
  		wr_en <= 1'b1;
  	end
  end
  
  
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		rd_en <= 1'b0; 
  	end
  	else if (cur_status == READ && rd_cnt == 'd127 && rd_en) begin
  		rd_en <= 1'b0;
  	end
  	else if (cur_status == READ) begin
  		rd_en <= 1'b1;
  	end
  end
  
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		wr_cnt <= 8'd0;
  	end
  	else if (wr_en && wr_cnt == 'd127) begin
  		wr_cnt <= 8'd0;
  	end
  	else if(wr_en) begin
  		wr_cnt <= wr_cnt + 1'b1;
  	end
  end
  
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		rd_cnt <= 8'd0;
  	end
  	else if (rd_cnt == 'd127 && rd_en) begin
  		rd_cnt <= 8'd0;
  	end
  	else if (rd_en) begin
  		rd_cnt <= rd_cnt + 1'b1;
  	end
  end
  
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		delay_cnt <= 4'd0;
  	end
  	else if (cur_status == WAIT1 || cur_status == WAIT2) begin
  		delay_cnt <= delay_cnt + 1'b1;
  	end
  	else begin
  		delay_cnt <= 4'd0;
  	end
  end
  
  always @(posedge clk or negedge reset_n) begin
  	if (~reset_n) begin
  		din <= 18'd0;
  	end
  	else if (wr_en) begin
  		din <= din + 1'b1;
  	end
  end
  
  /*------------------------------------------*\
                   例化FIFO
  \*------------------------------------------*/
  fifo_generator_0  fifo_generator_0(
    .clk  (clk),      // input wire clk
    .din  (din),      // input wire [17 : 0] din
    .wr_en(wr_en),  // input wire wr_en
    .rd_en(rd_en),  // input wire rd_en
    .dout (dout),    // output wire [17 : 0] dout
    .full (full),    // output wire full
    .empty(empty)  // output wire empty
  );
  
  
  endmodule
  ```

  

```verilog
//tb_top
`timescale 1ns / 1ps
module tb_top();
	reg clk;
	reg reset_n;

	initial begin
		clk = 0;
		forever #10 
		clk = ~clk;
	end

	initial begin
		reset_n  = 0;
		#100
		reset_n  = 1;
	end

	top top 
	(
		 .clk(clk),
		 .reset_n(reset_n)
    );
endmodule

```







