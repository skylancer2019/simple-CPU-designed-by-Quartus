# simple-CPU-designed-by-Quartus
## 模型机总体结构
### 模型机总体框图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/structure.jpg)
该模型机采用总线结构，各个寄存器都从总线得到数据，寄存器的值和计算结果通过选择
器“Choser6”返回总线。R1 与 R2 为 16 位寄存器用于存储计算相关的数据，“ALU”可进行 16
位 ALU 中相关计算，”MULTIPLIER”进行 8 位乘法运算。R3，R4，R5，R6 为 16 位通用寄存
器，都可从总线得到数据，和把数据写回总线。其中 R3,R4 可以负责存储比较器“compare”
中要比较的数据。”compare”比较的结果存入”store”中，”store”直接控制 PC 的置位端，用
于支持跳转指令。PC 为 8 位寄存器，PC 的值可以由总线的低 8 位得到或通过累加得到。
MAR 位 8 位寄存器，其值来源于总线低 8 位或者 PC，MAR 用作主存的地址寄存器。MDR
为 16 位数据寄存器，数据来源于总线或者主存，可将内容写入总线或者主存。IR 为 16 位
指令寄存器，IR 中的数据传递给 CROM。
### 计算部分
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/structure.jpg)
左图为总体框图中负责进行计算的部分，右图为 R1，R2 与 ALU,MULTIPLIER。4 个 74273
作为两个 16 位寄存器 R1 与 R2。R1 与 R2 的输出会输入 16ALU，R1 与 R2 的低 8 位输出会
输入 MUTItest(MULTIPLIER)。ALU 的输入为两个 16 位数据，LM,DM,RM 的移位选择，
CPR0,CPR1,CPR2 的脉冲，CN,M,S0,S1,S2,S3 的控制端。输出为计算后的 16 位数据。
MULTIPLIER 输入为两个 8 位数据，输出为一个 16 位数据。ALU 和 MUTItest(MULTIPLIER)
的输出会进入 16choser(16 位选择器)决定二者中哪个输入总线。
#### ALU原理图

最左侧四个寄存器分别存放输入的 16 位数据 A 和 B，由 CPR0 和 CPR1 控制两个寄存器的写入。采用 4 片 74181 和 1 片 74182 构造进位链，CN,M,S0,S1,S2,S3 控制进行的计算的类
型。进位链输出的 16 位结果输入移位器“shifter16”中，由 DM 直传，LM 左移，RM 右移控
制移位器的操作。移位器输出的结果在打入脉冲 CPR2 下存入寄存器。
其中 shifter16 以 16 位数据和 DM,LM,RM 为输入，输出移位之后的数据.
#### MULTIPLIER原理图
74285 和 74284 可用于四位乘法，其中 74285 输出低四位结果，74284 输出高四位结果。
本 MULTIPLIER 将 8 位乘法转化为四位乘法:


两个 12adder 为 12 位加法器，分别负责 A1xB1 与 A1xB2 相加和 A2xB1 与 A2xB2 相加。相
加时，A1xB1 应为低 8 位，高位补 0；A1xB2 为高 8 位，低位补 0；记相加结果位 M。
A2xB1 应为低 8 位，高位补 0；A2xB2 为高 8 位，低位补 0,记相加结果为 N。用 adder16
完成 M,N 化为 16 位数后的相加，其中 M 为低 12 位，高位补 0；N 为高 12 位，低位补
0。adder16 计算的结果即为 8 位乘法所得的结果。

其中为了保证乘法过程中不需要消耗时钟周期，12adder 与 adder16 都是直接相加。
12adder 与 adder16 都是由多个全加器组成。

### 比较部分

