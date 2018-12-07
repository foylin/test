# Compile Ubuntu firmware(GPT)
In order to facilitate the user, the official provides a SDK for Linux development. This chapter explains in detail the specific usage of the SDK.
## Ready to work
Download repo tools：
```
mkdir linux
cd linux
git clone https://github.com/FireflyTeam/repo.git
```
Download Linux-SDK:

init repo.git:
```
mkdir linux-sdk
cd linux-sdk
../repo/repo init --repo-url https://github.com/FireflyTeam/repo.git -u https://github.com/FireflyTeam/manifests.git -b linux-sdk -m rk3399/rk3399_linux_release.xml
```
Download code:
```
../repo/repo sync -c
```
If the network speed is too slow, the download will be interrupted. You can use the following script to synchronize the code:
```
#! /bin/bash

../repo/repo sync -c

while [ $? -ne 0 ] ; 
do  
	 ../repo/repo sync -c ; 
done
```
## Linux_SDK Directory
```
├── linux_sdk
│   ├── app
│   ├── buildroot                                            	compile buildroot rootfs directory
│   ├── build.sh -> device/rockchip/common/build.sh          	automatic compile script
│   ├── device 							config file
│   ├── distro 						        debian_root.img directory
│   ├── docs 							document
│   ├── envsetup.sh -> buildroot/build/envsetup.sh		
│   ├── external
│   ├── kernel 							kernel directory
│   ├── Makefile -> buildroot/build/Makefile				
│   ├── mkfirmware.sh -> device/rockchip/common/mkfirmware.sh 	update rockdev script
│   ├── prebuilts
│   ├── rkbin 
│   ├── rkflash.sh -> device/rockchip/common/rkflash.sh 	flashing firmware script 
│   ├── rootfs 							compile debian rootfs directory
│   ├── tools 							flashing and package tools
│   └── u-boot 							uboot directory
```
## Config
Config file firefly-rk3399.mk:
```
./build.sh firefly-rk3399.mk
```
file path:`device/rockchip/rk3399/firefly-rk3399.mk` If the configuration is successful，file will be link to`device/rockchip/.BoardConfig.mk`,check this file.  

<font color=#ff0000>WARNNING:</font>`firefly-rk3399.mk` is a configuration file for compiling buildroot firmware. At the same time, users can also refer to this configuration to generate a new configuration file to adapt the firmware they need. Important configuration:(If you need diy firmware, you may need to modify the following configuration information)
```
# Uboot defconfig
export RK_UBOOT_DEFCONFIG=firefly-rk3399  	
# Kernel defconfig
export RK_KERNEL_DEFCONFIG=firefly_linux_defconfig   
# Kernel dts
export RK_KERNEL_DTS=rk3399-firefly   
# parameter for GPT table
export RK_PARAMETER=parameter-ubuntu.txt   
# packagefile for make update image 
export RK_PACKAGE_FILE=rk3399-ubuntu-package-file   
# rootfs image path
export RK_ROOTFS_IMG=xxxx/xxxx.img   
```
## parameter & package-file
**parameter:** `parameter.txt` contains the partition information of the firmware. You can find some `parameter.txt` files in the `device/rockchip/rk3399` directory. The following is an example of `parameter-debian.txt`:
```
FIRMWARE_VER: 8.1 
MACHINE_MODEL: RK3399
MACHINE_ID: 007 
MANUFACTURER: RK3399
MAGIC: 0x5041524B
ATAG: 0x00200800
MACHINE: 3399
CHECK_MASK: 0x80
PWR_HLD: 0,0,A,0,1
TYPE: GPT 
CMDLINE: mtdparts=rk29xxnand:0x00002000@0x00004000(uboot),0x00002000@0x00006000(trust),0x00002000@0x00008000(misc),0x00010000@0x0000a000(boot),0x00010000@0x0001a000(recovery),0x00010000@0x0002a000(backup),0x00020000@0x0003a000(oem),0x00700000@0x0005a000(rootfs),-@0x0075a000(userdata:grow)
uuid:rootfs=614e0000-0000-4b53-8000-1d28000054a9
```
we care CMDLINE. Take uboot as an example. `0x00002000@0x00004000(uboot)` in `0x00004000` is the starting position of the uboot partition `0x00002000` is the size of the partition. BehindThe partitioning rules are the same.  

Users can modify the partition information according to their needs, but please keep at least the uboot, trust, boot, rootfs partition, which is the prerequisite for the machine to start normally. This is the simplest partitioning scheme used in `parameter-ubuntu.txt.`

