# Building Embedded Linux Kernel 

Reference MELP Third Edition-Page 91.

## Choose a Kernel Release to Build.

The first previous step to build a kernel for your target device, is to chose a reliable release of 
Linux Kernel in agreement with your development or production objetvies.

A deep description on this topic is covered on our reference book at ```Page 88 ```.

For purposes of the present project, we choose the ```last long term release``` 
till the moment that I write this document. 

Steps to get it are:

- Go through https://www.kernel.org/ and identify stable and long-term releases.
- Chose one of them depending on your interests. 
- Fetch and extract your releease tarball 

At this moment ```longterm: 	6.18.52``` is a reliable release

To fetch and extract our kernel release use the following commands. 

```bash 
# Clone Linux kernel stable tree
cd <Path to save your download>
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.18.52.tar.xz
tar xf linux-6.18.52.tar.xz
mv linux-6.18.52 linux-stable
``` 

Main directories for our purposes on the extracted files are:

- ```arch```
- ```Documentation ```
- ```drivers```
- ```fs```
- ```include```
- ```init```
- ```kernel```
- ```mm```
- ```net```
- ```scripts```
- ```tools ```

names are very descriptive, but for a better explanation look at: ```Page 92, MELP Book```


