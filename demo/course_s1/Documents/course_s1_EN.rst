illustrate
===========

First of all, thank you for purchasing the Versal development board produced by Xinyi Electronic Technology (Shanghai) Co., Ltd.!
Your support and trust in us and our products gives us the confidence and courage to move forward.

This tutorial is part of the FPGA and bare metal tutorial. Through continuous practice, you can master the basic process of FPGA and bare metal development. Although it does not explain many major principles, practice makes perfect. Practice more and gradually master the secrets.


Preparation work and precautions
==================================

Software Environment
------------------------

The software development environment is based on Vivado 2023.2

Hardware environment
-----------------------

+----------------------------------+--------------------------------------------+
| Development board model          | Chip model                                 |
+----------------------------------+--------------------------------------------+
| VD100                            | xcve2302-sfva784-1LP-es                    |
+----------------------------------+--------------------------------------------+

Script to create Vivado project
==================================

There is a script to generate vivado under each project, which is used to rebuild the vivado project. There are two methods to use. One is to use the batch file and right-click to edit create_project.bat

.. image:: images/media/image4.png
  :width: 1.16181in
  :height: 0.24653in

.. image:: images/media/image5.png
  :width: 2.26528in
  :height: 0.91042in

Change the path to the vivado installation path on your computer, save it, and then double-click to generate the vivado project.

.. image:: images/media/image6.png
  :width: 5.09931in
  :height: 0.28889in

The second method is to open the vivado software, first use the cd command to enter the auto_create_project directory, and then run source
./create_project.tcl command.

Script to create Vitis project
================================

Since the Vitis project takes up a lot of space after compilation, in order to save everyone's precious time, we provide the python script of the Vitis project. There is a vitis folder under each project, which contains the hardware description file xx.xsa, and the automatic Script to create project

.. image:: images/media/image7.png
  :width: 2.77083in
  :height: 0.77292in

What you need to do is to right-click on the builder.py file and change the vitis path to the installation path of your computer.

.. image:: images/media/image8.png
  :width: 3.76042in
  :height: 0.47014in

After saving, open the vitis software

.. image:: images/media/image9.png
  :width: 3.18611in
  :height: 2.00833in

Open a new terminal

.. image:: images/media/image10.png
  :width: 3.92222in
  :height: 0.67569in

Use the cd command to enter the vitis project path, and enter vitis -i to enter the vitis CLI.

.. image:: images/media/image11.png
  :width: 4.89236in
  :height: 1.2125in

Type run -t builder.py and press Enter

.. image:: images/media/image12.png
  :width: 4.07917in
  :height: 0.43889in

Creation completed

.. image:: images/media/image13.png
  :width: 4.22778in
  :height: 1.40972in

Click Open Workspace to switch to the vitis working directory

.. image:: images/media/image14.png
  :width: 4.29444in
  :height: 2.11319in

You can see the created project

.. image:: images/media/image15.png
  :width: 3.68264in
  :height: 1.84306in

At this time, the APP project and platform may not be related well, and they need to be related manually. You can compile the platform first.

.. image:: images/media/image16.png
  :width: 3.67222in
  :height: 0.95764in

Select component, click settings, click switch platform

.. image:: images/media/image17.png
  :width: 5.22153in
  :height: 4.05833in

.. image:: images/media/image18.png
  :width: 5.09306in
  :height: 1.38611in

Build the project again and you can use it

.. image:: images/media/image19.png
  :width: 4.05625in
  :height: 1.15278in

Chapter 1 Introduction to Versal
=================================

Versal includes Cortex-A72 processor and Cortex-R5 processor, PL programmable logic part, PMC platform management controller, AI
Engine and other modules are different from the previous ZYNQ
Unlike MPSoC 7000, Versal is interconnected internally through the NoC on-chip network.

.. image:: images/media/image20.png
  :width: 5.83819in
  :height: 5.02917in

Overall block diagram of the Versal chip

PS: Processing System is the part of ARM SoC that has nothing to do with FPGA.

PL: Programmable Logic, which is the FPGA part.

NoC architecture
-----------------

Versal programmable network-on-chip (NoC) is an AXI interconnect network used to share data between programmable logic PL, processor system PS, etc. The previous Versal series used the AXI cross-interconnect module, which is Versal's the difference.

NoC is designed for scalability. It consists of a series of interconnected horizontal (HNoC) and vertical (VNoC) paths supported by a set of customizable hardware implementation components that can be configured in different ways to meet design timing, speed and logic utilization requirements . The following is the structure diagram of the NoC

.. image:: images/media/image21.png
  :width: 5.84931in
  :height: 3.97153in

From the structure diagram of NoC, we can see that it mainly consists of NMU (NoC master units), NSU (NoC slave
units), NPI (NoC programming interface), NPS (NoC packet
switch). The PS side can connect to NMU and then access DDRMC through NPS connection. Similarly, the PL side can also access DDRMC through NMU and NPS. Access each module flexibly through NPS routing.

.. image:: images/media/image22.png
  :width: 5.71319in
  :height: 3.05764in

NMU structure

.. image:: images/media/image23.png
  :width: 6.00069in
  :height: 2.40208in

NSU structure

From the above NMU,
It can be seen from the NSU structure that the interface to the user is still the AXI bus. Inside it, AXI data is packaged or unpacked and connected to the NoC network.

.. image:: images/media/image24.png
  :width: 2.71458in
  :height: 2.71944in

NPS structure

Both NMU and NSU are connected to the NPS, which is equivalent to a router and forwards data to the destination device. It's a full 4x4
switch, each port supports 8 virtual channels in each direction, using credit-based flow control, similar to TCP's sliding window.

NoC is a very important component in the development of Versal. The PS side accesses DDR and the PL side accesses DDR through NoC. Different from Versal, versal does not have a DDR controller on the PS side and all accesses through NoC. Therefore, understanding the NoC structure is It is necessary. For more information, please refer to the official pg313 document.

PMC architecture
------------------

PMC (Platform Management Controller) manages the platform during startup, configuration, and operation. As can be seen from the structure diagram below, PMC consists of ROM
Code Unit, Platform Processing Unit, PMC I/O
It is composed of Peripherals and other units and has rich functions. Here we mainly introduce how PMC bootstraps startup.

.. image:: images/media/image25.png
  :width: 5.84861in
  :height: 6.04444in

PMC structure diagram

.. image:: images/media/image26.png
  :width: 4.77431in
  :height: 3.00417in

The first stage: Pre-Boot

1. PMC detects PMC power supply and POR_B release

2. PMC reads the boot mode pin and stores it in the boot mode register

3. PMC sends reset to RCU (ROM code unit)

.. image:: images/media/image27.png
  :width: 4.62431in
  :height: 3.62917in

The second stage: Boot Setup

4. RCU executes BootROM from RCU ROM

5. BootROM reads the boot mode register and selects the boot device

6. BootROM reads PDI (programmable device image) from the boot device and verifies it

7. BootROM releases the PPU reset, loads the PLM into the PPU RAM and verifies it. After verification, PPU wakes up and PLM
The software starts executing.

8. BootROM enters sleep state

.. image:: images/media/image28.png
  :width: 4.77431in
  :height: 3.27153in

The third stage: Load Platform

9. PPU starts executing PLM from PPU RAM

10. PLM starts to read and run the PDI module

11. PLM uses PDI content to configure other parts of Versal

11a: PLM configures data for the following modules: PMC, PS clocks

(MIO, clocks, resets, etc.) (CDO file)

