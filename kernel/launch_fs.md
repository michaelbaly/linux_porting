### initramfs
* Q: mkimage command not found - u-boot images will not be built
* A: apt install u-boot-tools

+ 1. uncompress initramfs.bz2 to /nfs
```
tar xjf initramfs.bz2 -C /nfs
```
+ 2. Build uImage and cp to ftp server
```
make uImage
cp uImage /srv/tftp
```
+ 3. set up boot and kernel parameters
```
# launch file system from nfs
set bootargs root=/dev/rootfs nfsroot=192.168.1.5:/nfs/initramfs ip=192.168.1.230 console=ttySAC0,115200
set bootcmd tftp uImage \; bootm
```
