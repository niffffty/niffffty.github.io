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

![Image](https://github.com/user-attachments/assets/9030e407-f54f-4e71-9c94-8e13e36da8eb)











