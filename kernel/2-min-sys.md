1. make initramfs file system

2. enable kernel option and set file system path


    ```
    General setup -> Initial RAM filesystem and RAM disk -> Initramfs source file(s)
    
    For example: /path/to/initramfs
    ```


3. compile kernel image

4. load kernel from tftp
   ```
   tftp zImage-2-min-sys
   go 50008000
   ```

5. test result
   ```
   mice: PS/2 mouse device common for all mice
   i2c /dev entries driver
   sdhci: Secure Digital Host Controller Interface driver
   sdhci: Copyright(c) Pierre Ossman
   VFP support v0.3: implementor 41 architecture 1 part 20 variant b rev 5
   Freeing init memory: 732K

   Please press Enter to activate this console. 
   / # ls
   bin           home          lib           root          tmp
   dev           init          mnt           sbin          usr
   etc           initrd.image  proc          sys           var

   ```
