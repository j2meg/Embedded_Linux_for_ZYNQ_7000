# 07 Booting with ext2 and NFS

Reference (```MELP Third edition | page 152``

## Create a filesystem image with device table. 

As we have a complete filesystem to configure zybo with a small 
userspace. 
Let's to create a different format to load the FS in our system. 

In this case, let's use  the ```ext2``` format. This is usual in managed flash memory such as
our SD-Card. 

Our first step is to get the ```genext2fs``` tool in our host system. 
For ubuntu, get it through: 

```bash
sudo apt install genext2fs
```

In Fedora:
```bash 
sudo dnf install genext2fs
```

The tool uses a device table whose format is descripted in the ```page 152-153``` 
of our reference book. 

A relevant detail of this tool is that we does not have to specify every file
in our filesystem, we just hace to point at ```the staging directory``` and 
list the changes and exceptions we need to make in the final filesystem. 

a conservative example of a device table for our purposes is available at ```Ch5/util_scripts/device-table.txt```
its content is just the next: 

```bash 
/dev d 755 0 0 - - - - -
/dev/null c 666 0 0 1 3 0 0 -
/dev/console c 600 0 0 5 1 0 0 -
```

Once that the device table file is saved, we can use it to generate the ext2 fs. with the next command 

```bash
genext2fs -b 4096 -d <path to your rootfs>/rootfs -D <path to device table>/device-table.txt -U rootfs.ext2
```

Now we can use the rootfs.ext2 file to boout our target system. 

## To load file on Zybo 

we can use the ```dd``` linux command pointing to the second partition of the ZYBO SD-Card. as follows

```bash 
sudo dd if=rootfs.ext2 of=/dev/mmcblk0p2
```

**Please verify the name of your SD-Card in the host name** it might be different and prevent write a different disk and loss important information. 

Then, slot the micro-SD card into the ZYBO development board and set the kernel command to
```root=/dev/mmcblk0p2``` 

The full U-Boot sequence is:

```bash 
# Open a Minicom port communication with zybo
minicom -D /dev/ttyUSB1 -b 115200
```

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
setenv bootargs console=ttyPS0,115200 root=/dev/mmcblk0p2
bootz 0x2000000 - 0x1f00000
```

# Congrats, you have been booted your ZYBO using the updated filesystem 
# from a ext2 file mounted on second partition of your SD Card! 

All the changes that we made before can be tried, such as the network connection, 
sys and proc processes, and the use of devtempfs to manage device nodes. 
A reference of the expected communication betwen your host and zybo during the kernel loading process is 
available at ```Ch5/reference_files/ext2_boot_log.txt```.

# A last part on this tutorial consist on mount the same rootfs filesystem through NFS.

