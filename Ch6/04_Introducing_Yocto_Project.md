# 02 Introducing Buildroot
Reference (```MELP Third edition | Page 179```)

Yocto Project is a more complex tool than buildroot. 
It does not only build: 
- a toolchain
- a bootloader
- a kernel 
- a root filesystem

for your target device... 

It can **generate an entire Linux distribution for you with binary packages**
 **that can be installed at runtime**.
Yocto building process is managed around groups of recipes written in a combination of Python and Shell script. 
It has a task scheduler called **BitBake** and its 
Oficial documentation is abailable at '''https://www.yoctoproject.org```.

# To add... Components of Yocto Project 
## Installing Yocto Project
To get a stable release and long-term support, remember that

To get and install on your host device, clone their oficial repository as:

```bash
cd <path to save on your local host>
git clone https://git.yoctoproject.org/poky.git
cd poky
# Check for the lastest stable release 
git fetch --tags
git for-each-ref refs/tags \
  --sort=creatordate \
  --format='%(creatordate:short) %(refname:short)'

git tag --list '202*' --sort=version:refname
# Change your branch to your prefereed release
# in our case 2026.08
git checkout 2026.08
```

## Configuring Buildroot


You also can get an ofical *TAR* file available at: 
https://buildroot.org/downloads

