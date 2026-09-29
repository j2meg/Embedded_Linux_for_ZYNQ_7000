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

To implement this part of tutorial, we require
- an Ethernet Cable to connect directelly from host to zybo. 

The benefits of this type of FS mounting are:

- It gives you access to the massive amount of storage for your target on your host machine. 
- You can add debug tools and executables with large symbol tables.
- **Updates made to the root filesystem on the development machine are made available on the target immediately. 
- You can access all the target's log files from the host. 

Firts, lets verify that our kernel allows us to mount the filesystem from NFS. 

```bash 
cd <path to kernel repository>/linux-stable
grep CONFIG_ROOT_NFS .config
# Expected output 
CONFIG_ROOT_NFS=y
``` 
If the variable is enabled, continue with the tutorial, if not, enable it and rebuild the kernel and copy the zimage again to your SD Card. 

## Configurations required. 

The first step is to install and configure an NFS server on your host device. 
The recommendation is to use ```nfs-kernel-server```
To install the package on ubuntu: 

```bash
sudo apt install nfs-kernel-server
```

For fedora: 
```bash 
sudo dnf install nfs-utils
sudo dnf install libnfsidmap
sudo dnf install sssd-nfs-idmap
```

The next step is told to the NFS server which directories are being exported to the network.
This information is saved in the ```/etc/exports``` file of your host machine. 
There is one line for each export. 
Refer to ```page 154``` of our reference book for a extended explanation of the next command. 

To expose the rootfs directory on my machine, I have to add the next line to /etc/export file of my host machine. 

```bash
# open the exports file of the host machine
sudo nano /etc/exports
# add the next line to your file 
/home/j2meg/Documentos/EmbeddedLinux/Experiments/SwInstallers/Filesystem_generation/rootfs *(rw,sync,no_subtree_check,no_root_squash)
# save the changes and exit of the file.
```

* exports the directory to any address on my local network, and the commands between parentheses list the 
permissions and options for the directory sharing, take care of some of them, because can be not secure if you connect your device at 
non local or non secure networks. 
A more secure configuration can be done if you have time to identify your target device IP address and configure it carefully. 
For our objectives it is enough. 

Now, restart your NFS server to pick them up... 
On your host device applye the next interactions

```bash
# Review the NFS server status
systemctl status nfs-server
○ nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: inactive (dead)
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
```
```bash 
# Reload the exportations file 
sudo exportfs -rav
# verify
sudo exportfs -v
showmount -e localhost

# Start the NFS server
sudo systemctl enable --now nfs-server
# expected output 
Created symlink '/etc/systemd/system/multi-user.target.wants/nfs-server.service' → '/usr/lib/systemd/system/nfs-server.service'.
# Verify the status again 
systemctl status nfs-server
sudo exportfs -rav
sudo exportfs -v
# Expected output 
nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
             /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Mon 2026-09-28 13:23:12 CST; 51s ago
 Invocation: ebfbfd11891341c18b8076dbd9bbfd4c
       Docs: man:rpc.nfsd(8)
             man:exportfs(8)
    Process: 160473 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 160475 ExecStart=/bin/sh -c /usr/sbin/nfsdctl autostart || /usr/sbin/rpc.nfsd (code=exited, status=0/S>
    Process: 160505 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (c>
   Main PID: 160505 (code=exited, status=0/SUCCESS)
   Mem peak: 2.4M
        CPU: 52ms

sep 28 13:23:11 fedora-vaio systemd[1]: Starting nfs-server.service - NFS server and services...
sep 28 13:23:12 fedora-vaio systemd[1]: Finished nfs-server.service - NFS server and services.

## In Fedora it is needed to add a Firewalld configuration 
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --add-service=rpc-bind
sudo firewall-cmd --zone=public --add-service=mountd
# with Zybo connected 
# Check the firewall zone for enp3s0
sudo firewall-cmd --get-zone-of-interface=enp3s0
# Expected output 
no zone
# verify the active firewall zones
sudo firewall-cmd --get-active-zones
public (default)
  interfaces: wlp1s0
# wlp1s0 is our wifi hardware device
# we have to move enp3s0 to the public zone to allow the traffic Zybo to host
sudo firewall-cmd --permanent --zone=public --add-interface=enp3s0
sudo firewall-cmd --reload
# Verify the changes with
sudo firewall-cmd --get-active-zones
# Expected output.
public (default)
  interfaces: enp3s0 wlp1s0

```

Till now, we configurate the NFS-server. 
It is time to connect with ZYBO.

In Fedora, we need to identify the Ethernet interface attached to zybo. 

Use the command ```ip addr```

```bash
# Expected output 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 30:f9:ed:b2:0d:1d brd ff:ff:ff:ff:ff:ff
    altname enx30f9edb20d1d
    inet 192.168.10.1/24 scope global enp3s0
       valid_lft forever preferred_lft forever
3: wlp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ca:79:e1:2e:b8:86 brd ff:ff:ff:ff:ff:ff permaddr 08:ed:b9:be:42:8d
    altname wlx08edb9be428d
    inet 192.168.1.219/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp1s0
       valid_lft 30545sec preferred_lft 30545sec
    inet6 2806:10a6:d:4824:dba:12d2:f487:b6a1/64 scope global dynamic noprefixroute 
       valid_lft 4294750858sec preferred_lft 4294750858sec
    inet6 2806:10a6:d:aa0b:5d20:932b:3618:a7c/64 scope global dynamic noprefixroute 
       valid_lft 4294246011sec preferred_lft 4294246011sec
    inet6 fe80::51f:ff5d:1da3:c7cf/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
En este caso la interfaz conectada a ZYBO a traves del ethernet es ```enp3s0```.
Y la IP del host al conectarse a ZYBO es ```192.168.10.1```, furthermore 
when we launch U-Boot on zybo, this have no IP configured yet. 

Then, in the U-Boot prompt of Zybo, connected to host through usb serial communication, but also through 
direct ethernet conection to the host pc. 

we have to execute the next commands:

```bash 
# Configure your network and nfs 
setenv serverip 192.168.10.1
setenv ipaddr 192.168.10.101
## npath must be the variable that storages our stagging directory
setenv npath /home/j2meg/Documentos/EmbeddedLinux/Experiments/SwInstallers/Filesystem_generation/rootfs
setenv bootargs console=ttyPS0,115200 root=/dev/nfs rw nfsroot=${serverip}:${npath} ip=${ipaddr}

# Load the image, dtb and boot
# Load the Device Tree Blob
fatload mmc 0:1 0x1f00000 zynq-zybo.dtb
# Allocate additional space in the Device Tree Blob.
# U-Boot modifies the FDT before passing it to Linux.
fdt addr 0x1f00000
fdt resize 0x10000
# Commands to load the kernel, device tree and filesystem from bootloader 
fatload mmc 0:1 0x2000000 zImage
bootz 0x2000000 - 0x1f00000
```

With this sequence you have to start your system with an NFS Filesystem
If you achieve arrive till this step!!! 

# Congrats, you have a development environment to experiment with ZYBO!! 
A reference of the communication log for our system running thorug NFS is available at ```Ch5/reference_files/NSF_bootLog.txt```

# The last part of this tutorial consides to use TFTP to load the kernel and dtb files

That part will be configurated soon...

