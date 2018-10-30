# 编译 Ubuntu 16.04 固件( GPT )
## 准备工作

#### 编译环境初始化
* Ubuntu 14.04 软件包安装：
```
$ sudo apt-get install git gnupg flex bison gperf build-essential \
zip tar curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-glx:i386 \
libgl1-mesa-dev g++-multilib mingw32 cmake tofrodos \
python-markdown libxml2-utils xsltproc zlib1g-dev:i386 lzop
$ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so
```
* 安装 ARM 交叉编译工具链和编译内核相关软件包
```
$ sudo apt-get install gcc-arm-linux-gnueabihf \
gcc-aarch64-linux-gnu device-tree-compiler lzop libncurses5-dev \
libssl1.0.0 libssl-dev
```

#### Linux SDK 目录结构

   ```
$ cd linux/
$ tree
.
├── build
├── kernel
├── rkbin
└── u-boot
```
*  下载 Linux SDK
```
# build
$ git clone -b debian https://github.com/FireflyTeam/build.git
# kernel
$ git clone -b firefly https://github.com/FireflyTeam/kernel.git
# rkbin
$ git clone -b master https://github.com/FireflyTeam/rkbin.git
# u-boot
$ git clone -b release https://github.com/FireflyTeam/u-boot.git
```
* Rootfs 可通过 Android_Tool 高级功能来解包官方发布固件获取 rootfs.img
* 开发板编译配置
```
build/board_configs.sh
```
例如，Firefly-RK3399 配置如下：
```
"rk3399-firefly")
	DEFCONFIG=firefly_linux_defconfig
	UBOOT_DEFCONFIG=firefly-rk3399_defconfig
	DTB_MAINLINE=rk3399-firefly.dtb
	DTB=rk3399-firefly.dtb
	export ARCH=arm64
	export CROSS_COMPILE=aarch64-linux-gnu-
	CHIP="rk3399"
	;;
```
<font color="#dd0000">注意：</font><br /> 
gpt固件，编译打包升级步骤可参考 `ROC-RK3328-CC` 版型

## 编译 U-Boot
```
./build/mk-uboot.sh rk3399-firefly
```

## 编译 kernel
```
./build/mk-kernel.sh rk3399-firefly
```

## 打包 gpt 原始固件
```
./build/mk-image.sh -c rk3399 -t system -r out/rootfs.img
```
这条命令根据[《存储映射》]所描述的布局，将分区映像文件写到指定位置，最终打包成 `out/system.img`

## 烧写到设备
#### 安装 rkdeveloptool
首先是下载、编译和安装 rkdeveloptool
```
#install libusb and libudev
sudo apt-get install pkg-config libusb-1.0 libudev-dev libusb-1.0-0-dev dh-autoreconf
# clone source and make
git clone https://github.com/rockchip-linux/rkdeveloptool
cd rkdeveloptool
autoreconf -i
./configure
make
sudo make install
```
#### 烧写原始固件
使用 rkdeveloptool 烧写原始固件到 eMMC 的步骤如下：
1. 强制设备进入 Maskrom 模式
2. 运行以下命令：
	```
    build/flash_tool.sh -c rk3399  -p system -i out/system.img
    ```
3. 断电重启


<font color="#dd0000">注意：</font><br /> 
RK3288 系列操作步骤基本类似，不做一一说明

详细说明可参考 `ROC-RK3328-CC` 新手教程

## 开发板 DTS 说明
#### RK3399 系列
* AIO-3399J
```
|--- rk3399-firefly-aio-3399j.dts
|--- rk3399-firefly-aio-3399j-lvds.dts (LVDS HSX101H40C)
```
* AIO-3399C
```
|--- rk3399-firefly-aio-3399c.dts
|--- rk3399-firefly-aio-3399c-lvds.dts (LVDS HSX101H40C)
```
* Firefly-RK3399
```
|--- rk3399-firefly.dts
|--- rk3399-firefly-edp.dts (EDP LP079QX1)
|--- rk3399-firefly-mipi.dts (MIPI B079XAN01)
```
#### RK3288 系列
* AIO-3288J
```
|--- rk3288-firefly-aio-3288j.dts
|--- rk3288-firefly-aio-3288j-lvds.dts (LVDS HSX101H40C)
```
* AIO-3288C
```
|--- rk3288-firefly-aio-3288C.dts
|--- rk3288-firefly-aio-3288C-lvds.dts (LVDS HSX101H40C)
|--- rk3288-firefly-aio-3288C-vga.dts (VGA)
```
* Firefly-RK3288
```
|--- rk3288-firefly.dts
|--- rk3288-firefly-vga.dts (VGA)
```

[《存储映射》]: http://opensource.rock-chips.com/wiki_Partitions#Default_storage_map
