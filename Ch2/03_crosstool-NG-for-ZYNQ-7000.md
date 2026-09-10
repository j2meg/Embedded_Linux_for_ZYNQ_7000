# Crosstool-NG Configuration for ZYNQ 7000

This document describe configurations to succesfully build a toolchain targeting ZYNQ 7000 boards.

Note: The resulting toolchain is intended for the ARM Processin System (PS)
of the ZYNQ-7000 SoC and will be used throughout the Embedded Linux development
process. 

## Objective 
To build a reproducible cross-compilation toolchain for the 
**ARM Cortex-a9 processors* integrated on Zynq-7000 SoC.

The toolchain will provide the *compiler, linker, C library* and related development utilities. 

#### Following the MELP philosophy
First clean any configuration for any other platform.
For purposes of this implementation  
It is recommendable to list sample configurations with

'''bin/ct-ng list-samples'''

to identify if there are a similar configuration to our desirable
*arm-cortexa9-neon-linux-gnueabihf* ...
In the case of this implementation the best fit for ZYNQ-7000 SoC is the
*arm-cortexa9_neon-linux-gnueabihf* configuration. 
Then we start our toolchain building process. 

```bash
cd <path to your crosstool-ng repository>
bin/ct-ng distclean
# List the configuration for our target 
bin/ct-ng show-arm-cortexa9_neon-linux-gnueabihf
```
Example output:
```bash
[L..X] arm-cortexa9_neon-linux-gnueabihf 
Languages : C,C++ 
OS : linux-7.1 
Binutils : binutils-2.47 
Compiler : gcc-16.2.0 
Linkers : 
C library : glibc-2.44 
Debug tools : gdb-17.2 
Companion libs : expat-2.7.1 gettext-0.26 gmp-6.3.0 isl-0.27 libiconv-1.18 
                 mpc-1.3.1 mpfr-4.2.2 ncurses-6.5 zlib-1.3.1 zstd-1.5.7 
Companion tools :
```
Verify configuration 
```bash
bin/ct-ng show-tuple
# Desirable output 
# arm-cortexa9_neon-linux-gnueabihf

```

Following book recommendation

```
# Choose our sample configuration
bin/ct-ng arm-cortexa9_neon-linux-gnueabihf
```

**change the configuration on menuconfic
on Paths and misc options > disable Render the toolchain readonly 
 This allows to modify internal libraries and configurations on your final 
 toolchain**

```bash
bin/ct-ng menuconfig
# Disable the Render the toolchain readonly field
```

#### Start the building process for our toolchain
```bash
bin/ct-ng build
``` 

#### Final Configuration
Once that your toolchain have been created

```bash
#Add a toolchain created using CrosstoolNG to your path
# and export ARCH and CROSS_COMPILE variables ready to 
# compile U-Boot, Linux, Busybox and anything else using
# the Kconfig/Kbuild scripts

PATH=${HOME}/x-tools/arm-unknown-linux-gnueabi/bin/:$PATH
export CROSS_COMPILE=arm-unknown-linux-gnueabi-
export ARCH=arm
```

