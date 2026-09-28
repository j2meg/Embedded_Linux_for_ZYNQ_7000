# Complementary Init tasks and network connection configurations.
Through this tutorial we will explore some of the tasks developed 
during the initialization process, some of them configurated in the 
```init``` program or as auxiliary processes. 
some of those tasks are: 
- To Create the etc/init.d/rcS file 
- To Starting a daemon process (syslogd)
- To Configure User Accounts
- To Add user accounts to the root filesystem 
- To Use alternative ways to manage device nodes
- To Configure the Network 

##  1. Analyzing the init program
For UNIX systems, there is a firts program to execute after boot.
Its name is  ```init```, it starts and monitors other programs. 

```init``` manages the life cycle of the system from bootup to shutdown. 

In our implementation, the init program was installed with busybox, and is located
at ```rootfs/sbin/init```, if we check into the details of that file we will see: 
```bash 
file sbin/init 
sbin/init: symbolic link to ../bin/busybox
```
looking into the details of busybox,

```bash
file bin/busybox 
bin/busybox: ELF 32-bit LSB executable, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, stripped
```

There are other init programs different to busybox, such as System V and systemd, for a deep understanding 
on these files read ```Chapter 13, Starting Up - The Init program``` of our reference book. 

The init program begins by reading the configuration file ```/etc/inittab```
We have to provide this file, 

In our case we must to create a file with the next content in our rootfs. 

```bash
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/ash
```

The first line in the inittab file calls a bashscript called ```rcS``` that 
we also have to create. 
The last line tells to execute an ash terminal with job control enabled. 

For the ```/etc/init.d/rcS``` file, we can put into the initialization 
tasks that allos us to manage our system and resources, for example 
a basic rcS file might be the next. 

```bash 
#!/bin/sh 
mount -t proc proc /proc 
mount -t sysfs sysfs /sys
```

After create your rcS file, grant it execution permissions: 

```bash
cd <path to your filesystem directory>/rootfs
chmod +x etc/init.d/rcS
```
### Starting daemon procecess 
To run daemons (background procecess) that monitors the system and manage 
resources through our busybox applet versions, 
We have to add the next lines to our ```etc/inittab```.

```bash
::respawn:/sbin/syslogd -n
::respawn:/sbin/klogd -n
```
Both of this programs save a log of messages form other programs and from the kernel. 
The log is written in ```/var/log/messages```, so they are available in the permanent storage.

### Configuration of user accounts
For a detailed description of user groups and user accounts on Linux 
refer to ```MELP | page 146``` on this part of the tutorial we just 
implement the process of adding user accounts sowed in ```page 147```.

####  To add user accounts to the root filesystem 
We have to implement 4 steps: 

1. In our ```rootfs``` directory, create the user account files:

```bash 
cd <path to your rootfs directory>/rootfs
touch etc/passwd etc/shadow etc/group
```

2. Make sure of grant permissions for ```etc/shadow``` of ```0600```.

```bash
cd <path to your rootfs directory>/rootfs
chmod 600 etc/shadow 
```

3. Initiate the logging procedure by starting a program called getty 
   using the busybox version. To do it, add the next lines into 
   ```inittab``` file. 

```bash
::respawn:/sbin/getty 115200 console 
``` 

4. The next step is rebuild the ```ramdisk``` and try it on zybo, but before to do it, 
let's explore how to implement better methods to managing device nodes. 

## Managin device nodes
In the reference book there are descripted 3 different methods to 
automatically create device nodes. 

1. ```devtmpfs```
2. ```mdev```
3. ```udev```

For a better understanding on its function and use cases, refer to ```MELP | page 148```

As, ```devtmpfs``` is the only maintanable way to generate device nodes prior to user space start up
we will start decribing its launching process. 

### Mount devtempfs

The first step to implement devtempfs, is to verify that you can launch it, looking at your kernel configuration. 

