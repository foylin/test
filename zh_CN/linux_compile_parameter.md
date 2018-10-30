# 编译 Ubuntu 16.04 固件( MBR )
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
├── kernel-4.4
├── prebuilts
├── rootfs
└── u-boot
```
* 获取交叉编译工具链
```
$ git clone -b master https://github.com/T-Firefly/prebuilts.git
```
*  获取 Kernel
```
$ git clone -b firefly https://github.com/FireflyTeam/kernel.git
```
* 获取 U-Boot
```
$ git clone -b master https://github.com/FireflyTeam/u-boot.git
```
RK3399 系列也可使用 Android 7.1 u-boot源码, 配置文件 `rk3399_linux_defconfig`
* Rootfs 可通过 Android_Tool 高级功能来解包官方发布固件获取 `rootfs.img`

## 编译 Kernel
#### RK3399 系列
* 编译前执行如下命令配置环境变量：
```
$ export ARCH=arm64
# export CROSS_COMPILE=aarch64-linux-gnu-
```
*  编译 Firefly-RK3399
```
$ make firefly_linux_defconfig
$ make rk3399-firefly.img -j4
```
编译完成后，会在 kernel 目录下生成 kernel.img 和 resource.img.
* MIPI7.85 编译, 对应dts 为 rk3399-firefly-mipi.dts， 则
```
$ make rk3399-firefly-mipi.img -j4
```

#### RK3288 系列
* 编译前执行如下命令配置环境变量：
```
$ export ARCH=arm
# export CROSS_COMPILE=arm-linux-gnueabihf-
```
*  编译 firefly-rk3288
```
$ make firefly_linux_defconfig
$ make rk3288-firefly.img -j4
```
编译完成后，会在 kernel 目录下生成 kernel.img 和 resource.img.
* VGA 编译, 对应dts 为 rk3288-firefly-vga.dts， 则
```
$ make rk3288-firefly-vga.img -j4
```

## 编译 U-Boot
* RK3399 系列
```
make rk3399_linux_defconfig
./mkv8.sh
```
* RK3288 系列
```
make rk3288_defconfig
./mkv7.sh
```

## 升级分区镜像
### 查看 parameter 文件
```
CMDLINE:console=ttyS2,115200 root=/dev/mmcblk2p6 rw rootwait \
mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00002000@0x00004000(trust),\
0x00008000@0x00006000(resource),0x0000A000@0x0000E000(kernel),\
0x00002000@0x00018000(backup),-@0x0001A000(boot)
```
<font color="#dd0000">注意：</font><br />
root=/dev/mmcblk2p6         ---> rootfs 分区
```
如 根文件分区大小异常，执行
$ resize2fs /dev/mmcblk2p6

 0x0000A000@0x0000E000(kernel)        ---> kernel 分区
   |-- kernel                           --->  分区名称
   |-- 0x0000E000                       -->  分区起始地址
   |-- 0x0000A000                       --->  分区大小，一般不需要关注
 ```

### Linux upgrade_tool 升级分区镜像（ xx.img ）

<font color="#dd0000">注意：</font><br />
parameter 分区对应的名称，是否相对应
```
sudo upgrade_tool di kernel kernel.img
sudo upgrade_tool di boot rootfs.img
...

kernel                  --->  上点 parameter 所述分区名称
kernel.img              --->  编译生产的 kernel.img 分区镜像
```

### Windows Android_Tool 工具升级分区镜像
<font color="#dd0000">注意：</font><br />
parameter 分区对应的起始地址，是否相对应

![](img/linux_compile_kernel.png)

## 开发板 DTS 说明
#### RK3399 系列
* AIO-3399J
```
|--- rk3399-firefly-aio.dts
|--- rk3399-firefly-aio-lvds.dts (LVDS HSX101H40C)
```
* AIO-3399C
```
|--- rk3399-firefly-aioc.dts
|--- rk3399-firefly-aioc-lvds.dts (LVDS HSX101H40C)
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
|--- rk3288-firefly-aio.dts
|--- rk3288-firefly-aio-lvds.dts (LVDS HSX101H40C)
```
* AIO-3288C
```
|--- rk3288-firefly-aioc.dts
|--- rk3288-firefly-aioc-lvds.dts (LVDS HSX101H40C)
|--- rk3288-firefly-aioc-vga.dts (VGA)
```
* Firefly-RK3288
```
|--- rk3288-firefly.dts
|--- rk3288-firefly-vga.dts (VGA)
```

## 烧写到设备

参考《[升级固件](upgrade_firmware.html)》，选择生成的 kernel.img，resource.img 和修改过的 parameter 文件，分别烧写到 "kernel" ，"resource" 和 "parameter" 分区，则可完成内核的更新。

如果还没有烧写根文件系统的，可以下载预先做好的镜像，或定制自己的根文件系统，并烧写到 parameter 文件指定的根分区中。
