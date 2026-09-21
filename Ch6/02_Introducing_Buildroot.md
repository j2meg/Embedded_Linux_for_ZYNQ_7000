# 02 Introducing Buildroot
Reference (```MELP Third edition | Page 165```)

Current versions of Buildroot are capable of 
- Building a toolchain
- a bootloader
- a kernel 
- a root filesystem

Oficial documentation is abailable at '''https://buildroot.org/docs.html``` and 
```https://buildroot.org/downloads/manual/manual.html```

## Installing Buildroot
To get a stable release and long-term support, remember that
Buildroot produce stable releases four times a year. 

To get and install on your host device, clone their oficial repository as:

```bash
cd <path to save on your local host>
git clone git://git.buildroot.net/buildroot 
cd buildroot 
# Check for the lastest stable release 
git fetch --tags
git tag --list '202*' --sort=version:refname
# Change your branch to your prefereed release
# in our case 2026.08
git checkout 2026.08
```

## Configuring Buildroot


You also can get an ofical *TAR* file available at: 
https://buildroot.org/downloads


