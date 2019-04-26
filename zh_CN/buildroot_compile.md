# 编译 Buildroot 固件

本章介绍编译Buildroot固件的具体流程。

## 准备工作

### 配置编译环境

安装编译所需工具，确保工具都正确安装

```bash
sudo apt-get install repo git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler \
gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev python-linaro-image-tools \
linaro-image-tools autoconf autotools-dev libsigsegv2 m4 intltool libdrm-dev curl sed make \
binutils build-essential gcc g++ bash patch gzip bzip2 perl tar cpio python unzip rsync file bc wget \
libncurses5 libqt4-dev libglib2.0-dev libgtk2.0-dev libglade2-dev cvs git mercurial rsync openssh-client \
subversion asciidoc w3m dblatex graphviz python-matplotlib libc6:i386 libssl-dev texinfo \
liblz4-tool genext2fs lib32stdc++6
```

### 配置编译文件

选择开发板对应的配置文件。配置文件会链接到`device/rockchip/.BoardConfig.mk`，查看该文件可确认当前所使用的配置文件

```bash
./build.sh firefly-rk3399.mk

#文件路径在`device/rockchip/rk3399/firefly-rk3399.mk`
```

`.mk` 配置文件说明

```bash
# Buildroot config
export RK_CFG_BUILDROOT=rockchip_rk3399     #buildroot根文件系统配置文件

#文件路径在`buildroot/configs/rockchip_rk3399_defconfig`
```

```bash
# rootfs image path
export RK_ROOTFS_IMG=buildroot/output/$RK_CFG_BUILDROOT/images/rootfs.$RK_ROOTFS_TYPE   #buildroot根文件系统镜像路径

#此例中，文件路径在`buildroot/output/rockchip_rk3399/images/rootfs.ext4`
#注：该文件路径将在首次编译根文件系统后生成
```

## 编译固件

执行编译命令时，将会根据`.mk`文件进行编译

### 全自动编译

全自动编译会编译并打包固件`update.img`，生成固件目录`rockdev/`

```bash
./build.sh
```

### 部分编译

#### 编译 kernel

```bash
./build.sh kernel
```

#### 编译 u-boot

```bash
./build.sh uboot
```

#### 编译 rootfs

编译buildroot根文件系统，将会在`buildroot/output/$RK_CFG_BUILDROOT`生成根文件系统目录。首次编译后，也可在该目录下执行`make`进行编译

```bash
./build.sh buildroot
```

## 固件打包

### 更新链接

为确保`rockdev/`目录下文件链接正确，更新各部分镜像链接

```bash
./mkfirmware.sh
```

### 打包固件

将`rockdev`目录的各部分镜像打包成固件`update.img`

```bash
./build.sh updateimg
```