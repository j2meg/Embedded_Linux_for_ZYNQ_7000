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

### First Boot and expected output. 

Press the ```PS-SRST``` button and you will see the next message on the screen 

