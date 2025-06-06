## launch u-boot from tftp

1. config ip/server ip/netmask for device
2. make sure peer server is alive
   ```
   ping <dot server ip>
   ```
3. load u-boot to memory
   ```
   tftp u-boot.bin
   ```
4. write u-boot to flash if necessary
   ```
   // erase u-boot partition
   nand erase u-boot
   // write to flash from memory - nand write <mem addr> <part-offset> <part-size>
   nand write 50008000 0 80000
   ```