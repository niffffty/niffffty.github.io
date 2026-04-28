ram的ip核：Block Memory Generater

- 信号：
  - clka
  - ena：ena和wea同时为高，就是写
  - wea：1是写，0是读
  - addra：地址
  - dina：往RAM写入的数据
  - douta：从RAM读出来的数据
- 配置：
  - 第一页Memory Type：single port RAM
  - 第二页Memory Size：16/16; 128/128
  - 第二页Primitives Output Register：勾选后，读比写延时两个时间单位，不勾选则延时一个单位（与fifo不一样，不能对齐）


![Image](https://github.com/user-attachments/assets/ac84edb0-3860-42ff-bb04-7b860b9ba495)


```verilog
//top.v
//往RAM写128数据，等待10，读128，等10，写，循环
`timescale 1ns / 1ps

module top(
	input wire		clk,
	input wire      reset_n
    );

/*------------------------------------------*\
                 状态机定义
\*------------------------------------------*/
reg [3:0] cur_status;
reg [3:0] nex_status;

localparam IDLE  = 4'b0000; //独热码
localparam WRITE = 4'b0001;
localparam WAIT1 = 4'b0010;
localparam READ  = 4'b0100;
localparam WAIT2 = 4'b1000;
/*------------------------------------------*\
                 RAM端口信号定义
\*------------------------------------------*/
reg         ena;
reg         wea;
reg  [6: 0] addra;
reg  [15:0] dina;
wire [15:0] douta;

/*------------------------------------------*\
                 计数器信号定义
\*------------------------------------------*/
reg [7:0] wr_cnt;
reg [7:0] rd_cnt;
reg [3:0] delay_cnt;

/*------------------------------------------*\
         三段状态机第一段：次态打一拍给现态
\*------------------------------------------*/
always @(posedge clk or negedge reset_n) begin
	if (~reset_n) begin
		cur_status <= IDLE;
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
		nex_status <= IDLE;
	end
	else begin
        case(cur_status)		//状态：IDLE-WRITE-WAIT1-READ-WAIT2-循环
			IDLE : begin
				nex_status <= WRITE;
			end
			WRITE :begin
				if(wr_cnt == 'd127 && wea && ena)
					nex_status <= WAIT1;
				else 
					nex_status <= cur_status;		//保持现态
			end
			WAIT1 : begin
				if (delay_cnt == 'd9)
					nex_status <= READ;
				else 
					nex_status <= cur_status;
			end
			READ : begin
				if (rd_cnt == 'd127 && ~wea && ena)
					nex_status <= WAIT2;
			end
			WAIT2 : begin
				if (delay_cnt == 'd9)
					nex_status <= WRITE;
				else 
					nex_status <= cur_status;
			end			
			default : nex_status <= IDLE;
		endcase
	end
end

/*------------------------------------------*\
       三段状态机第三段：设计状态机输出
\*------------------------------------------*/
    always @(posedge clk or negedge reset_n) begin		//wea：写/读信号，1写，0读
        if (~reset_n) begin								//根据WRITE-READ状态，拉高拉低
		wea <= 1'b0;
	end
	else if (cur_status == WRITE) begin
		wea <= 1'b1;
	end
	else if (cur_status == READ) begin
		wea <= 1'b0;
	end
end

    always @(posedge clk or negedge reset_n) begin		//ena：使能信号，1开，0关
        if (~reset_n) begin								//根据读写状态，写完/读完拉低
		ena <= 1'b0;
	end
	else if (wr_cnt == 'd127 && wea && ena && cur_status == WRITE) begin
		ena <= 1'b0;
	end
	else if (rd_cnt == 'd127 && ~wea && ena && cur_status == READ) begin
		ena <= 1'b0;
	end		
	else if (cur_status == READ) begin
		ena <= 1'b1;
	end
	else if (cur_status == WRITE)begin
		ena <= 1'b1;
	end
end

    always @(posedge clk or negedge reset_n) begin		//addr：地址
        if (~reset_n) begin								//0写到127清零，使能时递增写下一个地址
		addra <= 7'd0;
	end
	else if(addra == 'd127 && ena) begin
		addra <= 7'd0;
	end
	else if (ena) begin
		addra <= addra + 1'b1;
	end
end

    always @(posedge clk or negedge reset_n) begin			//din：写进来的数据
        if (~reset_n) begin									//假设数据为0-127
		dina <= 16'd0;
	end
	else if(wea && ena) begin
		dina <= dina + 1'b1;
	end
end

    always @(posedge clk or negedge reset_n) begin			//写数据计数器，写完128个清零
	if (~reset_n) begin
		wr_cnt <= 8'd0;
	end
	else if (cur_status == WRITE && wea && ena && wr_cnt == 'd127) begin
		wr_cnt <= 8'd0;
	end
	else if (cur_status == WRITE && wea && ena) begin
		wr_cnt <= wr_cnt + 1'b1;
	end
end

    always @(posedge clk or negedge reset_n) begin			//读数据计数器，读完128个清零
	if (~reset_n) begin
		rd_cnt <= 8'd0;
	end
	else if (cur_status == READ && ~wea && ena && rd_cnt == 'd127) begin
		rd_cnt <= 8'd0;
	end
	else if (cur_status == READ && ~wea && ena) begin
		rd_cnt <= rd_cnt + 1'b1;
	end
end

    always @(posedge clk or negedge reset_n) begin			//延时计数器，根据WAIT1状态清零
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

    blk_mem_gen_0  blk_mem_gen_0(								//例化ip核
  .clka (clk),    // input wire clka
  .ena  (ena),      // input wire ena
  .wea  (wea),      // input wire [0 : 0] wea
  .addra(addra),  // input wire [6 : 0] addra
  .dina (dina),    // input wire [15 : 0] dina
  .douta(douta)  // output wire [15 : 0] douta
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











