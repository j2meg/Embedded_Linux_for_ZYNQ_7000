# ZYBO First Boot
## Objectives
1. To Review the U-Boot compilation products. 
2. To format the Boot SD-Card for ZYBO
3. To implement and analyze the first Boot on ZYBO. 

## U-Boot compilation products. 

Once that U-Boot have compiled our bootloader using the toolchain gnerated in the 
files of chapter 2 we can to find the next compilation products in our U-Boot directory. 

```bash
4.0K drwxr-xr-x 1 j2m j2m 4.0K sep 11 11:59 spl
144K -rwxr-xr-x 1 j2m j2m 143K sep 11 11:58 System.map
4.0K drwxr-xr-x 1 j2m j2m 4.0K sep 11 00:31 test
 44K drwxr-xr-x 1 j2m j2m  44K sep 11 11:55 tools
8.4M -rwxr-xr-x 1 j2m j2m 8.5M sep 11 11:58 u-boot
1.1M -rwxr-xr-x 1 j2m j2m 1.1M sep 11 11:58 u-boot.bin
 24K -rwxr-xr-x 1 j2m j2m  22K sep 11 11:40 u-boot.cfg
 16K -rwxr-xr-x 1 j2m j2m  15K sep 11 11:59 u-boot.dtb
1.1M -rwxr-xr-x 1 j2m j2m 1.1M sep 11 11:58 u-boot-dtb.bin
1.3M -rwxr-xr-x 1 j2m j2m 1.3M sep 11 11:59 u-boot-dtb.img
1.1M -rwxr-xr-x 1 j2m j2m 1.1M sep 11 11:59 u-boot.elf
 512 -rwxr-xr-x 1 j2m j2m   70 sep 11 11:59 u-boot-elf.lds
1.1M -rwxr-xr-x 1 j2m j2m 1.1M sep 11 11:59 u-boot-elf.o
1.3M -rwxr-xr-x 1 j2m j2m 1.3M sep 11 11:59 u-boot.img
4.0K -rwxr-xr-x 1 j2m j2m 1.4K sep 11 11:58 u-boot.lds
1.6M -rwxr-xr-x 1 j2m j2m 1.6M sep 11 11:58 u-boot.map
1.1M -rwxr-xr-x 1 j2m j2m 1.1M sep 11 11:58 u-boot-nodtb.bin
3.1M -rwxr-xr-x 1 j2m j2m 3.1M sep 11 11:58 u-boot.srec
292K -rwxr-xr-x 1 j2m j2m 290K sep 11 11:58 u-boot.sym
 ```
A description of different extension files such as ```u-boot, u-boot.bin, .cfg, .dtb, .elf, etc.``` refeer to pag. 65 of MELP. 
An interesting distinction for  ```spl/``` directory is that it contains the **Secondary Program Loader (SPL)** and its 
associated build products. 

For the ZYNQ boot flow, the SPL is responsible for the initial SW set up and for loading the ```full U-Boot image```.

To boot on ZYBO important files that will be loaded into our Boot SD-Card are: 
```bash
ls -lh spl/boot.bin u-boot.img
-rwxr-xr-x 1 j2m j2m 126K sep 12 13:45 spl/boot.bin
-rwxr-xr-x 1 j2m j2m 1.3M sep 12 13:45 u-boot.img

```
Remaining files are useful for inspection, debugging, development, or other boot configurations. 

To explore the files content use the CLI tool ```mkimage```contained in your linux software package ```u-boot-tools``` or compilable from the 
U-boot software tree. 

With that we can see the next output for our implementation files. 
```bash
/u-boot$ mkimage -l spl/boot.bin 
Image Type   : Xilinx Zynq Boot Image support
Image Offset : 0x000008c0
Image Size   : 128772 bytes (128772 bytes packed)
Image Load   : 0x00000000
User Field   : 0x00000000
Checksum     : 0xfd166579

/u-boot$ mkimage -l u-boot.img 
FIT description: Firmware image with one or more FDT blobs
Created:         Sat Sep 12 13:45:21 2026
 Image 0 (firmware-1)
  Description:  U-Boot 2026.07 for zynq board
  Created:      Sat Sep 12 13:45:21 2026
  Type:         Firmware
  Compression:  uncompressed
  Data Size:    1068960 Bytes = 1043.91 KiB = 1.02 MiB
  Architecture: ARM
  OS:           U-Boot
  Load Address: 0x04000000
  Hash algo:    crc32
  Hash value:   ec6470d0
 Image 1 (fdt-1)
  Description:  zynq-zybo
  Created:      Sat Sep 12 13:45:21 2026
  Type:         Flat Device Tree
  Compression:  uncompressed
```

