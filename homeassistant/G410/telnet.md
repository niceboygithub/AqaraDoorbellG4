# Enable Telnet of Aqara Doorbell G410

The easy way to enable telnet of Aqara Doorbell G410 is that flash customized firmware.

What is the modification in the customzied firmware?
1. Remove th login password of telnet
2. Run telnetd as default.

The method:
1. Remove the power of the gateway.
2. Copy the [linux.bin and rootfs.bin](https://github.com/niceboygithub/AqaraCameraHubfw/tree/main/modified/G410/uboot) to USB disk in FAT32 format.
3. Plug-in the USB disk to the gateway.
4. Press the front button of the gateway without release
5. Plug-in the power cord to the gateway
6. When the LED turn to the purple, release the button.
7. After the flash is completed, the gateway will reboot to normal.
