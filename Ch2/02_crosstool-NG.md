# Installing crosstool-NG 
Tool set to build customized toolchains

## Once that the requirements are covered:
- create a new directory to manage your installation. 
```bash 
mkdir crosstool-installation
cd crosstool-installation
git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng
```

## Check for the lastes version of crosstool in the repo

```bash
git branch --show-current
git describe --tags --always
git log -1 --oneline
git fetch --tags
git tag --list 'crosstool-ng-*' --sort=version:refname
```

### Change to the lastest stable release or the most reliable for your hardware

```bash
git checkout crosstool-ng-1.29.0
git status
git describe --tags --always
```
## Installation 
Configure and install, use --prefix={PWD} to do not install on your
current directory, this avoids the requirement of permisions 
to execute or modify your installation

```bash
./bootstrap
./configure --prefix=${PWD}
make
make install 
```
## How to use CrossTool-NG?

To access to the crosstool-ng menu: 

```bash
cd /home/j2meg/Documentos/EmbeddedLinux/Experiments/SwInstallers/crosstool-installation/crosstool-ng
bin/ct-ng
```

To explore the samples and display data for an specific architecture. 

```bash
bin/ct-ng list-samples
bin/ct-ng show-arm-cortex_a8-linux-gnueabi
```
Example of Output

```bash
bin/ct-ng show-arm-cortex_a8-linux-gnueabi
[L...]   arm-cortex_a8-linux-gnueabi
    Languages       : C,C++
    OS              : linux-7.1
    Binutils        : binutils-2.47
    Compiler        : gcc-16.2.0
    Linkers         :
    C library       : glibc-2.44
    Debug tools     : duma-2_5_21 gdb-17.2 ltrace-0.7.3 strace-7.1
    Companion libs  : expat-2.7.1 gettext-0.26 gmp-6.3.0 isl-0.27 libelf-0.8.13 libiconv-1.18 mpc-1.3.1 mpfr-4.2.2 ncurses-6.5 zlib-1.3.1 zstd-1.5.7
    Companion tools :
```
-- Start to correct from here --
--To chose and to configure your owm toolchain: 
bin/ct-ng arm-cortex_a8-linux-gnueabi
bin/ct-ng menuconfig

--build the toolchain 
bin/ct-ng build


-- To create a new toolchain for other architecture
bin/ct-ng distclean
bin/ct-ng arm-unknown-linux-gnueabi 

-change the configuration on menuconfic
-on Paths and misc options > disable Render the toolchain readonly 

bin/ct-ng build

# Once that your toolchain have been created

# Add a toolchain created using CrosstoolNG to your path
# and export ARCH and CROSS_COMPILE variables ready to 
# compile U-Boot, Linux, Busybox and anything else using
# the Kconfig/Kbuild scripts

# Chris Simmonds, chris@2net.co.uk

PATH=${HOME}/x-tools/arm-unknown-linux-gnueabi/bin/:$PATH
export CROSS_COMPILE=arm-unknown-linux-gnueabi-
export ARCH=arm
