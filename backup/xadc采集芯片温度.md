### xadc硬核

- 概念

  - xadc是7系列芯片集成的硬核，功能：测fpga芯片温度、类似于adc功能
  - ip核搜XADC Wizard

- 配置

  - 第一页Basic：勾选Channel Sequencer、取消reset_in
  - 第四页Channel Sequencer：
    - 勾选TEMPERATURE的Enable、vauxp0到5的Enable
    - 取消勾选VP/VN

- 计算温度

  - 输出的温度值为0-4096，需要通过公式计算（一般工程中讲数字值传输到单片机计算温度）
  - 此次demo使用的板对应`ug480_xadc`，温度=`adc_code*503.975/4086 - 273.15`

- 使用现成的xadc_ctrl.v和top.v来驱动硬核

  ```verilog
  //top.v
  
  `timescale 1ns / 1ps
  module top(
  	input 	wire		clk
      );
  
  
  	wire        vauxp0;
  	wire        vauxn0;
  	wire        vauxp1;
  	wire        vauxn1;
  	wire        vauxp2;
  	wire        vauxn2;
  	wire        vauxp3;
  	wire        vauxn3;
  	wire        vauxp4;
  	wire        vauxn4;
  	wire        vauxp5;
  	wire        vauxn5;
  	wire [11:0] analog_wave_vaux0;
  	wire [11:0] analog_wave_vaux1;
  	wire [11:0] analog_wave_vaux2;
  	wire [11:0] analog_wave_vaux3;
  	wire [11:0] analog_wave_vaux4;
  	wire [11:0] analog_wave_vaux5;
  	wire [11:0] Temperature;
  
  	xadc_ctrl xadc_ctrl
  		(
  			.clk               (clk),
  			.vauxp0            (vauxp0),
  			.vauxn0            (vauxn0),
  			.vauxp1            (vauxp1),
  			.vauxn1            (vauxn1),
  			.vauxp2            (vauxp2),
  			.vauxn2            (vauxn2),
  			.vauxp3            (vauxp3),
  			.vauxn3            (vauxn3),
  			.vauxp4            (vauxp4),
  			.vauxn4            (vauxn4),
  			.vauxp5            (vauxp5),
  			.vauxn5            (vauxn5),
  			.analog_wave_vaux0 (analog_wave_vaux0),
  			.analog_wave_vaux1 (analog_wave_vaux1),
  			.analog_wave_vaux2 (analog_wave_vaux2),
  			.analog_wave_vaux3 (analog_wave_vaux3),
  			.analog_wave_vaux4 (analog_wave_vaux4),
  			.analog_wave_vaux5 (analog_wave_vaux5),
  			.Temperature       (Temperature)
  		);
  
  
  ila_0  ila_0(
  	.clk(clk), // input wire clk
  	.probe0(Temperature) // input wire [11:0] probe0
  );
  
  
  
  endmodule
  
  ```

  

  ```verilog
  //xadc_ctrl.v
  //更改了ip核端口后，需要在本文件添加对应的端口
  
  `timescale 1ps/1ps
  module xadc_ctrl(
     input             clk,
     input             vauxp0,		//外部输入一对差分信号
     input             vauxn0,		//输入到内部经过adc转换成数字信号
     input             vauxp1,
     input             vauxn1,
     input             vauxp2,
     input             vauxn2,
     input             vauxp3,
     input             vauxn3,
     input             vauxp4,
     input             vauxn4,
     input             vauxp5,
     input             vauxn5,
    
     output reg [11:0] analog_wave_vaux0,    //输出的各模拟量端口的数字值
     output reg [11:0] analog_wave_vaux1,
     output reg [11:0] analog_wave_vaux2,
     output reg [11:0] analog_wave_vaux3,
     output reg [11:0] analog_wave_vaux4,
     output reg [11:0] analog_wave_vaux5,
     output reg [11:0] Temperature           //输出温度 
     
      );
  	
     localparam  ONE_NS      = 1000;
     integer    count       = 0   ;
  
     wire EOC_TB;   //转换信号结束
     wire EOS_TB;   //顺序采集结束
     reg [6:0] DADDR_TB; //每个采样通道的具体地址
     reg DEN_TB;  //enable signal for drp
     reg DWE_TB;  //write enable for drp
     reg [15:0] DI_TB;  //input databus for drp
     wire [15:0] DO_TB;  //output databus for drp
     wire DRDY_TB;   //data ready signal for drp
     wire ALARM_OUT_TB;
     wire VCCAUX_ALARM_TB;
     wire VCCINT_ALARM_TB;
     wire USER_TEMP_ALARM_TB;
     wire BUSY_TB;   //ADC busy signal
     wire [4:0] CHANNEL_TB;  //通道选择输出
     wire OT_TB;     //过温报警输出
     wire DCLK_TB;    //DRP的时钟输入
  
         
  assign DCLK_TB = clk;  
  always @(posedge DCLK_TB)
     begin
       DADDR_TB = {2'b00,CHANNEL_TB};
       DI_TB = 16'h0000;
       DWE_TB = 1'b0;
       DEN_TB = EOC_TB;
     end
     
  always @(posedge DCLK_TB)
   begin
     if (DRDY_TB)
       if (DADDR_TB == 7'h10)
          analog_wave_vaux0 <= DO_TB[15:4];
   end
  
  always @(posedge DCLK_TB)
   begin
     if (DRDY_TB)
       if (DADDR_TB == 7'h11)
          analog_wave_vaux1 <= DO_TB[15:4];
   end        
  
  always @(posedge DCLK_TB)
   begin
     if (DRDY_TB)
       if (DADDR_TB == 7'h12)
          analog_wave_vaux2 <= DO_TB[15:4];
   end 
   
  always @(posedge DCLK_TB)
    begin
      if (DRDY_TB)
        if (DADDR_TB == 7'h13)
           analog_wave_vaux3 <= DO_TB[15:4];
    end  
    
  always @(posedge DCLK_TB)
     begin
       if (DRDY_TB)
         if (DADDR_TB == 7'h14)
            analog_wave_vaux4 <= DO_TB[15:4];
     end 
     
  always @(posedge DCLK_TB)
    begin
      if (DRDY_TB)
        if (DADDR_TB == 7'h15)
           analog_wave_vaux5 <= DO_TB[15:4];
    end
    
    //可输出芯片的温度
  always @(posedge DCLK_TB)begin 
     if(DRDY_TB == 1'b1 && DADDR_TB == 7'h00)begin // 地址DADDR_TB配置为ox00，可输出芯片温度
         Temperature <= DO_TB[15:4];              //  地址DADDR_TB 可以理解为寄存器
     end else begin
         Temperature <= Temperature;
     end
  end
  
  xadc_wiz_0  xadc_wiz_0
  (
     .daddr_in           (DADDR_TB[6:0]),            // Address bus for the dynamic reconfiguration port           
     .dclk_in            (~DCLK_TB),             // Clock input for the dynamic reconfiguration port                
     .den_in             (DEN_TB),              // Enable Signal for the dynamic reconfiguration port  转换结束可读了       
     .di_in              (DI_TB[15:0]),               // Input data bus for the dynamic reconfiguration port          
     .dwe_in             (DWE_TB),              // Write Enable for the dynamic reconfiguration port
  
     .vauxp0             (vauxp0),              // Auxiliary channel 0                                                 
     .vauxn0             (vauxn0),                                                                                     
     .vauxp1             (vauxp1),              // Auxiliary channel 1                                                 
     .vauxn1             (vauxn1),                                                                                     
     .vauxp2             (vauxp2),              // Auxiliary channel 5                                                 
     .vauxn2             (vauxn2),                                                                                     
     .vauxp3             (vauxp3),              // Auxiliary channel 8                                                 
     .vauxn3             (vauxp3),                                                                                     
     .vauxp4             (vauxp4),              // Auxiliary channel 9                                                 
     .vauxn4             (vauxn4), 
     .vauxp5             (vauxp5),              // Auxiliary channel 9                                                 
     .vauxn5             (vauxn5),                                                                                   
     .busy_out           (BUSY_TB),            // ADC Busy signal                                                  
     .channel_out        (CHANNEL_TB[4:0]),         // Channel Selection Outputs                                
     .do_out             (DO_TB[15:0]),              // Output data bus for dynamic reconfiguration port             
     .drdy_out           (DRDY_TB),            // Data ready signal for the dynamic reconfiguration port           
     .eoc_out            (EOC_TB),             // End of Conversion Signal                                          
     .eos_out            (EOS_TB),             // End of Sequence Signal                                            
     .ot_out             (OT_TB),              // Over-Temperature alarm output                                      
     .vccaux_alarm_out   (VCCAUX_ALARM_TB),    // VCCAUX-sensor alarm output                               
     .vccint_alarm_out   (VCCINT_ALARM_TB),    //  VCCINT-sensor alarm output                              
     .user_temp_alarm_out(USER_TEMP_ALARM_TB), // Temperature-sensor alarm output                       
     .alarm_out          (ALARM_OUT_TB),                                                                          
     .vp_in              (1'b0),               // Dedicated Analog Input Pair                                         
     .vn_in              (1'b0)            //一般情况置0                                                                           
     );  
     
  
    
     
  endmodule
  ```

  