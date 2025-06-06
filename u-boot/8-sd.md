## Update kernel from SD card

1. load u-boot from tftp and run it from memory
   ```
   tftp u-boot-8-sd.bin
   go 50008000
   ```
2. copy uImage to SD card
3. init SD with below command
   ```
   mmc init
   ```
4. check current files on SD card
   ```
   fatls mmc 0:1
   ```
5. load kernel image from SD to memory
   ```
   fatload mmc 0:1 50008000 uImage_My6410
   ```
6. erase and write new kernel image (if necessary)
   ```
   nand erase kernel
   nand write 50008000 kernel
   ```