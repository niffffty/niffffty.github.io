- 进制：4'b0000为二进制的0，四位宽

  - 二进制： Binary 
  - 八进制：Octal
  - 十进制：Decimal
  - 十六进制：Hexadecimal







- 单位换算：
  - 1bit = 1位（最小信息单位）
  - 8 bit = 1字节 = 1B
  - 1KB = 1024 字节
  - 1MB = 1024 KB
  - 1GM = 1024 MB
  - 二进制与十进制不同，可能导致实际容量略有差异（十进制：1KB=1000B）





- 数电基础：

  - 与或非

  - D触发器：在时钟上升沿的时候，将D赋给Q

    ```verilog
    // Verilog实现D触发器
    module d_trigger(
    	input wire clk,
        input wire d,
        output reg q		//q与时间有关，需要reg类型
    );
        always @(posedge clk)begin
        	q<=d;
        end
    endmodule
    ```

    

- verilog数据类型：

  - wire：线型，默认数据类型，输入信号默认为wire
  - reg：与时间有关的信号
    - 特例：在always块中被赋值的语句都定义为reg，即使这个信号为组合逻辑，即使与时间无关

- 二路选择器：

  ```verilog
  module sel(
      input wire sel,
      input wire a,
      input wire b,
      output reg c		
  );
      
      //第一种写法
      assign c = sel==1'b1 ? a:b;		//但是c可能为:0,1，不定态，高阻态
      
      //第二种写法
      always @(sel,a,b)begin
          if(sel==1'b)begin
          	c=a;
          end
          else if(sel==1'b0)begin
          	c=b;
          end
          
      end
  endmodule
  ```

  

- 运算符

  - 位运算符：&,|,~ 是按位相与/或/非
  - 逻辑运算符：&&,||,!，一般用在if条件里
  - 移位运算符：《《   ， 》》
  - 拼接运算符：{a,b}

- if运算：

  ```verilog
  always@(posedge clk)begin
      if(条件1)begin
      	语句1			//如果满足条件1，就执行语句1，执行完后跳出if
      end
      else if(条件2)begin
      	语句2
      end
  end
  ```

  

- case语句

  ```verilog
  //组合逻辑实现3_8译码器
  module decoder_3_8(
      input wire [2:0] sel,
      output reg [7:0] out
  );
      always@(sel) begin
          case(sel)
              3'b000:out = 8'b0000_0001;
              3'b001:out = 8'b0000_0010;
              3'b010:out = 8'b0000_0100;
              3'b011:out = 8'b0000_1000;
              3'b100:out = 8'b0001_0000;
              3'b101:out = 8'b0010_0000;
              3'b110:out = 8'b0100_0000;
              3'b111:out = 8'b1000_0000;
              default:out = 8'b1000_0001;
           endcase
      end
  ```

  

