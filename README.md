# Embedded_Linux_for_ZYNQ_7000
The present repository implements reference examples from Mastering Embedded Linux Programming Third Edition for Xilinx zynq_7000 SoC. Specific targets are QEMU, ZYBO and PYNQ-Z2

## Main Objective
The main objective of this implementation is to provide an alternative guide to bring up boards following the learning methodologies of the third edition of MELP and guide the user on the challenges to handle alternative development boards. 

Aligned with the main objective, documentations and implementations follows the chapters structure of the MELP book allowing to the user to have a complementary Hands-on Learning on embedded systems crossing experience from MELP examples and porting the knowledge to ZYNQ-7000 Platforms. 

## Table of Contents

 
* [Chapter 1 — Starting Out](Ch1/)
  * [01 QEMU Installation](Ch1/01_Install_QEMU.md)
* [Chapter 2 — Learning about Toolchains](Ch2/)
  * [01 CrossTool-NG Requirements](Ch2/01_crosstool-NG.md)
  * [CrossTool-NG installation](Ch2/02_crosstool-NG_installation.md)
  * [CrossTool-NG for ZYNQ-7000 targets](Ch2/03_crosstool-NG-for-ZYNQ-7000.md)
  * [CrossTool-NG reference building output](Ch2/04_crosstool-NG-reference-building-log.txt)
* [Chapter 3 — All about Bootloaders](Ch3/)
  * [01 Chapter Requirements](Ch3/01_Chapter_Requirements.md)
  * [02 Building U-Boot](Ch3/02_Build_U_Boot.md)
  * [03 Building U-Boot for specific target](Ch3/03_Build_u_boot_for_specific_target.md)
  * [04 Comments on U-Boot for Zynq 7000 based boards](Ch3/04_Comments_on_U_Boot_for_ZYNQ_7000.md)
  * [05 U-Boot for ZYBO TroubleShooting](Ch3/05_U-boot_for_Zybo_Troubleshooting.md)
  * [06 U-Boot for ZYBO expected output](Ch3/06_U-boot_for_zybo_expected_output.txt)
  * [07 ZYBO First Boot](Ch3/07_ZYBO_First_Boot.md)
  * [Util scripts](Ch3/util_scripts/)
* [Chapter 4 — Configuring and Building the Kernel](Ch4/)
  * [01 Technical Requirements](Ch4/01_Technical_Requirements.md)
  * [02 Building Kernel for ZYBO](Ch4/02_Building_kernel.md)
  * [03 Booting Kernel on Zybo](Ch4/03_Booting_Kernel_on_Zybo.md)
* [Chapter 5 - Building a Root filesystem](Ch5)
  * [01 Topics and Technical Requirements](Ch5/01_Topics_and_Requirements.md)
  * [02 Root filesystem layout](Ch5/02_Root_Filesystem_layout.md)
  * [03 Populating the root filesystem](Ch5/03_Populating_Filesystem.md)
  * [04 Transferring filesystem to ZYBO](Ch5/04_Transfer_Filesystem_To_target.md)
