# Customize Firmware
## Introduction

To customize Android firmware, there are two methods:

* Modify source code, then compile and package to new firmware.
* Patch based on existing firmware.

The former can customize Android on any level freely, but require compiling environment with high performance, and personal knowledgeable skill in Linux kernel and Android. Please see the detail in [Build_android](compile_android7.1_firmware.html).

We introduce the latter one here. It is divided in three stages: extract, customize and package. We use Linux as the host OS, and utilities which are all open source.
## Image Format

Our new update image, release_update.img here, contains bootloader loader.img and the actual firmware data update.img. That is, release_undate.img is a wrapper of loader.img and update.img.

release_update.img
```
|- loader.img
`- update.img
```
update.img is a composite file, which contains multiple files described by package-file. A typical package-file is:
```
# NAME Relative path package-file package-file bootloader Image/MiniLoaderAll.bin parameter       Image/parameter.txt trust           Image/trust.img uboot           Image/uboot.img misc Image/misc.img resource Image/resource.img kernel Image/kernel.img boot            Image/boot.img recovery Image/recovery.img system Image/system.img backup RESERVED #update-script update-script #recover-script recover-script
```
* package-file
* Package description of update.img. In update.img, there is a copy of package-file.
* Image/MiniLoaderAll.bin
* Bootloader
* Image/parameter.txt
* parameter file
* which can specify kernel command line. The command line has rich information, such as partition of the flash storage.
* Image/trust.img
* trust.img second loader of U-boot.
* Image/misc.img
* Image of misc partition. It is a boot mode selector. If it is empty (all zero), system boots normally. If it contains recovery command, system boots into recovery mode with corresponding command and arguments.
* Image/kernel.img 
* Android Linux kernel.
* Image/resource.img 
* Resource image, has kernel startup logo and DTB (Device Tree Blob) inside.
* Image/boot.img
* Android initial ram disk, which is the first root filesystem to load and has important system initialization.
* Image/recovery.img
* Image of Android 
* recovery mode, has kernel and recovery mode root filesystem inside.
* Image/system.img 
* Image of Android's system partition, which will has extensive customization below.

Extracting firmware, will involve in extracting update.img from release_update.img, and then extract files described by package-file inside update.img.

Packaging is just the reverse, which packs the files described by package-file into update.img, and then mix with loader.img into the final image file - release_update.img.
## Prepare the Utilities
```
git clone https://github.com/TeeFirefly/rk2918_tools.gitcd rk2918_tools
make
sudo cp afptool img_unpack img_maker mkkrnlimg /usr/local/bin
```
## Extract Firmware

* Extract release_update.img
```
$ cd /path/to/your/firmware/dir
$ img_unpack Firefly-RK3399_20161027.img img
rom version: 6.0.1
build time: 2016-10-27 14:58:18
chip: 33333043
checking md5sum....OK
```
* Update update.img
```
$ cd img
$ afptool -unpack update.img update
Check file...OK
------- UNPACK -------
package-file	0x00000800	0x00000280
Image/MiniLoaderAll.bin	0x00001000	0x0003E94E
Image/parameter.txt	0x00040000	0x00000350
Image/trust.img	0x00040800	0x00400000
Image/uboot.img	0x00440800	0x00400000
Image/misc.img	0x00840800	0x0000C000
Image/resource.img	0x0084C800	0x0003FE00
Image/kernel.img	0x0088C800	0x00F5D00C
Image/boot.img	0x017EA000	0x0014AD24
Image/recovery.img	0x01935000	0x013C0000
Image/system.img	0x02CF5000	0x2622A000
RESERVED	0x00000000	0x00000000
UnPack OK!
```
* Look at the file tree under the update directory
```
$ cd update/
$ tree
.
├── Image
│   ├── boot.img
│   ├── kernel.img
│   ├── MiniLoaderAll.bin
│   ├── misc.img
│   ├── parameter.txt
│   ├── recovery.img
│   ├── resource.img
│   ├── system.img
│   ├── trust.img
│   └── uboot.img
├── package-file
└── RESERVED

1 directory, 12 files
```
The firmware is extracted successfully. Let's start to customize.
## Customization
### Customize system.img

system.img is an image file with ext4 filesystem format, which can be mounted into system to modify directly:
```
sudo mkdir -p /mnt/system
sudo mount -o loop Image/system.img /mnt/system
cd /mnt/system
# Add or delete apks inside. Pay attention to the free space. You canno add too many apks.
# Edit complete. Unmount it.
cd /
sudo umount /mnt/system
```
Please note, the free space of system.img is nearly zero. If you want more, you have to expand it, and change partition settings in parameter file accordingly.

Here is an example of how the expand the image. Before expanding, run mount to make sure that system.img is not mounted (Unmount it if it is):

* Add 128M disk space
```
dd if=/dev/zero bs=1M count=128 >> Image/system.img
# Update filesystem records
e2fsck -f Image/system.img
resize2fs Image/system.img
```
## Packaging

First, you need to check the size of system.img. Make sure it is matched with partition setting in parameter file. If not, modify the parameter file to adjust the partitions.

For example, you can find the "CMDLINE" line in parameter file rk312x.parameter.txt, which has the string "system":
```
0x00200000@0x000B0000(system)
```
The number before symbol @, is the size of the partition, in unit of 512 bytes (traditional disk sector size). Therefore this system partition has a size of:
```
$ echo $(( 0x00200000 * 512 / 1024 / 1024))M
1024M
```
If size of system.img does not go beyond 1024M, there is no need to change parameter file.

If the partitions do not changed, you can flash the new system.img to the device for a test. Otherwise, you need to package a new firmware, flash it and test.

Here are the steps to  package update.img:

* package update.img ：
```
# The current directory is still update/, which has package-file. All files listed in package-file are available.
# Copy the parameter file, which is the default one used by  afptool.
$ cp Image/parameter.txt parameter
$ afptool -pack . ../update_new.img
------ PACKAGE ------
Add file: ./package-file
Add file: ./Image/MiniLoaderAll.bin
Add file: ./Image/parameter.txt
Add file: ./Image/trust.img
Add file: ./Image/uboot.img
Add file: ./Image/misc.img
Add file: ./Image/resource.img
Add file: ./Image/kernel.img
Add file: ./Image/boot.img
Add file: ./Image/recovery.img
Add file: ./Image/system.img
Add file: ./RESERVED
Add CRC...
------ OK ------
Pack OK!
```
* Package release_update.img ：
```
$ img_maker -rk33 loader.img update_new.img release_update_new.img
generate image...
append md5sum...
success!
```
Congratulations, release_update_new.img is the final firmware image file ready to flash.
## FAQs
### Q1: How to change firmware version?

A1: In parameter file, find the following line and modify it. The version can only contain digital numbers, and the two dots can not be omitted.
```
FIRMWARE_VER: 6.0.1
```