- ```spl/boot.bin```: Zynq boot image containing the U-Boot SPL, 
packaged in the format expected by the Zynq BootROM. It provides the 
first software stage executed after power-on and is responsible for 
loading the full U-Boot image.
- ```u-boot.img```: U-Boot FIT image containing the full U-Boot 
firmware and the associated Device Tree data. 
It is the image loaded by the SPL to start the complete 
U-Boot environment.

# SD Card Format and files load for ZYBO 
To boot in ZYBO, we require an SD Card with the next partition structure taken from 

https://github.com/jinchenglee/zybo_linux_setup_doc

- **BOOT**: 512 MiB FAT32 partition used to store the boot files required by the Zynq boot process.
- **ROOT_FS**: Remaining space formatted as EXT4, intended for the Linux root filesystem.

An adapted version of the formatting script of MELP, is avaliable on this repository or at:
https://github.com/j2meg/Embedded_Linux_for_ZYNQ_7000/blob/main/Ch3/util_scripts/format-sdcard.sh

Execute it with:

```bash 
sudo ./format_sd.sh <drive>
```
To identify the sd card device in your computer use the command ```lsblk```
and don't worry, the ``` format_sd.sh``` script has safe ward to avoid to format devices with storage greater to 32 GB, avoiding accidentally to format your local storage system. 

In my case the SD device is named ```mmcblk0``` the command to format it is:

```bash
sudo ./format_sd.sh mmcblk0
```
new partitions are mounted in ```/media/<user>/``` directory
in my case 

```bash 
ls /media/j2m/ -ls
total 16
4 drwxr-xr-x 2 j2m  j2m  4096 dic 31  1969 BOOT
4 drwxr-xr-x 3 root root 4096 sep 11 17:06 ROOT_FS
```
## COPY BOOT FILES INTO SD
Last important step with the SD card is to copy the compiled ```boot.bin``` and 
 ```u-boot.img``` files from your local ```u-boot``` directory into your ```BOOT``` SD card partition.   
this is done with the command

```bash 
cp <local u-boot directory path>/spl/boot.bin /media/<user directory>/BOOT/
cp <local u-boot directory path>/u-boot.img /media/<user directory>/BOOT/
sync
```

in my case 
```bash 
cp spl/boot.bin /media/j2m/BOOT/
cp u-boot.img /media/j2m/BOOT/
sync
```
Once that the copy is done, unmount the SD-card from your PC and go to the next step. 

# ZYBO Hardware configuration and boot
To prepare the ZYBO evaluation board for booting, execute next steps. 
- Turn off the device Power Switch and unplugg from any power source. 
- Introduce the boot SD-Card in your ZYBO SD-Card slot
- Conect the physical board jumpers with the next configuration: 
    - ```JP7``` in  ```USB power supply``` configuration. 
    - ```JP5``` in ```SD boot configuration``` it means (```QSPI and SD```).
- Connect the ZYBO to the PC through an usb wire.  
- Turn On the ```ZYBO POWER SWITCH```

You will see the  ```PG00D``` led blinking. 

In your host PC Open a serial terminal software, in my case  ```minicom``` 
and start communication with your board, in my case: 

```bash 
minicom -D /dev/ttyUSB1 -b 115200
```

### First Boot and expected output. 

Press the ```PS-SRST``` button and you will see the next message on the screen 

```bash 
Welcome to minicom 2.8

OPTIONS: I18n 
Port /dev/ttyUSB1, 14:59:00

Press CTRL-A Z for help on special keys


U-Boot SPL 2026.07 (Sep 11 2026 - 11:55:00 -0600)
Silicon version:        3
Trying to boot from MMC1


U-Boot 2026.07 (Sep 11 2026 - 11:55:00 -0600)

CPU:   Zynq 7z010
Silicon: v3.1
Model: Digilent Zybo board
DRAM:  ECC disabled 512 MiB
Core:  22 devices, 16 uclasses, devicetree: board
Flash: 0 Bytes
NAND:  0 MiB
MMC:   mmc@e0100000: 0
Loading Environment from FAT... *** Error - No Valid Environment Area found
*** Warning - bad env area, using default environment

In:    serial@e0001000
Out:   serial@e0001000
Err:   serial@e0001000
Net:   
ZYNQ GEM: e000b000, mdio bus e000b000, phyaddr 0, interface rgmii-id

Warning: ethernet@e000b000 (eth0) using random MAC address - 2e:a0:10:b9:c5:85
eth0: ethernet@e000b000
Hit any key to stop autoboot: 0

```

This message is the boot sequence working on your ZYBO board, to look for what 
kind of tasks you can develop from serial communication with U-Boot shell **Look at page 69 of MELP book* 
specifically **Using U-Boot** section.

# Congrats, you are succesfully booting your ZYBO board. 
# This is a great step towards your Embedded Linux implementation. 