NoC initialization and NPI module (DDR controller, NoC,

GT, XPIPE, I/Os, clocking and other NPI modules

PLM loads the application ELF of APU and RPU into storage space,

Such as DDR, OCM, TCM, etc.

11b: PL side logic configuration

PL side data (CFI file)

AI Engine configuration (AI Engine CDO)

.. image:: images/media/image28.png
  :width: 4.77431in
  :height: 3.27153in

The fourth stage: Post-Boot

12. PLM continues to operate until the next POR or system reset. And responsible for DFX reconfiguration, power management, subsystemsRestart, error management, security services.

An introduction to the Versal chip development process
--------------------------------------------------------

Since Versal integrates the CPU and FPGA, developers need to design not only ARM operating system applications and device drivers, but also the hardware logic design of the FPGA part. During development, you need to understand the Linux operating system and system architecture, and you also need to build a hardware design platform between FPGA and ARM systems. Therefore, the development of Versal requires collaborative design and development by software personnel and hardware personnel. This is the so-called "software and hardware co-design" in Versal development.

The design and development of the hardware system and software system of the Versal system requires the following development environment and debugging tools:
Xilinx
Vivado. The Vivado design suite implements the design and development of the FPGA part, pin and timing constraints, compilation and simulation, and implements the RTL to bitstream design process.

Xilinx
Vitis is the Xilinx software development kit (SDK). Based on the Vivado hardware system, the system will automatically configure some important parameters, including tool and library paths, compiler options, JTAG and flash memory settings, debugger connection and bare metal board support package (BSP). SDK is also available for all supported Xilinx
The IP hard core provides drivers. Vitis supports collaborative debugging of IP hard core (on FPGA) and processor software. We can use high-level C or C++ language to develop and debug ARM and FPGA systems to test whether the hardware system is working properly. Vitis software also comes with Vivado software and does not need to be installed separately.

The development of Versal is also a hardware-first-software approach. The specific process is as follows:

1) Create a new project on Vivado and add an embedded source file.

2) Add and configure some basic peripherals of PS and PL in Vivado, or need to add customized peripherals.

3) Generate the top-level HDL file in Vivado and add the constraint file. Then compile and generate bitstream file ( XX.pdi).

4) Export the hardware information to the Vitis software development environment. In the Vitis environment, you can write some debugging software to verify the hardware and software, and combine the bitstream files to debug the Versal system alone.

5) Generate u-boot.elf and bootloader images in the VMware virtual machine.

6) Generate a BOOT.pdi file from the bitstream file and u-boot.elf file in Vitis.

7) Generate Ubuntu kernel image file Zimage and Ubuntu root file system in VMware. In addition, you need to write a driver for the FPGA's custom IP.

8) Put the BOOT, kernel, device tree, and root file system files into the SD card, turn on the power of the development board, and the Linux operating system will boot from the SD card.

What skills are required to learn Versal?
-------------------------------------------

Learning Versal is more demanding than learning traditional tool development such as FPGA, MCU, ARM, etc. Learning Versal well is not something that can be accomplished overnight.

software developer
~~~~~~~~~~~~~~~~~~~~~~~~

- Principles of computer composition

- C, C++ language

- Computer operating system

- tcl script

- Good foundation in English reading

logic developer
~~~~~~~~~~~~~~~~~~

- Principles of computer composition

- C language

- Basics of digital circuits

Chapter 2 PL's "Hello World" LED experiment
============================================

**Experimental Vivado project for "led".**

For Versal, PL (FPGA) development is crucial. This is where Versal has an advantage over other ARMs. It can customize many ARM-side peripherals. Before customizing ARM-side peripherals, let us first go through an LED example. Cheng Lai is familiar with the development process of PL (FPGA) and the basic operation of Vivado software. This development process is exactly the same as that of FPGA chips without ARM.

In this routine, what we are going to do is an LED light control experiment. We control the LED light on the development board to flip once every second to achieve on, off, on, and off control.

LED hardware introduction
-------------------------------

The PL part of the development board is connected to a red user LED light. This 1 light is completely controlled by PL. If PL_LED1 is high level, the three-stage tube is turned on, and the light will be on, otherwise it will be off.

.. image:: images/media/image29.png
  :width: 3.22222in
  :height: 2.47569in

Create a Vivado project
-------------------------

1) Start Vivado. In Windows, you can start it by double-clicking the Vivado shortcut.

2) Click "Create New Project" in the Vivado development environment to create a new project.

.. image:: images/media/image30.png
  :width: 4.90245in
  :height: 3.54576in

3) A wizard for creating a new project will pop up, click "Next"

.. image:: images/media/image31.png
  :width: 4.82126in
  :height: 4.08408in

4) In the pop-up dialog box, enter the project name and the directory where the project is stored. Here we take an LED project name. Need to pay attention to the project path "Project
location" cannot have Chinese spaces, and the path name cannot be too long.

.. image:: images/media/image32.png
  :width: 4.85347in
  :height: 3.06944in

5) Select "RTL Project" in the project type

.. image:: images/media/image33.png
  :width: 5.26181in
  :height: 3.32917in

6) Target language “Target
language" select "Verilog". Although Verilog is selected, VHDL can also be used to support multi-language mixed programming.

.. image:: images/media/image34.png
  :width: 5.20556in
  :height: 3.27708in

7) Click "Next" without adding any files

.. image:: images/media/image35.png
  :width: 5.39514in
  :height: 3.34097in

8) Select "xc2302-sfva784-1LP-eS"

.. image:: images/media/image36.png
  :width: 5.13403in
  :height: 4.59444in

9) Click "Finish" to complete the creation of the future project named "led".

.. image:: images/media/image37.png
  :width: 5.40347in
  :height: 3.40417in

10) Vivado software interface

.. image:: images/media/image38.png
  :width: 4.61346in
  :height: 3.97672in

Create Verilog HDL file to light up LED
------------------------------------------

1) Click the Add Sources icon under Project Manager (or use the shortcut Alt+A)

.. image:: images/media/image39.png
  :width: 3.88736in
  :height: 2.26719in

2) Select "Add or create design sources" and click "Next"

.. image:: images/media/image40.png
  :width: 5.11453in
  :height: 3.45338in

3) Select “Create File”

.. image:: images/media/image41.png
  :width: 5.19748in
  :height: 3.5094in

4) Set the file name "File name" to "led" and click "OK"

.. image:: images/media/image42.png
  :width: 4.86244in
  :height: 3.28317in

5) Click "Finish" to complete adding the "led.v" file

.. image:: images/media/image43.png
  :width: 4.89769in
  :height: 3.30698in

6) In the pop-up module definition "Define
Module", you can specify the module name "Module" of the "led.v" file
name", the default here will not be "led", you can also specify some ports, but do not specify them here for the time being, click "OK".

.. image:: images/media/image44.png
  :width: 4.48908in
  :height: 3.21372in

7) Select "Yes" in the pop-up dialog box

.. image:: images/media/image45.png
  :width: 4.33533in
  :height: 3.10366in

8) Double-click "led.v" to open the file and then edit

.. image:: images/media/image46.png
  :width: 4.52898in
  :height: 3.45462in

9) Write "led.v", which defines a 32-bit register timer.
Used to count 0~199999999 (1 second) in a loop. When counting to 199999999 (1 second),
The register timer becomes 0 and the four LEDs are toggled. In this way, if the original LED is off, it will light up; if the original LED is on, it will go out. Since the input clock is a 200MHz differential clock, the IBUFDS primitive needs to be added to connect the differential signal. The code after writing is as follows:

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

