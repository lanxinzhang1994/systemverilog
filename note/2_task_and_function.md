# 2 Task and Function      

​    task和function在verilog中就已经存在，然而systemverilog为了便于工程使用对它们增加了许多新的特性。

## 1 task与function最大的区别有两点       

（1）task可以添加消耗时间的语句，而function不可以消耗时间。    

（2）task可以调用task和function，而function仅能调用function。

还有一点要提醒新手：

**task和function中是不能使用initial和always的。**    

## 2 task和function在SV中的新特性    

### 2.1 关于task/function参数定义风格     
**（1）类C的参数声明风格**      

在verilog中声明一个task的示例如下：

```   verilog
task task_name;
    input  [31:0]   x  ;
    output [31:0]   y  ;
    wire   [31:0]   x  ;
    reg    [31:0]   y  ;
    
    //      task 内容
    
endtask   
```

而在system verilog中，除了可以继续使用verilog中task/function的声明风格外，还提供了一种类似C语言的声明风格：

```verilog
task task_name(
    input        [31:0]   x,
    output logic [31:0]   y    //可以使用logic替代reg与wire
);
    
  //   task 内容  
endtask
```

**(2) 默认参数**         

在声明task和function时，可以设置默认的参数值，这样当使用task和function时对应参数缺省时，可以自动带入默认参数。     

```verilog
function multi(
	input int a = 10 ,
    input int b = 2  
);
    int c ;
    
    return c = a*b ;
    
endfunction
```

  **(3) 形式参数**   

在高级计算机语言中，形式参数是一种非常好的工具，使用形式参数可以节约内存且很多时候可以使程序更加灵活化，然而在verilog中并没有使用形式参数的方式，而在systemverilog中引入了形式参数。以下示例摘自《system verilog for verification》。


```verilog
function void print _checksum(const ref [31:0] a[]);
    bit [31:0] checksum=0;
    for(int i=0; i<a.size();i++)
        checksum^=a[i] ;
    $display("the array checksum is %0d",checksum);   
endfunction
```

在上例中，关键字ref表示该参数为形式引用。函数体内如果直接改变形式参数，会引起改变量的全局性改变，有时这种改变会比较“危险”，因为我们并不总希望某个函数能改变某个全局变量，特别是该全局变量比较重要的时候。为了解决这个问题，可以使用示例中的关键字const，使用const修饰的形式参数，对于函数来说依然具有形式参数的性质，同时在该函数的作用域内，该参数又可以看作是一个不可改变的常量。     




### 2.2 关于task/function返回值      

verilog中的task和function不能够使用return语句返回返回值并结束task/function，在systemverilog中增加了这一特性。

值得注意的是，由于在system verilog中task和function本身就可以使用output参数来输出数据，因此return更大的意义在于提供了一种便于灵活结束task和function的机制。     



##  3 automatic    

很多习惯C语言等高级语言的使用者常常直接将SV中的task/function和C语言中的函数使用方法等同，其实这是不正确的，C语言同时调用多个函数时函数内的变量是一个动态的变量，而SV中是静态变量。**更为直白的讲，当在程序的不同位置同时调用同一个task或function时，编译器会认为两个task/function的变量的存储空间是共享的，在一处改变某个变量，另一处程序调用的位置，该变量也会发生改变**。    

如果想让两处task/function的调用位置的变量相互独立，可以使用automatic关键字修饰task/function。

 

