# What should be in the root filesystem and directory layout
Reference: ```MELP Third Edition | page 123```)

As is commented on the reference book. 
A minimal root filesystem need the next components. 

- ```init```: This is the program that starts everything off, usually by running a series of scripts. 
- ```shell```: It is a command prompt, but more than it, this component executes the shell scripts called by ```init``` and other programs. 
- ```Daemons```: Background programs that provides service to others, such as ```syslogd```, or ```sshd```. They are launched by the ```init``` program that is a daemon in fact. 
- ```Shared Libraries```: They are used to execute programs dinamically linked at runtime, so they must be present in the root filesystem. 
- ```Configuration files```: The configuration for the ```init``` and other daemons is stored in a series of text files, usually in the ```/etc``` directory.
- ```Device nodes```: These are special files that give access to various device drivers.
- ```proc and sys```: Two pseudofilesystems that represent kernel data structures as a hierarchy of directories and files. They are mounted at ```/proc``` and ```/sys```.
- ```kernel modules```: Those parts of the kernel that have been configurated as modules, need to be installed in the root filesystem, usually in  ```/lib/modules/[kernel version]/```. 

## Directory layout

For the linux kernel there is not a mandatory layout of files and directories. Beyond of the existance of the ```init=``` program or ```rdinit=```.
So, filesystems such as Android and Linux for desktop are very different. 

Furthermore, for software development purposes, the filesystem layout is  
standarized on the **Filesystem Hierarchy Standard (FHS)** available at 
https://refspecs.linuxfoundation.org/fhs.shtml
. 

For Embedded systems, FHS tends to include the next structure:

- ```/bin```: Essential programs for all users
- ```/dev```: Device nodes and other special files
- ```/etc```: System Configuration files
- ```/lib```: Essential shared libraries, for example the C library
- ```/proc```: Information about processes represented as virtual files
- ```/sbin```: Essential programs for the system administrator
- ```/sys```: Information about devices and their drivers represented as virtual files
- ```/tmp```: A place to put temporary or volatile files
- ```/usr/bin```: Additional programs, libraries and system administrator utilities ordered in the respective directories ```/usr/bin```, ```/usr/lib``` and ```/usr/sbin```:
- ```/var```: A hierarchy of files and directories that may be modified at runtime, for example, log messages, some of which must be retained after boot. 


# Stagging directory

To implement our filesystem on an Embedded System, we should star by creating a ```staging``` 
directory on our host computer. 

At this directory, we can assemble the files that will eventually be transferred 
to the target. for purposes of this tutorial we will create the path ```rootfs```.

To generate our ```filesystem skeleton``` execute the next commands 

```bash 
cd <path to your experiments directory on host system>
mkdir rootfs
cd rootfs
mkdir bin dev etc home lib proc sbin sys tmp usr var
mkdir usr/bin usr/lib usr/sbin
mkdir -p var/log
```

The expected output for our tree directory is: 

```bash 
tree
.
├── bin
├── dev
├── etc
├── home
├── lib
├── proc
├── sbin
├── sys
├── tmp
├── usr
│   ├── bin
│   ├── lib
│   └── sbin
└── var
    └── log
```

A last step on this root filesystem creation is to grant appropriate presmissions to the recent created directories. 

A deep discussion on that topic is available on page ```127``` of the reference book. 

For porposes of this tutorial, we will relie on the next configurations to assign ownership of the staging directory to the ```root``` user. 

```bash
cd <path to our experiments directory>
cd rootfs
sudo chown -R root:root *