```bash
# To verify, execute next two greps
grep '^CONFIG_DEVTMPFS=' .config
grep '^CONFIG_DEVTMPFS_MOUNT=' .config
# expected output
CONFIG_DEVTMPFS=y
CONFIG_DEVTMPFS_MOUNT=y
```
If the first variable is enabled, your compilation is able to launch devtempfs,
If both are enabled, in an automated linux launch, devtempfs is self mounted (this is not our case as we are using busybox init and a customized inittab)
If either is enabled, enable them and rebuild your linux kernel, (and use the output files to boot your target board).

As in my case they are enableb, the next step is to mount ```devtmpfs``` 
in our filesystem. It can be done at runtime executing 

```bash 
mount -t devtmpfs devtmpfs /dev
``` 
but, to make this change permanent for our system, we have to add that line into the 
```etc/init.d/rcS```
The final ```rcS``` file content is:

```bash
#!/bin/sh
mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs devtmpfs /dev
```

### the alternative method to manage device nodes is using mdev
It is a little tricky than devtmpfs, but also is documented at ```page 149``` of the book. 

## Network configurations
The last configurations for our system previous to reboot our system is configuring network connections. 

**Assumtions**: There is an Ethernet interface in the board called ```eth0``` and that 
we will only need a simple IPv4 configuration.

To implement the network connection, we will use the network utilities part of 
BusyBox: ```ifup``` and ```ifdown```. Documentation about them is available on 
internet. 

The main network configurations are stored in the ```/etc/network/interfaces``` file.
We must to create the next directories:

```bash 
cd <path to rootfs>/rootfs
mkdir -p etc/network
mkdir -p etc/network/if-pre-up.d
mkdir -p etc/network/if-up.d
mkdir -p var/run
```

The **if** directories are associated to the network utilities and can start being empty.

The next step is to create the ```/etc/network/interfaces``` file. 
The reference book shows the content for both, **static IP** and **dynamic IP**
In our case implement the dynmic version. 

```bash
cd <path to rootfs>/rootfs
nano etc/network/interfaces

# content for the file 
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
```

As this file only describe the configuration, we require to configure 
a Dynamic Host Configuration Protocol (DHCP). 
To acomplish this objective, we first should to verify that our busybox 
compilation have the required tools to enable the network. 

to verify it, execute the next commands:
```bash
cd <path to rootfs>/rootfs
# Looking for tools command
tree | grep -E 'ifup|ifdown|udhcpc'
# if the ooutput is similar to the next, the required tools are installed
│   ├── ifdown -> ../bin/busybox
│   ├── ifup -> ../bin/busybox
│   ├── udhcpc -> ../bin/busybox
│   │   ├── udhcpc6 -> ../../bin/busybox
```

Now, to configure the DHCP, we require a configuration script in the direction 
```rootfs/usr/share/udhcpc/default.script```
BusyBox provides this document as part of its examples, to add it just copy 
the file ```<busybox repository path>/busybox/examples/udhcp/simple.script```

just use the next command to   copy it. 

```bash
cd <path to rootfs directory>/rootfs
mkdir -p usr/share/udhcpc
cp <busybox-path>/busybox/examples/udhcp/simple.script usr/share/udhcpc/default.script
```

## Network components for glibc 
Glibc uses Name Service Switch (NSS) to get the address resolution for web sites. 
Those process are configured through the ```/etc/nsswitch.conf``` file. 
Whose content will be filled as:

```bash
passwd: files
group: files
shadow: files
hosts: files dns
networks: files
protocols: files
services: files
```

We have to provide the files listed in the nsswitch.conf file,
some of them have been added previously on this tutorial, others must be added 
as we will mention next. 

For the hosts file

Create the file ```rootfs/etc/hosts```
with the next line of text
```bash
127.0.0.1 localhost
```
The files ```networks```, ```protocols``` and ```services``` can be copied from
your host device instalation. 
To do that, first, verify that the files exist in your host device: 

```bash
# commands to list the files
ls -l /etc/networks
ls -l /etc/protocols
ls -l /etc/services
# expected output
-rw-r--r--. 1 root root 58 ene 16  2026 /etc/networks
-rw-r--r--. 1 root root 6963 ene 16  2026 /etc/protocols
-rw-r--r--. 1 root root 701707 ene 16  2026 /etc/services
```

If the files exist, copy them to your rootfs directory:

