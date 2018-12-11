# Compile Ubuntu firmware(MBR)
## Before You Start
### Initialize compiling environment
* Ubuntu 14.04 packages install
```
$ sudo apt-get install git gnupg flex bison gperf build-essential \
zip tar curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
libgl1-mesa-dev g++-multilib mingw32 cmake tofrodos \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 lzop
$ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
```
* Install ARM cross compiling toolchain and related package to compile Linux kernel
```
$ sudo apt-get install gcc-arm-linux-gnueabihf \
gcc-aarch64-linux-gnu device-tree-compiler lzop libncurses5-dev \
libssl1.0.0 libssl-dev
```

## Linux SDK Directory Structure
```
$ cd linux/
$ tree
.
├── kernel-4.4
├── prebuilts
├── rootfs
└── u-boot
```
* Get the cross-compilation toolchain
```
$ git clone -b master https://github.com/T-Firefly/prebuilts.git
```
* Get the kernel
```
$ git clone -b firefly https://github.com/FireflyTeam/kernel.git
```
* Get the u-boot
```
$ git clone -b master https://github.com/FireflyTeam/u-boot.git
```
* rootfs:Download official [rootfs](http://en.t-firefly.com/doc/download/page/id/3.html#other_117)

## Compile Kernel
* Before compiling, execute the following command
```
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
```
* Compile firefly-rk3399
```
make firefly_linux_defconfig
make rk3399-firefly.img -j4
```
After the compilation is complete , the kernel.img and resource.img should be generated in the kernel folder.  

If you need to compile a xxx.dts in the `arch/arm64/boot/dts/rockchip/` directory, then
```
make xxx.img -j4
```

## Compile U-Boot
```
make rk3399_linux_defconfig
./mkv8.sh
```
## Flash partition images
### View parameter file
```
 CMDLINE:console=ttyFIQ0 root=/dev/mmcblk1p6 rw rootwait \
mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00002000@0x00004000(trust),\
0x00008000@0x00006000(resource),0x0000A000@0x0000E000(kernel),\
0x00002000@0x00018000(backup),-@0x0001A000(boot)
```
* be careful
```
root=/dev/mmcblk1p6                             ---> rootfs partition
If the size of the root file partition is abnormal,
 resize2fs /dev/mmcblk1p6
 
 0x0000A000@0x0000E000(kernel)        ---> kernel partition
 kernel                               --->  partition name
 0x0000E000                            --->  partition start address
 0x0000A000                            --->  partition size, generally do not need to pay attention
```

### Linux upgrade_tool flash partition images（ xx.img ）
Note whether the corresponding name of the parameter partition corresponds to
```
sudo upgrade_tool di kernel kernel.img
sudo upgrade_tool di boot    rootfs.img
...
kernel                                                         --->  on the parameter partition name
kernel.img                                                 --->  Compiled production kernel.img partition image
```
### Windows AndroidTool tool upgrade partition images
Note whether the starting address of the parameter partition corresponds to
![](img/linux_compile_mbr.png)
## Flash to Device
Please reference [Flash image](flash-image.html), choose the newly created boot.img and modified parameter files, flash to "boot" and "parameter" partitions respectively. Then the kernel update is done.  

If the root filesystem image is not flashed yet, you can download the prebuilt image, or customize your own one, and flash it to the partition specified in parameter file.

