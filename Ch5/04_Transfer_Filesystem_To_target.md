# Transferring the root filesystem to target

Reference MELP Book Third edition | ```Page 139```

For a discussion on the different ways to transfer a filesystem to 
our target device, there are three methods explained on detail on 
page ```138``` of our reference book. 

For purposes of this tutorial, we will to develop strategies to load it 
from:

- An **initramfs**: or ```ramdisk```, that is an image loadded into the RAM by the bootloader.

- A **Disk image**: This is a copy of the root filesystem formatted and ready to be loaded onto a mass storage device on the target.

- A **Network Filesystem**: An exposition of the staging directory to the network via NFS server and mounted by the target device on the boot. 

On this tutorial we will delving deeper on how to start the root 
filesystem by these three ways. 

## Creating a boot initramfs

An initramfs is a compressed ```cpio``` archive, to achive it, we need to 
configure our kernel with ```CONFIG_BLK_DEV_INITRD```. 

Verify that in the kernel that we prevously built this configuration is enabled 
with the command: 

```bash
grep '^CONFIG_BLK_DEV_INITRD' .config
```

If the output is something similar to 

```bash
CONFIG_BLK_DEV_INITRD=y
```

compatibillity with initramfs is enabled and we can continue with the process, in other case, enable it in the ```.config``` file of the linux kernel building directory, 
and rebuild the project. 

There are three ways to create a boot ramdisk, they are: a ```standalone cpio``` file, a ```cpio``` embedded in the kernel image and, as a ```device table```
 that the kernel procecess as part of the build. 

In this tutorial we will implement the standalone initramfs. 

## Standalone initramfs
### The next commands create the archive, compresses it and adds a U-Boot header ready to load onto the target

```bash
cd <path to your filesystem directory>
cd rootfs
# comprisse the filesystem on a cpio file
find . | cpio -H newc -ov --owner root:root > ../initramfs.cpio
# note --owner root:root makes that everithing in the cpio file archive have a UID and GID of 0.
cd .. 
# comprisse again the cpio file into a .gz file
gzip initramfs.cpio
# add the U-Boot header for loading onto the target
mkimage -A arm -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
```

Expected final output:

```bash
 mkimage -A arm -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
Image Name:   
Created:      Sat Sep 19 19:02:23 2026
Image Type:   ARM Linux RAMDisk Image (gzip compressed)
Data Size:    1771689 Bytes = 1730.17 KiB = 1.69 MiB
Load Address: 00000000
Entry Point:  00000000
```

### Booting the initramfs 
At this point we require the SD card formatted at Ch4 with the next files in the ```BOOT``` partition.

```bash
tree /media/j2m/BOOT/
/media/j2m/BOOT/
├── boot.bin
├── u-boot.img
├── zImage
└── zynq-zybo.dtb

0 directories, 4 files

```

The next step is to copy the ```uRamdisk``` file to the boot partition of the micro SD Card and use it to boot the ZYBO development board. 
Boot it as in the Chapter 4 of this repository and enter the next commands: 

```bash
# command to start minicom serial communication
minicom -D /dev/ttyUSB1 -b 115200
```
To identify the directions to mount our boot files, we require support of the next U-boot shell commands

```bash
# Review information about the board and memory map 
bdinfo 
# expected output
boot_params = 0x00000000
DRAM bank   = 0x00000000
-> start    = 0x00000000
-> size     = 0x20000000
flashstart  = 0x00000000
flashsize   = 0x00000000
flashoffset = 0x00000000
baudrate    = 115200 bps
relocaddr   = 0x1feef000
reloc off   = 0x1beef000
Build       = 32-bit
current eth = ethernet@e000b000
ethaddr     = 02:71:52:f2:40:0e
IP addr     = <NULL>
fdt_blob    = 0x1eac8e60
lmb_dump_all:
 memory.count = 0x1
 memory[0]      [0x0-0x1fffffff], 0x20000000 bytes, flags: none
 reserved.count = 0x2
 reserved[0]    [0x1dabf000-0x1dac7fff], 0x9000 bytes, flags: no-notify, e
 reserved[1]    [0x1dac8e40-0x1fffffff], 0x25371c0 bytes, flags: no-overwe
devicetree  = board
arch_number = 0x00000000
TLB addr    = 0x1fff0000
irq_sp      = 0x1eac8e50
sp start    = 0x1eac8e40
ARM frequency = 650 MHz
DSP frequency = 0 MHz
DDR frequency = 525 MHz
Early malloc usage: 714 / 800

# Get recommended addres to mount Kernel image
Zynq> printenv kernel_addr_r
## Output
kernel_addr_r=0x2000000
# Get recommended addres to mount initramfs (uRamdisk)
Zynq> printenv ramdisk_addr_r
## Output
ramdisk_addr_r=0x3100000
# Get recommended addres to mount device tree
Zynq> printenv fdt_addr_r
## Output
fdt_addr_r=0x1f00000
```
With that information, the commands to load our kernel files are:
```bash
# Load the Device Tree Blob
fatload mmc 0:1 0x1f00000 zynq-zybo.dtb

# Allocate additional space in the Device Tree Blob.
# U-Boot modifies the FDT before passing it to Linux.
fdt addr 0x1f00000
fdt resize 0x10000
# Commands to load the kernel, device tree and filesystem from bootloader 
fatload mmc 0:1 0x2000000 zImage
fatload mmc 0:1 0x1f00000 zynq-zybo.dtb 
fatload mmc 0:1 0x3100000 uRamdisk 
setenv bootargs console=ttyPS0,115200 rdinit=/bin/sh
bootz 0x2000000 0x3100000 0x1f00000
```

