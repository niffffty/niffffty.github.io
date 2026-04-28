### 模块例化
     
- 模块例化=c语言中的函数调用

```verilog
//被例化的文件，d触发器,右键生成testbench
module d_trigger(
	input wire clk,
    input wire d,
    output reg q
);
    always @(posedge clk) begin
    	q<=d;
    end
endmodule
```

```verilog
//调用例化的文件，实现d触发器
module top(
	input wire clk,
    input wire d2,
    output reg q2
);
    d_trigger d_trigger_inst (		//调用d_trigger
        .clk	(clk),				//将两个模块的线连上
        .d		(d2),
        .q		(q2)
    );
    
endmodule
```



### 4种计数器

- 计数器1,每个上升沿反转

![Image](https://github.com/user-attachments/assets/602c906d-187e-4e6d-a079-943aef2beb84)

  ```verilog
  //根据以上时序图实现每一个上升沿led反转一次
  module led1(
  	input wire clk,				
      input wire reset_n,
      output reg led
  );
      always @(posedge clk or negedge reset_n) begin
          if(~reset_n) begin
          	led <=1'b0;
          end
          else begin
          	led <= ~led;
          end
      end
  endmodule
  ```

- 计数器2，两个上升沿反转

  
![Image](https://github.com/user-attachments/assets/d3e3c764-cd9d-4172-8434-8bf79dde85c3)

  ```verilog
  //用计数器实现每两个上升沿led反转一次
  module led2(
  	input wire clk,				
      input wire reset_n,
      output reg led
  );
      reg [11:0] cnt;
  //反转 
      always @(posedge clk or negedge reset_n) begin
          if(~reset_n) begin
          	led <=1'b0;
          end
          else begin
          	led <= ~led;
          end
      end
  //计数器
      always @(posedge clk or negedge reset_n) begin
          if(~reset_n) begin			//复位cnt清0
          	cnt <='d0;
          end
          else if(cnt == 'd1)begin	//计数到xx值清0
          	cnt <='d0;
          end
          else begin					//正常计数
          	cnt <= cnt + 1'd1;
          end
      end
  endmodule
  ```

- 计数器3，1s反转

  ```verilog
  //用计数器实现每1s,led反转一次
  module led3(
  	input wire clk,			//50M	
      input wire reset_n,
      output reg led
  );
      reg [25:0] cnt;			//cnt26位宽
      
  always @(posedge clk or negedge reset_n) begin
      if(~reset_n) begin
      	led <=1'b0;
      end
      else if(cnt == 'd49_999_999) begin //主频50M=每秒50_000_000个时钟周期
      	led <= ~led;				   //当计数器计时到50M时，反转led
      end
  end
  
  always @(posedge clk or negedge reset_n) begin
      if(~reset_n) begin			//复位cnt清0
      	cnt <='d0;
      end
      else if(cnt == 'dd49_999_999)begin	//计数到xx值清0
      	cnt <='d0;
      end
      else begin					//正常计数
      	cnt <= cnt + 1'd1;
      end
  end
  endmodule
  ```

- 计时器4，1min反转

  
![Image](https://github.com/user-attachments/assets/900d5005-bb99-468c-a756-c0c3a2f26f06)

  ```verilog
  //用计数器实现每1min,led反转一次
  module led3(
  	input wire clk,			//50M	
      input wire reset_n,
      output reg led
  );
      reg [25:0] cnt;			//cnt26位宽
      reg [7:0]  min_cnt;		//min_cnt分钟计数器
  //反转
  always @(posedge clk or negedge reset_n) begin
      if(~reset_n) begin
      	led <=1'b0;
      end
      else if(cnt == 'd49_999_999) begin //主频50M=每秒50_000_000个时钟周期
      	led <= ~led;				   //当计数器计时到50M时，反转led
      end
  end
  //秒计时
  always @(posedge clk or negedge reset_n) begin
      if(~reset_n) begin			//复位cnt清0
      	cnt <='d0;
      end
      else if(cnt == 'dd49_999_999)begin	//计数到xx值清0
      	cnt <='d0;
      end
      else begin					//正常计数
      	cnt <= cnt + 1'd1;
      end
  end
  //分钟计时
      always @(posedge clk or negedge reset_n) begin
      if(~reset_n) begin			//复位cnt清0
      	min_cnt <='d0;
      end
      else if(cnt == 'd49_999_999 && min_cnt=='d59)begin	//计数到xx值清0
      	min_cnt <='d0;
      end
      else if(cnt=='d49_999_999)begin					//正常计数
      	min_cnt <= min_cnt + 1'd1;
      end
  end
  endmodule
  ```

  