partition:
```
uboot : flashing uboot.img.
trust : flashing trust.img.
misc  : flashing misc.img,responsible for starting the mode switch and first aid mode parameter transfer.
boot  : flashing boot.img,contain kernel.img and resource.img
recovery : flashing recovery.img.recovery mode image.
backup : Reserved
oem   : flashing oem.img,Used by the manufacturer to store the app or data. Read only. Mounted in the /oem directory.
rootfs : flashing rootfs.img.
userdata : flashing userdata.img mount /userdata directory.
```
**package-file:** This file should be consistent with `parameter.txt` for firmware packaging. The relevant files can be found in `tools/linux/Linux_Pack_Firmware/rockdev`. Take `rk3399-ubuntu-package-file` as an example:
```
# NAME          Relative path
#
#HWDEF          HWDEF
package-file    package-file
bootloader      Image/MiniLoaderAll.bin
parameter       Image/parameter.txt          
trust           Image/trust.img
uboot           Image/uboot.img
boot            Image/boot.img
rootfs:grow     Image/rootfs.img
backup          RESERVED
```
Only package the img file you use according to `parameter.txt`.
## Compile Environment
buildroot:
```
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-
compiler gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-
tools linaro-image-tools autoconf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-
client subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo
liblz4-tool genext2fs lib32stdc++6
```
debian,ubuntu:
```
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-
compiler gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-
tools linaro-image-tools gcc-4.8-multilib-arm-linux-gnueabihf gcc-arm-linux-gnueabihf libssl-dev
gcc-aarch64-linux-gnu g+conf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-
client subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo
liblz4-tool genext2fs lib32stdc++6
```
## Automatic Compile
After all the above work is completed:
```
./build.sh
```
Automatic compile default build buildroot rootfs. Generate the firmware directory rockdev/ and back it up in IMAGE.
## Compile separately
### kernel
```
./build.sh kernel
```
### u-boot
```
./build.sh u-boot
```
### recovery
```
./build.sh recovery
```
### rootfs

* buildroot:
```
./build.sh rootfs
```
* debian:

```
cd rootfs/
1:
#Building base debian system by ubuntu-build-service from linaro
sudo apt-get install binfmt-support qemu-user-static live-build
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f

2:
#Compile 32-bit debian:
RELEASE=stretch TARGET=desktop ARCH=armhf ./mk-base-debian.sh
#Compile 64-bit debian:
RELEASE=stretch TARGET=desktop ARCH=arm64 ./mk-base-debian.sh

# If you encounter the following problem
noexec or nodev issue /usr/share/debootstrap/functions: line 1450: ..../rootfs/ubuntu-build-
service/stretch-desktop-armhf/chroot/test-dev-null: Permission denied E: Cannot install into target
'/home/foxluo/work3/rockchip/rk_linux/rk3399_linux/rootfs/ubuntu-build-service/stretch-
desktop-armhf/chroot' mounted with noexec or nodev
# Solution: 
mount -o remount,exec,dev xxx (xxx is the mount place), then rebuild it.

3:
# Compile 32-bit debian:
VERSION=debug ARCH=armhf ./mk-rootfs-stretch.sh
# Compile 64-bit debian:
VERSION=debug ARCH=arm64 ./mk-rootfs-stretch-arm64.sh

4:
./mk-image.sh
mv linaro-rootfs.img ../distro/

5:
# Modify firefly-rk3399.mk
vim device/rockchip/rk3399/firefly-rk3399.mk

# Modify RK_ROOTFS_IMG 
RK_ROOTFS_IMG=distro/linaro-rootfs.img
```
* ubuntu: Download official [rootfs](https://pan.baidu.com/s/1DuCzTGARDi7APxyKs9Nl1A#list/path=%2F)  

Download、Decompression
```
tar -xvf rk3399_ubuntu18.04_LXDE.img.tgz
```
linux-sdk/ubunturootfs
```
mkdir ubunturootfs
mv rk3399_ubuntu18.04_LXDE.img ubunturootfs/
```
Modify firefly-rk3399-ubuntu.mk
```
vim device/rockchip/rk3399/firelfy-rk3399.mk
#Modify RK_ROOTFS_IMG
RK_ROOTFS_IMG=ubunturootfs/rk3399_ubuntu18.04_LXDE.img
```

<font color=#ff0000>WARNNING:</font>Ubuntu root file system image storage path can not be wrong  
Excuting `./mkfirmware.sh` to update `rockdev/rootfs.img` link.

### Synchronously image

Make sure the file link in the `rockdev/` directory is correct before packaging the firmware:
tree
```
├── boot.img -> ~/project/linux_sdk/kernel/boot.img
├── idbloader.img -> ~/project/linux_sdk/u-boot/idbloader.img
├── linaro-rootfs.img
├── MiniLoaderAll.bin -> ~/project/linux_sdk/u-boot/rk3399_loader_v1.14.115.bin
├── misc.img -> ~/project/linux_sdk/device/rockchip/rockimg/wipe_all-misc.img
├── oem.img
├── parameter.txt -> ~/project/linux_sdk/device/rockchip/rk3399/parameter-ubuntu.txt
├── recovery.img -> ~/project/linux_sdk/buildroot/output/rockchip_rk3399_recovery/images/recovery.img
├── rootfs.img -> ~/project/linux_sdk/ubunturootfs/rk3399_ubuntu18.04_LXDE.img
├── trust.img -> ~/project/linux_sdk/u-boot/trust.img
├── uboot.img -> ~/project/linux_sdk/u-boot/uboot.img
└── userdata.img
```
Excuting `./mkfirmware.sh` to update link
```
./mkfirmware.sh
```

## Pack
<font color=#ff0000>WARNNING：</font>Please confirm that tools/linux/Linux_Pack_Firmware/rockdev/package-file is correct before packaging. Image will be partitioned and packaged according to this file. This file link will be updated when the ./b uild.sh roc-rk3399-pc.mk command is used. If the configuration is incorrect, please return to the [Configuration] section to reconfigure it.
```
./build.sh updateimg
```
## Flashing
please follow [《Flash Image 》]()Flash Image  

<font color=#ff0000>WARNNING：</font>Please clearly understand the type of firmware, find a suitable method to flashing
