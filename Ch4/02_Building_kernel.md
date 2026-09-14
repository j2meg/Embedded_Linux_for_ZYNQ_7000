# Building Embedded Linux Kernel 

Reference MELP Third Edition-Page 91.

## Choose a Kernel Release to Build.

The first previous step to build a kernel for your target device, is to chose a reliable release of 
Linux Kernel in agreement with your development or production objetvies.

A deep description on this topic is covered on our reference book at ```Page 88 ```.

For purposes of the present project, we choose the ```last stable release``` 
till the moment that I write this document. 

Steps to get it are:

- To clone Linux kernel stable tree on your local PC. 
- Review for the different releases in the tree
- Identify those compatible with your hardware on the  ```arch/ ``` directory of your release.
- Go to next sections on this tutorial to compile.

Commands to support your search are listed next, execute each one separately. 

```bash 
# Clone Linux kernel stable tree
git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/



``` 

