# Technical requirements

## Requirements list
```bash
autoconf, automake, bison, bzip2, cmake, flex, g++,
gawk, gcc, gettext, git,gperf,help2man,libncurses5-dev.libstdc++6,
libtool,libtool-bin,make, patch,python3-dev,rsync,texinfo,unzup,wget,
xz-utils 
```

## Installation process 

### For Fedora
sudo dnf install autoconf  automake bison bzip2 cmake flex g++ gawk gcc\
gettext git gperf help2man libncurses5-dev libstdc++6 \
libtool libtool-bin make  patch python3-dev rsync texinfo unzip wget\
xz-utils
### For ubuntu
```bash
sudo apt-get install autoconf  automake bison bzip2 cmake flex g++ gawk\
gcc gettext git gperf help2man libncurses5-dev libstdc++6 libtool \
libtool-bin make  patch python3-dev rsync texinfo unzip wget xz-utils
```


### Some modifications for Fedora  
libncurses5-dev==ncurses-devel
libstdc++6==libstdc++
libtool-bin==libtool
python3-dev==python3-devel
xz-utils==xz

