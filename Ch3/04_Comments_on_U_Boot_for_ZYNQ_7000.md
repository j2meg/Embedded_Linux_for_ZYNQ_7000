# Comments on U-Boot Compatibility for ZYNQ-boards

As the objective of this repository is to document compatibility
between MELP third edition book with ZYNQ 7000 SoC based boards. 

And our specific targets are ZYBO (Ffirst edition board) and 
PYNQ-Z2 (second edition built by Tul). 

It is important to document that **target specifc compatibility with U-Boot mainline tools**
Next lines give details about it. 

## U-Boot support for ZYBO
As it is documented in https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841973/Build+U-Boot

Support for zybo in the U-Boot mainline comes in two different ways in agreement with the U-Boot 
work U-Boot version. 

Also, it is interesting to note that Xilinx/amd has its own U-Boot repository 
available at: https://github.com/Xilinx/u-boot-xlnx/ that is a mirror of the 
oficial U-Boot tree mantained by denx git clone https://source.denx.de/u-boot/u-boot.git
Posible support for specific Xilinx devices may be in its own mirror, but denx
has a wide support for oficial Xilinx devices. 

### Zybo in U-Boot (< 2020.1 Release)

For minor releases of U-Boot than 2020.1 zybo is supported by a '[board]_defconfig' file
that depends on your specific device.
´´´
Digilent ZYBO (1st Edition) ---> zynq_zybo_defconfig

Digilent ZYBO-7Z (2nd Edition) --> zynq_zybo_z7_defconfig
´´´

In the case of this implementation, we use zynq_zybo_defconfig.

An example to buildU-Boot for zybo is 

```bash
# Into the U-boot repository path
make distclean
make zynq_zybo_defconfig
make
```

### Zybo in U-Boot (>= 2020.1 Release)
After the build process completes the target u-boot elf-file is created in the top level source directory, named u-boot/u-boot.elf. Additionally in the tools/ directory the mkimage utility is created, which is used in other tasks to wrap images into u-boot format.

For 2020.1 and above releases common defconfig is being made. 

```bash
All Zynq (except for mini) ---> xilinx_zynq_virt_defconfig
```
To build U-boot for zybo board, follow below steps.

```bash
make distclean
make xilinx_zynq_virt_defconfig
export DEVICE_TREE="zynq-zybo"
make 
```

device tree can be found under ```arch/arm/dts/``` with the name ```zynq-zybo.dts```

A compiled device tree blob (.dtb) with the name ```zynq-zybo.dtb``` will be generated under ```arch/arm/dts``` after the build

Similarly one can find for all other zynqmp/zynq boards with matching name.

If u-boot is build without exporting DEVICE_TREE, the DTB file needs to be loaded at 0x100000 in primary DDR memory (configurable with ```CONFIG_XILINX_OF_BOARD_DTB_ADDR```).

device tree should be loaded before loading u-boot.elf.

The DTB load address (0x100000) is same for ```zynq/zynqmp``` by default.


To make mkimage available in other steps, it is recommended to add the tools directory to your $PATH.

```bash
cd tools
export PATH=`pwd`:$PATH
```

