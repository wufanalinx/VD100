说明
==

首先感谢大家购买芯驿电子科技（上海）有限公司出品的Versal的开发板！
您对我们和我们产品的支持和信任,给我们增添了永往直前的信心和勇气。

本教程为FPGA和裸机部分教程，通过不断练习，掌握FPGA和裸机开发的基本流程，虽然没有讲解很多大道理，但是熟能生巧，多多练习，逐渐掌握其中的奥秘。


准备工作及注意事项
==================

软件环境
--------

软件开发环境基于Vivado 2023.2

硬件环境
--------

+---------------------------------+------------------------------------+
| 开发板型号                      | 芯片型号                           |
+---------------------------------+------------------------------------+
| VD100                           | xcve2302-sfva784-1LP-e-s           |
+---------------------------------+------------------------------------+

脚本建立Vivado工程
==================

每个工程下面都有一个生成vivado的脚本，用于重建vivado工程，有两种方法可以使用，一是利用批处理文件，右键编辑create_project.bat

.. image:: images/media/image4.png
   :width: 1.16181in
   :height: 0.24653in

.. image:: images/media/image5.png
   :width: 2.26528in
   :height: 0.91042in

将路径换成自己电脑上的vivado安装路径，保存，然后双击即可生成vivado工程。

.. image:: images/media/image6.png
   :width: 5.09931in
   :height: 0.28889in

第二种方法是打开vivado软件，先用cd命令进入auto_create_project目录，然后运行source
./create_project.tcl命令。

脚本建立Vitis工程
=================

由于Vitis工程编译后占用空间较大，因此为了节省大家宝贵的时间，我们提供了Vitis工程的python脚本，在每个工程下都有个vitis文件夹，里面包含硬件描述文件xx.xsa，以及自动创建工程的脚本

.. image:: images/media/image7.png
   :width: 2.77083in
   :height: 0.77292in

大家需要做的是右键编辑builder.py文件，vitis路径换成自己电脑的安装路径

.. image:: images/media/image8.png
   :width: 3.76042in
   :height: 0.47014in

保存之后，打开vitis软件

.. image:: images/media/image9.png
   :width: 3.18611in
   :height: 2.00833in

打开新的terminal

.. image:: images/media/image10.png
   :width: 3.92222in
   :height: 0.67569in

使用cd命令进入vitis工程路径，并输入vitis -i，进入vitis CLI

.. image:: images/media/image11.png
   :width: 4.89236in
   :height: 1.2125in

输入run -t builder.py，回车

.. image:: images/media/image12.png
   :width: 4.07917in
   :height: 0.43889in

创建完成

.. image:: images/media/image13.png
   :width: 4.22778in
   :height: 1.40972in

点击Open Workspace切换到vitis工作目录

.. image:: images/media/image14.png
   :width: 4.29444in
   :height: 2.11319in

可以看到创建好的工程

.. image:: images/media/image15.png
   :width: 3.68264in
   :height: 1.84306in

这个时候，APP工程和platform可能没有关联好，要手动关联。可以先把platform编译一遍。

.. image:: images/media/image16.png
   :width: 3.67222in
   :height: 0.95764in

选中component，点设置，点击switch platform

.. image:: images/media/image17.png
   :width: 5.22153in
   :height: 4.05833in

.. image:: images/media/image18.png
   :width: 5.09306in
   :height: 1.38611in

再build工程，即可使用

.. image:: images/media/image19.png
   :width: 4.05625in
   :height: 1.15278in

Versal介绍
==========

Versal包含了Cortex-A72处理器和Cortex-R5处理器，PL端可编程逻辑部分，PMC平台管理控制器，AI
Engine等模块，与以往的ZYNQ
7000和MPSoC不同，Versal内部是通过NoC片上网络进行互联。

.. image:: images/media/image20.png
   :width: 5.83819in
   :height: 5.02917in

Versal芯片的总体框图

PS: 处理系统 （Processing System) , 就是与FPGA无关的ARM的SoC的部分。

PL: 可编程逻辑 (Progarmmable Logic), 就是FPGA部分。

NoC架构 
--------

Versal可编程片上网络（NoC)是一种AXI互连网络，用于在可编程逻辑PL，处理器系统PS等之间共享数据，而之前的Versal系列采用的AXI交叉互联模块，这是Versal的不同之处。

NoC是为可扩展性而设计的。它由一系列相互连接的水平（HNoC）和垂直（VNoC）路径，由一组可定制的硬件实现组件支持，这些组件可以以不同的方式进行配置，以满足设计时序、速度和逻辑利用率要求。以下是NoC的结构图

.. image:: images/media/image21.png
   :width: 5.84931in
   :height: 3.97153in

从NoC的结构图，可以看到，其主要由NMU（NoC master units），NSU（NoC slave
units），NPI（NoC programming interface），NPS（NoC packet
switch）组成。PS端可以连接到NMU，再通过NPS连接访问到DDRMC，同样PL端也可以通过NMU，NPS访问到DDRMC。通过NPS路由的方式，灵活地访问各模块。

.. image:: images/media/image22.png
   :width: 5.71319in
   :height: 3.05764in

NMU结构

.. image:: images/media/image23.png
   :width: 6.00069in
   :height: 2.40208in

   NSU结构

   从以上的NMU,
   NSU结构可以看到，对用户的接口仍然是AXI总线，在其内部，将AXI数据进行组包或解包，连接到NoC网络。

.. image:: images/media/image24.png
   :width: 2.71458in
   :height: 2.71944in

NPS结构

而NMU和NSU都是连接到NPS上的，它相当于一个路由器，将数据转发给目的设备。它是一个全双工的4x4
switch，每个端口在每个方向支持8个虚拟通道，采用基于信用的流控，类似于TCP的滑动窗口。

NoC是Versal开发中非常重要的部件，PS端访问DDR，PL端访问DDR都是通过NoC，与Versal不同的是，versal在PS端没有DDR控制器，都是通过NoC访问，因此了解NoC结构是很有必要的，更多的内容可以参考官方的pg313文档。

PMC架构
-------

PMC（平台管理控制器）在启动，配置，运行时做平台的管理。从下图的结构图中可以看出，PMC由ROM
Code Unit，Platform Processing Unit，PMC I/O
Peripherals等单元组成，功能丰富。在这里主要介绍一下PMC是如何引导程序启动的。

.. image:: images/media/image25.png
   :width: 5.84861in
   :height: 6.04444in

PMC结构图

.. image:: images/media/image26.png
   :width: 4.77431in
   :height: 3.00417in

第一阶段：Pre-Boot

1. PMC检测PMC电源和POR_B释放

2. PMC读取启动模式引脚并存入boot mode寄存器

3. PMC发送复位给RCU（ROM code unit)

.. image:: images/media/image27.png
   :width: 4.62431in
   :height: 3.62917in

第二阶段：Boot Setup

4. RCU从RCU ROM中执行BootROM

5. BootROM读出boot mode寄存器，选择启动设备

6. BootROM从启动设备读取PDI（programmable device image)并校验

7. BootROM释放PPU的复位，将PLM加载到PPU RAM并校验。校验后，PPU唤醒，PLM
软件开始执行。

8. BootROM进入睡眠状态

.. image:: images/media/image28.png
   :width: 4.77431in
   :height: 3.27153in

第三阶段：Load Platform

9. PPU开始从PPU RAM中执行PLM

10. PLM开始读取并运行PDI模块

11. PLM利用PDI内容配置Versal其他部分

11a: PLM为以下模块配置数据： PMC， PS clocks

