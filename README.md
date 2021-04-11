# simple-CPU-designed-by-Quartus
## 实现的指令
### 指令组

以下指令中的寄存器可以是任意一个通用寄存器。
MOV1 R1 R2            将 R2 中数据移到 R1 中

MOV2 # R1             将 R1 中数据移到#指向的地址

MOV3 R1 立即数        将立即数移到 R1 中

MOV4 S R1             将 R1 中的数据移到 S 指向的地址所指向的地址

MOV5 R1 #             将#指向地址中的内容移动到 R1 中

MOV6 R1 S             将 S 指向地址所指向的地址中的内容移动到 R1 中

JUMP1 立即数          无条件转移到立即数所表示的地址处

JUMP2 #               无条件转移到#指向的地址中的数据所表示的地址处

JUMP3 R1              无条件转移到 R1 中数据所表示的地址处

JUMP1= 立即数         相等条件转移到立即数表示的地址处

JUMP2= #              相等条件转移到#指向的地址中数据表示的地址处

JUMP3= R1             相等条件转移到 R1 中数据所表示的地址处

JUMP1> 立即数         大于条件转移到立即数表示的地址处

JUMP2> #              大于条件转移到#指向的地址中数据表示的地址处

JUMP3> R1             大于条件转移到 R1 中数据所表示的地址处

JUMP1< 立即数         小于条件转移到立即数表示的地址处

JUMP2< #              小于条件转移到#指向的地址中数据表示的地址处

JUMP3< R1             小于条件转移到 R1 中数据所表示的地址处

CLEAR                 清空 PC 中的内容

ADD                   加法

LOADD                 逻辑加

SUB                   减法

INTERSECT             逻辑交

XOR                   异或

LL A                  左移

RR A                  右移

NOT A                 非

ADD1                  A 加 1

SUB1                  A 减 1

ADD2                  A 加 B 加 1

MULTI                 八位整数乘法

HALT                  终止

运算指令的计算结果默认存储在 R6 中。

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
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/compute.jpg)
左图为总体框图中负责进行计算的部分，右图为 R1，R2 与 ALU,MULTIPLIER。4 个 74273
作为两个 16 位寄存器 R1 与 R2。R1 与 R2 的输出会输入 16ALU，R1 与 R2 的低 8 位输出会
输入 MUTItest(MULTIPLIER)。ALU 的输入为两个 16 位数据，LM,DM,RM 的移位选择，
CPR0,CPR1,CPR2 的脉冲，CN,M,S0,S1,S2,S3 的控制端。输出为计算后的 16 位数据。
MULTIPLIER 输入为两个 8 位数据，输出为一个 16 位数据。ALU 和 MUTItest(MULTIPLIER)
的输出会进入 16choser(16 位选择器)决定二者中哪个输入总线。
#### ALU原理图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/ALU.jpg)
最左侧四个寄存器分别存放输入的 16 位数据 A 和 B，由 CPR0 和 CPR1 控制两个寄存器的写入。采用 4 片 74181 和 1 片 74182 构造进位链，CN,M,S0,S1,S2,S3 控制进行的计算的类
型。进位链输出的 16 位结果输入移位器“shifter16”中，由 DM 直传，LM 左移，RM 右移控
制移位器的操作。移位器输出的结果在打入脉冲 CPR2 下存入寄存器。
其中 shifter16 以 16 位数据和 DM,LM,RM 为输入，输出移位之后的数据.
#### MULTIPLIER原理图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/multiplier.jpg)
74285 和 74284 可用于四位乘法，其中 74285 输出低四位结果，74284 输出高四位结果。
本 MULTIPLIER 将 8 位乘法转化为四位乘法:
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/multiply.jpg)
两个 12adder 为 12 位加法器，分别负责 A1xB1 与 A1xB2 相加和 A2xB1 与 A2xB2 相加。相
加时，A1xB1 应为低 8 位，高位补 0；A1xB2 为高 8 位，低位补 0；记相加结果位 M。
A2xB1 应为低 8 位，高位补 0；A2xB2 为高 8 位，低位补 0,记相加结果为 N。用 adder16
完成 M,N 化为 16 位数后的相加，其中 M 为低 12 位，高位补 0；N 为高 12 位，低位补
0。adder16 计算的结果即为 8 位乘法所得的结果。

其中为了保证乘法过程中不需要消耗时钟周期，12adder 与 adder16 都是直接相加。
12adder 与 adder16 都是由多个全加器组成。

### 比较部分
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/compare_struct.jpg)
上图为总体框图中和比较相关的部分。
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/compare.jpg)
上图为比较部分相关的原理图。左边四个 74273 为两个 16 位寄存器 R3 和 R4。四个 7485
构成了 16 位比较器。16 位比较器的输入为两个 16 位数，输出为 ALB,AEB,AGB。若 ALB 输
出 0，则 A 小于 B; 若 AEB 输出 0，则 A=B; 若 AGB 输出 0，则 A 大于 B。7485 右侧用
74273 的高三位作为 store 寄存器。Store 寄存器控制 PC 的置位端。Store 的打入脉冲为
CPSC。

通过 store 控制 PC 置位，实现跳转指令：
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/jump.jpg)
图中最上面的 and2 的输入为 AGB 的结果和跳转指令中的判定条件 A 大于 B 的信号。若二
者都为 1 则说明满足跳转条件，则 PC 置位。第二个 and2 的输入为 AEB 的结果和跳转指令
中判定 A 等于 B 的信号，若二者都为 1 则满足跳转条件，PC 置位。第三个 and2 的输入为
ALB 的结果和跳转指令中的判定条件 A 小于 B 的信号，若二者都为 1 则满足跳转条件，PC
置位。JUMP 和 JUMP=指令的跳转条件是 AEB,JUMP<是 ALB,JUMP>是 AGB。在判定跳转
条件之前，要转移到的地址已经提前放在总线上；一旦置位，则 PC 的值变成要转移到的
地址。