```bash
cd <path to rootfs>/rootfs
cp -a /etc/networks etc/
cp -a /etc/protocols etc/
cp -a /etc/services etc/
```

Now, we can verify our files 
```bash 
ls -l etc/passwd etc/group etc/shadow
cat etc/passwd
cat etc/group
ls -l etc/shadow
```
Finally 
**Copy the NSS header libraries from $SYSROOT directory to rootfs/lib**
to to it, first 

```bash
# 1. Set the environment variables 
PATH=${HOME}/x-tools/arm-cortexa9_neon-linux-gnueabihf/bin/:$PATH
export CROSS_COMPILE=arm-cortexa9_neon-linux-gnueabihf-
export ARCH=arm
export SYSROOT=$(${CROSS_COMPILE}gcc -print-sysroot)
# 2. Verify the existance of the required libreries (command)
ls -l $SYSROOT/lib/libnss*
ls -l $SYSROOT/lib/libresolv*
### expected output
-rwxr-xr-x. 1 j2meg j2meg 159184 sep 10 18:30 /home/j2meg/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot/lib/libnss_compat.so.2
-rwxr-xr-x. 1 j2meg j2meg 135448 sep 10 18:30 /home/j2meg/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot/lib/libnss_db.so.2
-rwxr-xr-x. 1 j2meg j2meg   9720 sep 10 18:30 /home/j2meg/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot/lib/libnss_dns.so.2
-rwxr-xr-x. 1 j2meg j2meg   9720 sep 10 18:30 /home/j2meg/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot/lib/libnss_files.so.2
-rwxr-xr-x. 1 j2meg j2meg  63940 sep 10 18:30 /home/j2meg/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot/lib/libnss_hesiod.so.2
-rwxr-xr-x. 1 j2meg j2meg 217848 sep 10 18:30 /home/j2meg/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot/lib/libresolv.so.2
(base) bash-5.3$ 

# 3. Copy the libraries to your rootfs directory
cd <path to rootfs>/rootfs
cp -a $SYSROOT/lib/libnss* lib/
cp -a $SYSROOT/lib/libresolv* lib/

# 4. Verify the copy
ls lib/ -als
## expected output
total 2696
   4 drwxr-xr-x.  3 j2meg j2meg    4096 sep 27 01:44 .
   4 drwxr-xr-x. 13 j2meg j2meg    4096 sep 22 16:48 ..
 176 -rwxr-xr-x.  1 j2meg j2meg  178480 sep 22 16:54 ld-linux-armhf.so.3
1468 -rwxr-xr-x.  1 j2meg j2meg 1499708 sep 22 16:54 libc.so.6
 444 -rwxr-xr-x.  1 j2meg j2meg  452132 sep 22 16:54 libm.so.6
 156 -rwxr-xr-x.  1 j2meg j2meg  159184 sep 10 18:30 libnss_compat.so.2
 136 -rwxr-xr-x.  1 j2meg j2meg  135448 sep 10 18:30 libnss_db.so.2
  12 -rwxr-xr-x.  1 j2meg j2meg    9720 sep 10 18:30 libnss_dns.so.2
  12 -rwxr-xr-x.  1 j2meg j2meg    9720 sep 10 18:30 libnss_files.so.2
  64 -rwxr-xr-x.  1 j2meg j2meg   63940 sep 10 18:30 libnss_hesiod.so.2
 216 -rwxr-xr-x.  1 j2meg j2meg  217848 sep 10 18:30 libresolv.so.2
   4 drwxr-xr-x.  3 j2meg j2meg    4096 sep 22 17:12 modules
```

# Finally, we completed our filesystem

##
Once that those changes are made in our filesystem, we must to reload it into our 
initramfs and load it into our target as a standalone file or embedded in our 
kernel image as in our two previous tutorials. 

**To launch the init program from u-boot shell**
set this envronment variables before boot the kernel 
```bash
setenv bootargs console=ttyPS0,115200 rdinit=/sbin/init
```

### In the next tutorial 
We will see how to create an ext2 fs to mount on our SD card for ZYBO
How to Boot it
How to mounting the rootfs using NFS
