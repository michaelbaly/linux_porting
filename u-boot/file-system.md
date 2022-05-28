# Yaffs
+ 1. go to yaffs/utils and make to generate **mkyaffs2image** tool
+ 2. for 4k + 218 pages, modify mkyaffs2image.c file and recompile
```
#define chunkSize 4096
#define spareSize 218
#define pagesPerBlock 128
```
+ 3. uncompressing rootfs.bz2 (generate *rootfs* dir)
+ 4. make fs image
```
./mkyaffs2image rootfs rootfs.bin
chmod 755 rootfs.bin
sudo cp rootfs.bin /srv/tftpd
```
+ 5. update u-boot
```
nand erase 0 80000
nand read 50008000 0 10 ; md.b 50008000 10  # read and dump memory
tftp u-boot.bin ; nand write 50008000 0 80000 # update u-boot partition
```
+ 6. update file-system
```
nand erase 700000 2000000   # erase 32M space start from file-system partition
tftp rootfs.bin ; nand write.yaffs ${fileaddr} 700000 ${filesize} # burning rootfs.bin
```
+ 7. set boot args for kernel
```
set bootargs root=/dev/mtdblock2 console=ttySAC0,115200
```
+ 8. configure bootcmd
```
set bootcmd tftp uImage \; bootm
```
+ 9. reset the board

# Ubifs
+ 1. uncomressing mtd-utils.tar.gz
```
tar zxf mtd-utils.tar.gz
```
+ 2. install packets
```
apt install libacl1-dev liblzo2-dev uuid-dev
```
+ 3. generate tool
```
cd mtd-utils && make    # mkfs.ubifs
```
+ 4. make fs image
```
mkfs.ubifs -m 4096 -c 8176 -e 516096 -r rootfs -o rootfs.bin
```
+ 5. update u-boot (like yaffs)
+ 6. enter u-boot CL and check the partition
```
MY6410 # mtdparts
device nand0 <NAND>, # parts = 5
 #: name                size            offset          mask_flags
 0: u-boot              0x00080000      0x00000000      0
 1: param               0x00080000      0x00080000      0
 2: reserve             0x00100000      0x00100000      0
 3: kernel              0x00500000      0x00200000      0
 4: file-system         0xff900000      0x00700000      0
active partition: nand0,0 - (u-boot) 0x00080000 @ 0x00000000
defaults:
mtdids  : nand0=NAND
mtdparts: mtdparts=NAND:512k(u-boot),512k(param),1m(reserve),5m(kernel),-(file-system)
```
+ 7. erase && update kernel partition
```
nand erase kernel
tftp uImage ; nand write ${fileaddr} kernel
```
+ 8. erase file-system partition
```
nand erase file-system
ubi part file-system
ubi info
```
+ 9. create rootfs volume(size equals with 512M)
```
ubi create rootfs 20000000
```
+ 10. create home dir
```
ubi create home        # with left LEB size
```
+ 11. download from ftp server and buring to rootfs volume
```
ubi write ${fileaddr} rootfs ${filesize}
# mount rootfs volume
ubifsmount rootfs
ubifsls /
```
+ 12. set args for both u-boot and kernel and reset the board
```
set bootargs ubi.mtd=2 root=ubi0:rootfs rootfstype=ubifs console=ttySAC0,115200
set bootcmd nand read 50008000 kernel \; bootm
```
