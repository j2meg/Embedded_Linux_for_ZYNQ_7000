# Booting the Kernel in ZYBO

Reference ```MELP Book | page 108```

Booting Linux is highly device dependent. 
There are examples in our reference book to boot on Raspberry-Pi 4,
Beagle Bone Black and QEMU, on this document, we develop the same process
for ZYBO. 

## Requirements
- A partitioned micro-SD Card with the Bootloader generated on Ch3 (see ```Ch3/07_ZYBO_First_Boot.md```).
- ZYBO evaluation board. 
- A linux based host system
- USB wire to attach ZYBO to the host device

## Copying the Image files to SD card. 
With our kernel compiled on host pc. 
- Insert the micro-SD in your host pc sd slot.
- Copy the next files from your ```linux-stable``` directory to your ```SD-Card Boot partition```
    - ```arch/arm/boot/zImage```
    - ```arch/arm/boot/<Correct to the zybo path>```
- Unmount the SD-card from the host device and insert it on the ZYBO board. 

Code to copy Kernel files to SD-Card
```bash
cd <path to linux repository>/linux-stable
cp arch/arm/boot/zImage /media/j2m/BOOT/
cp arch/arm/boot/dts/xilinx/zynq-zybo.dtb /media/j2m/BOOT/


```
Next comments help you to configure ZYBO to Boot.
## ZYBO Hardware configuration and boot
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

## Manual Kernel Boot and expected output. 

Once that your system is turned on and communication was established throug ```minicom```. 
- Press the ```PS-SRST``` button and you will see the next message on the screen 

```bash
Welcome to minicom 2.8                                                                                       
                                                                                                             
OPTIONS: I18n                                                                                                
Port /dev/ttyUSB1, 15:16:00                                                                                  
                                                                                                             
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

Warning: ethernet@e000b000 (eth0) using random MAC address - 92:51:f9:57:ff:d0
eth0: ethernet@e000b000
Hit any key to stop autoboot: 5

```

This indicates that the u-boot sequence of the ```Second Program Loader (SPL)```
Is working and ready to load a kernel operative system. 
Furthermore, at this moment we do not have a defined sequence to load our ```zImage``` and ```zynq-zybo.dtb``` files. 
Then we have to load them by hand. 

### Process to load kernel files
- Before that the counter comes to zero: 

```bash
Warning: ethernet@e000b000 (eth0) using random MAC address - 92:51:f9:57:ff:d0
eth0: ethernet@e000b000
Hit any key to stop autoboot: 5
```

Hit any key to stop the system autoboot. 
- from the mincom serial console, list the files in our SD Card First Partition (BOOT). 

```bash
# Zynq Command 
Zynq> fatls mmc 0:1
# Expected Output
   128772   boot.bin
  1351968   u-boot.img
 12046848   zImage
    11087   zynq-zybo.dtb
```

Here you must to see our previously copied boot files. 

- next, load into our U-boot shell the ```zImage``` kernel file

```bash
# Load z Image command
Zynq> fatload mmc 0:1 0x02000000 zImage
# expected output
12046848 bytes read in 660 ms (17.4 MiB/s)
```

- now, load the zybo device tree binary file ```zynq-zybo.dtb```.

```bash
# Load device tree command
Zynq> fatload mmc 0:1 0x03000000 zynq-zybo.dtb
# Expected output
11087 bytes read in 12 ms (901.4 KiB/s)
```

- Finally, boot our kernel system from the serial command  interface.

```bash
# Command to boot from the loadded kernel address.
Zynq> bootz 0x02000000 - 0x03000000
# Expected output
Kernel image @ 0x2000000 [ 0x000000 - 0xb7d200 ]
## Flattened Device Tree blob at 03000000
   Booting using the fdt blob at 0x3000000
Working FDT set to 3000000
   Loading Device Tree to 1dab5000, end 1dabab4e ... OK
Working FDT set to 1dab5000

Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 6.18.52 (j2m@j2m-Aspire-A515-51) (arm-cortexa9_neon-linux-gnueabihf-gcc (crosstool-NG 1.29.0) 16.2.0, GNU ld (crosstool-6
[    0.000000] CPU: ARMv7 Processor [413fc090] revision 0 (ARMv7), cr=18c5387d
[    0.000000] CPU: PIPT / VIPT nonalia...
```


This indicates that Linux 6.18.52 was successfully initialized the Zynq platform and
detected the Zybo board, CPU cores, memory, Ethernet controller
and SD card.

The boot process stopped when Linux attempted to mount the root
filesystem because no root filesystem had been provided yet. 

Communication finishes with a very descriptive ```Kernel Panic Message```

```bash
[    2.630351] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
[    2.638637] CPU: 1 UID: 0 PID: 1 Comm: swapper/0 Not tainted 6.18.52 #1 NONE 
[    2.645798] Hardware name: Xilinx Zynq Platform
[    2.650335] Call trace: 
[    2.650351]  unwind_backtrace from show_stack+0x10/0x14
[    2.658149]  show_stack from dump_stack_lvl+0x54/0x68
[    2.663244]  dump_stack_lvl from vpanic+0xc4/0x2ec
[    2.668070]  vpanic from __do_trace_suspend_resume+0x0/0x48
[    2.673747] ---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0) ]---
```

To see our complete interaction with the platform see at the document on this repository 
```Ch4/reference_files/Linux_booting_ZYBO_serial_output.txt```.

## Congrats, there is loaded a Embedded Linux Kernel.
## Next Step: Create and load the Linux File System.
