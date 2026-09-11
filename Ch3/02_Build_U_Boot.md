# U-Boot Installation Process 
Reference MLPE - Page 64

Das U-Boot is aopen source bootloader for embeded boards. 
It borned as a PowerPC boards bootloader, and, then it was ported to
arm, mips and sh based boards. 

## References
docs  https://docs.u-boot-project.org/en/latest/
mailing list at u-boot@lists.denx.de
mailing list form https://lists.denx.de/listinfo/u-boot

## Building U-Boot
 Recommended way: 
 getting the source code cloning the .git file and check out the
 tag you inted to use (lastest)

```bash 
git clone git://git.denx.de/u-boot.git
cd u-boot
git branch --show-current
git describe --tags --always
git log -1 --oneline
git fetch --tags
git tag --list 'v*' --sort=version:refname
```

Book examples work with v2021.01 version, but as we try to migrate 
towards ZYNQ-7000 ARM Cortex A9 processor, we have to check the version that
work for us: 

```bash
git checkout v2021.01
# althoug they checkout to v2021.01, support for arm versatile still
# just til 2015, then we use if we want another board look for different
# stable releases of u-boot and identify your correct version
git checkout v2026.07
git status 
git describe --tags --always 
``` 

#### the lastest versions of u boot are:
v2026.01
v2026.01-rc1
v2026.01-rc2
v2026.01-rc3
v2026.01-rc4
v2026.01-rc5
v2026.04
v2026.04-rc1
v2026.04-rc2
v2026.04-rc3
v2026.04-rc4
v2026.04-rc5
v2026.07
v2026.07-rc1
v2026.07-rc2
v2026.07-rc3
v2026.07-rc4
v2026.07-rc5
v2026.10-rc1
v2026.10-rc2

#### The lastest stable release is v2026.07
## To look for ZYNQ-7000 devices, see next document on this repository.
