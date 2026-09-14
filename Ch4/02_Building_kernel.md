# Building Embedded Linux Kernel 

Reference MELP Third Edition-Page 91.

## Choose a Kernel Release to Build.

The first previous step to build a kernel for your target device, is to chose a reliable release of 
Linux Kernel in agreement with your development or production objetvies.

A deep description on this topic is covered on our reference book at ```Page 88 ```.

For purposes of the present project, we choose the ```last long term release``` 
till the moment that I write this document. 

Steps to get it are:

- Go through https://www.kernel.org/ and identify stable and long-term releases.
- Chose one of them depending on your interests. 
- Fetch and extract your releease tarball 

At this moment ```longterm: 	6.18.52``` is a reliable release

To fetch and extract our kernel release use the following commands. 

```bash 
## Clone Linux kernel stable release
cd <Path to save your download>
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.18.52.tar.xz
tar xf linux-6.18.52.tar.xz
mv linux-6.18.52 linux-stable
``` 

Main directories for our purposes on the extracted files are:

- ```arch```
- ```Documentation ```
- ```drivers```
- ```fs```
- ```include```
- ```init```
- ```kernel```
- ```mm```
- ```net```
- ```scripts```
- ```tools ```

names are very descriptive, but for a better explanation look at: ```Page 92, MELP Book```

## Linux Kenel Compilation Process

The list of commands used to build Linux Kernel for Zybo are listed next. 

```bash
cd <Path to your linux release extracted files>/linux-stable
# Add our toolchain generated with crosstool-ng for ZYBO
# To our local PATH environment variable
PATH=${HOME}/x-tools/arm-cortexa9_neon-linux-gnueabihf/bin/:$PATH

# Set temporal architecture and cross compiler environment variables to pass to make
export ARCH=arm
export CROSS_COMPILE=arm-cortexa9_neon-linux-gnueabihf-

# Bring our repository to a clean state using make mrproper
make mrproper

# Configure the kernel for ARMv7 platforms
# This configuration includes support for Xilinx Zynq
make multi_v7_defconfig

# Build the Linux kernel
make -j4 zImage

# Build kernel modules
make -j4 modules

# Build the Device Tree for the Digilent Zybo
make -j4 xilinx/zynq-zybo.dtb

```

## Expected outputs
```bash
# Command
make multi_v7_defconfig
# Output
  HOSTCC  scripts/basic/fixdep
  HOSTCC  scripts/kconfig/conf.o
  HOSTCC  scripts/kconfig/confdata.o
  HOSTCC  scripts/kconfig/expr.o
  LEX     scripts/kconfig/lexer.lex.c
  YACC    scripts/kconfig/parser.tab.[ch]
  HOSTCC  scripts/kconfig/lexer.lex.o
  HOSTCC  scripts/kconfig/menu.o
  HOSTCC  scripts/kconfig/parser.tab.o
  HOSTCC  scripts/kconfig/preprocess.o
  HOSTCC  scripts/kconfig/symbol.o
  HOSTCC  scripts/kconfig/util.o
  HOSTLD  scripts/kconfig/conf
#
# configuration written to .config
#

```
