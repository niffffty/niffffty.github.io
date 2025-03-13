### fpga开发流程

1. 明确功能
2. 设计模块波形
3. 写代码
4. 仿真
5. 管脚约束
6. 综合
7. 布局布线
8. 生成bit流
9. 上板验证
10. 使用工具抓信号



### led_demo

- 创建工程，选择文件位置，勾选创建子目录，next，勾选不定义源文件

- 选择芯片型号（此处使用正点原子达芬奇XC7A35T-2FGG484C）

- add source---创建设计源文件---创建文件---文件名---

- subline打开，写代码（**核心：verilog代码和时序图的对应关系，看到波形图，可以写出波形图所对应代码；看到verilog代码，可以画出代码所对应的波形图**）

  ```verilog
  //led.v
  module led(
  	input  wire clk,
      input  wire reset_n,
      output reg  led
  );
      
      localparam  CNT_MAX = 'd49_999_999;
      reg [25:0] cnt;
  //计数器
      always @(posedge clk or negedge reset_n)begin
          if(~reset_n)begin
          	cnt <='d0;
          end
          else if(cnt ==CNT_MAX)begin
          	cnt <='d0;
          end
          else begin
          	cnt <=cnt+1'b1;
          end
      end
  //控制led反转
      always @(posedge clk or negedge reset_n)begin
          if(~reset_n)begin
          	led <=1'b0;
          end
          else if(cnt ==CNT_MAX)begin
          	led<=~led;
          end
      end
  endmodule
  ```

  

- 仿真

  - add sources-仿真文件-创建文件tb_led-finish

  - 仿真格式

    ```verilog
    'timescale				//仿真单位/仿真精度
    module Test_bench();	//仿真模块通常无输入/输出	
    	//1.信号或变量声明（输入一般定义reg，输出定义为wire）		//2.使用initial或always语句产生激励
        //3.例化待测试模块
        //4.监控和比较输出响应
    
    endmodule
    ```

  - 仿真代码

    ```verilog
    //tb_led.v
    
    'timescale 1ns / 1ps				
    module tb_led();		
    	reg clk;
        reg reset_n;
    //激励
        initial begin 	//可以从testbench文件复制
        	clk = 0;
            forecer #10
            clk = ~clk;
        end
        
        initial begin 	//可以从testbench文件复制
        	reset_n = 0;
            #100
            reset_n = 1;
        end
        
    //例化待测试模块,
        led inst_led(		 //可以从testbench文件复制
            .clk(clk),
            .reset_n(reset_n),
            .led(led));
       
    
    endmodule
    ```

    

- 开启仿真

  - Run Simulation
  - 把Objects的信号去掉，把Scope-inst_led中的信号右键添加到仿真图中（Add to Wave Window）
  - Restart-- Run all--暂停
  - 在Name右键--Radix选择进制--Unsigned Decimal
  - （改代码以后重新run Simulation；添加Divider分割；添加组group）

- 添加管脚约束

  - Schematic--3 I/O Ports或右上角I/O Planning
  - 原理图中找到GCLK、Reset、led引脚 ，绑定
  - 电平标准选择LVCMOS33*
  - Ctrl+S保存，文件名led

- 生成bit流

  - Generate Bitsteam
  - Open Hardware Manager
  - auto connect，左键xc7a...，右键选择Program Device，选择bit文件



### 流水灯_demo

功能：

```verilog
//led_shift.v
module led_shift(
    input wire clk,
    input wire reset_n,
    output reg [3:0] led
);
    reg [25:0] cnt;  // 1秒计数器
    reg [1:0] cnt_shift;  // 移位计数器，2位足够表示0~3
    localparam CNT_MAX = 49_999_999;  // 最大计数值

    always @(posedge clk or negedge reset_n) begin
        if (~reset_n) begin
            cnt <= 0;  // 重置计数器
            cnt_shift <= 0;  // 重置移位计数器
            led <= 4'b0001;  // 初始状态
        end
        else begin
            // 计数器逻辑
            if (cnt == CNT_MAX) begin
                cnt <= 0;  // 重置计数器
                // LED 移位逻辑
                if (cnt_shift == 3) begin
                    cnt_shift <= 0;  // 重置移位计数器
                    led <= 4'b0001;  // 重置LED状态
                end
                else begin
                    led <= led << 1;  // 左移一位
                    cnt_shift <= cnt_shift + 1;  // 移位计数器加1
                end
            end
            else begin
                cnt <= cnt + 1;  // 计数器加1
            end
        end
    end
endmodule
```









































