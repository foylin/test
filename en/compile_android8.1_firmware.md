# Build Android 8.1

## Preparation
To compile Android, requirement of PC is:
* 64 bit CPU
* 16GB physical memory + swap memory
* 30GB free disk space to build. The source code tree take another 8GB space.

64 bit Ubuntu 12.04 is officially recommended. Yet newer  64 bit Linux OS is also supported, providing the requirements of software and hardware in [http://source.android.com/source/building.html](http://source.android.com/source/building.html) are met.  
To initialize compiling environment, please reference [http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.htm) .
* Install OpenJDK 8:
```
   sudo add-apt-repository ppa:openjdk-r/ppa
   sudo apt-get update
   sudo apt-get install openjdk-8-jdk
```
* Ubuntu 12.04 packages install:
```
 sudo apt-get install git gnupg flex bison gperf build-essential \
 zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
 libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
 g++-multilib mingw32 tofrodos gcc-multilib ia32-libs \
 python-markdown libxml2-utils xsltproc zlib1g-dev:i386
```
* Ubuntu 13.10/14.04 packages install:
```
 sudo apt-get install git-core gnupg flex bison gperf libsdl1.2-dev \
 libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl \
 libncurses5-dev zlib1g-dev pngcrush schedtool libxml2 libxml2-utils  \
 xsltproc lzop libc6-dev schedtool g++-multilib lib32z1-dev lib32ncurses5-dev \
 lib32readline-gplv2-dev gcc-multilib libswitch-perl
```
* Install ARM cross compiling toolchain and related package to compile Linux kernel:
```
 sudo apt-get install gcc-arm-linux-gnueabihf \
 lzop libncurses5-dev \
 libssl1.0.0 libssl-dev
```

## Download Android SDK

The size of SDK is huge. Please download Firefly-RK3399_Android8.1_git_SDK_20180901.7z.001/002 from the following cloud storage:
* Android8.1 SDK GoogleDriver Download

Please check the md5 checksum before proceeding:
```
$ md5sum /path/to/Firefly-RK3399_Android8.1_git_SDK_20180901.7z.001
$ md5sum /path/to/Firefly-RK3399_Android8.1_git_SDK_20180901.7z.002
8efed8db2687a395df0c0d2791202cc0  Firefly-RK3399_Android8.1_git_SDK_20180901.7z.001
05b5db991bbb9b763d330fa476f2ed76  Firefly-RK3399_Android8.1_git_SDK_20180901.7z.002
```
If it is correct, uncompress it:
```
cd ~/proj/
7z x ./Firefly-RK3399_Android8.1_git_SDK_20180901.7z.001 -oFirefly-rk3399
cd ./Firefly-rk3399
git reset --hard
```
<font color=#ff0000>Attention: You must first update the remote repository after decompression .</font>  
From now on, you can pull from gitlab:
```
git remote add gitlab https://gitlab.com/TeeFirefly/firenow-oreo-rk3399.git
git pull gitlab firefly-rk3399:firefly-rk3399
```
You can also browse the source codes online at [https://gitlab.com/TeeFirefly/firenow-oreo-rk3399]([https://gitlab.com/TeeFirefly/firenow-oreo-rk3399)
Use Firefly's script to compile
* Only compile the kernel:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -k -j8
```
* Only compile the Uboot:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -u -j8
```
* Only compile the Android:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -a -j8
```
* Compile Ubooot, Kernel, Android:
```
cd ~/proj/firefly-rk3399/
./FFTools/make.sh -j8
```

## Product Firefly-RK3399 complie
### Default Compilation ： HDMI+DP
```
./FFTools/make.sh -j8
./FFTools/mkupdate/mkupdate.sh
```
### Compile EDP7.85
```
./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_mid-userdebug
./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_mid-userdebug
```
## Manual compilation
### Firefly-RK3399

Before compiling, execute the following command:
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```
* Compile Kernel：
```
cd ~/proj/firefly-rk3399/kernel/
make ARCH=arm64 firefly_defconfig
make -j8 ARCH=arm64 rk3399-firefly.img
```
* Compile Uboot：
```
cd ~/proj/firefly-rk3399/u-boot/
make rk3399_defconfig
make ARCHV=aarch64 -j8
```
* Compile Android：
```
cd ~/proj/firefly-rk3399/
source build/envsetup.sh
lunch rk3399_firefly_mid-userdebug
make -j8
./mkimage.sh
```

### Create update.img

Compiled with Firefly's script can be packaged into update.img, run: `./FFTools/mkupdate/mkupdate.sh` update After the package is finished, the update.img will be generated under `rockdev/Image-rk3399_firefly_mid/`  
Create update.img in Windows is simple. Just copy the files to AndroidTool's `rockdev\Image` directory as previous step. Then run the batch file `mkupdate.bat` in rockdev directory, which will create update.img under `rockdev\Image`.
### Flashing partition images
`./mkimage.sh` at previous step will repack boot.img and system.img, and copy other related image files to the `rockdev/Image-rk3399_firefly_mid/` directory. The common image files are listed below:
* boot.img : Android's initramfs, to initialize and mount system partition.
* kernel.img : Kernel image.
* misc.img : Misc partition image, to switch boot mode and pass parameter in recovery mode.
* recovery.img : Recovery mode image.
* resource.img : Resource image, containing boot logo and kernel's device tree info.
* system.img : System partition image with ext4 filesystem format.
* trust.img ：File about sleep
* rk3399_loader_v1.12.112.bin ：Loader
* uboot.img ：uboot
* oem : preset package
* vendor : driver package of the vendor

Please flash the image according to [Flash image](flash-image.html).  
If you are using Windows system to flash the images, please copy the image files mentioned above to `rockdev\Image` directory in AndroidTool (Image flash tool in Windows). Then follow instructions in the flash image guide. This is the easy way, using default configuration, no need to modify image file's path.  
update.img is convenient to distribute to end users to upgrade the system, while partition images are more frequently used during development.