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
-- Emulating an instance 

qemu-system-arm -machine vexpress-a9 -m 256M -drive file=rootfs.ext4,sd -net nic -net use -kernel zImage -dtb vexpress- v2p-ca9.dtb -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic
