* Toolchains have a loose name convention like *arch[-vendor][-os]-abi.*

+ 1. **arch** is for architecture: **arm, mips, x86, i686**...
+ 2. **vendor** is tool chain supplier: **apple,**
+ 3. **os** is for operating system: **linux, none (bare metal)**
+ 4. **abi** is for application binary interface convention: **eabi, gnueabi, gnueabihf**

### Install gnu toolchain(ubuntu20.04)
+ 1. compiler
```
apt install gcc-arm-linux-gnueabi g++-arm-linux-gnueabi
```
