# Enable Telnet of Aqara DoorBell G4


## Hard way (High Risk, do it at your own risk)
1. Open the case
2. Connect to UART, reference to the picture. Suggestion use soldering.
<img src="/images/g4_uart.png" alt="uart" height="520" width="460">

3. Interrupt uboot.
  - Use putty to open COM port. Baud Rate is 115200.
  - While boot up (power up the , keep pressed the key "Enter". If not stop uboot, please try again.

4. If it stopped on uboot cli, enter the following command

```
nand info
```
 Check if nand is properly initialized.
```
Device 0: nand0, sector size 128 KiB
  Page size      2048 b
  OOB size         64 b
  Erase size   131072 b
```
If NAND is initialized, run the next command to continue.
```
printenv bootargs
```
   Save the bootargs to note, then replace the string "/linuxrc" to "/bin/sh".
   For example, the new bootarg likes
```
setenv bootargs root=/dev/mtdblock7 rootfstype=squashfs ro init=/bin/sh LX_MEM=0x7FE0000 mma_heap=mma_heap_name0,miu=0,sz=0x500000 cma=2M mmap_reserved=fb,miu=0,sz=0x300000,max_start_off=0x7C00000,max_end_off=0x7F00000 mtdparts=nand0:1664k@0x140000(BOOT0),1664k(BOOT1),256k(ENV),256k(ENV1),128k(KEY_CUST),5m(KERNEL),5m(KERNEL_BAK),16m(rootfs),16m(rootfs_bak),1m(factory),20m(RES),-(UBI)
```
   Then enter the following commands
```
run bootcmd
```
5. It will continue to boot kernel and stops shell console. Then enter the following command.
```
ubifs_mount.sh 0 /res
ubifs_mount.sh 1 /data
```
6. Then delete the 'passwd' file.
```
rm /res/passwd
```
7. Create the post_init.sh
```
mkdir -p /data/scripts; chattr -i /data/scripts/post_init.sh; echo -e "#\!/bin/sh\n\nfw_manager.sh -r\nfw_manager.sh -t -k\n" > /data/scripts/post_init.sh; chmod a+x /data/scripts/post_init.sh; chattr +i /data/scripts/post_init.sh
```
   if you want enable rtsp as default
```
mkdir -p /data/scripts; chattr -i /data/scripts/post_init.sh; echo -e "#\!/bin/sh\n\nfw_manager.sh -r\nfw_manager.sh -t -k\nrtsp &\n" > /data/scripts/post_init.sh; chmod a+x /data/scripts/post_init.sh; chattr +i /data/scripts/post_init.sh
```
8. Restart Doorbell G4, then telnet to G4 to clear password
```
passwd -d root
```