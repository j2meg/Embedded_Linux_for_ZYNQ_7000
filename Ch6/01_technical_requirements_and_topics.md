# Chapter 6. Selecting a Build System
Reference (```MELP Third edition | Page 161)

In this chapter we will cover the next topics:
- Comparing Build Systems (Buildroot and Yocto)
- Distribting Binary Files
- Downloading and Installing Buildroot
- Downloading and installing Yocto Project

# Technical requirements 

- A linux based host system with at least 60Gb of available storage. 
- Etcher for Linux
- Micro-SD card to mount our Linux images in the target device (ZYBO)
- A ZYBO development Board and probabbly PYNQ-Z2 development board
- Ethernet cable and port for network connectivity
- Micro-USB wire to connect the Development board to the host system. 

## Main objective
In previous chapters we built an Embedded Linux system manually, 
this process is called **Roll your Own**. With the advantage of to have complete
control of the software and each dependancy. 
The main drawback of this process is that as a project grows, **RYO** become not manageable. 

Our main objective in this chapter is to get Hands-On Experience with Buildroot and
 Yocto project as building systems, to automate the steps to built a system: 
- Toolchain
- Bootloader
- Kernel 
- Root filesystem 

