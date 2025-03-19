# 时钟ip核使用、分频倍频demo

- xilinx的ip核：软核、硬核
  - 硬核：FPGA内镶嵌其他的芯片，如dsp、GT
  - 软核：相当于一个.v文件，有输入有输出，可以调用
- 时钟ip核
  - 时钟分频、倍频：50M时钟到100M，就是倍频，使用一个ip核，叫做锁相环PLL
  - PLL是一个硬核，可以用赛灵思给的软核Clock间接控制PLL
  - 操作：vivado左侧的IP Catalog，搜clock，找到Clocking Wizard



- 案例：

  - 功能：

    - 输入50M
    - 输出10M、20M、30M、50M、100M、150M、200M

  - 操作：

    - IP核设置页面，第一页修改输入Primary为50；Source选Single

      - Source：单端、差分、No buffer（先经过一个时钟，再传出来的时钟）

    - 第二页配置输出，勾选clk-out后，修改Requested输出，10 20 30 50 100 150

      - 如果7个输出不够，就需要级联，先保存第一个ip核，再点击一个ip核，输入50M、no buffer，输出200M，生成

    - 创建设计文件top.v，例化clk的ip核，

      - vivado-Sources-IP Source-点击clk_wiz_1打开clk_wiz_1.veo，下拉找到clk_wiz_1 instance_name函数；clk_wiz_2.veo中复制例化函数

        ```verilog
        //top.v
        `timescale 1ns / 1ps
        module top(
        	input 		clk
        );
            wire locked1;	//在系统初始化阶段，需要等待locked信号为高电平后，
            wire clk_out1;	//才能安全地使用时钟管理模块输出的时钟信号，以避免因时钟不稳定导致的电路异常行为
            wire clk_out2;
            wire clk_out3;
            wire clk_out4;
            wire clk_out5;
            wire clk_out6;
            wire clk_out7;
            
            wire lock2;
            wire clk_out8;
        //clk的ip核1
            clk_wiz_1 clk_wiz_1
            (
                .clk_out1(clk_out1),
                .clk_out2(clk_out2),
                .clk_out3(clk_out3),
                .clk_out4(clk_out4),
                .clk_out5(clk_out5),
                .clk_out6(clk_out6),
                .clk_out7(clk_out7),
                .reset(1'd0),
                .locked(locked1),
                .clk_in1(clk)
            );
        //clk的ip核2
            clk_wiz_2 clk_wiz_2
            (
                .clk_out1(clk_out8),
                .reset(1'd0),
                .locked(locked2),
                .clk_in1(clk_out5)
            );
        endmodule
        ```

        ```verilog
        //top_tb.v
        //Subline里右键生成Testbench
        module tb_top();
            
            reg clk;
            
            initial begin
            	clk = 0;
                forever #(10)
                clk = ~clk;    
            end
            
            top inst_top(
                .clk(clk)
            );
        endmodule
        ```

        