After execute those commands, our system will be loaded, and it will show something like: 

```bash
Hit any key to stop autoboot: 2Hit any key to stop autoboot: 0
Zynq> fatload mmc 0:1 0x1f00000 zynq-zybo.dtb
11087 bytes read in 12 ms (901.4 KiB/s)
Zynq> fdt addr 0x1f00000
Working FDT set to 1f00000
Zynq> fdt resize 0x10000
Zynq> fatload mmc 0:1 0x2000000 zImage
12046848 bytes read in 661 ms (17.4 MiB/s)
Zynq> fatload mmc 0:1 0x1f00000 zynq-zybo.dtb 
11087 bytes read in 12 ms (901.4 KiB/s)
Zynq> fatload mmc 0:1 0x3100000 uRamdisk 
1771753 bytes read in 106 ms (15.9 MiB/s)
Zynq> setenv bootargs console=ttyPS0,115200 rdinit=/bin/sh
Zynq> bootz 0x2000000 0x3100000 0x1f00000
Kernel image @ 0x2000000 [ 0x000000 - 0xb7d200 ]
## Loading init Ramdisk from Legacy Image at 03100000 ...
   Image Name:   
   Created:      2026-09-20   1:02:23 UTC
   Image Type:   ARM Linux RAMDisk Image (gzip compressed)
   Data Size:    1771689 Bytes = 1.7 MiB
   Load Address: 00000000
   Entry Point:  00000000
   Verifying Checksum ... OK
## Flattened Device Tree blob at 01f00000
   Booting using the fdt blob at 0x1f00000
Working FDT set to 1f00000
   Loading Ramdisk to 1d90a000, end 1daba8a9 ... OK
   Loading Device Tree to 1d904000, end 1d909b4e ... OK
Working FDT set to 1d904000

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 6.18.52 (j2m@j2m-Aspire-A515-51) (arm-cortexa9_neon-linux-gnueabihf-gcc (crosstool-NG 1.29.0) 16.2.0, GNU ld (crosstool-NG 1.29.0) 2.47.20260726) #1 SMP Mon Sep 14 14:00:15 CST 2026
[    0.000000] CPU: ARMv7 Processor [413fc090] revision 0 (ARMv7), cr=18c5387d
[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
[    0.000000] OF: fdt: Machine model: Digilent Zybo board
[    0.000000] Memory policy: Data cache writealloc
[    0.000000] efi: UEFI not found.
[    0.000000] cma: Reserved 64 MiB at 0x19800000
[    0.000000] Zone ranges:
[    0.000000]   DMA      [mem 0x0000000000000000-0x000000001fffffff]
[    0.000000]   Normal   empty
[    0.000000]   HighMem  empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000000000000-0x000000001fffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000000000000-0x000000001fffffff]

... 
...
...
...

/bin/sh: can't access tty; job control turned off
~ # 

```
Last line shows a command shell enabled to execute shell commands though busybox. 

For a Full reference about the load of the kernel and mount of the filesystem 
with uRamDisk, see ```Ch5/reference_files/minicom_log1.txt``` on this repository. 

We will close this tutorial at this step. 

## Congrats!!! you have been mounted a small functional Linux based OS on ZYBO!!
## next tutorials on this chapter will show you alternative methods to load filesystem. 
## Play with it and enjoy a lot! 