10) Save the code after writing it

Add pin constraints
------------------------

The constraint file format used by Vivado is xdc file. The xdc file mainly completes the pin constraints and clock constraints.
and group constraints. Here we need to assign the input and output ports in the led.v program to the real pins of the FPGA.

1) Create a new constraint file

.. image:: images/media/image47.png
  :width: 5.99722in
  :height: 2.96736in

2)Create File

.. image:: images/media/image48.png
  :width: 4.95556in
  :height: 3.31319in

.. image:: images/media/image49.png
  :width: 2.33472in
  :height: 1.8in

3) Bind the reset signal rst_n to the button on the PL side, assign pins and level standards to the LED and clock, and the constraints are as follows

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

Generate pdi file
--------------------

1) The compilation process can be subdivided into synthesis, placement and routing, bit file generation, etc. Here we directly click "Generate
Device Image", directly generate pdi files.

.. image:: images/media/image51.png
  :width: 1.8375in
  :height: 0.75069in

2) In the pop-up dialog box, you can select the number of tasks, which is related to the number of CPU cores. Generally, the larger the number, the faster the compilation. Click "OK"

.. image:: images/media/image52.png
  :width: 2.2739in
  :height: 1.78158in

3) An error was reported during compilation.

.. image:: images/media/image53.png
  :width: 5.98611in
  :height: 0.78264in

[DRC CIPS-2] Versal CIPS exists check - wdi: Versal designs must
contain a CIPS IP in the netlist hierarchy to function properly.
Please create an instance of the CIPS IP and configure it. Without a
CIPS IP in the design, Vivado will not generate a CDO for the PMC,
an elf for the PLM.

Judging from the error report, the versa design must include CIPS, that is, the PS side, so the CIPS core needs to be added.

4) Select Create Block Design

.. image:: images/media/image54.png
  :width: 2.26458in
  :height: 2.29792in

.. image:: images/media/image55.png
  :width: 3.19792in
  :height: 1.73125in

5) Add CIPS

.. image:: images/media/image56.png
  :width: 5.19167in
  :height: 2.67778in

.. image:: images/media/image57.png
  :width: 2.63333in
  :height: 2.09792in

6) Double-click CIPS, select PL_Subsystem, only the logic on the PL side

.. image:: images/media/image58.png
  :width: 4.18542in
  :height: 3.7875in

7) Right-click Generate Output products

.. image:: images/media/image59.png
  :width: 2.89653in
  :height: 1.85833in

.. image:: images/media/image60.png
  :width: 2.08403in
  :height: 2.85278in

8) Then right-click to create HDL

.. image:: images/media/image61.png
  :width: 3.44167in
  :height: 1.77569in

.. image:: images/media/image62.png
  :width: 3.06875in
  :height: 1.50694in

9) Instantiate the PS side file in led.v

.. image:: images/media/image63.png
  :width: 1.49444in
  :height: 0.55972in

.. image:: images/media/image64.png
  :width: 3.28958in
  :height: 1.52986in

10) Then Generate
Bitstream, there are no errors in the compilation, the compilation is completed, a dialog box pops up allowing us to choose subsequent operations, you can select "Open
Hardware Manger", of course, you can also choose "Cancel", we choose here
"Cancel", don't download it yet.

.. image:: images/media/image65.png
  :width: 2.51597in
  :height: 1.51181in

Vivado simulation
-------------------

Next, we might as well try our best and use Vivado's own simulation tool to output waveforms to verify whether the flow lamp program design results are consistent with our expectations (note: you can also simulate before generating the bit file). Specific steps are as follows:

1. Set the simulation configuration of Vivado, right-click Simulation Settings in SIMULATION.

.. image:: images/media/image66.png
  :width: 2.71162in
  :height: 2.82275in

2. In Simulation
In the Settings window, configure as shown below. Here, set it to 50ms (set it as needed). For other settings, follow the default settings. Click OK to complete.

.. image:: images/media/image67.png
  :width: 4.16967in
  :height: 3.68114in

3. Add the incentive test file and click Add under Project Manager Sources icon, click Next after setting as shown below.

.. image:: images/media/image68.png
  :width: 4.24388in
  :height: 2.19655in

4. Click Create File to generate the simulation stimulus file.

.. image:: images/media/image69.png
  :width: 3.47146in
  :height: 2.72528in

Enter the name of the stimulus file in the pop-up dialog box. Here we enter the name vtf_led_test.

.. image:: images/media/image70.png
  :width: 2.21088in
  :height: 1.80096in

5. Click the Finish button to return.

.. image:: images/media/image71.png
  :width: 3.95375in
  :height: 3.03139in

We will not add IO Ports here, click OK.

.. image:: images/media/image72.png
  :width: 3.1395in
  :height: 2.2426in

In Simulation
There is an additional vtf_led_test file just added in the Sources directory. Double-click to open this file, and you can see that there is only the definition of the module name and nothing else.

.. image:: images/media/image73.png
  :width: 4.14019in
  :height: 2.71368in

6. Next we need to write the content of this vtf_led_test.v file. First define the input and output signals, and then instantiate the led_test module to make the led_test program part of this test program. Then add reset and clock excitation. The completed vtf_led_test.v file is as follows:

