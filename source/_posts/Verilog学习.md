---
title: Verilog学习
date: 2022-05-09 15:21:26
tags: 计算机组成
---

# HDL语言（Verilog）学习要点(预习)

## 知识点

### Z与x

HDL用**Z**表示浮空值，代表高阻状态，即与电路断开，用**x**代表无效的逻辑电平（*未知状态或冲突状态*）。

- 例如在初始化时，状态节点会被初始化为**x**，代表未知状态。
- 例如一条总线被两个使能的三态缓冲器驱动为不同的值，则也输出**x**，代表冲突状态。
- 接收浮空输入，无效输入，未初始化的输入都会输出**x**。

----------------------

### case，if语句

**case**语句一般都要写上default，且这两种语法都必须出现在**always**语句块中。

------------

### 阻塞赋值与非阻塞赋值

- **=** 为阻塞赋值，与编程语言一样，需要按顺序执行,组合逻辑一般使用阻塞赋值，可以使用多个assign，也可以使用always语句块。例如如下实现全加器，两种方法均可 **(都是用了阻塞赋值)** ：

- <=为非阻塞赋值，当信号发生变化时并行执行，但一般不用于组合逻辑中，使用非阻塞赋值可以得到正确的结果但会执行多次always中的语句，使仿真速度变慢，参考 *《数字设计与计算机体系结构 P126》* 。

------------------

### 定义状态机时 `typedef enum`的使用 (不过这个好像只能systemVerilog 使用，我们使用的verilog没有这个语法，后来在发现。。。。。。。)

```Verilog
typedef enum reg [1:0] {S0,S1,S2} statustype;
statustype [1:0] status;
```

以上代码自动赋值S0,S1,S2,相当于

```Verilog
parameter S0=0;
parameter S1=1;
parameter S2=2;
```

**当状态机的状态非常多时**，可以不用一个个定义独热码，非常实用。

### 模块实例化

