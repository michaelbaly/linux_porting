1. System configuration
   ```
   a. uImage (Linux version 2.6.29-g0373fd3-dirty (gypsice§gypsice-vm) 
      (gcc version 4.2.2) #15 Fri Jun 6 13:23:55 CST 2025)

   b. NFS server: ubuntu22.04 and it's config
      1. make sure NFS server is NFSv3 compatibility 
         (+:- -> enabled:disabled in kernel)
         -> cat /proc/fs/nfsd/versions 
         +3 -4 -4.0 -4.1 -4.2 
      2. nfs rootpath on server (check /etc/exports for details)
         -> exportfs
         /nfs            127.0.0.1
         /nfs            <world>

         Content from /etc/exports
         /nfs *(rw,sync,no_root_squash,no_subtree_check,fsid=0) 127.0.0.1(rw,sync,no_root_squash,no_subtree_check)
   c. ufw on ubuntu is disabled. (otherwise, ufw rule needs to be configured)
      -> sudo ufw status
      Status: inactive
   d. IMPORTANT: use TCP when loading file system from NFS server, otherwise, it will
      always timeout(from U-boot) or kernel panic(since can't load file system)
      set bootargs root=/dev/nfs nfsroot=10.28.1.110:/nfs/rootfs,nfsvers=3,tcp ip=10.28.1.254 console=ttySAC0,115200

      bootargs used for kernel
      Kernel command line: root=/dev/nfs nfsroot=10.28.1.110:/nfs/rootfs,nfsvers=3,tcp ip=10.28.1.254 console=ttySAC0,115200
   ```

2. Disable buiding initramfs source file(s) and enable nfs correspondig options
    
    Enable Network support in kernel
    ```
    Networking support -> Networking options -> TCP/IP networking
    ```

    Enable DM9000 driver
    ```
    Device driver -> Network device support -> Ethernet (10 or 100Mbit) -> 
    DM9000 support
    ```

    For NFS root file system
    ```
    File systems -> Network File System -> NFS client support
                                           -> NFS client support for NFS version 3
                                           -> Root file system on NFS (/nfs generally)
    ```

    Disable initramfs as showed in min-sys
    ```
    General setup -> Initial RAM filesystem and RAM disk -> Initramfs source file(s)
    
    For example: /path/to/initramfs
    ```

3. compile kernel image
   
   make
   make uImage
   cp arch/arm/boot/zImage /tftpboot/
   cp arch/arm/boot/uImage /tftpboot/

4. load kernel from tftp
   
   For manually load
   ```
   tftp zImage
   go 50008000
   ```

   And for automatically load from U-boot
   ```
   set bootcmd tftp uImage \; bootm
   sav
   ```
5. test result

   Kernel will automatically load file system from NFS server based on **kernel command line**
   ```
   eth0: link down
   eth0: link up, 100Mbps, full-duplex, lpa 0xCDE1
   IP-Config: Guessing netmask 255.0.0.0
   IP-Config: Complete:
        device=eth0, addr=10.28.1.254, mask=255.0.0.0, gw=255.255.255.255,
        host=10.28.1.254, domain=, nis-domain=(none),
        bootserver=255.255.255.255, rootserver=10.28.1.110, rootpath=
   Looking up port of RPC 100003/3 on 10.28.1.110
   Looking up port of RPC 100005/3 on 10.28.1.110
   VFS: Mounted root (nfs filesystem) on device 0:11.
   Freeing init memory: 100K

   Please press Enter to activate this console. 

   ```

Notes:
1. Modify nfsd.conf if necessary -> sudo vim /etc/nfsd.conf
```
[nfsd]
# debug=0
# threads=8
# host=
# port=0
# grace-time=90
# lease-time=90
# udp=n
# tcp=y
# vers2=n
# vers3=y
# vers4=y
vers4=n
# vers4.0=y
vers4.0=n
# vers4.1=y
vers4.1=n
# vers4.2=y
vers4.2=n
# rdma=n
# rdma-port=20049
rsize=1024
wsize=1024
```
2. Restart nfs server and rpcbind
```
sudo systemctl restart nfs-kernel-server
sudo systemctl restart rpcbind
```
3. Verify if U-boot can load rootfs by udp via NFS

   a. Edit /etc/nfs.conf
   ```
   [nfsd]
   vers3=y
   vers4=y
   udp=y       # Explicitly allow UDP for v3
   # You can also pin mountd/statd to fixed ports if your firewall needs it:
   mountd-port=20048
   statd-port=32765
   ```
   b. Restart service
   ```
   sudo systemctl restart nfs-kernel-server rpcbind
   ```
   c. Verify that NFSv3 IS listening on UDP
   ```
   sudo ss -ulpn | grep nfsd

   udp   UNCONN  0  0  0.0.0.0:2049  0.0.0.0:*   users:(("nfsd",pid=...))
   udp   UNCONN  0  0  0.0.0.0:111   0.0.0.0:*   users:(("rpcbind",pid=...))
   udp   UNCONN  0  0  0.0.0.0:20048 0.0.0.0:*   users:(("mountd",pid=...))
   ```
   If none of the NFS ports appear under UDP, your server is still not serving UDP.
   
   d. Confirm from a Linux client

   ```
   # add rule to ufw
   sudo ufw allow 2049/udp
   sudo ufw allow 111/udp
   sudo ufw allow 20048/udp
   ```
   ```
   showmount -e 10.28.1.110
   # Should list your export (e.g. /nfs)
   mount -o udp,vers=3 10.28.1.110:/nfs /mnt
   ls /mnt        # Should succeed over UDP

   ```