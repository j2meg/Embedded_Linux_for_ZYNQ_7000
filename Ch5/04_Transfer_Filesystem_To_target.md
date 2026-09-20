# Transferring the root filesystem to target

Reference MELP Book Third edition | ```Page 139```

For a discussion on the different ways to transfer a filesystem to 
our target device, there are three methods explained on detail on 
page ```138``` of our reference book. 

For purposes of this tutorial, we will to develop strategies to load it 
from:

- An **initramfs**: or ```ramdisk```, that is an image loadded into the RAM by the bootloader.

- A **Disk image**: This is a copy of the root filesystem formatted and ready to be loaded onto a mass storage device on the target.

- A **Network Filesystem**: An exposition of the staging directory to the network via NFS server and mounted by the target device on the boot. 

On this tutorial we will delving deeper on how to start the root 
filesystem by these three ways. 

## Creating a boot initramfs

An initramfs is a compressed ```cpio``` archive, to achive it, we need to 
configure our kernel with ```CONFIG_BLK_DEV_INITRD```. 

Verify that in the kernel that we prevously built this configuration is enabled 
with the command: 

```bash
grep '^CONFIG_BLK_DEV_INITRD' .config
```

If the output is something similar to 

```bash
CONFIG_BLK_DEV_INITRD=y
```

compatibillity with initramfs is enabled and we can continue with the process, in other case, enable it in the ```.config``` file of the linux kernel building directory, 
and rebuild the project. 

There are three ways to create a boot ramdisk, they are: a ```standalone cpio``` file, a ```cpio``` embedded in the kernel image and, as a ```device table```
 that the kernel procecess as part of the build. 

In this tutorial we will implement the standalone initramfs. 

## Standalone initramfs
### The next commands create the archive, compresses it and adds a U-Boot header ready to load onto the target

```bash
cd <path to your filesystem directory>
cd rootfs
# comprisse the filesystem on a cpio file
find . | cpio -H newc -ov --owner root:root > ../initramfs.cpio
# note --owner root:root makes that everithing in the cpio file archive have a UID and GID of 0.
cd .. 
# comprisse again the cpio file into a .gz file
gzip initramfs.cpio
# add the U-Boot header for loading onto the target
mkimage -A arm -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
```

Expected final output:

```bash
 mkimage -A arm -O linux -T ramdisk -d initramfs.cpio.gz uRamdisk
Image Name:   
Created:      Sat Sep 19 19:02:23 2026
Image Type:   ARM Linux RAMDisk Image (gzip compressed)
Data Size:    1771689 Bytes = 1730.17 KiB = 1.69 MiB
Load Address: 00000000
Entry Point:  00000000
```

