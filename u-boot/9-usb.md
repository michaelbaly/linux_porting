## Update kernel from USB storage

1. load u-boot from tftp and run it from memory
   ```
   tftp u-boot-9-usb.bin
   go 50008000
   ```
2. copy uImage to USB storage
3. Start USB with below command
   ```
   usb start
   ```
4. check current files on USB storage
   ```
   fatls usb 0:4   // where 0 represents device, and 4 represent partition

         boot/
         efi/
         sources/
         support/
         ...
   ```

5. load kernel image from USB storage to memory
   ```
   fatload usb 0:4 50008000 setup.exe

   reading setup.exe
   .......

   73904 bytes read
   MY6410 # md.b 50008000 128
   50008000: 4d 5a 90 00 03 00 00 00 04 00 00 00 ff ff 00 00    MZ..............
   50008010: b8 00 00 00 00 00 00 00 40 00 00 00 00 00 00 00    ........@.......
   50008020: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
   50008030: 00 00 00 00 00 00 00 00 00 00 00 00 e0 00 00 00    ................
   50008040: 0e 1f ba 0e 00 b4 09 cd 21 b8 01 4c cd 21 54 68    ........!..L.!Th
   50008050: 69 73 20 70 72 6f 67 72 61 6d 20 63 61 6e 6e 6f    is program canno
   50008060: 74 20 62 65 20 72 75 6e 20 69 6e 20 44 4f 53 20    t be run in DOS 
   50008070: 6d 6f 64 65 2e 0d 0d 0a 24 00 00 00 00 00 00 00    mode....$.......
   50008080: 4e 97 0b e1 0a f6 65 b2 0a f6 65 b2 0a f6 65 b2    N.....e...e...e.
   50008090: 51 9e 66 b3 08 f6 65 b2 51 9e 61 b3 18 f6 65 b2    Q.f...e.Q.a...e.
   500080a0: 51 9e 60 b3 09 f6 65 b2 51 9e 64 b3 07 f6 65 b2    Q.`...e.Q.d...e.
   500080b0: 0a f6 64 b2 8f f6 65 b2 51 9e 6c b3 01 f6 65 b2    ..d...e.Q.l...e.
   500080c0: 51 9e 9a b2 0b f6 65 b2 51 9e 67 b3 0b f6 65 b2    Q.....e.Q.g...e.
   500080d0: 52 69 63 68 0a f6 65 b2 00 00 00 00 00 00 00 00    Rich..e.........
   500080e0: 50 45 00 00 64 86 06 00 ce 37 6c dd 00 00 00 00    PE..d....7l.....
   500080f0: 00 00 00 00 f0 00 22 0c 0b 02 0e 0f 00 18 00 00    ......".........
   50008100: 00 e6 00 00 00 00 00 00 70 21 00 00 00 10 00 00    ........p!......
   50008110: 00 00 00 40 01 00 00 00 00 10 00 00 00 02 00 00    ...@............
   50008120: 0a 00 00 00 0a 00 00 00    ........
   MY6410 # 
   ```

6. erase and write new kernel image (if necessary)
   ```
   nand erase kernel

   nand write 50008000 kernel
   ```

## Transfer file from Linux host

note: compile dnw and its driver before any opertion, you may install gcc in case host kernel was built with another gcc like gcc-12
```
make -C /lib/modules/6.8.0-52-generic/build M=/home/gypsice/project/bd6410/porting/u-boot-porting/6410/9-usb/resources/dnw_linux/dnw-driver modules
make[1]: Entering directory '/usr/src/linux-headers-6.8.0-52-generic'
warning: the compiler differs from the one used to build the kernel
  The kernel was built by: x86_64-linux-gnu-gcc-12 (Ubuntu 12.3.0-1ubuntu1~22.04) 12.3.0
  You are using:           
  CC [M]  /home/gypsice/project/bd6410/porting/u-boot-porting/6410/9-usb/resources/dnw_linux/dnw-driver/dnw-driver.o
/bin/sh: 1: gcc-12: not found
make[3]: *** [scripts/Makefile.build:243: /home/gypsice/project/bd6410/porting/u-boot-porting/6410/9-usb/resources/dnw_linux/dnw-driver/dnw-driver.o] Error 127
make[2]: *** [/usr/src/linux-headers-6.8.0-52-generic/Makefile:1925: /home/gypsice/project/bd6410/porting/u-boot-porting/6410/9-usb/resources/dnw_linux/dnw-driver] Error 2
make[1]: *** [Makefile:240: __sub-make] Error 2
make[1]: Leaving directory '/usr/src/linux-headers-6.8.0-52-generic'
make: *** [Makefile:11: default] Error 2

```
```
sudo apt install gcc-12
```


1. install kernel module 
   ```
    sudo insmod dnw-driver.ko  (chmod to 755 before insmod)
   ```
2. listen dnw on 6410 device
   ```
   dnw 50008000
   ```
3. connect host to device with a OTG cable
4. start dnw service on host to send the file
   ```
   sudo ./dnw filename
   ```