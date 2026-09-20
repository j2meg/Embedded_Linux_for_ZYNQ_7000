# Populating Filesystem 
Reference (MELP 3rd edition |```Page 129```)
To populate our recent created filesystem in the staging directory, so this can be functional for our target device 
it requires the next type of files into:

- Programs for the root filesystem
    - The init program
    - Shell
    - Utilities
- Libraries
- Device Nodes
- Pseudo filesystems
    - ```proc```
    - ```sys```
- Kernel modules

In the next sections we will describe the process to create, copy or configure these files and pass them into our staging directory for the filesystem.

## The init program

## Shell and Utilities
There are multiple ways to get s shell and utilities software for our target. 
In the reference book are listed some options such as ```BusyBox```  and ```ToyBox```

About the shell program, we can get options such as the classical desktop ```bash``` but it is too weight for an embedded system, and lighter versions such as ```ash``` and ```hush``` for each of these options **test the libraries and commands once that you start your device** because they have different dependencies and it can produce errors to function. 

### Configuring and Building BusyBox

To get and config busybox is similar to get and build the Linux Kernel.

Just type next commands and to clone the source repository and use ```menuconfig``` to configure your set up. 

```bash
cd <Path to download busybox repository>
git clone git://busybox.net/busybox.git
cd busybox
git fetch --tags
git tag --list  --sort=version:refname
git checkout 1_38_0 # last stable release at this moment
```

Use Make to clean and start with the default configuration.

```bash
make distclean
make defconfig
```

Use ```menuconfig``` to add specific configurations and assign the 
install path of **busybox** to our staging path ```rootfs``` with the next commands:

```bash
make menucofnig 
# navigate to settings> --- Installation Options ("make install" behavior) 
# in Destination path for 'make install' set your rootfs path or staging directory.
```

Other way to see the building configurations is to see into the ```.config``` file in the busybox repository. 
The next step is to cross compile your busybox distribution for your specific target. 

In a similar way to what we did for the kernel building:

```bash
# Set your make environment variables as
PATH=${HOME}/x-tools/arm-cortexa9_neon-linux-gnueabihf/bin/:$PATH
export CROSS_COMPILE=arm-cortexa9_neon-linux-gnueabihf-
export ARCH=arm

# build your busybox installation
make
make install 
# The last command will copy the binary to the directory configured in CONFIG_PREFIX
```

You can see reference outputs of this building and installation proces at: ```Ch5/reference_files``` on this repository. 


## Copying Libraries in the root filesystem
If your target programs are not linked statically, you should to copy shared libraries into your filesystem to be available in your target device. 

This increase the ammount of storage required, then be careful with that. 

To complete this process:
- Copy required ```.so``` files from the ```sysroot``` directory of your toolchain. 

With that objective we can suppose that our image will eventually need them all. 

Be careful about the amount of storage of ```glibc```, you can use instead ```musl libc``` or  ```uClibc-ng```

Other approach is
- to cherry pick those libraries that you require through your library dependencies, using ```readelf``` command to your image file. 

In our case: 

```bash
cd <path to filesystem directory>
cd rootfs
# Load temporal environment variables
PATH=${HOME}/x-tools/arm-cortexa9_neon-linux-gnueabihf/bin/:$PATH
export CROSS_COMPILE=arm-cortexa9_neon-linux-gnueabihf-
export ARCH=arm

# Apply readelf to your target cross compiler 
# As in previous paragraphs we exported a cross compiler env variable
${CROSS_COMPILE}readelf -a bin/busybox | grep 'program interpreter'
${CROSS_COMPILE}readelf -a bin/busybox | grep 'Shared library'
```

This outputs something like: 

```bash 
[Requesting program interpreter: /lib/ld-linux-armhf.so.3]
 0x00000001 (NEEDED)                     Shared library: [libm.so.6]
 0x00000001 (NEEDED)                     Shared library: [libresolv.so.2]
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
```

Now we need to find those ```.so``` files in the toolschain ```sysroot``` directory and copy them to our staging directory. 

Those steps are executed as

```bash
# Identify your sysroot path 
${CROSS_COMPILE}gcc -print-sysroot

#output
~/x-tools/arm-cortexa9_neon-linux-gnueabihf/arm-cortexa9_neon-linux-gnueabihf/sysroot

## Save the sysroot path into a new env variable 
export SYSROOT=$(${CROSS_COMPILE}gcc -print-sysroot)
```

For each file in our requirements, let's verify the type of file that they are. 