(MIO ,clocks, resets等）(CDO文件）

NoC初始化和NPI模块（DDR控制器，NoC，

GT,XPIPE,I/Os,clocking和其他NPI模块

PLM加载APU和RPU的应用程序ELF到存储空间，

如DDR，OCM，TCM等

11b: PL端逻辑配置

PL端数据（CFI文件）

AI Engine配置（AI Engine CDO)

.. image:: images/media/image28.png
   :width: 4.77431in
   :height: 3.27153in

第四阶段：Post-Boot

12.
PLM继续运行，直到下一次POR或系统复位。并负责DFX重配置，电源管理，子系统
重启，错误管理，安全服务。

Versal芯片开发流程的简介
------------------------

由于Versal将CPU与FPGA集成在了一起，开发人员既需要设计ARM的操作系统应用程序和设备的驱动程序，又需要设计FPGA部分的硬件逻辑设计。开发中既要了解Linux操作系统，系统的构架，也需要搭建一个FPGA和ARM系统之间的硬件设计平台。所以Versal的开发是需要软件人员和硬件硬件人员协同设计并开发的。这既是Versal开发中所谓的"软硬件协同设计”。

Versal系统的硬件系统和软件系统的设计和开发需要用到一下的开发环境和调试工具：
Xilinx
Vivado。Vivado设计套件实现FPGA部分的设计和开发，管脚和时序的约束，编译和仿真，实现RTL到比特流的设计流程。

Xilinx
Vitis是Xilinx软件开发套件(SDK),在Vivado硬件系统的基础上，系统会自动配置一些重要参数，其中包括工具和库路径、编译器选项、JTAG和闪存设置，调试器连接已经裸机板支持包(BSP)。SDK也为所有支持的Xilinx
IP硬核提供了驱动程序。Vitis支持IP硬核（FPGA上）和处理器软件协同调试，我们可以使用高级C或C++语言来开发和调试ARM和FPGA系统，测试硬件系统是否工作正常。Vitis软件也是Vivado软件自带的，无需单独安装。

Versal的开发也是先硬件后软件的方法。具体流程如下：

1) 在Vivado上新建工程，增加一个嵌入式的源文件。

2) 在Vivado里添加和配置PS和PL部分基本的外设，或需要添加自定义的外设。

3) 在Vivado里生成顶层HDL文件，并添加约束文件。再编译生成比特流文件（*.pdi）。

4) 导出硬件信息到Vitis软件开发环境，在Vitis环境里可以编写一些调试软件验证硬件和软件，结合比特流文件单独调试Versal系统。

5) 在VMware虚拟机里生成u-boot.elf、 bootloader 镜像。

6) 在Vitis里将比特流文件和u-boot.elf文件生成一个BOOT.pdi文件。

7) 在VMware里生成Ubuntu的内核镜像文件Zimage和Ubuntu的根文件系统。另外还需要要对FPGA自定义的IP编写驱动。

8) 把BOOT、内核、设备树、根文件系统文件放入到SD卡中，启动开发板电源，Linux操作系统会从SD卡里启动。

学习Versal要具备哪些技能
------------------------

学习Versal比学习FPGA、MCU、ARM等传统工具开发要求更高，想学好Versal也不是一蹴而就的事情。

软件开发人员
~~~~~~~~~~~~

-  计算机组成原理

-  C、C++语言

-  计算机操作系统

-  tcl脚本

-  良好的英语阅读基础

逻辑开发人员
~~~~~~~~~~~~

-  计算机组成原理

-  C语言

-  数字电路基础

PL的"Hello World"LED实验
========================

**实验Vivado工程为“led”。**

对于Versal来说PL（FPGA）开发是至关重要的，这也是Versal比其他ARM的有优势的地方，可以定制化很多ARM端的外设，在定制ARM端的外设之前先让我们通过一个LED例程来熟悉PL（FPGA）的开发流程，熟悉Vivado软件的基本操作，这个开发流程和不带ARM的FPGA芯片完全一致。

在本例程中，我们要做的是LED灯控制实验，每秒钟控制开发板上的LED灯翻转一次，实现亮、灭、亮、灭的控制。

LED硬件介绍
-----------

开发板的PL部分连接了1个红色的用户LED灯。这1个灯完全由PL控制。如果PL_LED1为高电平，三级管导通，灯则会亮，否则会灭。

.. image:: images/media/image29.png
   :width: 3.22222in
   :height: 2.47569in

创建Vivado工程
--------------

1) 启动Vivado，在Windows中可以通过双击Vivado快捷方式启动

2) 在Vivado开发环境里点击“Create New Project”，创建一个新的工程。

.. image:: images/media/image30.png
   :width: 4.90245in
   :height: 3.54576in

3) 弹出一个建立新工程的向导，点击“Next”

.. image:: images/media/image31.png
   :width: 4.82126in
   :height: 4.08408in

4) 在弹出的对话框中输入工程名和工程存放的目录，我们这里取一个led的工程名。需要注意工程路径“Project
   location”不能有中文空格，路径名称也不能太长。

.. image:: images/media/image32.png
   :width: 4.85347in
   :height: 3.06944in

5) 在工程类型中选择“RTL Project”

.. image:: images/media/image33.png
   :width: 5.26181in
   :height: 3.32917in

6) 目标语言“Target
   language”选择“Verilog”，虽然选择Verilog，但VHDL也可以使用，支持多语言混合编程。

.. image:: images/media/image34.png
   :width: 5.20556in
   :height: 3.27708in

7) 点击“Next”，不添加任何文件

.. image:: images/media/image35.png
   :width: 5.39514in
   :height: 3.34097in

8) 选择“xc2302-sfva784-1LP-e-S”

.. image:: images/media/image36.png
   :width: 5.13403in
   :height: 4.59444in

9) 点击“Finish”就可以完成以后名为“led”工程的创建。

.. image:: images/media/image37.png
   :width: 5.40347in
   :height: 3.40417in

10) Vivado软件界面

.. image:: images/media/image38.png
   :width: 4.61346in
   :height: 3.97672in

创建Verilog HDL文件点亮LED
--------------------------

1) 点击Project Manager下的Add Sources图标（或者使用快捷键Alt+A）

.. image:: images/media/image39.png
   :width: 3.88736in
   :height: 2.26719in

2) 选择添加或创建设计源文件“Add or create design sources”,点击“Next”

.. image:: images/media/image40.png
   :width: 5.11453in
   :height: 3.45338in

3) 选择创建文件“Create File”

.. image:: images/media/image41.png
   :width: 5.19748in
   :height: 3.5094in

4) 文件名“File name”设置为“led”，点击“OK”

.. image:: images/media/image42.png
   :width: 4.86244in
   :height: 3.28317in

5) 点击“Finish”,完成“led.v”文件添加

.. image:: images/media/image43.png
   :width: 4.89769in
   :height: 3.30698in

6) 在弹出的模块定义“Define
   Module”,中可以指定“led.v”文件的模块名称“Module
   name”,这里默认不变为“led”，还可以指定一些端口，这里暂时不指定，点击“OK”。

.. image:: images/media/image44.png
   :width: 4.48908in
   :height: 3.21372in

7) 在弹出的对话框中选择“Yes”

.. image:: images/media/image45.png
   :width: 4.33533in
   :height: 3.10366in

8) 双击“led.v”可以打开文件，然后编辑

.. image:: images/media/image46.png
   :width: 4.52898in
   :height: 3.45462in

9) 编写“led.v”,这里定义了一个32位的寄存器timer,
   用于循环计数0~199999999(1秒钟), 计数到199999999(1秒)的时候,
   寄存器timer变为0，并翻转四个LED。这样原来LED是灭的话，就会点亮，如果原来LED为亮的话，就会熄灭。由于输入时钟为200MHz的差分时钟，因此需要添加IBUFDS原语连接差分信号，编写好后的代码如下：

