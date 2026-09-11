# Build U-Boot for specific target

Reference MELP | Page 65.

## Objective
- To build an U-Boot Instance for a ZYNQ-7000 CortexA9 processor
   using the toolchain generated on Chapter 2. 
## Implementation
The original example compiles a U-Boot instance for **Beagle Bone Black**
evaluation board. 

Following the MELP methodology, our first step is to identify 
the **u-boot configuration file** for our target in:  

```bash
<u-boot repository path>/configs/<vendor>/<target>/
# For example:
# u-boot/configs/ti/am335x
```

The name of the file that we are looking for has a format

```bash
[board]_defconfig
```
        
Configuration file is used to define the presets for your target. 
other way to identify your device is to look on the board directory
```bash
<u-boot repository path>/board/<vendor>/<target>/
```
**For ZYNQ-7000 boards** a wide discussion about ```defconfig``` files, is available at the next
document of this repository. ```04_Comments_on_U_Boot_for_ZYNQ_7000.md```

## To work ZYBO with U-Boot v2026.07 (lastest stable available)

The steps required are show next. 



https://www.instructables.com/Booting-Linux-on-the-ZYBO/         
### Inform to U-Boot your Cross Compiler prefix by setting make variable
```CROSS_COMPILE``` and selecting the configuration file using a 
command of the make  ```[board]_defconfig``` type.

===============================================================
-- EXAMPLE for zybo
===============================================================
(An equivalent example for the toolchain of QEMU ARM Versatile PB evaluation board is available in page 65 of the MELP book.

```bash    
# Once that your toolchain have been created with crosstool-ng
# Add a toolchain created using CrosstoolNG to your path
# and export ARCH and CROSS_COMPILE variables ready to 
# compile U-Boot, Linux, Busybox and anything else using
# the Kconfig/Kbuild scripts
# For the ZYBO development board

PATH=${HOME}/x-tools/arm-cortexa9_neon-linux-gnueabihf/bin/:$PATH
export CROSS_COMPILE=arm-cortexa9_neon-linux-gnueabihf-
export ARCH=arm
```

# Before three lines are saved in the file 
```set-path-arm-cortexa9_neon-linux-gnueabihf``` of this directory.

To build U-Boot, call:

```bash
source <path to script directory>/set-path-arm-cortexa9_neon-linux-gnueabihf
## use that toolchain to make with the specific u-boot target _defconfig
## file
make distclean
make xilinx_zynq_virt_defconfig
export DEVICE_TREE="zynq-zybo"
make
```