```bash 
cd $SYSROOT 
ls -l lib/ld-linux-armhf.so.3
ls -l lib/libm.so.6
ls -l lib/libresolv.so.2
ls -l lib/libc.so.6
file lib/ld-linux-armhf.so.3
file lib/libm.so.6
file lib/libresolv.so.2
file lib/libc.so.6


# output 
-rwxr-xr-x 1 j2m j2m 1264692 sep 10 13:45 lib/ld-linux-armhf.so.3
-rwxr-xr-x 1 j2m j2m 2149832 sep 10 13:43 lib/libm.so.6
-rwxr-xr-x 1 j2m j2m 217640 sep 10 13:44 lib/libresolv.so.2
-rwxr-xr-x 1 j2m j2m 11246608 sep 10 13:44 lib/libc.so.6
lib/ld-linux-armhf.so.3: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, with debug_info, not stripped
lib/libc.so.6: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, with debug_info, not stripped
lib/libm.so.6: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (GNU/Linux), dynamically linked, for GNU/Linux 3.2.0, with debug_info, not stripped
lib/libresolv.so.2: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, for GNU/Linux 3.2.0, with debug_info, not stripped
``` 

Some of them can be simbolic links, in that case, take care of to copy both files, symbolic link file and the source wich it points

In my case, I just have to copy the four files: 

```bash
cp -a $SYSROOT/lib/ld-linux-armhf.so.3 lib
cp -a $SYSROOT/lib/libm.so.6 lib
cp -a $SYSROOT/lib/libresolv.so.2 lib
cp -a $SYSROOT/lib/libc.so.6 lib
```

### Reducing the size of the libraries (stripping)
for each of the recent created libraries the output of the file command outputs at the end the note 
```not stripped```. Strip is the process of reduce the size of your libraries stripping the binaries of symbol tables and other elements that are part of our libraries, to do it: 

```bash
cd <path to your filesystem>
cd rootfs

${CROSS_COMPILE}strip lib/ld-linux-armhf.so.3
${CROSS_COMPILE}strip lib/libm.so.6
${CROSS_COMPILE}strip lib/libresolv.so.2
${CROSS_COMPILE}strip lib/libc.so.6
```

Now, our files must be stripped and its total size have been reduced from original. 

```bash
file lib/libc.so.6 
lib/libc.so.6: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, stripped
```
## Device Nodes 
Most devices in linux are represented by **device nodes**. 
Remembering the UNIX  philosophy, *everything is a file* (except network interfaces, which are sockets).

A deep discussion about Device Nodes is available in MELP third edition page 134.
Remember that we will need to create device nodes for each device that we wanto to access on our system, through the ```mknod``` command. 
For purposes of this tutorial, as we are implementing a very minimal root filesystem, 
we will just to create two device nodes ```null``` and ```console```.  

```console``` only needs to be accesible to ```root```, the owner of the device node 
so the access permisions are ```600 (rw-------)```. The ```null``` device node should be readable and writable by everyone, so the mode is ```666 (rw-rw-rw-)```. Use the ```-m``` option for mknode to set the mode when creating the node. And we require to be in root mode to create device nodes. 

```bash
cd <path to the staging directory>
cd rootfs
sudo mknod -m 666 dev/null c 1 3 
sudo mknod -m 600 dev/console c 5 1 
ls -l dev

# expected output 
total 1
crwxr-xr-x 1 j2m j2m 5, 1 sep 17 21:38 console
crwxr-xr-x 1 j2m j2m 1, 3 sep 17 21:38 null
 
```

to delete a device node, it is enough to use the file remover command ```rm ``` because they are also files. 


## Mounting the proc and sys pseudofilesystems

For an extended explanation of the ```proc``` and  ```sys``` filesystems 
refer to MELP third edition book, page 136.

For purposes of this tutorial, they are pseudofilesystems that provide a nearest view of the kernel activities. 
They represent kernel data as files in a hierarchy of directories.  

The Directories where the ```proc``` and  ```sys``` filesystems should be mounted 
(```rootfs/proc``` and (```rootfs/sys```) 
have been created before, we just have to take care about mount the filesystems
when our SO launches the initi program with the commands:

```bash
mount -t proc proc /proc
mount -t sysfs sysfs /sys
```

## Kernel Modules

If your linux configuration have kernel modules, they need to be installed into the root filesystem
 using the ```modules_install``` kernel make target. 
This will copy the configuration files into de directory called ```lib/modules/<kernel version> 
together with the configuration files needed by the ```modprobe``` command.

```bash
make modules_install \
    INSTALL_MOD_PATH=~rootfs
```

In our case, there is no need to install any module yet. 
