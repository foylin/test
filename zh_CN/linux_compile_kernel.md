# Ubuntu 16.04 固件
## 准备工作
### 编译环境初始化

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

### 获取内核源码

git clone -b firefly https://github.com/FireflyTeam/kernel.git

## 编译内核

* 编译前执行如下命令配置环境变量：
```
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
```
*  编译 firefly-rk3399
```
make firefly_linux_defconfig
make rk3399-firefly-linux.img -j4
```

编译完成后，会在 kernel 目录下生成 kernel.img 和 resource.img.
如需编译 arch/arm/boot/dts/rockchip/ 目录下某个 xxx.dts，则
```
make xxx.img -j4
```
## 升级分区镜像
### 查看 parameter 文件
```
CMDLINE:console=ttyS2,115200 root=/dev/mmcblk2p6 rw rootwait \
mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),0x00002000@0x00004000(trust),\
0x00008000@0x00006000(resource),0x0000A000@0x0000E000(kernel),\
0x00002000@0x00018000(backup),-@0x0001A000(boot)
```
* 注意点

root=/dev/mmcblk2p6                             ---> rootfs 分区
如 根文件分区大小异常，
```
 resize2fs /dev/mmcblk2p6
```

 0x0000A000@0x0000E000(kernel)        ---> kernel 分区
 kernel                                                        --->  分区名称
 0x0000E000                                             --->  分区起始地址
 0x0000A000                                            --->  分区大小，一般不需要关注

### Linux upgrade_tool 升级分区镜像（ xx.img ）

注意，parameter 分区对应的名称，是否相对应

```
sudo upgrade_tool di kernel kernel.img
sudo upgrade_tool di boot rootfs.img
...
```
kernel                                                         --->  上点 parameter 所述分区名称
kernel.img                                                 --->  编译生产的 kernel.img 分区镜像

### Windows AndroidTool 工具升级分区镜像

注意，parameter 分区对应的起始地址，是否相对应
![](img/linux_compile_kernel.png)

## 修改 parameter 文件

Linux 的根文件系统（RFS）可能在不同的分区或存储设备上（eMMC、TF 卡或 U 盘），所以需要在内核的参数中指定。修改 parameter 文件中的 CMDLINE 行：
```
CMDLINE:console=tty0 console=ttyS2 ... mtdparts=rk29xxnand:0x00002000@0x00002000(uboot),...,-@0x00394000(user)
```
根据实际情况加入以下之一（# 后是注释，不需要加入）：
```
root=/dev/block/mtd/by-name/linuxroot        # 名为 "linuxroot" 的 nand 分区
root=/dev/mmcblk0p1          # TF 卡的第一个分区
root=/dev/sda1               # U 盘或 USB 硬盘的第一个分区
root=LABEL=linuxroot         # 卷标为 "linuxroot" 的分区，可以是任一存储设备
```
## 烧写到设备

参考《[升级固件](upgrade_firmware.html)》，选择生成的 boot.img 和修改过的 parameter 文件，分别烧写到 "boot" 和 "parameter" 分区，则可完成内核的更新。

如果还没有烧写根文件系统的，可以下载预先做好的镜像，或定制自己的根文件系统，并烧写到 parameter 文件指定的根分区中。