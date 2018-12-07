# U-BOOT
## Introduction
RK UBOOT is designed basing on the open UBOOT resource. It support boot loader mode and download mode. Boot loader mode is the normal mode of UBOOT. Device usually release under this mode. This mode mainly load the kernel from FLASH to SDRAM and run the Operating System. User need to flash the device under the download mode. Keep press the recover key on board and power on the board, it will enter download mode. This chapter mainly talk about UBOOT.
## Compile
The compiling steps of UBOOT are similar to kernel compiling. Before compiling, you need to write the configuration to ".config"，run the command:
```
make rk3399_box_defconfig
Linux,
make rk3399_linux_defconfig
```
If you need to modify the relative option, you can run:
```
make menuconfig
```
Run below command to Compile:
```
make ARCHV=aarch64
```
After compiling successfully, there will be a new file created at the follow path:
```
u-boot/uboot.img
u-boot/trust.img
u-boot/RK3399MiniLoaderAll_Vx.xx.bin
```
## Flash Image
Run the Flash tool, connect the Firefly to your computer through USB Type-C wire. Keep press the recover key and then power on the board. After entering the Download Mode, you can flash the Loader.

![](img/Uboot_rk3399_download.jpg)

If you flash the loader successfully, there will be some information output from the debug UART as the follow log shows:
```
#Boot ver: 2016-12-19#1.05
```
If the time shows the same as your compile time,it means you flash the loader successfully.  

Since Firefly is mainly used for development, there are default 1 second countdown before running the Operating System. During the 1 second, any input from the debug UART can make the UBOOT enter the command mode. If you need to set the UBOOT doesn’t enter the command mode default, you can edit the file: u-boot/include/configs/rk33plat.h Change the code
```
#define CONFIG_BOOTDELAY               1
```
to
```
#define CONFIG_BOOTDELAY               0
```
## Level 1 Loader

U-BOOT as a level 1 Loader mode, then only support EMMC storage device, after the completion of the compiler generated:
```
RK3288LoaderU-BOOT_V2.17.01.bin
```
V2.17.01 is the release number, rockchip definition U-Boot loader version, which is defined in accordance with the storage version 2.17, customers must not modify this version, 01 is a small version of the U-Boot definition, the user according to actual needs in the Makefile to modify.
## Secondary Loader
U-Boot as a secondary Loader mode, then the firmware supports all the storage devices, this mode, the need for MiniLoader support, through the macro CONFIG_MERGER_MINILOADER configuration generation. At the same time the introduction of Arm Trusted, Firmware will generate trust image, the configuration through the macro CONFIG_MERGER_TRUSTIMAGE generated.  

RK3399 uses the secondary loader, the compiled image is:
```
u-boot/uboot.img
u-boot/trust.img
u-boot/RK3399MiniLoaderAll_V1.05.bin
```
V1.05 is the version number of the release, rockchip defines the U-Boot loader version, which is based on the 1.05 version of the definition of storage, customers must not modify this version. uboot.img is U-Boot as a secondary loader package. trust.img is U-Boot as a secondary loader package.
