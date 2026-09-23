# Other ways to transferring fs to ZYBO

Reference (```MELP third edition | pages 141-159)

In previous documents, we succesfully boot Linux with a RYO Filesystem using
a standalone initramfs. 

On this document we will explore other ways to transfer our filesystem to ZYBO 
Such as:
- Building an ```initramfs into the kernel image``` 
- Building an ```initramfs using a device table```

We also will cover important tasks to succesfully use our new linux image into 
our target device:

- Build the init program 
- Starting a daemon process 
- Configuring user accounts
- Managing device nodes
- Configuring network connections
- And mounting our filesystem using NFS


## Building an initramfs into the kernel image

This is useful for targets that does not have the abillity to load an ```initramfs``` as a standalone file. 
To address this problem, linux can incorporate the filesystem into the kernel image. 

**Note**: This process implies to rebuild our kernel system, **take care of to save a copy (backup) of your previous configurations or documentation before make other changes on your system**.
backup your configurations with the next commands 

```bash
cd <Linux repository path>
cd linux-stable
cp .config .config.zybo-working
make savedefconfig
cp defconfig defconfig.zybo-working
```

save ```.config.zybo-working``` and ```defconfig.zybo-working``` in a safe place. 

To do it: 
- Change the kernel configuration, setting ```CONFIG_INITRAMFS_SOURCE``` to the full path of the ```cpio```  file that we created in previous tutorial. 
    - Using ```menuconfig```, go to **General setup | Initramfs source file(s)**.
    - Verify your setting with:
```bash
# command
grep 'CONFIG_INITRAMFS_SOURCE' .config
# expected output
CONFIG_INITRAMFS_SOURCE="/media/j2m/Almacenamiento/EmbeddedLinux/Experiments/FileSystem_Generation/initramfs.cpio"
```
- Build the kernel. 
```bash
cd <Path to your linux release extracted files>/linux-stable
# Add our toolchain generated with crosstool-ng for ZYBO
# To our local PATH environment variable
PATH=${HOME}/x-tools/arm-cortexa9_neon-linux-gnueabihf/bin/:$PATH

# Set temporal architecture and cross compiler environment variables to pass to make
export ARCH=arm
export CROSS_COMPILE=arm-cortexa9_neon-linux-gnueabihf-
 
# Just Build the Linux kernel image + the initramfs in the same file
make -j4 zImage
# Modules and DTB are the same than in previous building.
```
- Copy the file ```zImage``` into your micro SD-Card, (in this case there is no uRamdisk file, remove it if you have a backup of it). 
```bash
cp arch/arm/boot/zImage /media/j2m/BOOT/
```
- Boot your ZYBO development board with the next U-Boot prompt commands

```bash
# Open the serial port communication through minicom
minicom -D /dev/ttyUSB1 -b 115200

# once that the boot have been completed, to attend a problem with the DTB file
# Load the Device Tree Blob
fatload mmc 0:1 0x1f00000 zynq-zybo.dtb

# Allocate additional space in the Device Tree Blob.
# U-Boot modifies the FDT before passing it to Linux.
fdt addr 0x1f00000
fdt resize 0x10000

# Load the kernel zImage (with the embedded initramfs) and the device tree blob from the U-boot prompt. 
fatload mmc 0:1 0x2000000 zImage
fatload mmc 0:1 0x1f00000 zynq-zybo.dtb 
# Open a shell prompt
setenv bootargs console=ttyPS0,115200 rdinit=/bin/sh
bootz 0x2000000 - 0x1f00000

# note the dash in the last command to indicate that there is no a 
# standalone initramfs loaded as other memory block.
``` 
The communication log that documents this interaction is storaged in 
```Ch5/reference_files/busybox_building_log.txt```. 


The problem with this configuration is that **You must to**
**regeneratethe cpio file each time you change the contents**
**of the root filesystem and then, rebuild the kernel**

## Build an initramfs using a device table 

A device table is  a textfile that lists the files, directories, device nodes and links that go into an archive or filesystem. 
It allows to create entries in te archive file tat are owned by the ```root``` user or any other UID, without having root privileges yourself. 

You can create device nodes without root privileges. The archive is just a data file, the content is expanded at boot time and real files and directories get created using the attributes you have specified. 

To enable the use of device table when creating an ```initramfs```, you have to write the device table file and then point ```CONFIG_INITRAM_FS``` at it. 
For more details about **device tables** refeer to ```page 142-143``` of our reference book. 

To create an ```initramfs device table``` from scratch, previous versions of Linux kernel had a **Linux kernel script** ```usr/gen_initramfs_list.sh``` which creates a device table from a given directory. 

In our case (working on ```linux-6.18.52```) these file have been **deprecated**, but we can to use an scritp to create the device table (as intermediate step) for our initramfs from the ```rootfs``` directory, change the ownership to user and group ID 0, use the next commands:

```bash 
cd <path to linux repository>
cd linux-stable
# Command to create the device table and modify the UID and GID
bash usr/gen_initramfs.sh -u 1000 -g 1000 <path to rootfs> > initramfs.cpio
# In my case:

bash usr/gen_initramfs.sh -u 1000 -g 1000 /home/j2meg/Documentos/EmbeddedLinux/Experiments/SwInstallers/Filesystem_generation/rootfs > /home/j2meg/Documentos/EmbeddedLinux/Experiments/SwInstallers/Filesystem_generation/initramfs.cpio
``` 

**note**: the script only works with a ```bash``` shell.
The output can be compressed and transferred to the target device in the same way that we did in the ```Ch5/04_Transfer_Filesystem_to_Target.md```

```bash
# comprise the initramfs.cpio file 
gzip -k   initramfs.cpio
# Use one of the previous methods to load the filesystem in U-Boot 
# as a standalone file 
# or embedded in the kernel zImage file
```

**An interesting task at this point is to review, usr/gen_initramfs.sh file and try to extract the intermediate device table or device list, to explore its structure.**
By now, that homework is out of the scope of this repository, but it can change in some time. 

Other altenative way to create our ramdisk is to use **initrd** but, this method, is also out of the scope of this document. 