+-----------------------------------------------------------------------+
| \`timescale 1ns **/** 1ps                                             |
|                                                                       |
| **module** led\ **(**                                                 |
|                                                                       |
| //Differential system clock                                           |
|                                                                       |
| **input** sys_clk_p\ **,**                                            |
|                                                                       |
| **input** sys_clk_n\ **,**                                            |
|                                                                       |
| **input** rst_n\ **,**                                                |
|                                                                       |
| **output** **reg** led                                                |
|                                                                       |
| **);**                                                                |
|                                                                       |
| **reg[**\ 31\ **:**\ 0\ **]** timer_cnt\ **;**                        |
|                                                                       |
| **wire** sys_clk **;**                                                |
|                                                                       |
| IBUFDS IBUFDS_inst **(**                                              |
|                                                                       |
| **.**\ O\ **(**\ sys_clk\ **),** // Buffer output                     |
|                                                                       |
| **.**\ I\ **(**\ sys_clk_p\ **),** // Diff_p buffer input (connect    |
| directly to top-level port)                                           |
|                                                                       |
| **.**\ IB\ **(**\ sys_clk_n\ **)** // Diff_n buffer input (connect    |
| directly to top-level port)                                           |
|                                                                       |
| **);**                                                                |
|                                                                       |
| **always@(posedge** sys_clk\ **)**                                    |
|                                                                       |
| **begin**                                                             |
|                                                                       |
| **if** **(!**\ rst_n\ **)**                                           |
|                                                                       |
| **begin**                                                             |
|                                                                       |
| led **<=** 1'b0 **;**                                                 |
|                                                                       |
| timer_cnt **<=** 32'd0 **;**                                          |
|                                                                       |
| **end**                                                               |
|                                                                       |
| **else** **if(**\ timer_cnt **>=** 32'd199_999_999\ **)** //1 second  |
| counter, 200M-1=199999999                                             |
|                                                                       |
| **begin**                                                             |
|                                                                       |
| led **<=** **~**\ led\ **;**                                          |
|                                                                       |
| timer_cnt **<=** 32'd0\ **;**                                         |
|                                                                       |
| **end**                                                               |
|                                                                       |
| **else**                                                              |
|                                                                       |
| **begin**                                                             |
|                                                                       |
| led **<=** led\ **;**                                                 |
|                                                                       |
| timer_cnt **<=** timer_cnt **+** 32'd1\ **;**                         |
|                                                                       |
| **end**                                                               |
|                                                                       |
| **end**                                                               |
|                                                                       |
| **endmodule**                                                         |
+-----------------------------------------------------------------------+

10) 编写好代码后保存

添加管脚约束
------------

Vivado使用的约束文件格式为xdc文件。xdc文件里主要是完成管脚的约束,时钟的约束,
以及组的约束。这里我们需要对led.v程序中的输入输出端口分配到FPGA的真实管脚上。

1) 新建约束文件

.. image:: images/media/image47.png
   :width: 5.99722in
   :height: 2.96736in

2) Create File

.. image:: images/media/image48.png
   :width: 4.95556in
   :height: 3.31319in

.. image:: images/media/image49.png
   :width: 2.33472in
   :height: 1.8in

3) 将复位信号rst_n绑定到PL端的按键，给LED和时钟分配管脚、电平标准，约束如下

.. image:: images/media/image50.png
   :width: 4.82986in
   :height: 1.96389in

+-----------------------------------------------------------------------+
| set_property PACKAGE_PIN AB23 [get_ports sys_clk_p]                   |
|                                                                       |
| set_property PACKAGE_PIN F21 [get_ports rst_n]                        |
|                                                                       |
| set_property PACKAGE_PIN E20 [get_ports led]                          |
|                                                                       |
| set_property IOSTANDARD LVCMOS15 [get_ports led]                      |
|                                                                       |
| set_property IOSTANDARD LVCMOS15 [get_ports rst_n]                    |
|                                                                       |
| set_property IOSTANDARD LVDS15 [get_ports sys_clk_p]                  |
|                                                                       |
| create_clock -period 5.000 -name sys_clk_p -waveform {0.000 2.500}    |
| [get_ports sys_clk_p]                                                 |
+-----------------------------------------------------------------------+

生成pdi文件
-----------

1) 编译的过程可以细分为综合、布局布线、生成bit文件等，这里我们直接点击“Generate
   Device Image”,直接生成pdi文件。

.. image:: images/media/image51.png
   :width: 1.8375in
   :height: 0.75069in

2) 在弹出的对话框中可以选择任务数量，这里和CPU核心数有关，一般数字越大，编译越快，点击“OK”

.. image:: images/media/image52.png
   :width: 2.2739in
   :height: 1.78158in

3)  编译的时候发现有报错

    .. image:: images/media/image53.png
       :width: 5.98611in
       :height: 0.78264in

    [DRC CIPS-2] Versal CIPS exists check - wdi: Versal designs must
    contain a CIPS IP in the netlist hierarchy to function properly.
    Please create an instance of the CIPS IP and configure it. Without a
    CIPS IP in the design, Vivado will not generate a CDO for the PMC,
    an elf for the PLM.

    从报错来看，versal设计是必须包含CIPS的，也就是PS端，因此需要添加CIPS核。

4)  选择Create Block Design

    .. image:: images/media/image54.png
       :width: 2.26458in
       :height: 2.29792in

    .. image:: images/media/image55.png
       :width: 3.19792in
       :height: 1.73125in

5)  添加CIPS

    .. image:: images/media/image56.png
       :width: 5.19167in
       :height: 2.67778in

    .. image:: images/media/image57.png
       :width: 2.63333in
       :height: 2.09792in

6)  双击CIPS，选择PL_Subsystem，只有PL端的逻辑

    .. image:: images/media/image58.png
       :width: 4.18542in
       :height: 3.7875in

7)  右键Generate Output products

    .. image:: images/media/image59.png
       :width: 2.89653in
       :height: 1.85833in

    .. image:: images/media/image60.png
       :width: 2.08403in
       :height: 2.85278in

8)  之后右键创建HDL

    .. image:: images/media/image61.png
       :width: 3.44167in
       :height: 1.77569in

    .. image:: images/media/image62.png
       :width: 3.06875in
       :height: 1.50694in

9)  在led.v中例化PS端文件

    .. image:: images/media/image63.png
       :width: 1.49444in
       :height: 0.55972in

    .. image:: images/media/image64.png
       :width: 3.28958in
       :height: 1.52986in

10) 之后再Generate
    Bitstream，编译中没有任何错误，编译完成，弹出一个对话框让我们选择后续操作，可以选择“Open
    Hardware Manger”，当然，也可以选择“Cancel”，我们这里选择
    “Cancel”，先不下载。

.. image:: images/media/image65.png
   :width: 2.51597in
   :height: 1.51181in

Vivado仿真
----------

接下来我们不妨小试牛刀，利用Vivado自带的仿真工具来输出波形验证流水灯程序设计结果和我们的预想是否一致（注意：在生成bit文件之前也可以仿真）。具体步骤如下：

1. 设置Vivado的仿真配置，右击SIMULATION中Simulation Settings。

.. image:: images/media/image66.png
   :width: 2.71162in
   :height: 2.82275in

2. 在Simulation
   Settings窗口中进行如下图来配置，这里设置成50ms（根据需要自行设定）,其它按默认设置，单击OK完成。

.. image:: images/media/image67.png
   :width: 4.16967in
   :height: 3.68114in

3. 添加激励测试文件，点击Project Manager下的Add
   Sources图标,按下图设置后单击Next。

.. image:: images/media/image68.png
   :width: 4.24388in
   :height: 2.19655in

4. 点击Create File生成仿真激励文件。

.. image:: images/media/image69.png
   :width: 3.47146in
   :height: 2.72528in

在弹出的对话框中输入激励文件的名字，这里我们输入名为vtf_led_test。

.. image:: images/media/image70.png
   :width: 2.21088in
   :height: 1.80096in

5. 点击Finish按钮返回。

.. image:: images/media/image71.png
   :width: 3.95375in
   :height: 3.03139in

这里我们先不添加IO Ports，点击OK。

.. image:: images/media/image72.png
   :width: 3.1395in
   :height: 2.2426in

在Simulation
Sources目录下多了一个刚才添加的vtf_led_test文件。双击打开这个文件，可以看到里面只有module名的定义，其它都没有。

.. image:: images/media/image73.png
   :width: 4.14019in
   :height: 2.71368in

6. 接下去我们需要编写这个vtf_led_test.v文件的内容。首先定义输入和输出信号，然后需要实例化led_test模块，让led_test程序作为本测试程序的一部分。再添加复位和时钟的激励。完成后的vtf_led_test.v文件如下：

+-----------------------------------------------------------------------+
| \`timescale 1ns **/** 1ps                                             |
|                                                                       |
| /////////////                                                         |
| ///////////////////////////////////////////////////////////////////// |
|                                                                       |
| // Module Name: vtf_led_test                                          |
|                                                                       |
| /////////////                                                         |
| ///////////////////////////////////////////////////////////////////// |
|                                                                       |
| **module** vtf_led_test\ **;**                                        |
|                                                                       |
| // Inputs                                                             |
|                                                                       |
| **reg** sys_clk_p\ **;**                                              |
|                                                                       |
| **reg** rst_n **;**                                                   |
|                                                                       |
| **wire** sys_clk_n\ **;**                                             |
|                                                                       |
| // Outputs                                                            |
|                                                                       |
| **wire** led\ **;**                                                   |
|                                                                       |
| // Instantiate the Unit Under Test (UUT)                              |
|                                                                       |
| led uut **(**                                                         |
|                                                                       |
| **.**\ sys_clk_p\ **(**\ sys_clk_p\ **),**                            |
|                                                                       |
| **.**\ sys_clk_n\ **(**\ sys_clk_n\ **),**                            |
|                                                                       |
| **.**\ rst_n\ **(**\ rst_n\ **),**                                    |
|                                                                       |
| **.**\ led\ **(**\ led\ **)**                                         |
|                                                                       |
| **);**                                                                |
|                                                                       |
| **initial**                                                           |
|                                                                       |
| **begin**                                                             |
|                                                                       |
| // Initialize Inputs                                                  |
|                                                                       |
| sys_clk_p **=** 0\ **;**                                              |
|                                                                       |
| rst_n **=** 0\ **;**                                                  |
|                                                                       |
| // Wait for global reset to finish                                    |
|                                                                       |
| **#**\ 1000\ **;**                                                    |
|                                                                       |
| rst_n **=** 1\ **;**                                                  |
|                                                                       |
| **end**                                                               |
|                                                                       |
| //Create clock                                                        |
|                                                                       |
| **always** **#**\ 2.5 sys_clk_p **=** **~** sys_clk_p\ **;**          |
|                                                                       |
| **assign** sys_clk_n **=** **~**\ sys_clk_p **;**                     |
|                                                                       |
| **endmodule**                                                         |
+-----------------------------------------------------------------------+

7) 编写好后保存，vtf_led_test.v自动成了这个仿真Hierarchy的顶层了，它下面是设计文件led_test.v。

.. image:: images/media/image74.png
   :width: 2.63408in
   :height: 2.45107in

8) 点击Run Simulation按钮，再选择Run Behavioral
   Simulation。这里我们做一下行为级的仿真就可以了。

.. image:: images/media/image75.png
   :width: 2.88031in
   :height: 3.23482in

如果没有错误，Vivado中的仿真软件开始工作了。

10.
在弹出仿真界面后如下图，界面是仿真软件自动运行到仿真设置的50ms的波形。

.. image:: images/media/image76.png
   :width: 6.00417in
   :height: 1.23611in

由于LED[3：0]在程序中设计的状态变化时间长，而仿真又比较耗时，在这里观测timer[31:0]计数器变化。把它放到Wave中观察(点击Scope界面下的uut，
再右键选择Objects界面下的timer， 在弹出的下拉菜单里选择Add Wave
Window)。

.. image:: images/media/image77.png
   :width: 3.82425in
   :height: 2.22484in

添加后timer显示在Wave的波形界面上，如下图所示。

.. image:: images/media/image78.png
   :width: 4.75283in
   :height: 1.31547in

11. 点击如下标注的Restart按钮复位一下，再点击Run
All按钮。（需要耐心！！！），可以看到仿真波形与设计相符。（注意：仿真的时间越长，仿真的波形文件占用的磁盘空间越大，波形文件在工程目录的xx.sim文件夹）

.. image:: images/media/image79.png
   :width: 4.16502in
   :height: 1.82527in

.. image:: images/media/image80.png
   :width: 6.00417in
   :height: 1.37986in

我们可以看到led的信号会变成1，说明LED灯会变亮。

下载
----

1) 连接好开发板的JTAG接口，给开发板上电，注意拔码开关要选择JTAG模式，也就是全部拔到”ON”，“ON”代表的值是0，不用JTAG模式，下载会报错。

.. image:: images/media/image81.png
   :width: 5.50347in
   :height: 3.82569in

.. image:: images/media/image82.png
   :width: 4.09375in
   :height: 2.23403in

2) 在“HARDWARE MANAGER”界面点击“Auto Connect”，自动连接设备

.. image:: images/media/image83.png
   :width: 3.01461in
   :height: 2.12162in

3) 选择芯片，右键“Program Device...”

.. image:: images/media/image84.png
   :width: 3.34583in
   :height: 2.10347in

4) 在弹出窗口中点击“Program”

.. image:: images/media/image85.png
   :width: 3.53194in
   :height: 1.88056in

5) 等待下载

.. image:: images/media/image86.png
   :width: 3.18855in
   :height: 0.87404in

6) 下载完成以后，我们可以看到PL
   LED开始每秒变化一次。到此为止Vivado简单流程体验完成。后面的章节会介绍如果把程序烧录到Flash，需要PS系统的配合才能完成，只有PL的工程不能直接烧写Flash。在”体验ARM，裸机输出”Hello
   World”一章的常见问题中有介绍。

实验总结
--------

本章节介绍了如何在PL端开发程序，包括工程建立，约束，仿真等方法，在后续的代码开发方式中皆可参考此方法。

PL通过NoC读写DDR4实验
=====================

**实验VIvado工程为“pl_rw_ddr”。**

硬件介绍
--------

开发板的PL端有4颗16bit ddr4

.. image:: images/media/image87.png
   :width: 4.39028in
   :height: 2.6in

Vivado工程建立
--------------

Versal的DDR4是通过NoC访问，因此需要添加NoC IP进行配置。

创建一个Block design并配置NoC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1)  选择Create Block Design

    .. image:: images/media/image54.png
       :width: 2.26458in
       :height: 2.29792in

    .. image:: images/media/image88.png
       :width: 3.01319in
       :height: 1.87153in

2)  添加CIPS

    .. image:: images/media/image56.png
       :width: 5.19167in
       :height: 2.67778in

    .. image:: images/media/image57.png
       :width: 2.63333in
       :height: 2.09792in

3)  双击CIPS，选择PL_Subsystem，只有PL端的逻辑

    .. image:: images/media/image58.png
       :width: 4.18542in
       :height: 3.7875in

4)  添加NoC IP

    .. image:: images/media/image89.png
       :width: 2.42222in
       :height: 2.80486in

5)  配置NoC

    选择一个AXI Slave和AXI Clock，选择”Single Memory Controller”

    .. image:: images/media/image90.png
       :width: 5.60972in
       :height: 3.17778in

    选择Inputs为PL

    .. image:: images/media/image91.png
       :width: 6in
       :height: 1.225in

    连接port

    .. image:: images/media/image92.png
       :width: 6.01389in
       :height: 1.39028in

    DDR4配置

    .. image:: images/media/image93.png
       :width: 5.39792in
       :height: 3.20069in

    .. image:: images/media/image94.png
       :width: 5.99583in
       :height: 2.42569in

    配置完成，点击OK

6)  配置CIPS，添加复位

    .. image:: images/media/image95.png
       :width: 1.79444in
       :height: 0.89931in

    .. image:: images/media/image96.png
       :width: 3.64028in
       :height: 3.11458in

    .. image:: images/media/image97.png
       :width: 3.52014in
       :height: 3.04236in

    .. image:: images/media/image98.png
       :width: 2.83056in
       :height: 2.25486in

    点击Finish

7)  添加Clocking Wizard，配置输出时钟150MHz，作为PL端读写时钟

    .. image:: images/media/image99.png
       :width: 1.37014in
       :height: 0.62917in

    .. image:: images/media/image100.png
       :width: 5.625in
       :height: 1.73681in

8)  添加IBUFDS为NoC和Clocking
    Wizard提供参考时钟，并导出S00_AXI，CH0_DDR4_0等总线，添加axi_clk,axi_resetn为PL端提供时钟和复位。

    .. image:: images/media/image101.png
       :width: 5.99167in
       :height: 2.18958in

    双击参考时钟引脚，并配置频率为200MHz

    .. image:: images/media/image102.png
       :width: 2.75208in
       :height: 1.58056in

    双击AXI总线，并配置

    .. image:: images/media/image103.png
       :width: 4.45972in
       :height: 3.44375in

    .. image:: images/media/image104.png
       :width: 4.12431in
       :height: 2.81597in

9)  分配地址

    .. image:: images/media/image105.png
       :width: 5.42708in
       :height: 1.325in

    .. image:: images/media/image106.png
       :width: 6.00278in
       :height: 1.41458in

10) Create HDL

    .. image:: images/media/image107.png
       :width: 4.37083in
       :height: 1.55972in

添加其他测试代码
~~~~~~~~~~~~~~~~

其他代码主要功能是读写ddr4并比较数据是否一致，这里不做详细介绍，可参考工程代码。

.. image:: images/media/image108.png
   :width: 3.17708in
   :height: 2.13056in

1) 在mem_test.v中添加mark_debug调试

.. image:: images/media/image109.png
   :width: 3.94143in
   :height: 2.8396in

2) 引脚绑定

   .. image:: images/media/image110.png
      :width: 1.65069in
      :height: 1.32917in

3) 综合

   .. image:: images/media/image111.png
      :width: 1.95694in
      :height: 0.85278in

3. 综合完成后点击Set up debug

   .. image:: images/media/image112.png
      :width: 1.72292in
      :height: 2.53125in

   .. image:: images/media/image113.png
      :width: 3.80139in
      :height: 2.40208in

   .. image:: images/media/image114.png
      :width: 3.98681in
      :height: 2.53333in

   根据需求设置采样点数

   .. image:: images/media/image115.png
      :width: 4.25069in
      :height: 2.7125in

   .. image:: images/media/image116.png
      :width: 4.31111in
      :height: 2.74792in

   之后保存，并生成pdi文件

   .. image:: images/media/image51.png
      :width: 1.8375in
      :height: 0.75069in

下载调试
--------

生成pdi文件以后，使用JTAG下载到开发板，在MIG_1窗口会显示DDR4校准等信息

.. image:: images/media/image117.png
   :width: 6.00278in
   :height: 3.32917in

在hw_ila_1中可以查看调试信号

.. image:: images/media/image118.png
   :width: 6in
   :height: 3.0125in

.. _实验总结-1:

实验总结
--------

本实验通过PL端Verilog代码直接读写ddr4，主要了解NoC的配置方法，如何通过NoC访问DDR4，后续的实验中都要用到此配置。

LVDS液晶屏显示实验
==================

**实验Vivado工程为“lvds_lcd”。**

本章介绍lvds lcd液晶屏的color bar显示。

.. _硬件介绍-1:

硬件介绍
--------

ALINX黑金7寸LCD屏模块(AN7000)采用IVO的7寸TFT LCD液晶屏,
液晶屏的型号为M070AWAD R0。AN7000 LCD屏模块由TFT
液晶屏和驱动板组成，具体参数可以参考AN7000的用户手册。AN7000实物照片如下：

.. image:: images/media/image119.png
   :alt: \_K4A5291
   :width: 5.37431in
   :height: 3.34722in

AN7000 LCD屏正面图

程序设计
--------

1) 与PL的“Hello World” LED实验一样，添加一个block
   design，并添加CIPS核，配置成PL Subsystem

   .. image:: images/media/image120.png
      :width: 2.17639in
      :height: 1.05556in

2. 添加LVDS LCD的控制器IP

   .. image:: images/media/image121.png
      :width: 1.78542in
      :height: 1.19028in

3. 添加Advanced IO Wizard，并配置

   .. image:: images/media/image122.png
      :width: 4.32222in
      :height: 3.34167in

   .. image:: images/media/image123.png
      :width: 4.3in
      :height: 2.89028in

   .. image:: images/media/image124.png
      :width: 4.62847in
      :height: 2.30694in

4. 连接如下

   .. image:: images/media/image125.png
      :width: 5.68681in
      :height: 2.65486in

5. 添加color bar文件，并拖到block design中，并连接

   .. image:: images/media/image126.png
      :width: 3.91597in
      :height: 1.97222in

   在video_define.v中定义VIDEO_1280_720，因为LCD分辨率是1280*720

   .. image:: images/media/image127.png
      :width: 1.94861in
      :height: 0.59722in

6. 生成HDL文件

   .. image:: images/media/image128.png
      :width: 2.46181in
      :height: 1.31875in

7. 添加其他一些信号

   .. image:: images/media/image129.png
      :width: 5.85069in
      :height: 2.89861in

8. 约束引脚

   .. image:: images/media/image130.png
      :width: 2.33611in
      :height: 1.44097in

9. 生成pdi文件

   .. image:: images/media/image51.png
      :width: 1.8375in
      :height: 0.75069in

实验现象
--------

连接好LCD屏，下载程序，即可看到彩条显示。

.. image:: images/media/image131.png
   :width: 3.72014in
   :height: 4.87708in

.. image:: images/media/image132.png
   :width: 5.35347in
   :height: 3.80694in

GTYP收发器误码率测试IBERT实验
=============================

**实验VIvado工程为“ibert_test”,目录中还有一个“ibert_ex”，是生成的测试工程。**

Vidado软件为我们提供了强大的误码率测试器IBERT，不但可以测试误码率还能测试眼图，给我们使用高速收发器带来很大的便利，本实验做个抛砖引玉，简单介绍IBERT的使用。

.. _硬件介绍-2:

硬件介绍
--------

使用IBERT测试误码率和眼图必须有个收发环通的硬件，开发板上有2个SFP光纤接口，本实验把2个光接口收发两两连接，形成2个收发环通链路。

.. _vivado工程建立-1:

Vivado工程建立
--------------

1) 新建一个工程名为“ibert_test”

2) 在“IP Catalog”中搜索“gt”快速找到“Versal ACAPs Transceivers
   Wizard”,双击

.. image:: images/media/image133.png
   :width: 5.99722in
   :height: 1.49167in

3)  “Component Name”改为”ibert”，并修改preset为“Aurora 64B/66B”

    .. image:: images/media/image134.png
       :width: 6.00208in
       :height: 3.88889in

4)  点击Transceiver Configs Protocol 0，配置发送和接收参数，点击OK

    .. image:: images/media/image135.png
       :width: 3.72083in
       :height: 1.70903in

    .. image:: images/media/image136.png
       :width: 6.00347in
       :height: 4.52292in

    .. image:: images/media/image137.png
       :width: 5.99722in
       :height: 4.48472in

5)  点击Generate

    .. image:: images/media/image138.png
       :width: 2.625in
       :height: 3.27153in

6)  右键“Open IP Example Design...”,选择example工程路径

    .. image:: images/media/image139.png
       :width: 3.3875in
       :height: 2.54236in

    .. image:: images/media/image140.png
       :width: 3.84653in
       :height: 1.75556in

7)  添加buffer连接到apb3clk

    .. image:: images/media/image141.png
       :width: 5.9875in
       :height: 3.14722in

8)  添加反向器连接到复位

    .. image:: images/media/image142.png
       :width: 5.99514in
       :height: 1.95069in

9)  其他一些信号配置为常数0

    .. image:: images/media/image143.png
       :width: 3.93056in
       :height: 3.19722in

10) 删除输出信号

    .. image:: images/media/image144.png
       :width: 2.025in
       :height: 1.57778in

11) 配置sfp_disable为0

    .. image:: images/media/image145.png
       :width: 4.46458in
       :height: 1.00556in

12) 将CIPS改成PL Subsystem

    .. image:: images/media/image146.png
       :width: 5.47014in
       :height: 4.74514in

13) 约束引脚

    .. image:: images/media/image147.png
       :width: 5.99583in
       :height: 5.09167in

14) 生成pdi文件

    .. image:: images/media/image148.png
       :width: 1.72431in
       :height: 0.79444in

.. _下载调试-1:

下载调试
--------

1) 插入光模块，然后使用光纤将2个光口对接，连接好JTAG下载线，给开发板上电

.. image:: images/media/image149.png
   :width: 5.99028in
   :height: 3.39931in

2) 使用JTAG下载BIT文件到开发板，可以看到速度接近10.3125Gbps。

.. image:: images/media/image150.png
   :width: 2.70625in
   :height: 3.36528in

3) 选择IBERT，右键，选择“Create Links”

.. image:: images/media/image151.png
   :width: 3.33819in
   :height: 1.68889in

参考原理图，光纤连接到了Quad104的CH0和CH1，选择Link 0为Quad_104 CH_0
TX和CH1 RX对应，Link 1为Quad_104 CH_1 TX和CH0 RX对应

.. image:: images/media/image152.png
   :width: 5.99931in
   :height: 3.93542in

4) 修改配置，码流选择PRBS 31，Loopback配置成None

   .. image:: images/media/image153.png
      :width: 5.99028in
      :height: 0.55903in

5) 配置完，可以点击BERT Reset,可以看到Errors都是0，重新开始测试。

   .. image:: images/media/image154.png
      :width: 5.99722in
      :height: 1.33472in

6) 选择一个链路，右键“Create Scan...”

.. image:: images/media/image155.png
   :width: 3.30208in
   :height: 1.98889in

.. image:: images/media/image156.png
   :width: 3.36944in
   :height: 3.56319in

7) 默认配置出来的眼图，注意：使用不同的软件版本，测量眼图可能会有差异。

.. image:: images/media/image157.png
   :width: 5.99792in
   :height: 3.05069in

体验ARM，裸机输出“Hello World”
==============================

**从本章开始由FPGA工程师与软件开发工程师协同实现。**

前面的实验都是在PL端进行的，可以看到和普通FPGA开发流程没有任何区别，ZYNQ的主要优势就是FPGA和ARM的合理结合，这对开发人员提出了更高的要求。从本章开始，我们开始使用ARM，也就是我们说的PS，本章我们使用一个简单的串口打印来体验一下Vivado
Vitis和PS端的特性。

前面的实验都是FPGA工程师应该做的事情，从本章节开始就有了分工，FPGA工程师负责把Vivado工程搭建好，提供好硬件给软件开发人员，软件开发人员便能在这个基础上开发应用程序。做好分工，也有利于项目的推进。如果是软件开发人员想把所有的事情都做了，可能需要花费很多时间和精力去学习FPGA的知识，由软件思维转成硬件思维是个比较痛苦的过程，如果纯粹的学习，又有时间，就另当别论了。专业的人做专业的事，是个很好的选择。

.. _硬件介绍-3:

硬件介绍
--------

我们从原理图中可以看到ZYNQ芯片分为PL和PS，PS端的IO分配相对是固定的，不能任意分配，而且不需要在Vivado软件里分配管脚，虽然本实验仅仅使用了PS，但是还要建立一个Vivado工程，用来配置PS管脚。虽然PS端的ARM是硬核，但是在ZYNQ当中也要将ARM硬核添加到工程当中才能使用。前面章节介绍的是代码形式的工程，本章开始介绍ZYNQ的图形化方式建立工程。

FPGA工程师工作内容
------------------

下面介绍FPGA工程师负责内容。

.. _vivado工程建立-2:

Vivado工程建立
--------------

1) 创建一个名为“ps_hello”的工程，建立过程不再赘述，参考“PL的”Hello
   World”LED实验”。

2) 点击“Create Block Design”，创建一个Block设计

.. image:: images/media/image54.png
   :width: 2.26458in
   :height: 2.29792in

3) “Design
   name”这里不做修改，保持默认“design_1”，这里可以根据需要修改，不过名字要尽量简短，否则在Windows下编译会有问题。

.. image:: images/media/image88.png
   :width: 3.01319in
   :height: 1.87153in

4) 点击“Add IP”快捷图标

.. image:: images/media/image56.png
   :width: 5.19167in
   :height: 2.67778in

5) 搜索“PS”，在搜索结果列表中双击”Control,Interfaces & Processing
   System”

.. image:: images/media/image57.png
   :width: 2.63333in
   :height: 2.09792in

6) 点击Run Block Automation

.. image:: images/media/image158.png
   :width: 5.25069in
   :height: 1.81389in

7)  配置如下，点击OK

    .. image:: images/media/image159.png
       :width: 4.79514in
       :height: 3.08958in

8)  自动连接如下

    .. image:: images/media/image160.png
       :width: 5.60139in
       :height: 2.27986in

9)  双击CIPS进行配置

    .. image:: images/media/image161.png
       :width: 4.58958in
       :height: 3.92361in

    .. image:: images/media/image162.png
       :width: 4.28125in
       :height: 3.73403in

    点击PSPMC进行配置

    .. image:: images/media/image163.png
       :width: 3.59444in
       :height: 0.93611in

10) 配置QSPI，EMMC，SD

    .. image:: images/media/image164.png
       :width: 5.21736in
       :height: 2.54306in

    .. image:: images/media/image165.png
       :width: 5.25in
       :height: 2.70556in

    .. image:: images/media/image166.png
       :width: 5.09861in
       :height: 2.69375in

    选择相应MIO

    .. image:: images/media/image167.png
       :width: 3.26667in
       :height: 2.32778in

11) 勾选USB 2.0，GEM0，UART0，TTC，GPIO等外设

    .. image:: images/media/image168.png
       :width: 5.39375in
       :height: 2.91806in

    配置外设

    .. image:: images/media/image169.png
       :width: 5.53472in
       :height: 3.48264in

12) 将MIO24配置成GPIO输入，对应PS端按键，MIO25配置成GPIO输出，对应PS端LED灯

    .. image:: images/media/image170.png
       :width: 4.39028in
       :height: 3.78889in

    .. image:: images/media/image171.png
       :width: 4.35347in
       :height: 3.87986in

13) 在clocking中，将参考时钟设置更精确些

    .. image:: images/media/image172.png
       :width: 4.75972in
       :height: 1.51597in

14) 将内部中断都勾选上，配置完成，点击OK

    .. image:: images/media/image173.png
       :width: 5.99236in
       :height: 2.18958in

15) 点击Finish

    .. image:: images/media/image174.png
       :width: 4.53958in
       :height: 3.93125in

16) 双击AXI NoC配置DDR4

    .. image:: images/media/image175.png
       :width: 1.77847in
       :height: 1.86667in

    .. image:: images/media/image176.png
       :width: 6.00208in
       :height: 3.89514in

    .. image:: images/media/image177.png
       :width: 6.00208in
       :height: 2.32847in

    选择参考时钟和system clock

    .. image:: images/media/image178.png
       :width: 5.21944in
       :height: 2.06736in

    DDR Address Region 1选择NONE，点击OK

    .. image:: images/media/image179.png
       :width: 5.99375in
       :height: 3.34444in

17) 修改引脚名称

    .. image:: images/media/image180.png
       :width: 5.99306in
       :height: 1.90556in

    双击配置sys_clk的频率为200MHz

    .. image:: images/media/image181.png
       :width: 3.59375in
       :height: 2.04861in

18) 选择Block设计，右键“Create HDL
    Wrapper...”,创建一个Verilog或VHDL文件，为block
    design生成HDL顶层文件。

.. image:: images/media/image182.png
   :width: 4.225in
   :height: 2.38819in

19) 保持默认选项，点击“OK”

.. image:: images/media/image183.png
   :width: 3.14452in
   :height: 1.81793in

20) 添加约束

    .. image:: images/media/image184.png
       :width: 5.64444in
       :height: 2.50208in

    .. image:: images/media/image185.png
       :width: 2.62708in
       :height: 2.05139in

    .. image:: images/media/image186.png
       :width: 5.22708in
       :height: 1.99375in

21) Generate Device Image

    .. image:: images/media/image187.png
       :width: 2.31944in
       :height: 0.92569in

22) 完成后取消

.. image:: images/media/image188.png
   :width: 2.59167in
   :height: 1.77153in

23) File->Export->Export Hardware...

.. image:: images/media/image189.png
   :width: 3.08958in
   :height: 2.575in

.. image:: images/media/image190.png
   :width: 3.82431in
   :height: 3.21875in

.. image:: images/media/image191.png
   :width: 4.03125in
   :height: 3.31806in

.. image:: images/media/image192.png
   :width: 4.10972in
   :height: 3.42708in

.. image:: images/media/image193.png
   :width: 4.21111in
   :height: 3.55833in

此时在工程目录下可以看到xsa文件，这个文件就包含了Vivado硬件设计的信息，可交由软件开发人员使用。

.. image:: images/media/image194.png
   :width: 2.01473in
   :height: 1.46875in

到此为止，FPGA工程师工作告一段落。

软件工程师工作内容
------------------

**Vitis工程目录为“ps_hello/vitis”**

以下为软件工程师负责内容。

Vitis调试
---------

创建Application工程
~~~~~~~~~~~~~~~~~~~

1) 新建一个文件夹，将vivado导出的xx.xsa文件拷贝进来。

2) Vitis是独立的软件，可以双击Vitis软件打开，也可以通过在Vivado软件中选择ToolsLaunch
   Vitis打开Vitis软件

.. image:: images/media/image9.png
   :width: 3.18611in
   :height: 2.00833in

在欢迎界面，点击Open Workspace，选择之前新建的文件夹，点击”OK”

.. image:: images/media/image195.png
   :width: 5.99931in
   :height: 2.57431in

3) 启动Vitis之后界面如下，点击“Create Platform
   Component”，这个选项会创建Platfrom工程，Platform工程类似于以前版本的hardware
   platform，包含了硬件支持的相关文件以及BSP。

.. image:: images/media/image196.png
   :width: 5.97778in
   :height: 2.38958in

4) 第一页填写Component name和路径，保持默认，点击Next

.. image:: images/media/image197.png
   :width: 5.98889in
   :height: 4.01319in

5) 选择(XSA，选择“Browse”，选择之前生成的xsa，点击打开，之后点击Next

.. image:: images/media/image198.png
   :width: 5.99306in
   :height: 3.99583in

6) 选择操作系统和处理器，这里保持默认

.. image:: images/media/image199.png
   :width: 5.99167in
   :height: 4.00556in

7)  点击Finish完成

    .. image:: images/media/image200.png
       :width: 5.99722in
       :height: 3.98403in

8)  生成之后出现窗口界面，以下是一些窗口介绍，与之前版本的Vitis界面有相似之处，但差别也比较大。

    .. image:: images/media/image201.png
       :width: 5.98611in
       :height: 3.26875in

9)  可以在Flow窗口编译平台

    .. image:: images/media/image202.png
       :width: 2.13472in
       :height: 0.70208in

    没有错误状态

    .. image:: images/media/image203.png
       :width: 2.13333in
       :height: 0.58333in

10) 点击左侧Example，这里面有很多官方的例程，与以前版本也比较类似，选择Hello
    World

    .. image:: images/media/image204.png
       :width: 1.89167in
       :height: 4.90069in

11) 点击创建工程

    .. image:: images/media/image205.png
       :width: 4.87361in
       :height: 2.50347in

12) 填写工程名称和路径，保持默认

    .. image:: images/media/image206.png
       :width: 4.04653in
       :height: 2.71181in

13) 选中平台

    .. image:: images/media/image207.png
       :width: 3.95486in
       :height: 2.64167in

14) 点击Next

    .. image:: images/media/image208.png
       :width: 3.99306in
       :height: 2.69167in

15) 完成

    .. image:: images/media/image209.png
       :width: 3.96111in
       :height: 2.65208in

16) 选中hello_world，点击Build

    .. image:: images/media/image210.png
       :width: 2.88194in
       :height: 3.22778in

.. _下载调试-2:

下载调试
~~~~~~~~

1) 连接JTAG线到开发板、UART的USB线到PC

   .. image:: images/media/image211.png
      :width: 4.27986in
      :height: 2.48125in

2) 在上电之前最好将开发板的启动模式设置到JTAG模式，拔到”ON”的位置

.. image:: images/media/image82.png
   :width: 4.09375in
   :height: 2.23403in

3) 开发板上电，并且打开串口调试工具，点击Flow中的Run

   .. image:: images/media/image212.png
      :width: 2.37153in
      :height: 1.08958in

4) 这个时候观察串口调试工具，即可以看到输出”Hello World”

.. image:: images/media/image213.png
   :width: 2.51458in
   :height: 2.28125in

固化程序
--------

普通的FPGA一般是可以从flash启动，或者被动加载，在第一章的PMC架构中已经介绍启动过程，这里不再介绍。

在Flow中选择Creat Boot
Image，弹出的窗口中可以看到生成的BIF文件路径，BIF文件是生成BOOT文件的配置文件，还有生成的Output
Image文件路径，也就是生成BOOT.pdi文件，它是我们需要的启动文件，可以放到SD卡启动，也可以烧写到QSPI
Flash。

.. image:: images/media/image214.png
   :width: 2.99306in
   :height: 1.34792in

.. image:: images/media/image215.png
   :width: 3.94653in
   :height: 4.93542in

在生成的目录下可以找到boot.pdi文件

.. image:: images/media/image216.png
   :width: 6.18611in
   :height: 0.72153in

SD卡启动测试
~~~~~~~~~~~~

1) 格式化SD卡，只能格式化为FAT32格式，其他格式无法启动

.. image:: images/media/image217.png
   :width: 1.62959in
   :height: 2.62898in

2) 放入boot.pdi文件，放在根目录

.. image:: images/media/image218.png
   :width: 2.32817in
   :height: 1.3048in

3) SD卡插入开发板的SD卡插槽

4) 启动模式调整为SD卡启动

.. image:: images/media/image219.png
   :width: 4.09653in
   :height: 2.91389in

5) 打开串口软件，上电启动，即可看到打印信息，红色框为FSBL启动信息，黄色箭头部分为执行的应用程序helloworld

.. image:: images/media/image220.png
   :width: 3.40694in
   :height: 2.99861in

QSPI启动测试
~~~~~~~~~~~~

1) 在Vitis菜单Vitis -> Program Flash

.. image:: images/media/image221.png
   :width: 2.77778in
   :height: 1.95347in

2) Image FIle文件选择要烧写的boot.pdi。选择Verify after flash，Flash
   Type选择qspi-x8-dual_parallel，在烧写完成后校验flash。

.. image:: images/media/image222.png
   :width: 4.70417in
   :height: 2.5in

3) 点击Program等待烧写完成

.. image:: images/media/image223.png
   :width: 3.61806in
   :height: 2.42986in

4) 设置启动模式为QSPI，再次启动，可以在串口软件里看到与SD同样的启动效果。

.. image:: images/media/image224.png
   :width: 3.06458in
   :height: 2.31667in

.. image:: images/media/image225.png
   :width: 3.58403in
   :height: 3.25347in

本章小结
--------

本章从FPGA工程师和软件工程师两者角度出发，介绍了Versal开发的经典流程，FPGA工程师的主要工作是搭建好硬件平台，提供硬件描述文件xsa给软件工程师，软件工程师在此基础上开发应用程序。本章是一个简单的例子介绍了FPGA和软件工程师协同工作，后续还会牵涉到PS与PL之间的联合调试，较为复杂，也是Versal开发的核心部分。

同时也介绍了FSBL，启动文件的制作，SD卡启动方式，QSPI下载及启动方式。

PS端以太网使用之lwip
====================

**vivado工程目录为“ps_hello/vivado”**

.. _软件工程师工作内容-1:

软件工程师工作内容
------------------

以下为软件工程师负责内容。

开发板有两路千兆以太网，通过RGMII接口连接，本实验演示如何使用Vitis自带的LWIP模板进行PS端千兆以太网TCP通信。

LWIP虽然是轻量级协议栈，但如果从来没有使用过，使用起来会有一定的困难，建议先熟悉LWIP的相关知识。

Vitis程序开发
-------------

LWIP库修改
~~~~~~~~~~

由于自带的LWIP库只能识别部分phy芯片，如果开发板所用的phy芯片不在默认支持范围内，要修改库文件。也可以直接使用修改过的库替换原有的库。

1) 找到库文件目录“x:\\Xilinx2023.2\\Vitis\\2023.2\\data\\embeddedsw\\ThirdParty\\sw_services”

.. image:: images/media/image226.png
   :width: 1.42708in
   :height: 2.45903in

2) 找到要修改的文件目录“lwip213_v1_1\\src\\contrib\\ports\\xilinx\\netif”中文件“xaxiemacif_physpeed.c”和“xemacpsif_physpeed.c”要修改。

.. image:: images/media/image227.png
   :width: 4.20694in
   :height: 2.40833in

主要添加了get_phy_speed_ksz9031，get_phy_speed_JL2121，以支持ksz9031和JL2121自协商获取速度。在资料中提供了修改好的lwip库，可直接替换。

.. image:: images/media/image228.png
   :width: 1.24028in
   :height: 0.19097in

创建APP工程时基于LWIP模板
~~~~~~~~~~~~~~~~~~~~~~~~~

1. BSP中添加lwip213库

   .. image:: images/media/image229.png
      :width: 5.22569in
      :height: 3.67986in

2. 配置dhcp功能为True

   .. image:: images/media/image230.png
      :width: 4.66528in
      :height: 3.54236in

   Build platform

   .. image:: images/media/image231.png
      :width: 3.29861in
      :height: 0.97153in

3. 选择lwIP Echo Server模板

   .. image:: images/media/image232.png
      :width: 4.29028in
      :height: 3.56597in

4. 生成模板

   .. image:: images/media/image233.png
      :width: 4.99444in
      :height: 2.95764in

   过程不再赘述，可参考体验ARM，裸机输出”Hello World“一章之6.3.1

5. Build

   .. image:: images/media/image234.png
      :width: 3.12569in
      :height: 1.42014in

.. _下载调试-3:

下载调试
--------

测试环境要求有一台支持dhcp的路由器，开发板连接路由器可以自动获取IP地址，实验主机和开发板在一个网络，可以相互通信。

以太网测试
~~~~~~~~~~

1) 连接串口打开串口调试终端，连接好PS端以太网网线到路由器，运行Vitis下载程序

.. image:: images/media/image235.png
   :width: 3.66319in
   :height: 2.08403in

.. image:: images/media/image236.png
   :width: 3.39028in
   :height: 1.48194in

2) 可以看到串口打印出一些信息，可以看到自动获取到地址为“192.168.1.63”，连接速度1000Mbps，tcp端口为7

.. image:: images/media/image237.png
   :width: 4.6125in
   :height: 3.15556in

3) 使用telnet连接

.. image:: images/media/image238.png
   :width: 2.92292in
   :height: 2.83194in

4) 当输入一个字符时，开发板返回相同字符

.. image:: images/media/image239.png
   :width: 3.92222in
   :height: 2.45764in

.. _实验总结-2:

实验总结
--------

通过实验我们更加深刻了解到Vitis程序的开发，本实验只是简单的讲解如何创建一个LWIP应用，LWIP可以完成UDP、TCP等协议，在后续的教程中我们会提供基于以太网的具体应用，例如摄像头数据通过以太网发送上位机显示。

整体工程及实验
==============

本章将板卡大部分外设集成到Vivado工程。

.. _vivado工程建立-3:

Vivado工程建立
--------------

整体框图如下，两路MIPI摄像头通过VDMA写入DDR4，LVDS
LCD通过VDMA从DDR4读出图像数据。具体搭建过程不再描述，可通过TCL脚本恢复Vivado工程。

.. image:: images/media/image240.png
   :width: 4.925in
   :height: 3.88403in

Vitis实验
---------

基于VDMA的LVDS LCD显示实验
~~~~~~~~~~~~~~~~~~~~~~~~~~

本实验主要功能是ARM在DDR中制作color bar，VDMA读取这块空间，发送给LVDS
LCD显示模块。下载程序：

.. image:: images/media/image241.png
   :width: 3.23472in
   :height: 0.99583in

.. image:: images/media/image242.png
   :width: 2.97986in
   :height: 1.30486in

显示结果如下：

.. image:: images/media/image132.png
   :width: 5.35347in
   :height: 3.80694in

MIPI摄像头采集显示实验
~~~~~~~~~~~~~~~~~~~~~~

本实验主要功能是配置单个或两个MIPI摄像头，将图像显示到LCD上面，也是通过VDMA实现。

.. image:: images/media/image243.png
   :width: 3.17083in
   :height: 2.15069in

Run下载程序

.. image:: images/media/image244.png
   :width: 3.38125in
   :height: 1.65833in

若是想显示单个或两个摄像头，可以修改config.h里的宏定义，重新编译下载即可

.. image:: images/media/image245.png
   :width: 5.12986in
   :height: 2.01389in

显示效果

.. image:: images/media/image246.png
   :width: 4.16944in
   :height: 3.74514in

MIPI摄像头双目采集以太网传输实验
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

前面介绍了MIPI摄像头的LCD显示，但有些场合下，需要把视频传输到上位机，便可以利用以太网进行数据的传输，本章利用LWIP的udp将摄像头数据传输到上位机。

下面介绍LWIP部分内容，与上位机通信时，采用UDP传输，在UDP数据包中自定义了协议，如下所示：

一、获取板卡信息

（1）询问命令（共5字节，由上位机通过以太网发送）

+--------------+--------------+---------------------------------------+
| 字节数       | 1            | 4                                     |
+--------------+--------------+---------------------------------------+
| 命令信息     | Header       | 0x00020001                            |
+--------------+--------------+---------------------------------------+

（2）应答命令（共16字节，由开发板通过以太网发送）

+----------+-----------------------------------------------------------+
| 字节数   | 命令信息                                                  |
+----------+-----------------------------------------------------------+
| 1        | Header|0x01                                               |
+----------+-----------------------------------------------------------+
| 4        | 0x00020001                                                |
+----------+-----------------------------------------------------------+
| 6        | 板卡MAC地址                                               |
+----------+-----------------------------------------------------------+
| 4        | 板卡IP地址                                                |
+----------+-----------------------------------------------------------+
| 1        | 0x02                                                      |
+----------+-----------------------------------------------------------+

二、获取数据

（1）控制命令（由上位机发送数据请求）

+----------+-----------------------------------------------------------+
| 字节数   | 命令信息                                                  |
+----------+-----------------------------------------------------------+
| 1        | Header                                                    |
+----------+-----------------------------------------------------------+
| 4        | 0x00020002                                                |
+----------+-----------------------------------------------------------+
| 6        | 板卡MAC地址                                               |
+----------+-----------------------------------------------------------+
| 1        | 摄像头通道选择，数值1代表仅打开摄像                       |
|          | 头1，数值2代表仅打开摄像头2，数值3代表同时打开两个摄像头  |
+----------+-----------------------------------------------------------+
| 1        | 启动信号，0表示关闭上位图像显示，其他表示打开图像显示     |
+----------+-----------------------------------------------------------+

（2）应答命令（由开发板发送）

+----------+-----------------------------------------------------------+
| 字节数   | 命令信息                                                  |
+----------+-----------------------------------------------------------+
| 1        | Header|0x 01                                              |
+----------+-----------------------------------------------------------+
| 3        | 0x 000200                                                 |
+----------+-----------------------------------------------------------+
| 1        | 通道标识，数值2代表摄像头1，数值3代表摄像头2              |
+----------+-----------------------------------------------------------+
| 3        | 序列号，以太网包序号，用于上位机识别                      |
+----------+-----------------------------------------------------------+
| N        | 图像数据                                                  |
+----------+-----------------------------------------------------------+

每个UDP包都包含有Header，在第一个字节，其格式如下：

+-----------------------+----------------------+----------------------+
| 比特位                | 值（0）              | 值（1）              |
+-----------------------+----------------------+----------------------+
| bit 0                 | 查询或控制           | 应答                 |
+-----------------------+----------------------+----------------------+
| bit1~bit7             | 随机数据             |                      |
+-----------------------+----------------------+----------------------+

注：当应答时，高7位随机数据保持不变，bit0设置为1

工作流程为：

1) 上位机发送询问命令

2) 开发板应答询问

3) 上位机发送控制命令请求数据

4) 开发板发送数据

5) 步骤3和4循环

实验步骤
^^^^^^^^

1. 如果在vitis中勾选lwip库

.. image:: images/media/image247.png
   :width: 5.70833in
   :height: 3.84861in

并且做参数配置

.. image:: images/media/image248.png
   :width: 5.32153in
   :height: 2.70347in

.. image:: images/media/image249.png
   :width: 4.04792in
   :height: 2.69861in

.. image:: images/media/image250.png
   :width: 3.94028in
   :height: 2.18611in

重新编译platform

.. image:: images/media/image251.png
   :width: 3.89931in
   :height: 1.09861in

2. Build工程，连接好板子摄像头，电源，串口，PS端网口ETH1，然后点击Run下载程序

   .. image:: images/media/image252.png
      :width: 5.53819in
      :height: 3.85764in

   .. image:: images/media/image253.png
      :width: 4.06181in
      :height: 1.75486in

3. 如果有DHCP服务器，会自动分配IP给开发板；如果没有DHCP服务器，默认开发板IP地址为192.168.1.10，需要将PC的IP地址设为同一网段，如下图所示。同时要确保网络里没有192.168.1.10的IP地址，否则会造成IP冲突，导致无法显示图像。可以在板子未上电前在CMD里输入ping
   192.168.1.10查看是否能ping通，如果ping通，说明网络中有此IP地址，就无法验证。

..

   没有问题之后打开串口软件。

.. image:: images/media/image254.png
   :width: 3.16215in
   :height: 3.95585in

4. 串口打印信息如下，检测出网卡速度，设置的IP地址

.. image:: images/media/image255.png
   :width: 5.41042in
   :height: 4.34167in

5. 打开Vivado工程文件夹，打开videoshow.exe

.. image:: images/media/image256.png
   :width: 1.08889in
   :height: 0.16181in

软件扫描到两个摄像头，可通过勾选来选择相应的摄像头显示，点击播放

.. image:: images/media/image257.png
   :width: 4.5375in
   :height: 3.54931in

显示效果如下，如果想重新选择显示通路，在软件屏幕上双击，回到选择界面，再次选择要显示的图像。

.. image:: images/media/image258.png
   :width: 5.98889in
   :height: 2.35486in

6. 打开任务管理器，可以看到网络带宽为750Mbps左右

.. image:: images/media/image259.png
   :width: 4.40208in
   :height: 3.85833in
