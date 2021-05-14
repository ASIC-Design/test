# test

# low power design

1. one hot or differential code style ( counter 尽量使用独热码，格雷码）
2. In MUX， high toggle signals use less chain（高翻转率的信号接到mux的级联最小的端， 通常是cell级别）
3. signal gating， 在decoder的case中添加enbale，减少翻转
4. bus上翻转bits达到一半时，损耗最大，超过一半后变小
5. clock gating： if（~reset_n）a<=0; else if(en) a<=data_in; else a<=a;  使用使能开关，最后一个eles 信号保持原值，在时序逻辑中可以不用写，保持原值比置零的功耗要低。


# FSM

https://www.huaweicloud.com/articles/1921d48efd5c0aa66efa57f27f6580cb.html

【摘要】时序电路的状态是一个状态变量集合，这些状态变量在任意时刻的值都包含了为确定电路的未来行为而必需考虑的所有历史信息。 状态机采用VerilogHDL语言编码，建议分为三个always段完成。 三段式建模描述FSM的状态机输出时，只需指定case敏感表为次态寄存器， 然后直接在每个次态的case分支中描述该状态的输出即可，不用考虑状态转移条件。 三段式描述方法虽然代码结构复杂了一些，但是换...
时序电路的状态是一个状态变量集合，这些状态变量在任意时刻的值都包含了为确定电路的未来行为而必需考虑的所有历史信息。

状态机采用VerilogHDL语言编码，建议分为三个always段完成。

三段式建模描述FSM的状态机输出时，只需指定case敏感表为次态寄存器， 然后直接在每个次态的case分支中描述该状态的输出即可，不用考虑状态转移条件。

三段式描述方法虽然代码结构复杂了一些，但是换来的优势是使FSM做到了同步寄存器输出，消除了组合逻辑输出的不稳定与毛刺的隐患，而且更利于时序路径分组，一般来说在FPGA/CPLD等可编程逻辑器件上的综合与布局布线效果更佳。

//第一个进程，同步时序always模块，格式化描述次态寄存器迁移到现态寄存器

always @ (posedge clk or negedge rst_n)  //异步复位
 if(!rst_n) current_state <= IDLE;
 else current_state <= next_state;//注意，使用的是非阻塞赋值

//第二个进程，组合逻辑always模块，描述状态转移条件判断
always @ (current_state)   //电平触发
  begin next_state = x;  //要初始化，使得系统复位后能进入正确的状态 case(current_state) S1: if(...) next_state = S2;  //阻塞赋值 ... endcase
end

//第三个进程，同步时序always模块，格式化描述次态寄存器输出
always @ (posedge clk or negedge rst_n)
//初始化
....
case(next_state)
S1: out1 <= 1'b1;  //注意是非阻塞逻辑
S2: out2 <= 1'b1;
default:...   //default的作用是免除综合工具综合出锁存器。
endcase
end
二、两段式有限状态机与三段式有限状态机的区别：

　　FSM将时序部分（状态转移部分）和组合部分（判断状态转移条件和产生输出）分开，写为两个always语句，即为两段式有限状态机。将组合部分中的判断状态转移条件和产生输入再分开写，则为三段式有限状态机。
区别：
　　二段式在组合逻辑特别复杂时适用，但要注意需在后面加一个触发器以消除组合逻辑对输出产生的毛刺。三段式没有这个问题，由于第三个always会生成触发器。
　　设计时注意方面：
　　1.编码原则，binary和gray-code适用于触发器资源较少，组合电路资源丰富的情况（CPLD），对于FPGA，适用one-hot code。这样不但充分利用FPGA丰富的触发器资源，还因为只需比较一个bit，速度快，组合电路简单。
　　2.FSM初始化问题：
　　GSR(Gobal Set/Reset)只是在加电时清零所有的reg和片内ram，并不保证FSM能进入初始化状态，要利用GSR，方案是适用one-hot code with zero idle，即初始状态编码为全零。已可以适用异步复位rst
　　3.FSM输出可以适用task
　　4.FSM中的case最好加上default，默认态可以设为初始态
　　5.尤其注意：
第二段的always(组合部分，赋值用=)里面判断条件一定要包含所有情况！可以用else保证包含完全。
　　6、第二段always中，组合逻辑电平要维持超过一个clock，仿真时注意。

三、总结

为了便于记忆，把二段、三段式的特点终结成几句话：
二段式：状态切换用时序逻辑，次态输出和信号输出用组合逻辑。
三段式：状态切换用时序逻辑，次态输出用组合逻辑，信号输出用时序逻辑。信号输出的process中，case语句用next state做条件，可以解决比组合逻辑输出慢一拍的问题。
有时候判断次态需要用到计数器怎么办呢（计数器是时序电路，用组合逻辑是实现不了的）？方法是独立实现一个计数器，而在组合逻辑里用使能信号（或清除、置位等）来控制它。

参考文档：https://www.cnblogs.com/jiang-ic/p/8119079.html


