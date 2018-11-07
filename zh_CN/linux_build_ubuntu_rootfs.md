# 创建 Ubuntu 根文件系统  

## 准备工作
### 安装qemu
在Linux PC主机上安装模拟器：
```
sudo apt-get install qemu-user-static
```
### 下载和解压 ubuntu-core

Firefly-rk3399 ubuntu根文件系统是基于Ubuntu base 16.04来创建的。用户可以到[ubuntu cdimg](http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/) 下载，选择下载ubuntu-base-16.04.1-base-arm64.tar.gz 。

下载完之后，创建临时文件夹并解压根文件系统：
```
mkdir temp
sudo tar -xpf ubuntu-base-16.04.1-base-arm64.tar.gz -C temp
```
## 修改根文件系统
### 准备工作

准备网络：
```
sudo cp -b /etc/resolv.conf temp/etc/resolv.conf
```
准备qemu
```
sudo cp /usr/bin/qemu-aarch64-static temp/usr/bin/
```
进入根文件系统进行操作：
```
sudo chroot temp
```
### 更新及安装

更新：
```
apt update 
apt upgrade
```
安装自己需要的功能
```
apt install vim git ....(根据自己需求添加)
```
安装xubuntu
```
apt install xubuntu-desktop
```
### 添加用户及设置密码

添加用户
```
useradd -s '/bin/bash' -m -G adm,sudo firefly
```
给用户设置密码：
```
passwd firefly
```
给root用户设置密码：
```
passwd root
```
修改完自己的根文件系统就可以退出了。
```
exit
```
## 制作根文件系统

制作自己的根文件系统，大小依据自己的根文件系统而定，注意依据temp文件夹的大小来修改count值
```
mkdir  rootfs
dd if=/dev/zero of=linuxroot.img bs=1M count=4000
mkfs.ext4 linuxroot.img
sudo mount linuxroot.img rootfs/
sudo cp -rfp temp/*  rootfs/
sudo umount rootfs/
e2fsck -p -f linuxroot.img
resize2fs  -M linuxroot.img
```
这样 linuxroot.img 就是最终的根文件系统映像文件了。
## FAQs
### 根文件系统加载后，大小不正常，未占满整个分区：

在系统正确加载后执行扩展文件系统命令：
```
 resize2fs /dev/mmcblk1p6    --> rootfs 分区

 查看 parameter文件中，root= 节点设备
```
