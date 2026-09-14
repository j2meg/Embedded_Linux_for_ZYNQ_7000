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

### Manual Kernel Boot and expected output. 

The kernel image and Device Tree were copied to the FAT boot
partition of the SD card.

From the U-Boot console, both files were loaded manually:

fatload mmc 0:1 0x02000000 zImage
fatload mmc 0:1 0x03000000 zynq-zybo.dtb

The kernel was then started with:

bootz 0x02000000 - 0x03000000

Linux 6.18.52 successfully initialized the Zynq platform and
detected the Zybo board, CPU cores, memory, Ethernet controller
and SD card.

The boot process stopped when Linux attempted to mount the root
filesystem because no root filesystem had been provided yet.
Press the ```PS-SRST``` button and you will see the next message on the screen 

