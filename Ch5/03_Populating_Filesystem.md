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


