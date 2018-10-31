 # 升级固件

## 前言

本文介绍了如何将主机上的固件文件，通过Type-C数据线，烧录到开发板的闪存中。   
升级时，需要根据主机操作系统和固件类型来选择合适的升级方式。

## 准备工作

* FireFly-RK3399 开发板
* 固件
* 主机
* 良好的Type-C数据线

固件文件一般有两种：

* 单个统一固件 update.img, 将启动加载器、参数和所有分区镜像都打包到一起，用于固件发布。
* 多个分区镜像,如 kernel.img, rootfs.img, recovery.img 等，在开发阶段生成。
* 可以在这里找到已编译好的统一[[RK3399固件]](http://www.t-firefly.com/doc/download/page/id/3.html#other_14)，下载后解压。也可以参考编译固件的说明自行编译。

主机操作系统支持：
> * Windows XP （32/64位）
> * Windows 7 (32/64位)
> * Windows 8 (32/64位)
> * Linux (32/64位)

## Windows

之前烧写 RK 的固件，需要用到以下两种工具：

* 量产工具 RKBatchTool，用于烧写统一固件（update.img）
* 开发者工具 RKDevelopTool，可单独烧写分区固件

后来 RK 发布了 AndroidTool 工具，在 RKDevelopTool 的基础上增加了统一固件（update.img）的烧写支持，因此现在仅需要这个工具即可。   
使用烧写工具前需要安装 RK USB 驱动。如果驱动已经安装好，可以跳过这步。

### 安装 RK USB 驱动

下载 [ Release_DriverAssistant.zip](http://www.t-firefly.com/doc/download/page/id/3.html#other_11) ，解压，然后运行里面的 DriverInstall.exe 。   
为了所有设备都使用更新的驱动，请先选择"驱动卸载"，然后再选择"驱动安装"。   

![](img/upgrade_firmware1.png)

### 连接设备

有两种方法可以使设备进入升级模式

* 一种方法是设备先断开电源适配器和Type-C数据线的连接：
   * USB数据线一端连接主机，Type-C一端连接开发板Type-C母口。
   * 按住设备上的 RECOVERY （恢复）键并保持。
   * 接上电源
   * 大约两秒钟后，松开 RECOVERY 键。

*    另一种方法，无需断开电源适配器和Type-C数据线的连接：
     * USB数据线一端连接主机，Type-C一端连接开发板Type-C母口。
     * 按住设备上的 RECOVERY （恢复）键并保持。
     * 短按一下 RESET（复位）键。
     * 大约两秒钟后，松开 RECOVERY 键

主机应该会提示发现新硬件并配置驱动。打开设备管理器，会见到新设备"Rockusb Device" 出现，如下图。如果没有，则需要返回上一步重新安装驱动。   

![](img/upgrade_firmware2.png)

## 烧写固件

下载 [AndroidTool](http://www.t-firefly.com/doc/download/page/id/3.html#windows_12)(**若系统是Android8.1则需要2.54以上版本**)，解压，运行 AndroidTool_Release_v2.38 目录里面的 AndroidTool.exe（注意，如果是 Windows 7/8,需要按鼠标右键，选择以管理员身份运行），如下图：   
![](img/upgrade_firmware3.png)

### 烧写统一固件 update.img

烧写统一固件 update.img 的步骤如下:

1. 切换至"升级固件"页。
2. 按"固件"按钮，打开要升级的固件文件。升级工具会显示详细的固件信息。
3. 按"升级"按钮开始升级。
4. 如果升级失败，可以尝试先按"擦除Flash"按钮来擦除 Flash，然后再升级。

**注意：如果你烧写的固件laoder版本与原来的机器的不一致，请在升级固件前先执行"擦除Flash"。**   

![](img/upgrade_firmware4.png)

### 烧写分区映像

烧写分区映像的步骤如下：

1. 切换至"下载镜像"页。
2. 勾选需要烧录的分区，可以多选。
3. 确保映像文件的路径正确，需要的话，点路径右边的空白表格单元格来重新选择。
4. 点击"执行"按钮开始升级，升级结束后设备会自动重启。

![](img/upgrade_firmware3.png)

## Linux

RK 提供了一个 Linux 下的命令行工具 upgrade_tool，支持统一固件 update.img 和分区镜像的烧写。

开源工具则有两个选择:

* [rkflashtool](https://github.com/Galland/rkflashtool_rk3066)

* [rkflashkit](https://github.com/linuxerwang/rkflashkit)

它们都仅支持分区映像烧写，不支持统一固件。rkflashtool 是命令行工具，rkflashkit 有图形界面，后加了命令行支持，更是好用。以下仅对 rkflashkit 做介绍。   
*** Linux 下无须安装设备驱动，参照 Windows 章节连接设备则可。***

### upgrade_tool

下载 [Linux_Upgrade_Tool](http://www.t-firefly.com/doc/download/page/id/3.html#linux_12)(**系统是Android8.1则需要Linux_Upgrde_Tool_for_android8.1**), 并按以下方法安装到系统中，方便调用：   
```
unzip Linux_Upgrade_Tool_v1.24.zip
cd Linux_UpgradeTool_v1.24
sudo mv upgrade_tool /usr/local/bin
sudo chown root:root /usr/local/bin/upgrade_tool
```

烧写统一固件 update.img：   
```
sudo upgrade_tool uf update.img
```

烧写分区镜像：   
```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di resource /path/to/resource.img
sudo upgrade_tool di -p paramater   #烧写 parameter
sudo upgrade_tool ul bootloader.bin # 烧写 bootloader
```

如果因 flash 问题导致升级时出错，可以尝试低级格式化、擦除 nand flash：   
```
sudo upgrade_tool lf update.img	# 低级格式化
sudo upgrade_tool ef update.img	# 擦除
```
### rkflashkit

安装：   
```
sudo apt-get install build-essential fakeroot 
git clone https://github.com/linuxerwang/rkflashkit
cd rkflashkit
./waf debian
sudo apt-get install python-gtk2
sudo dpkg -i rkflashkit_0.1.4_all.deb
```

图形界面：   
```
sudo rkflashkit
```

![](img/upgrade_firmware5.png)

命令行：   
```
$ rkflashkit --help
Usage:  [args] [ [args]...]

part                              List partition
flash @    Flash partition with image file
cmp @      Compare partition with image file
backup @   Backup partition to image file
erase  @               Erase partition
reboot                            Reboot device

For example, flash device with boot.img and kernel.img, then reboot:

sudo rkflashkit flash @boot boot.img @kernel.img kernel.img reboot
```

帮助信息里有使用示例，可以看出，一条命令就可以烧写多个映像文件并重启设备，对需要经常编译和烧写内核的开发者来说，是一大福音。

## 常见问题

### 如何强行进入 MaskRom 模式

如果板子进入不了 Loader 模式，此时可以尝试强行进入 MaskRom 模式。操作方法见[《如何进入 MaskRom 模式》](maskrom_mode.html)。
