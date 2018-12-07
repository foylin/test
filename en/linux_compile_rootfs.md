# Compile Ubuntu rootfs
## Before You Start
### install qemu
intall qemu in your computer
```
sudo apt-get install qemu-user-static
```
### Download & Decompression the ubuntu-core
Firefly-rk3399 create the root-file-system by the Ubuntu base 16.04. You can download from here [ubuntu cdimg download](http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/). please choose this one ： `ubuntu-base-16.04.1-base-arm64.tar.gz` .  

after download，create a temporary folder for decompression.
```
mkdir temp
sudo tar -xpf ubuntu-base-16.04.1-base-arm64.tar.gz -C temp
```
## Optimized the rootfs
### Preparations
get your network readyː
```
sudo cp -b /etc/resolv.conf temp/etc/resolv.conf
```
prepare qemu
```
sudo cp /usr/bin/qemu-aarch64-static temp/usr/bin/
```
Change root
```
sudo chroot temp
```
### update & install
update：
```
apt update
apt upgrade
```
install you need
```
apt install vim git...(something you like)
```
install xubuntu
```
apt install xubuntu-desktop
```
### user & password
```
useradd -s '/bin/bash' -m -G adm,sudo firefly
```
set user's password
```
passwd firefly
```
set the root user's password
```
passwd root
```
After all the work is done,exit
```
exit
```
## make the rootfs
make the rootfs,notice that you need change the "count" value according to the size of the "temp" folder.
```
dd if=/dev/zero of=linuxroot.img bs=1M count=2048
sudo  mkfs.ext4  linuxroot.img
mkdir  rootfs
sudo mount linuxroot.img rootfs/
sudo cp -rfp temp/*  rootfs/
sudo umount rootfs/
e2fsck -p -f linuxroot.img
resize2fs  -M linuxroot.img
```
The final root filesystem linuxroot.img is ready to serve.
### Something more
when the system is ready, remember resize the rootfs partition by
```
resize2fs /dev/mtd/by-name/linuxroot
```
