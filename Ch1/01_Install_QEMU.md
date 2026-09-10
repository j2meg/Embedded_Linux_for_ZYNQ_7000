# Installing QEMU

*Web site* 
https://www.qemu.org/download/
```
#on Ubuntu
sudo apt-get install qemu-system
#on Fedora: 
dnf install @virtualization

# Verify installation 
qemu-system-arm --version
# List the available hardware emulation architectures
qemu-system-arm -machine help
```
For purposes of this implementation we are looking for compatibility with 

```xilinx-zynq-a9       Xilinx Zynq Platform Baseboard for Cortex-A9```

A direct command to identify compatibility 

```qemu-system-arm -machine help | grep -Ei 'xilinx|zynq' ```

## Emulate an ARM Xilinx-zynq-a9 instance 
- to add...