+-----------------------------------------------------------------------+
| \`timescale 1ns **/** 1ps                                             |
|                                                                       |
| // Module Name: vtf_led_test                                          |
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

1) After writing, save, vtf_led_test.v automatically becomes the top level of this simulation Hierarchy, and below it is the design file led_test.v.

.. image:: images/media/image74.png
  :width: 2.63408in
  :height: 2.45107in

8) Click the Run Simulation button and select Run Behavioral Simulation. Here we can just do behavioral level simulation.

.. image:: images/media/image75.png
  :width: 2.88031in
  :height: 3.23482in

If there are no errors, the simulation software in Vivado starts working.

10. After the simulation interface pops up, as shown below, the interface is the waveform of 50ms when the simulation software automatically runs to the simulation setting.

.. image:: images/media/image76.png
  :width: 6.00417in
  :height: 1.23611in

Since the state change time of LED[3:0] designed in the program is long, and the simulation is relatively time-consuming, we observe the changes of the timer[31:0] counter here. Put it into Wave and observe it (click uut under the Scope interface,Then right-click and select timer under the Objects interface, and select Add Wave in the pop-up drop-down menu.Window).

.. image:: images/media/image77.png
  :width: 3.82425in
  :height: 2.22484in

After adding, the timer is displayed on the Wave interface, as shown in the figure below.

.. image:: images/media/image78.png
  :width: 4.75283in
  :height: 1.31547in

11. Click the Restart button marked below to reset, and then click RunAll button. (Patience is required!!!), you can see that the simulation waveform is consistent with the design. (Note: The longer the simulation time, the greater the disk space occupied by the simulated waveform file. The waveform file is in the xx.sim folder of the project directory)

.. image:: images/media/image79.png
  :width: 4.16502in
  :height: 1.82527in

.. image:: images/media/image80.png
  :width: 6.00417in
  :height: 1.37986in

We can see that the LED signal will change to 1, indicating that the LED light will brighten.

download
----------

1) Connect the JTAG interface of the development board and power on the development board. Note that the pull-out switch must select JTAG mode, that is, pull all the switches to "ON". The value represented by "ON" is 0. If JTAG mode is not used, an error will be reported when downloading. .

.. image:: images/media/image81.png
  :width: 5.50347in
  :height: 3.82569in

.. image:: images/media/image82.png
  :width: 4.09375in
  :height: 2.23403in

2) Click "Auto Connect" on the "HARDWARE MANAGER" interface to automatically connect to the device

.. image:: images/media/image83.png
  :width: 3.01461in
  :height: 2.12162in

3) Select the chip, right-click "Program Device..."

.. image:: images/media/image84.png
  :width: 3.34583in
  :height: 2.10347in

4) Click "Program" in the pop-up window

.. image:: images/media/image85.png
  :width: 3.53194in
  :height: 1.88056in

5) Wait for download

.. image:: images/media/image86.png
  :width: 3.18855in
  :height: 0.87404in

6) After the download is completed, we can see the PL
The LED starts changing every second. At this point, the Vivado simple process experience is completed. Later chapters will introduce that if you burn the program to Flash, you need the cooperation of the PS system to complete it. Only PL projects cannot directly burn Flash. Hello in "Experience ARM, Bare Metal Output"
It is introduced in the FAQ in the chapter "World".

Experiment summary
--------------------

This chapter introduces how to develop programs on the PL side, including project establishment, constraints, simulation and other methods. You can refer to this method in subsequent code development methods.

Chapter 3 PL reads and writes DDR4 experiment through NoC
==========================================================

**The experimental VIvado project is "pl_rw_ddr".**

Hardware introduction
-----------------------

The PL side of the development board has 4 16bit ddr4

.. image:: images/media/image87.png
  :width: 4.39028in
  :height: 2.6in

Vivado project set up
-----------------------

Versal's DDR4 is accessed through NoC, so NoC IP needs to be added for configuration.

Create a Block design and configure the NoC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Select Create Block Design

.. image:: images/media/image54.png
  :width: 2.26458in
  :height: 2.29792in

.. image:: images/media/image88.png
  :width: 3.01319in
  :height: 1.87153in

2) Add CIPS

.. image:: images/media/image56.png
  :width: 5.19167in
  :height: 2.67778in

.. image:: images/media/image57.png
  :width: 2.63333in
  :height: 2.09792in

3) Double-click CIPS, select PL_Subsystem, only the logic on the PL side

.. image:: images/media/image58.png
  :width: 4.18542in
  :height: 3.7875in

4) Add NoC IP

.. image:: images/media/image89.png
  :width: 2.42222in
  :height: 2.80486in

5) Configure NoC

Select an AXI Slave and AXI Clock, select "Single Memory Controller"

.. image:: images/media/image90.png
  :width: 5.60972in
  :height: 3.17778in

Select Inputs as PL

.. image:: images/media/image91.png
  :width: 6in
  :height: 1.225in

connection port

.. image:: images/media/image92.png
  :width: 6.01389in
  :height: 1.39028in

DDR4 configuration

.. image:: images/media/image93.png
  :width: 5.39792in
  :height: 3.20069in

.. image:: images/media/image94.png
  :width: 5.99583in
  :height: 2.42569in

Configuration is complete, click OK

6) Configure CIPS and add reset

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

Click Finish

7) Add Clocking Wizard and configure the output clock to 150MHz as the PL side read and write clock

.. image:: images/media/image99.png
  :width: 1.37014in
  :height: 0.62917in

.. image:: images/media/image100.png
  :width: 5.625in
  :height: 1.73681in

8) Add IBUFDS for NoC and Clocking
Wizard provides a reference clock and exports S00_AXI, CH0_DDR4_0 and other buses, and adds axi_clk and axi_resetn to provide clock and reset for the PL side.

.. image:: images/media/image101.png
  :width: 5.99167in
  :height: 2.18958in

Double-click the reference clock pin and configure the frequency to 200MHz

.. image:: images/media/image102.png
  :width: 2.75208in
  :height: 1.58056in

Double-click the AXI bus and configure

.. image:: images/media/image103.png
  :width: 4.45972in
  :height: 3.44375in

.. image:: images/media/image104.png
  :width: 4.12431in
  :height: 2.81597in

9) Assign address

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

Add additional test code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The main function of other codes is to read and write ddr4 and compare whether the data is consistent. I will not introduce it in detail here. You can refer to the engineering code.

.. image:: images/media/image108.png
  :width: 3.17708in
  :height: 2.13056in

1) Add mark_debug debugging in mem_test.v

.. image:: images/media/image109.png
  :width: 3.94143in
  :height: 2.8396in

2) Pin binding

.. image:: images/media/image110.png
  :width: 1.65069in
  :height: 1.32917in

3) Comprehensive

.. image:: images/media/image111.png
  :width: 1.95694in
  :height: 0.85278in

3. After the synthesis is completed, click Set up debug

.. image:: images/media/image112.png
  :width: 1.72292in
  :height: 2.53125in

.. image:: images/media/image113.png
  :width: 3.80139in
  :height: 2.40208in

.. image:: images/media/image114.png
  :width: 3.98681in
  :height: 2.53333in

Set the number of sampling points according to needs

.. image:: images/media/image115.png
  :width: 4.25069in
  :height: 2.7125in

.. image:: images/media/image116.png
  :width: 4.31111in
  :height: 2.74792in

Then save and generate pdi file

.. image:: images/media/image51.png
  :width: 1.8375in
  :height: 0.75069in

Download debugging
--------------------

After generating the pdi file, use JTAG to download it to the development board, and DDR4 calibration and other information will be displayed in the MIG_1 window.

.. image:: images/media/image117.png
  :width: 6.00278in
  :height: 3.32917in

Debug signals can be viewed in hw_ila_1

.. image:: images/media/image118.png
  :width: 6in
  :height: 3.0125in

.. _Experiment Summary-1:

Experiment summary
-----------------------

This experiment directly reads and writes ddr4 through the PL side Verilog code. It mainly understands the configuration method of NoC and how to access DDR4 through NoC. This configuration will be used in subsequent experiments.

Chapter 4 LVDS LCD screen display experiment
=============================================

**The experimental Vivado project is "lvds_lcd".**

This chapter introduces the color bar display of lvds lcd LCD screen.

.. _Hardware Introduction-1:

Hardware introduction
--------------------------

ALINX black gold 7-inch LCD screen module (AN7000) uses IVO's 7-inch TFT LCD screen.
The model number of the LCD screen is M070AWAD R0. AN7000 LCD screen module is made of TFT
It consists of an LCD screen and a driver board. For specific parameters, please refer to the AN7000 user manual. The actual photos of AN7000 are as follows:

.. image:: images/media/image119.png
  :alt: \_K4A5291
  :width: 5.37431in
  :height: 3.34722in

AN7000 LCD screen front view

programming
---------------

1) Like PL’s “Hello World” LED experiment, add a block
design, and add the CIPS core and configure it as PL Subsystem

.. image:: images/media/image120.png
  :width: 2.17639in
  :height: 1.05556in

2. Add LVDS LCD controller IP

.. image:: images/media/image121.png
  :width: 1.78542in
  :height: 1.19028in

3. Add Advanced IO Wizard and configure

.. image:: images/media/image122.png
  :width: 4.32222in
  :height: 3.34167in

.. image:: images/media/image123.png
  :width: 4.3in
  :height: 2.89028in

.. image:: images/media/image124.png
  :width: 4.62847in
  :height: 2.30694in

4. Connect as follows

.. image:: images/media/image125.png
  :width: 5.68681in
  :height: 2.65486in

5. Add the color bar file, drag it to the block design, and connect it

.. image:: images/media/image126.png
  :width: 3.91597in
  :height: 1.97222in

Define VIDEO_1280_720 in video_define.v because the LCD resolution is 1280*720

.. image:: images/media/image127.png
  :width: 1.94861in
  :height: 0.59722in

6. Generate HDL file

.. image:: images/media/image128.png
  :width: 2.46181in
  :height: 1.31875in

7. Add some other signals

.. image:: images/media/image129.png
  :width: 5.85069in
  :height: 2.89861in

8. Constraint pins

.. image:: images/media/image130.png
  :width: 2.33611in
  :height: 1.44097in

9. Generate pdi file

.. image:: images/media/image51.png
  :width: 1.8375in
  :height: 0.75069in

Experimental phenomena
-------------------------

Connect the LCD screen, download the program, and you can see the color bar display.

.. image:: images/media/image131.png
  :width: 3.72014in
  :height: 4.87708in

.. image:: images/media/image132.png
  :width: 5.35347in
  :height: 3.80694in

Chapter 5 GTYP transceiver bit error rate test IBERT experiment
=================================================================

**The experimental VIvado project is "ibert_test", and there is also "ibert_ex" in the directory, which is the generated test project.**

Vidado software provides us with the powerful bit error rate tester IBERT, which can not only test the bit error rate but also test the eye diagram, which brings great convenience to us in using high-speed transceivers. This experiment will serve as a starting point and briefly introduce the IBERT use.

.. _Hardware Introduction-2:

Hardware introduction
---------------------------

To use IBERT to test the bit error rate and eye diagram, you must have transceiver loopback hardware. There are two SFP optical interfaces on the development board. In this experiment, the two optical interfaces are connected in pairs to form two transceiver loopthrough links.

.. _vivado project creation-1:

Vivado project set up
------------------------

1) Create a new project named "ibert_test"

2) Search "gt" in "IP Catalog" to quickly find "Versal ACAPs Transceivers"
Wizard", double-click

.. image:: images/media/image133.png
  :width: 5.99722in
  :height: 1.49167in

3) Change "Component Name" to "ibert" and modify the preset to "Aurora 64B/66B"

.. image:: images/media/image134.png
  :width: 6.00208in
  :height: 3.88889in

4) Click Transceiver Configs Protocol 0, configure the sending and receiving parameters, and click OK

.. image:: images/media/image135.png
  :width: 3.72083in
  :height: 1.70903in

.. image:: images/media/image136.png
  :width: 6.00347in
  :height: 4.52292in

.. image:: images/media/image137.png
  :width: 5.99722in
  :height: 4.48472in

5) Click Generate

.. image:: images/media/image138.png
  :width: 2.625in
  :height: 3.27153in

6) Right-click "Open IP Example Design..." and select the example project path

.. image:: images/media/image139.png
  :width: 3.3875in
  :height: 2.54236in

.. image:: images/media/image140.png
  :width: 3.84653in
  :height: 1.75556in

7) Add buffer to connect to apb3clk

.. image:: images/media/image141.png
  :width: 5.9875in
  :height: 3.14722in

8) Add inverter connected to reset

.. image:: images/media/image142.png
  :width: 5.99514in
  :height: 1.95069in

9) Some other signals are configured as constant 0

.. image:: images/media/image143.png
  :width: 3.93056in
  :height: 3.19722in

10) Delete output signal

.. image:: images/media/image144.png
  :width: 2.025in
  :height: 1.57778in

11) Configure sfp_disable to 0

.. image:: images/media/image145.png
  :width: 4.46458in
  :height: 1.00556in

12) Change CIPS to PL Subsystem

.. image:: images/media/image146.png
  :width: 5.47014in
  :height: 4.74514in

13) Constraint pins

.. image:: images/media/image147.png
  :width: 5.99583in
  :height: 5.09167in

14) Generate pdi file

.. image:: images/media/image148.png
  :width: 1.72431in
  :height: 0.79444in

.. _Download Debug-1:

Download debugging
--------------------

1) Insert the optical module, then use optical fiber to connect the two optical ports, connect the JTAG download cable, and power on the development board

.. image:: images/media/image149.png
  :width: 5.99028in
  :height: 3.39931in

2) Use JTAG to download the BIT file to the development board. You can see that the speed is close to 10.3125Gbps.

.. image:: images/media/image150.png
  :width: 2.70625in
  :height: 3.36528in

3) Select IBERT, right-click and select "Create Links"

.. image:: images/media/image151.png
  :width: 3.33819in
  :height: 1.68889in

Referring to the schematic diagram, the optical fiber is connected to CH0 and CH1 of Quad104. Select Link 0 as Quad_104 CH_0
TX corresponds to CH1 RX, Link 1 corresponds to Quad_104 CH_1 TX and CH0 RX

.. image:: images/media/image152.png
  :width: 5.99931in
  :height: 3.93542in

4) Modify the configuration, select PRBS 31 for the code stream, and configure Loopback to None

.. image:: images/media/image153.png
  :width: 5.99028in
  :height: 0.55903in

5) After configuration, you can click BERT Reset. You can see that the Errors are all 0 and restart the test.

.. image:: images/media/image154.png
  :width: 5.99722in
  :height: 1.33472in

6) Select a link, right-click "Create Scan..."

.. image:: images/media/image155.png
  :width: 3.30208in
  :height: 1.98889in

.. image:: images/media/image156.png
  :width: 3.36944in
  :height: 3.56319in

7) The eye diagram configured by default. Note: The measured eye diagram may be different when using different software versions.

.. image:: images/media/image157.png
  :width: 5.99792in
  :height: 3.05069in

Chapter 6 Experience ARM, bare metal output "Hello World"
===========================================================

**From this chapter onwards, FPGA engineers and software development engineers collaborate to implement it.**

The previous experiments were all conducted on the PL side. It can be seen that there is no difference from the ordinary FPGA development process. The main advantage of ZYNQ is the reasonable combination of FPGA and ARM, which puts forward higher requirements for developers. Starting from this chapter, we start to use ARM, which is what we call PS. In this chapter, we use a simple serial port printing to experience Vivado
Vitis and PS side features.

The previous experiments are all things that FPGA engineers should do. From the beginning of this chapter, there is a division of labor. FPGA engineers are responsible for setting up the Vivado project and providing good hardware to software developers. Software developers can develop applications on this basis. . A good division of labor is also conducive to the advancement of the project. If a software developer wants to do everything, it may take a lot of time and energy to learn FPGA knowledge. Converting from software thinking to hardware thinking is a relatively painful process. If you just want to learn purely and have time, you can That's another matter. Professional people doing professional things is a good choice.

.. _Hardware Introduction-3:

Hardware introduction
----------------------------

We can see from the schematic diagram that the ZYNQ chip is divided into PL and PS. The IO allocation on the PS side is relatively fixed and cannot be allocated arbitrarily, and there is no need to allocate pins in the Vivado software. Although this experiment only used PS, it still To create a Vivado project to configure PS pins. Although the ARM on the PS side is a hard core, in ZYNQ the ARM hard core must be added to the project before it can be used. The previous chapters introduced projects in the form of codes. This chapter begins by introducing ZYNQ's graphical approach to building projects.

FPGA engineer job content
-------------------------------

The following introduces what FPGA engineers are responsible for.

.. _vivado project creation-2:

Vivado project set up
--------------------------

1) Create a project named "ps_hello". The establishment process will not be described in detail. Please refer to "PL's" Hello
World "LED Experiment".

2) Click "Create Block Design" to create a Block design

.. image:: images/media/image54.png
  :width: 2.26458in
  :height: 2.29792in

3) “Design
name" is not modified here, keep the default "design_1", you can modify it as needed, but the name should be as short as possible, otherwise there will be problems compiling under Windows.

.. image:: images/media/image88.png
  :width: 3.01319in
  :height: 1.87153in

4) Click the “Add IP” shortcut icon

.. image:: images/media/image56.png
  :width: 5.19167in
  :height: 2.67778in

5) Search for "PS" and double-click "Control, Interfaces & Processing" in the search results list
System"

.. image:: images/media/image57.png
  :width: 2.63333in
  :height: 2.09792in

6) Click Run Block Automation

.. image:: images/media/image158.png
  :width: 5.25069in
  :height: 1.81389in

7) Configure as follows, click OK

.. image:: images/media/image159.png
  :width: 4.79514in
  :height: 3.08958in

8) Automatic connection is as follows

.. image:: images/media/image160.png
  :width: 5.60139in
  :height: 2.27986in

9) Double-click CIPS to configure

.. image:: images/media/image161.png
  :width: 4.58958in
  :height: 3.92361in

.. image:: images/media/image162.png
  :width: 4.28125in
  :height: 3.73403in

select PS PMC to config

.. image:: images/media/image163.png
  :width: 3.59444in
  :height: 0.93611in

10) Config QSPI，EMMC，SD

.. image:: images/media/image164.png
  :width: 5.21736in
  :height: 2.54306in

.. image:: images/media/image165.png
  :width: 5.25in
  :height: 2.70556in

.. image:: images/media/image166.png
  :width: 5.09861in
  :height: 2.69375in

Select the corresponding MIO

.. image:: images/media/image167.png
  :width: 3.26667in
  :height: 2.32778in

11) Check USB 2.0, GEM0, UART0, TTC, GPIO and other peripherals

.. image:: images/media/image168.png
  :width: 5.39375in
  :height: 2.91806in

Configure peripherals

.. image:: images/media/image169.png
  :width: 5.53472in
  :height: 3.48264in

12) Configure MIO24 as GPIO input, corresponding to the PS side buttons, and configure MIO25 as GPIO output, corresponding to the PS side LED lights

.. image:: images/media/image170.png
  :width: 4.39028in
  :height: 3.78889in

.. image:: images/media/image171.png
  :width: 4.35347in
  :height: 3.87986in

13) In clocking, set the reference clock more accurately

.. image:: images/media/image172.png
  :width: 4.75972in
  :height: 1.51597in

14) Check all internal interrupts, the configuration is complete, and click OK

.. image:: images/media/image173.png
  :width: 5.99236in
  :height: 2.18958in

15) Click Finish

.. image:: images/media/image174.png
  :width: 4.53958in
  :height: 3.93125in

16) Double-click AXI NoC to configure DDR4

.. image:: images/media/image175.png
  :width: 1.77847in
  :height: 1.86667in

.. image:: images/media/image176.png
  :width: 6.00208in
  :height: 3.89514in

.. image:: images/media/image177.png
  :width: 6.00208in
  :height: 2.32847in

select reference clock and system clock

.. image:: images/media/image178.png
  :width: 5.21944in
  :height: 2.06736in

DDR Address Region 1, select NONE and OK

.. image:: images/media/image179.png
  :width: 5.99375in
  :height: 3.34444in

1)  Modify pin name

.. image:: images/media/image180.png
  :width: 5.99306in
  :height: 1.90556in

Double-click to configure the frequency of sys_clk to 200MHz

.. image:: images/media/image181.png
  :width: 3.59375in
  :height: 2.04861in

18) Select the Block design, right-click "Create HDLWrapper...", create a Verilog or VHDL file for blockdesign generates HDL top-level files.

.. image:: images/media/image182.png
  :width: 4.225in
  :height: 2.38819in

19) Keep the default options and click "OK"

.. image:: images/media/image183.png
  :width: 3.14452in
  :height: 1.81793in

20) Add constraint

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

22) Cancel after completion

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

At this time, you can see the xsa file in the project directory. This file contains Vivado hardware design information and can be used by software developers.

.. image:: images/media/image194.png
  :width: 2.01473in
  :height: 1.46875in

At this point, the work of the FPGA engineer comes to an end.

Software engineer job content
---------------------------------

**The Vitis project directory is "ps_hello/vitis"**

The following is the responsibility of software engineers.

Debugging
------------

Create Application project
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1) Create a new folder and copy the xx.xsa file exported by vivado.

2) Vitis is an independent software. You can double-click the Vitis software to open it, or select ToolsLaunch in the Vivado software.
VitisOpen Vitis software

.. image:: images/media/image9.png
  :width: 3.18611in
  :height: 2.00833in

On the welcome interface, click Open Workspace, select the previously created folder, and click "OK"

.. image:: images/media/image195.png
  :width: 5.99931in
  :height: 2.57431in

3) After starting Vitis, the interface is as follows, click "Create Platform"
Component", this option will create a Platform project, which is similar to previous versions of hardware
platform, including hardware support related files and BSP.

.. image:: images/media/image196.png
  :width: 5.97778in
  :height: 2.38958in

4) Fill in the Component name and path on the first page, keep the default, and click Next

.. image:: images/media/image197.png
  :width: 5.98889in
  :height: 4.01319in

5) Select (XSA, select "Browse", select the previously generated xsa, click to open, and then click Next

.. image:: images/media/image198.png
  :width: 5.99306in
  :height: 3.99583in

6) Select operating system and processor, keep the default here

.. image:: images/media/image199.png
  :width: 5.99167in
  :height: 4.00556in

7) Click Finish to complete

.. image:: images/media/image200.png
  :width: 5.99722in
  :height: 3.98403in

8) After generation, a window interface appears. The following are some window introductions. They are similar to the previous version of Vitis interface, but the differences are also quite large.

.. image:: images/media/image201.png
  :width: 5.98611in
  :height: 3.26875in

9) The platform can be compiled in the Flow window

.. image:: images/media/image202.png
  :width: 2.13472in
  :height: 0.70208in

no error status

.. image:: images/media/image203.png
  :width: 2.13333in
  :height: 0.58333in

10) Click Example on the left. There are many official routines here, which are similar to previous versions. Select Hello.
World

.. image:: images/media/image204.png
  :width: 1.89167in
  :height: 4.90069in

11) Click to create project

.. image:: images/media/image205.png
  :width: 4.87361in
  :height: 2.50347in

12) Fill in the project name and path and keep the default

.. image:: images/media/image206.png
  :width: 4.04653in
  :height: 2.71181in

13) Select the platform

.. image:: images/media/image207.png
  :width: 3.95486in
  :height: 2.64167in

14) Click Next

.. image:: images/media/image208.png
  :width: 3.99306in
  :height: 2.69167in

15) Complete

.. image:: images/media/image209.png
  :width: 3.96111in
  :height: 2.65208in

16) Select hello_world and click Build

.. image:: images/media/image210.png
  :width: 2.88194in
  :height: 3.22778in

.. _Download Debug-2:

Download debugging
~~~~~~~~~~~~~~~~~~~~~~~

1) Connect the JTAG cable to the development board and the UART USB cable to the PC

.. image:: images/media/image211.png
  :width: 4.27986in
  :height: 2.48125in

2) Before powering on, it is best to set the startup mode of the development board to JTAG mode and pull it to the "ON" position.

.. image:: images/media/image82.png
  :width: 4.09375in
  :height: 2.23403in

3) Power on the development board, open the serial port debugging tool, and click Run in Flow

.. image:: images/media/image212.png
  :width: 2.37153in
  :height: 1.08958in

4) At this time, observe the serial port debugging tool and you can see the output "Hello World"

.. image:: images/media/image213.png
  :width: 2.51458in
  :height: 2.28125in

firmware
-----------

Ordinary FPGAs can generally be started from flash or passively loaded. The startup process has been introduced in the PMC architecture in Chapter 1 and will not be introduced here.

Select Create Boot in Flow
Image, you can see the generated BIF file path in the pop-up window. The BIF file is the configuration file for generating the BOOT file, and the generated Output
Image file path, that is, the BOOT.pdi file is generated. It is the startup file we need. It can be placed in the SD card for startup, or it can be programmed to QSPI.
Flash.

.. image:: images/media/image214.png
  :width: 2.99306in
  :height: 1.34792in

.. image:: images/media/image215.png
  :width: 3.94653in
  :height: 4.93542in

The boot.pdi file can be found in the generated directory

.. image:: images/media/image216.png
  :width: 6.18611in
  :height: 0.72153in

SD card boot test
~~~~~~~~~~~~~~~~~~~~~~~

1) Format the SD card. It can only be formatted to FAT32 format. Other formats cannot be started.

.. image:: images/media/image217.png
  :width: 1.62959in
  :height: 2.62898in

2) Put the boot.pdi file into the root directory

.. image:: images/media/image218.png
  :width: 2.32817in
  :height: 1.3048in

3) Insert the SD card into the SD card slot of the development board

4) Adjust the startup mode to SD card startup

.. image:: images/media/image219.png
  :width: 4.09653in
  :height: 2.91389in

5) Open the serial port software, power on and start, you can see the printed information. The red box is the FSBL startup information, and the yellow arrow part is the executed application helloworld.

.. image:: images/media/image220.png
  :width: 3.40694in
  :height: 2.99861in

QSPI startup test
~~~~~~~~~~~~~~~~~~~~~~~

1) In the Vitis menu Vitis -> Program Flash

.. image:: images/media/image221.png
  :width: 2.77778in
  :height: 1.95347in

2) Select the boot.pdi to be burned in the Image FIle file. Select Verify after flash, Flash
Select qspi-x8-dual_parallel for Type, and verify the flash after programming is completed.

.. image:: images/media/image222.png
  :width: 4.70417in
  :height: 2.5in

3) Click Program and wait for programming to complete

.. image:: images/media/image223.png
  :width: 3.61806in
  :height: 2.42986in

4) Set the startup mode to QSPI, start it again, and you can see the same startup effect as SD in the serial port software.

.. image:: images/media/image224.png
  :width: 3.06458in
  :height: 2.31667in

.. image:: images/media/image225.png
  :width: 3.58403in
  :height: 3.25347in

chapter summary
--------------------

This chapter introduces the classic process of Versal development from the perspectives of both FPGA engineers and software engineers. The main job of FPGA engineers is to build a hardware platform and provide hardware description files xsa to software engineers, who then develop applications based on this. This chapter is a simple example that introduces the collaborative work of FPGA and software engineers. It will also involve joint debugging between PS and PL later, which is more complicated and is also the core part of Versal development.

At the same time, FSBL, startup file production, SD card startup method, QSPI download and startup method are also introduced.

Chapter 7 lwip used by PS side Ethernet
=========================================

**The vivado project directory is "ps_hello/vivado"**

.. _Software Engineer Job Content-1:

Software engineer job content
-------------------------------

The following is the responsibility of software engineers.

The development board has two channels of Gigabit Ethernet, connected through the RGMII interface. This experiment demonstrates how to use the LWIP template that comes with Vitis to perform Gigabit Ethernet TCP communication on the PS side.

Although LWIP is a lightweight protocol stack, if you have never used it before, it will be difficult to use it. It is recommended to familiarize yourself with the relevant knowledge of LWIP first.

Vitis program development
----------------------------

LWIP library modification
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Since the built-in LWIP library can only recognize some phy chips, if the phy chip used by the development board is not within the default support range, the library file must be modified. You can also directly use the modified library to replace the original library.

1) Find the library file directory "x:\\Xilinx2023.2\\Vitis\\2023.2\\data\\embeddedsw\\ThirdParty\\sw_services"

.. image:: images/media/image226.png
  :width: 1.42708in
  :height: 2.45903in

2) Find the files "xaxiemacif_physpeed.c" and "xemacpsif_physpeed.c" in the file directory "lwip213_v1_1\\src\\contrib\\ports\\xilinx\\netif" to be modified.

.. image:: images/media/image227.png
  :width: 4.20694in
  :height: 2.40833in

Mainly added get_phy_speed_ksz9031, get_phy_speed_JL2121 to support ksz9031 and JL2121 auto-negotiation to obtain speed. The modified lwip library is provided in the information and can be directly replaced.

.. image:: images/media/image228.png
  :width: 1.24028in
  :height: 0.19097in

Create an APP project based on the LWIP template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1. Add lwip213 library to BSP

.. image:: images/media/image229.png
  :width: 5.22569in
  :height: 3.67986in

2. Configure the dhcp function to True

.. image:: images/media/image230.png
  :width: 4.66528in
  :height: 3.54236in

Build platform

.. image:: images/media/image231.png
  :width: 3.29861in
  :height: 0.97153in

3. Select lwIP Echo Server template

.. image:: images/media/image232.png
  :width: 4.29028in
  :height: 3.56597in

4. Generate template

.. image:: images/media/image233.png
  :width: 4.99444in
  :height: 2.95764in

The process will not be described in detail. You can refer to Chapter 6 of Experience ARM, Bare Metal Output "Hello World"

5.Build

.. image:: images/media/image234.png
  :width: 3.12569in
  :height: 1.42014in

.. _Download Debug-3:

Download debugging
---------------------

The test environment requires a router that supports dhcp. The development board can automatically obtain an IP address when connected to the router. The experimental host and development board are on the same network and can communicate with each other.

Ethernet test
~~~~~~~~~~~~~~~~

1) Connect the serial port and open the serial debugging terminal, connect the PS end Ethernet cable to the router, and run the Vitis download program

.. image:: images/media/image235.png
  :width: 3.66319in
  :height: 2.08403in

.. image:: images/media/image236.png
  :width: 3.39028in
  :height: 1.48194in

2) You can see some information printed out by the serial port. You can see that the address automatically obtained is "192.168.1.63", the connection speed is 1000Mbps, and the tcp port is 7

.. image:: images/media/image237.png
  :width: 4.6125in
  :height: 3.15556in

3) Use telnet to connect

.. image:: images/media/image238.png
  :width: 2.92292in
  :height: 2.83194in

4) When a character is entered, the development board returns the same character

.. image:: images/media/image239.png
  :width: 3.92222in
  :height: 2.45764in

.. _Experiment Summary-2:

Experiment summary
--------------------

Through the experiment, we have a deeper understanding of the development of the Vitis program. This experiment simply explains how to create an LWIP application. LWIP can complete UDP, TCP and other protocols. In subsequent tutorials, we will provide specific applications based on Ethernet, such as cameras. The data is sent to the host computer via Ethernet for display.

Chapter 8 Overall engineering and experiments
================================================

This chapter integrates most of the peripherals of the board into the Vivado project.

.. _vivado project creation-3:

Vivado project set up
-------------------------

The overall block diagram is as follows. Two MIPI cameras write to DDR4 and LVDS through VDMA.
The LCD reads image data from DDR4 via VDMA. The specific construction process will not be described. The Vivado project can be restored through TCL scripts.

.. image:: images/media/image240.png
  :width: 4.925in
  :height: 3.88403in

Vitis experiment
--------------------

LVDS LCD display experiment based on VDMA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The main function of this experiment is that ARM makes a color bar in DDR, VDMA reads this space and sends it to LVDS
LCD display module. Download program:

.. image:: images/media/image241.png
  :width: 3.23472in
  :height: 0.99583in

.. image:: images/media/image242.png
  :width: 2.97986in
  :height: 1.30486in

The displayed results are as follows:

.. image:: images/media/image132.png
  :width: 5.35347in
  :height: 3.80694in

MIPI camera acquisition and display experiment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The main function of this experiment is to configure a single or two MIPI cameras and display images on the LCD, which is also implemented through VDMA.

.. image:: images/media/image243.png
  :width: 3.17083in
  :height: 2.15069in

Run downloader

.. image:: images/media/image244.png
  :width: 3.38125in
  :height: 1.65833in

If you want to display a single or two cameras, you can modify the macro definition in config.h, recompile and download it.

.. image:: images/media/image245.png
  :width: 5.12986in
  :height: 2.01389in

display effect

.. image:: images/media/image246.png
  :width: 4.16944in
  :height: 3.74514in

MIPI camera binocular acquisition Ethernet transmission experiment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The LCD display of the MIPI camera was introduced earlier, but in some cases, the video needs to be transmitted to the host computer, and the Ethernet can be used to transmit the data. This chapter uses LWIP udp to transmit the camera data to the host computer.

The following introduces part of the content of LWIP. When communicating with the host computer, UDP transmission is used, and the protocol is customized in the UDP data packet, as shown below:

1. Obtain board information

(1) Query command (5 bytes in total, sent by the host computer through Ethernet)

+----------------------+--------------+------------------------------+
| Number of bytes      | 1            | 4                            |
+----------------------+--------------+------------------------------+
| Command information  | Header       | 0x00020001                   |
+----------------------+--------------+------------------------------+

(2) Response command (16 bytes in total, sent by the development board through Ethernet)

+---------------+----------------------------------------------------+
|Number of bytes|Command information                                 |
+---------------+----------------------------------------------------+
| 1             | Header|0x01                                        |
+---------------+----------------------------------------------------+
| 4             | 0x00020001                                         |
+---------------+----------------------------------------------------+
| 6             | Board MAC address                                  |
+---------------+----------------------------------------------------+
| 4             | Board IP address                                   |
+---------------+----------------------------------------------------+
| 1             | 0x02                                               |
+---------------+----------------------------------------------------+

1. Obtain data

(1) Control command (data request sent by the host computer)

+---------------+-------------------------------------------------------------------------------------------------------------------------+
|Number of bytes|Command information                                                                                                      |
+---------------+-------------------------------------------------------------------------------------------------------------------------+
| 1             | Header                                                                                                                  |
+---------------+-------------------------------------------------------------------------------------------------------------------------+
| 4             | 0x00020002                                                                                                              |
+---------------+-------------------------------------------------------------------------------------------------------------------------+
| 6             | Board MAC address                                                                                                       |
+---------------+-------------------------------------------------------------------------------------------------------------------------+
| 1             | Camera channel selection, a value of 1 means only turning on camera                                                     |
|               | Header 1, the value 2 means opening only camera 2, the value 3 means opening both cameras at the same time              |
+---------------+-------------------------------------------------------------------------------------------------------------------------+
| 1             | Start signal, 0 means turning off the upper image display, other means turning on the image display                     |
+---------------+-------------------------------------------------------------------------------------------------------------------------+

(2) Response command (sent by development board)

+---------------+-----------------------------------------------------------------------------------------------+
|Number of bytes| Command information                                                                           |
+---------------+-----------------------------------------------------------------------------------------------+
| 1             | Header|0x 01                                                                                  |
+---------------+-----------------------------------------------------------------------------------------------+
| 3             | 0x 000200                                                                                     |
+---------------+-----------------------------------------------------------------------------------------------+
| 1             | Channel identification, the value 2 represents camera 1, the value 3 represents camera 2      |
+---------------+-----------------------------------------------------------------------------------------------+
| 3             | Serial number, Ethernet packet sequence number, used for host computer identification         |
+---------------+-----------------------------------------------------------------------------------------------+
| N             | Image data                                                                                    |
+---------------+-----------------------------------------------------------------------------------------------+

Each UDP packet contains a Header, in the first byte, with the following format:

+----------------------+----------------------+--------------------+
| bit                  | value(0)             | value(1)           |
+----------------------+----------------------+--------------------+
| bit 0                | Query or control     | Reply              |
+----------------------+----------------------+--------------------+
| bit1~bit7            | Random data          |                    |
+----------------------+----------------------+--------------------+

Note: When responding, the upper 7 bits of random data remain unchanged and bit0 is set to 1

The workflow is:

1) The host computer sends an inquiry command

2) Development board responds to inquiries

3) The host computer sends control command request data

4) The development board sends data

5) Cycle of steps 3 and 4

Experimental steps
^^^^^^^^^^^^^^^^^^^^^^^^

1. If you check the lwip library in vitis

.. image:: images/media/image247.png
  :width: 5.70833in
  :height: 3.84861in

And do parameter configuration

.. image:: images/media/image248.png
  :width: 5.32153in
  :height: 2.70347in

.. image:: images/media/image249.png
  :width: 4.04792in
  :height: 2.69861in

.. image:: images/media/image250.png
  :width: 3.94028in
  :height: 2.18611in

Recompile the platform

.. image:: images/media/image251.png
  :width: 3.89931in
  :height: 1.09861in

2. Build the project, connect the board camera, power supply, serial port, PS port ETH1, then click Run to download the program

.. image:: images/media/image252.png
  :width: 5.53819in
  :height: 3.85764in

.. image:: images/media/image253.png
  :width: 4.06181in
  :height: 1.75486in

3. If there is a DHCP server, the IP will be automatically assigned to the development board; if there is no DHCP server, the default development board IP address is 192.168.1.10. You need to set the IP address of the PC to the same network segment, as shown in the figure below. At the same time, make sure that there is no IP address of 192.168.1.10 in the network, otherwise it will cause an IP conflict and prevent the image from being displayed. You can enter ping in CMD before the board is powered on.
192.168.1.10 Check whether it can be pinged successfully. If it is successfully pinged, it means that this IP address exists in the network and cannot be verified.

..

After there is no problem, open the serial port software.

.. image:: images/media/image254.png
  :width: 3.16215in
  :height: 3.95585in

4. The serial port print information is as follows, the network card speed is detected and the IP address is set.

.. image:: images/media/image255.png
  :width: 5.41042in
  :height: 4.34167in

5. Open the Vivado project folder and open videoshow.exe

.. image:: images/media/image256.png
  :width: 1.08889in
  :height: 0.16181in

The software scans two cameras. You can select the corresponding camera to display by checking it, and click to play.

.. image:: images/media/image257.png
  :width: 4.5375in
  :height: 3.54931in

The display effect is as follows. If you want to reselect the display channel, double-click on the software screen to return to the selection interface and select the image to be displayed again.

.. image:: images/media/image258.png
  :width: 5.98889in
  :height: 2.35486in

6. Open the task manager and you can see that the network bandwidth is about 750Mbps

.. image:: images/media/image259.png
  :width: 4.40208in
  :height: 3.85833in