![pic](https://note.youdao.com/yws/api/personal/file/8391DC0390C7419EAC9C74CEE124CAED?method=download&shareKey=c5c78d09f579cb1e34b1c20331b53c4f)
且模块实例化可以改变引用模块中参数的位宽

```verilog
module Decode(A,F);
    parameter Width =1,polarity=1;
    ....
endmodule 
module Top;
    wire [3:0] A4;
    wire [15:0] B16;
    Decode #(4,0) D1(A4,B16);//将Decode中的两个parameter的值更改为4，1
end module 
```

----------------

### 变量

##### wire型

- 输出信号自动默认为wire类型
- 不能存储值，必须受到驱动器的驱动（assign，赋值语句）

##### reg类型

- 寄存器
- 默认初始值x，需要初始化

##### memory类型

- 通过定义寄存器数组实现
- 与C语言中数组的使用方法类似

```verilog
reg [7:0] mem[255:0];
//定义256个8位存储器，地址0~255
```

----------------------

### 运算符

- 位运算符 & ，|，~，^(异或)，~^（异或非）
- 逻辑运算符 &&，||，！
- 移位运算符 <<,>>（0填补）
- 位拼接运算符 { }

```v
{b,{3{a,b}}}={b,a,b,a,b,a,b}//注意重复法使用时需要再套一个括号
```

- 等式运算符 ==，！=，！==，===
  - ===是全等，每一位都要相等，包括x，z
  - ==是值相等，当有不定值时不能判断相等

### 块语句

- 顺序块 

```v
begin 

end
```

- 并行块
  - 进入并行块后块内语句同时执行
  - 每条语句延迟时间相对于程序进入块内的仿真时间
  - 

```v
fork://块名
//块内声明语句
    语句1；
    语句2；
    //...
```

-------------------

### 语句

#### if else

- 语法与C语言中类似
- if语句块若没有写明else，则会生成额外的寄存器，因此写if语句最好写完整。

```v
always @(*)
begin 
    if(al) q=d;
    //没有写else，当al=0时变量q没有新的赋值，变量将保持原值，也就是综合后生成一个关于q的锁存器，这个锁存器不是我们原先想要的
end
```

#### case 

```v
case (expression)
    s1:
    s2:
    //...
    default:
endcase
```

- case 不写明default会生成不必要的锁存器

#### 循环 forever

```v
forever begin
//多条语句
end 
```

- 用于产生周期性的波形
- 不能像always一样写在程序中，而要写在initial中

#### 循环 repeat

```v
parameter size=8;
reg [8:1]opa,opb;
reg [16:1]result;
begin:mult
    reg [16:1]shift_opa,shift_opb;
    shift_opa=opa;
    shift_opb=opb;
    result =0;
    repeat (size)//重复8次
    begin
        if(shift_opb[1])
            result = result+shift_opa;
        shift_opa =shift_opa <<1;
        shift_opb = shift_opb >>1;
    end
end
```

### 循环for与while

- **特别提一下for中一般用integer i做循环变量**

---------------------

### 结构说明语句

#### initial

- 只执行一次 

#### always

- 不断执行直到仿真过程结束
- 常与时序控制语句相结合

```v
always #period clk=~clk;
always@(posedge clk)begin
//...
end
always@(*)begin
//...
end
```

- 电平敏感时序控制

```v
always 
    wait(count_enable)  #20 count=count+1;
```

程序将持续监视count_enable,当其值为1时，执行后面的语句。

--------------------

### task及function说明语句

- 函数与主模块使用一个仿真时间单位，任务可以定义自己的时间单位
- 函数不能启动任务，任务可以启动其他任务及函数
- 函数至少有一个输入变量，任务可以没有
- 函数返回一个值，任务不返回值（但任务有输出值）
  **函数的目的是通过返回一个值响应输入信号的值，而任务可以支持多种目的，可以计算多个结果的值，这些结果枝只能通过task的任务输出或总线端口输出** 
  **可以这么理解，任务是一个小的module，可以有多种用途**

#### Task

```v
task my_task;
    input a,b;
    output c,e;
    //.....
    c=//...
    e=//...
endtask
my_task(a0,b0,c0,d0);//任务调用
```

任务可以启动其他的任务，可以启动的任务数无限制，也可以启动其他的函数。
一个例子(只是一个行为模块，不可综合)：

```v
module light(
    );
reg clock,red,amber,green;
parameter on=1,off=0,red_tics=350,
			amber_tics=30,green_tics=2000;
initial begin
	red=off;
	amber=off;
	green=off;
end
always begin
	red=on;
	light(red,red_tics);
	green=on;
	light(green,green_tics);
	amber=on;
	light(amber,amber_tics);
end
task light;//等待一定时间将该信号灯关掉
	output color;
	input [31:0] tics;
	begin
		repeat(tics)
			@(posedge clock);
		color=off;
	end
endtask
always 
	begin
		#100 clock=0;
		#100 clock=1;
	end
endmodule
```

#### Function（函数）

- 返回的是一个寄存器的值
- 没有return语句，最后函数运行完后返回寄存器的值是啥返回值就是啥
- 返回值自动定义为与函数名同名的寄存器
- 函数不能启动任务
- 函数不能包含任何时间控制语句（#，@，wait）

```v
function [7:0]getbyte;//函数名为getbyte，返回寄存器也是getbyte，且为8位，如果不定义位宽自动默认为1位宽
intput [15:0]address;
begin
    //....
    getbyte=//....
end
endfunction
//调用
word={getbyte(msbyte),getbyte(lsbyte)};
```

一个例子关于阶乘：

```v
module tryfact;
    function[31:0]factorial;
        input[3:0]operand;
        reg [3:0]index;
        begin
            factorial=1;
            for(index=2;index<=operand;index=index+1)
                factorial=index*factorial;
        end
    endfunction
reg[31:0]result;
reg[3:0]n;
initial begin
    result=1;
    for(n=2;n<=9;n=n+1)begin
        $display("n=%d,result=%d",n,result);
        result=n*factorial(n)/((n*2)+1);

    end
    $display("finalresult=%d",result);

end
endmodule
```

结果：
![A](https://note.youdao.com/yws/api/personal/file/66D3CCBB4BA7482D871D1BD37FEECF68?method=download&shareKey=6afbe6ea018f26f97b34af03d18077af)

#### 自动递归函数

- verilog中函数不能进行递归调用，因为没有栈，递归时将对同一区域进行操作，计算结果不确定。

```v
function automatic integer factorial;
input [31:0]oper;
integer i;
begin
    if(oper>=2)
        factorial=factorial(oper-1)*oper;//调用函数
    else
        factorial = 1;
end
endfuction
```

---------------------

### 常用系统任务

#### \$display and \$write

- 都是输出信息，display自动换行，write不自动换行
- %c，以ascll输出
- %t，输出时间
- %%，输出%号

#### \$readmemb and \$readmemh

参见语言模板 FILE I/O说明。

---------------------

### 编译预处理

- `define
- `include
- 条件编译命令

```v
    `ifdef 宏名
    程序段1；
    `else
    程序段2；
    `endif
```

**宏名若已定义，则编译程序段1，否则编译程序段2。**

-----------------------

## 代码块

***三态缓冲器***

```Verilog
module tristate (
    input [3:0]a,
    input en,
    output tri [3:0] y //使用tri类型而不是trireg类型
    
);
    assign y = en? a : 4'bz 
endmodule 
```

***4：1选择器（结构化建模）***

```Verilog
module mux4 (
    input [3:0]d0,d1,d2,d3,
    input [1:0]s,
    output [3:0] y 
);
    wire [3:0] low,high;
    //使用了之前编写好的2：1复用器
    mux2 lowmux(d0,d1,s[0],low);
    mux2 highmux(d2,d3,s[0],high);
    mux2 finalmux(low,high,s[1],y);
endmodule 
```

***Cpu_checker***
![有限状态机图](https://img-blog.csdnimg.cn/20210218115513426.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0plcmVteVpoYW8xOTk4,size_16,color_FFFFFF,t_70#pic_center)

```Verilog
`timescale 1ns / 1ps
module cpu_checker(
    input clk,
    input reset,
    input [7:0] char,
    input [15:0] freq,
    output  [1:0] format_type,
    output  [3:0] error_code
    );
//额外寄存器变量需要3个

parameter  S0=4'd0;
parameter  S1=4'd1;
parameter  Dextop=3'd4; 
parameter Hextop=4'd8;
parameter YES=1'b1;
parameter NO=1'b0;
parameter initial_reg=64'b0000_0000;
reg timewrong=1'b0;
reg pcwrong=1'b0;
reg addrwrong=1'b0;
reg grfwrong=1'b0;
reg [2:0] dexchecker;
reg [3:0] hexchecker;
reg [3:0] status;
reg typereg;
reg [63:0]Time=64'b0000_0000;
reg [63:0]PC=64'b0000_0000;
reg [63:0]GRF=64'b0000_0000;
reg [63:0]ADDR=64'b0000_0000;
assign  error_code = {grfwrong,addrwrong,pcwrong,timewrong} ;
initial begin
    status <= S0;
    dexchecker <= 3'd0;
    hexchecker <= 4'd0;
    typereg <= 1'b0;
end
wire digitchar = (char >= "0"&&char <="9")? YES : NO;
wire hexchar = (digitchar == YES)? YES :(char >= "a"&&char <= "f")? YES:NO;
assign  format_type= (status != 4'd13)? 2'b00:
                    (typereg == 1'b0)? 2'b01:2'b10;
always @(posedge clk) begin
    if(reset == 1'b1)begin
        status <= S0;
        dexchecker <= 3'd0;
        hexchecker <= 4'd0;
        typereg <= 1'b0;
        Time <= initial_reg;
        PC <= initial_reg;
        GRF <= initial_reg;
        ADDR <= initial_reg;
        timewrong <= 1'b0;
        pcwrong <= 1'b0;
        grfwrong <= 1'b0;
        addrwrong <= 1'b0;
    end
    case (status)
        4'd0:begin
            Time <= initial_reg;
            PC <= initial_reg;
            GRF <= initial_reg;
            ADDR <= initial_reg;
            timewrong <= 1'b0;
            pcwrong <= 1'b0;
            grfwrong <= 1'b0;
            addrwrong <= 1'b0;
            if(char == "^")status <= S1;
            else status <= S0;
        end
        4'd1:begin
            Time <= initial_reg;
            PC <= initial_reg;
            GRF <= initial_reg;
            ADDR <= initial_reg;
            timewrong <= 1'b0;
            pcwrong <= 1'b0;
            grfwrong <= 1'b0;
            addrwrong <= 1'b0;
            if(digitchar == YES)begin
                status <= 4'd2;
                dexchecker <= 3'd1;
                Time <= char - "0";
            end
            else if(char == "^")status <=S1;
            else status <= S0;
        end
        4'd2:begin
            if(digitchar == YES)begin
                dexchecker <= dexchecker +3'd1;
                if(dexchecker +3'b1<= Dextop)begin
                    Time <= (Time << 3)+(Time << 1) + (char -"0");
                    status <= 4'd2;
                end
                else status <= 4'd0;
            end
            else if(char == "@")status <= 4'd3;
            else if(char == "^")status <= S1;
            else status <= S0;
        end
        4'd3:begin
            if(hexchar == YES)
            begin
                status <= 4'd4;  
                hexchecker <= 4'd1;  
                PC <= (char-"0");
            end
            else if(char == "^")status <= S1;
            else status <= S0;
        end
        4'd4:begin
            if(hexchar == YES)begin
                hexchecker <= hexchecker + 4'd1;
                if(hexchecker + 4'b1<= Hextop)begin
                    PC <= (PC << 4)+((char <= "9") ? (char - "0") : (char - "a" + 64'd10));
                    status <= 4'd4;
                end
                else status <= S0;
            end
            else if(char == ":")begin
                if(hexchecker == 4'd8)status <= 4'd5;
                else status <= S0;
            end
            else if(char == "^") status <= S1;
            else status <= S0;
        end
        4'd5:begin
            if(char == " ")status <= 4'd5;
            else if(char =="^")status <= S1;
            else if( char == "$")status <= 4'd6;
            else if(char == 8'd42)status <=4'd7;
            else status <= S0;
        end
        4'd6:begin
            typereg <= 1'b0;
            if(digitchar == YES)begin
                dexchecker <= 3'd1;
                GRF <= char-"0";
                status <= 4'd8;
            end
            else if(char == "^")status <= S1;
            else status <=S0;
        end
        4'd8:begin
            if(digitchar ==YES)begin
                dexchecker <= dexchecker + 3'd1;
                if(dexchecker +3'b1<= Dextop)begin
                    GRF <= (GRF << 3)+(GRF << 1)+(char -"0");
                    status <= 4'd8;
                end
                else status <= S0;
            end
            else if(char == " ")status <= 4'd9;
            else if(char == "<")status <=4'd10;
            else if(char =="^")status <=S1;
            else status <=S0;
        end
        4'd9:begin
            if(char ==" ")status <= 4'd9;
            else if( char == "<")status <= 4'd10;
            else if(char == "^")status <= S1;
            else status <= S0;
        end
        4'd10:begin
            if(char == "=")status <= 4'd11;
            else if(char == "^")status <= S1;
            else status <= S0;
        end
        4'd11:begin
            if( char == " ")status <= 4'd11;
            else if(hexchar == YES)
            begin
                hexchecker<= 4'd1;
                status <= 4'd12; 
            end
            else if(char == "^")status <= S1;
            else status <= S0; 
        end
        4'd12:begin
            if(hexchar == YES)begin
                hexchecker <= hexchecker+4'd1;
                if(hexchecker +4'b1<=Hextop)status <=4'd12;
                else status <= S0;
            end
            else if(char == "^")status <=S1;
            else if( char =="#")begin
                if(hexchecker == 4'd8)begin
                    status <=4'd13;
                    //time wrong?
                    if((Time & (-Time))>=(freq >>1)||Time == initial_reg)timewrong <= 1'b0;
                    else timewrong <= 1'b1;
                    //Pcwrong?
                    if(PC >= 64'h0000_3000 && PC <= 64'h0000_4fff)begin
                        if((PC & (-PC) )>= 64'd4|| PC == initial_reg)pcwrong <= 1'b0;
                        else pcwrong <= 1'b1; 
                    end
                    else pcwrong <= 1'b1;
                    //grfwrong?
                    if (typereg==1'b0)begin
                        if(GRF >= 64'd0&&GRF <= 64'd31)grfwrong <= 1'b0;
                        else grfwrong <= 1'b1;
                    end
                    //addrwrong?
                    else if(typereg == 1'b1)begin
                        if(ADDR >= 64'h0000_0000 && ADDR <= 64'h0000_2fff)begin
                            if((ADDR & (-ADDR)) >=64'd4||ADDR == initial_reg)addrwrong <= 1'b0;
                            else addrwrong <= 1'b1;
                        end
                        else addrwrong <= 1'b1;
                    end
                end
                else status <=S0;
            end
            else status <=S0;
        end
        4'd13:begin
            timewrong <= 1'b0;
            pcwrong <= 1'b0;
            grfwrong <= 1'b0;
            addrwrong <= 1'b0;
            if(char  =="^")status  <=S1;
            else status <=S0;
        end
        4'd7:begin
            typereg <= 1'b1;
            if(hexchar == YES)begin
                hexchecker <=4'd1;
                ADDR <= char-"0";
                status <=4'd14;
            end
            else if(char == "^")status <=S1;
            else status <=S0;
        end
        4'd14:begin
            if(hexchar == YES)begin
                hexchecker <= hexchecker+4'd1;
                if(hexchecker +4'b1<=Hextop)begin
                    ADDR = (ADDR << 4)+((char <= "9") ? (char - "0") : (char - "a" + 32'd10));
                    status <=4'd14;
                end
                else status <= S0;
            end
            else if(char == " ")begin
                if(hexchecker == 4'd8)status <=4'd9;
                else status <=S0;
            end
            else if(char == "<")begin
                if(hexchecker == 4'd8)status <=4'd10;
                else status <=S0;
            end
            else if(char == "^")status <= S1;
            else status <=S0;
        end
        default:status <= S0;
            
    endcase
end
endmodule
```

在编写该程序时，由于对非阻塞赋值不了解，犯了如下<font color=red>错误</font>
![错误示例](https://note.youdao.com/yws/api/personal/file/0D55C111835844B697593EECEDB414C6?method=download&shareKey=ecab4c2038bce57928a125e985a8ad4f)

*上图易因为对非阻塞赋值的不理解而导致错误，非阻塞复制是在**本语句块结束之后**寄存器的值才发生改变，因此在进行大小判断时，**必须加上4'b1代表理论上的寄存器取值**，实际上寄存器的值还停留在上一周期。我写的时候就没有加4'b1,导致可以多读一位hexchecker。*

## **文末附一个ISE常用简便操作的网址**

[ISE使用](https://www.cnblogs.com/ninghechuan/p/6214706.html)