### 内存读写部分
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/memory_struct.jpg)
上图为与内存读写相关的部分。MAR 和 MDR 都有两个输入来源，输入来源的选择用
选择器实现。MAR 输出到 RAM 中，而 MDR 可以输出到 RAM 也可以输出到总线。本
模型机使用的 RAM 为 16 x 256，8 为地址，16 位数据。
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/memory.jpg)
上图为内存读写部分的原理图。MAR 通过 8choser 完成对 PC 和总线的选择，输出连接
RAM 的 address 端。MDR 用 16choser 选择 RAM 或总线，输出连接 RAM 的 data 端和写入
总线选择器 Choser6。

### 其他部分
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/others.jpg)
右上角的两个 74161 是 PC，两个 74161 的 ABCD 都连接到总线上，CLRN 与 clear 信号相
连，LDN 置位端连接上文所述的比较部分。为了让 PC 的 CLK 足够宽，用左下角的 74161
使得该 74161 的 QA 比系统时钟更宽。
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/map.jpg)

## 微程序实现的控制部件CU
### CU总体框图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/cu_struct.jpg)
IR 存储当前指令，IR 的高六位操作码和 uIR 的低 10 位作为微地址形成电路的地址来
源。若微地址由 IR 生成，则生成的 10 位微地址高六位为操作码，低四位为 0；该微地
址是指令对应的微程序的入口地址。微地址也可以直接由 uIR 的低 10 位直接给出。
uPC 存储当前微指令的地址，可自增或者由微地址形成电路置位。CROM 为控制寄存
器，存储微指令。uIR 为微指令寄存器，存储 CROM 中当前的微指令。IR 的三位目的
寄存器位输入 3-8 译码器译码，若译码结果指向一个寄存器，则说明该条指令需要向
该寄存器写入数据。在 uIR 中设置“toR“位，”toR“有效时，根据上述指向的寄存器发出
CP 脉冲。同时 CP 脉冲也可以直接由 uIR 中对应的三位译码产生。IR 的三位源寄存器
经 3-8 译码后若指向某个寄存器，说明该指令需要从该寄存器读数据，即该寄存器的
数据需要写入总线。在 uIR 中设置”fromR”位，若有效，则说明要将上述译码器指向的
寄存器内容写入总线，产生对应的写入总线信号。写入总线信号也可以由 uIR 对应的
三位经译码后产生。
微指令格式如下：
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/microinstruction.jpg)
### 每个部件原理图
#### IR原理图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/IR.jpg)
IR 为两个 74273 组成的 16 位寄存器。其输入与
16 位总线相连。高六位输出连接微地址形成电
路。低 8 位中和寄存器相关的六位与 uIR 中对应的
六位会对信号产生共同影响。
#### 微地址形成电路,CROM,uPC原理图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/crom_uPC.jpg)
上图中 16choser 为微地址形成电路，其中 A 输入的高六位连接 IR 的高六位，低四位
补 0；B 的 10 位输入连接 uIR 的低 10 位。依靠 Achoser 和 Bchoser 线对两种方式进行选择。uPC 由 3 个 74161 组成，生成 10 位地址。其低 10 位输入为 16choser 的 10 位
输出。CROM 为 32x1024 的控制寄存器。CROM 与 uPC 的 clock 直接与系统时钟相
连。

#### uIR原理图
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/uIR.jpg)
uIR 由 4 片 74273 组成，uIR 的输入为 CROM 的 32 位输出端 q[31..0]。uIR 不少输出都需要
经译码器译码后才能产生对应的信号。图中与 or 相关的译码器即为上述的与通用寄存器打
入脉冲相关的译码器和与写入总线入口选择有关的译码器。

### CROM内的微程序
CROM 为 32 x 1024 的 ROM, 每个指令的执行周期对应的微程序开始处的 10 位地址中，高
六位为 IR 的高六位，低四位为 0。即每个微程序占用的地址空间最多为2
4 = 16。微程序从
对应地址处开始，每个地址存放一个 32 位微指令。每个执行周期微程序由如上指令流程中
所述的执行周期的微指令码顺序组成。特别地，CROM 的 0 位开始存放实现取指周期的微
程序。每个指令的执行周期对应的微程序执行完后会使用 00000002 让 uPC 清零，即开始下
一个取指周期。
程序              CROM中的位置

取指周期          000

MOV1 R1 R2        010

MOV2 # R1         020

MOV3 R1 立即数    030

MOV4 S R1         040

MOV5 R1 #         050

MOV6 R1 S         060

JUMP1 立即数      070

JUMP2 #           080

JUMP3 R1          090

JUMP= 立即数      0a0

JUMP= #           0b0

JUMP= R1          0c0

JUMP> 立即数      0d0

JUMP> #           0e0

JUMP> R1          0f0

JUMP< 立即数      100

JUMP< #           110

JUMP< R1          120

CLEAR             130

ADD               140

LOADD             150

SUB               160

INTERSECT         170

XOR               180

LL                190

RR                1a0

NOT               1b0

ADD1              1c0

SUB1              1d0

ADD2              1e0

MULTI             1f0

HALT              200

CROM内部如下图所示：
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/CROM1.jpg)
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/CROM2.jpg)
![image](https://github.com/skylancer2019/simple-CPU-designed-by-Quartus/blob/main/pic/CROM3.jpg)
