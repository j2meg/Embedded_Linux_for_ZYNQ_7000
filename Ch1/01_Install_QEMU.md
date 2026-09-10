# Installing QEMU

*Web site* 
https://www.qemu.org/download/

####on Ubuntu
```
sudo apt-get install qemu-system
```

####on Fedora: 
```
sudo dnf install @virtualization #Install qemu
sudo dnf install qemu-system-arm # Install qemu support for arm architectures
```

Verify installation 
```
qemu-system-arm --version
# current version for ubuntu Ubuntu 22.04.5 LTS: 6.2.0
# current version for Fedora Linux 44 (Xfce): 10.2.2
```


#### List the available hardware emulation architectures
```
qemu-system-arm -machine help
```
For purposes of this implementation we are looking for compatibility with 

```xilinx-zynq-a9       Xilinx Zynq Platform Baseboard for Cortex-A9```

A direct command to identify compatibility 

```qemu-system-arm -machine help | grep -Ei 'xilinx|zynq' ```

## Emulate an ARM Xilinx-zynq-a9 instance 
- to add...
