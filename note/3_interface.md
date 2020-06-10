# 3 interface    

在我们设计数字系统或验证平台时，使用信号线将各个模块连接起来是一个非常重要的步骤。当我们使用verilog做设计时，每当我们想要对某一模块的接口进行修改的时候，我们常常会感到非常苦恼，因为我们可能因为一个模块接口的修改，而连带的修改好几个相关的模块接口，尤其是当系统比较复杂时，这样的修修补补非常容易留下错误。System Verilog引入了interface这一新结构类型来解决这一问题。

如果还是无法理解interface的作用，我们可以做一个形象的比喻：   

比如我们常用的usb接口，其中一共有四根数据线，VCC，VDD，D+，D-。如果这个世界没有usb接口，我们每次将计算机与usb设备链接时，就需要一根一根的将四根线分别连接才能通过计算机使用usb设备；而usb接口将四根线与插接方式打包固定了下来，当我们有了usb接口后，使用usb设备时直接插接接口即可，简单方便。   

对于sv中的interface来说，它与举例中的usb接口作用差不多，是将模块接口的连线封装起来，便于使用。



## 1 interface的声明与使用     

interface通过关键词`interface`来声明，声明方式与module非常类似，示例如下：

```verilog
interface module_if(input clk);

	logic port_a_0 ;
	logic port_a_1 ;
	logic port_b_0 ;
	logic port_b_1 ;

endinterface
```

下面我们来体会一下使用interface与不使用interface的区别。

假设我们有两个模块module_a与module_b，且在顶层分别例化与连接，示例如下：

```verilog
module module_a(
    input clk,
    input rst_n,
	input port_a_0 ,
    input port_a_1 ,
    output port_b_0 ,
    output port_b_1
);
    ......
    
endmodule

module module_b(
    input clk,
    input rst_n,
	input port_b_0 ,
    input port_b_1 ,
    output port_a_0 ,
    output port_a_1
);
    ......
    
endmodule

module top();
  
    logic clk ;
    logic rst_n ;
    
    logic port_a_0 ;
    logic port_a_1 ;
    logic port_b_0 ;
    logic port_b_1 ;
    
    always #10 clk = ~clk ;
    
    initial begin
    	rst_n = 0 ;
        #50;
        rst_n = 1 ;
    end
    
    
	module_a U_A(
        .clk(clk),
        .rst_n(rst_n),
        .port_a_0(port_a_0),
        .port_a_1(port_a_1),
        .port_b_0(port_b_0),
        .port_b_1(port_b_1)
	);
    
    module_b U_B(
        .clk(clk),
        .rst_n(rst_n),
        .port_b_0(port_b_0),
        .port_b_1(port_b_1),
        .port_a_0(port_a_0),
        .port_a_1(port_a_1)
	);
        
endmodule
```

显然，虽然代码非常简单，但是各种net还是非常凌乱，而且如果我们需要根据设计需要增加模块接口信号时，我们会增加超级多的工作，比如模块U_A和U_B均增加了一组交互信号port_c_0和port_c_1时，我们需要修改module_a的声明位置，module_b的声明位置， 以及例化U_A和U_B等4个位置的代码。

以上仅仅是简单的增加了两个信号的交互，如果增加更多的信号修改，或者某些信号修改位宽......简直不敢想象会有多少工时的加班。但是当有了interface这种结构，一切都变得简单了很多:


```verilog
interface module_if(input clk);
    logic rst_n,
	logic port_a_0 ;
    logic port_a_1 ;
    logic port_b_0 ;
    logic port_b_1 ;
endinterface

module module_a( module_if U_IF);
    ......
    
endmodule

module module_b( module_if U_IF);
    ......
    
endmodule

module top();
  
    logic clk ;
        
    always #10 clk = ~clk ;
    
    module_if U_IF(clk);
    
    initial begin
    	U_IF.rst_n = 0 ;
        #50;
        U_IF.rst_n = 1 ;
    end
        
    module_a U_A(U_IF);
    module_b U_B(U_IF);
        
endmodule
```

这样一来，不但代码少了很多，而且当我们需要更改模块的接口设计时，仅需要在interface内部一处修改。




## 2 使用modport分类接口       

但是如果向上面一样简单的使用interface可能还是会有点不太方便：

（1）interface中的信号可能有很多，并不是所有的模块都会用得到interface中声明的全部信号。

（2）不使用interface是，模块接口的input/output属性可以帮助我们检查连线方向的正确性，所以最好在interface中也可以规定好每一个信号在某个模块的方向。

其实在sv中，这两个问题已经得到了解决，那就是使用modport将interface中的信号进行分组打包。比如针对上面的例子，可以做一个修改：

```verilog
interface module_if(input clk);
    logic rst_n,
	logic port_a_0 ;
    logic port_a_1 ;
    logic port_b_0 ;
    logic port_b_1 ;
    
    modport A(
    	input rst_n ,
        input port_a_0 ,
        input port_a_1 ,
        output port_b_0 ,
        output port_b_1
    );
    
    modport B(
    	input rst_n ,
        output port_a_0 ,
        output port_a_1 ,
        input port_b_0 ,
        input port_b_1 
    );
    
endinterface

module module_a( module_if.A U_IF);
    ......
    
endmodule

module module_b( module_if.B U_IF);
    ......
    
endmodule

module top();
  
    logic clk ;
        
    always #10 clk = ~clk ;
    
    module_if U_IF(clk);
    
    initial begin
    	U_IF.rst_n = 0 ;
        #50;
        U_IF.rst_n = 1 ;
    end
        
    module_a U_A(U_IF);  //注意此处不需要再声明modport的名字了
    module_b U_B(U_IF);  //注意此处不需要再声明modport的名字了
        
endmodule
```

















