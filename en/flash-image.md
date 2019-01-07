# Flash Image
## Introduction
This page describes how to flash the image file from the host to the development board's eMMC, via the Type-C cable or USB OTG + Type-C adapter(Hereinafter referred as Type-C).

Please choose the right way according to the host OS and the type of image file .
## Before Start
What you need:
 * Firefly RK3399 development board
 * Image file
 * Host PC
 * Type-C Cable / Dual male usb data cable  

There are two types of image file:  

 * Packed image, often known as update.img, which contains the bootloader, parameter and all the partition image files. It is used for firmware release.
 * Partition image, like kernel.img, boot.img, recovery.img, etc, which are created during development.  

You can find the compiled unified [RK3399 firmware](http://en.t-firefly.com/doc/download/3.html#other_14) here, and extract it after downloading. You can also compile it yourself by referring to the instructions for compiling the firmware.  

Supported host OS:
 * Windows XP （32/64bit
 * Windows 7 (32/64bit)
 * Windows 8 (32/64bit)
 * Linux (32/64bit)

## Windows
### Install RK USB Driver
Download [Release_DriverAssistant.zip](https://drive.google.com/file/d/0B7HO8lbGgAqAQ0VORkxoR0RmSzA/view), then "驱动安装"(Driver install).), uncompress it, then run DriverInstall.exe inside.  

In order to use new driver for all the rockchip devices, please select "驱动卸载"(Driver uninstall), then "驱动安装"(Driver install).  

![](img/upgrade_firmware1.png)

### Connect Device
Firefly-RK3399  

There are two ways to switch the device to upgrade mode.  

The first one, device is cut off all the power sources, such as power adapter and Type-C connection:
 1. Keep Type-A cable connected with host PC.
 2. Press and hold RECOVERY key.
 3. Connect Type-C cable to the device.
 4. After around two seconds, release RECOVERY key.    

The other way:

 1. Use Type-C cable to connect host and device together.
 2. Press and hold RECOVERY key.
 3. Shortly press RESET key.
 4. After around two seconds, release RECOVERY key.
  
The host will prompt to have new device detected and configured. Open the Device Management, you'll find a new device name "Rockusb Device", as shown below. Return to previous step to reinstall driver if it is not shown.

![](img/upgrade_firmware2.png)

### Flash Image
Download [AndroidTool](http://en.t-firefly.com/doc/download/3.html#windows_12)(Android8.1 need AndroidTool_Relase_v2.54 or higher version). Uncompress it and change to directory AndroidTool_Release_v2.38.  

AndroidTool defaults to display in Chinese. We need to change it to English. Open config.ini with an text editor (like notepad). The starting lines are:
```
#选择工具语言:Selected=1(Chinese);Selected=2(English)
[Language]
Kinds=2
Selected=1
LangPath=Language\
```
Change "Selected=1" to "Selected=2", and save. From now on,  AndroidTool will display in English.  
Now, run AndroidTool.exe: (Note: If using Windows 7/8, you'll need to right click it, select to run it as Administrator)

![](img/upgrade_firmware3.png)

### Flash update.img
Steps of flashing update.img:
 1. Switch to "Upgrade Firmware" tab page.
 2. Click "Firmware" button and open the image file. Detail information of the image file, like version and chip, is shown.
 3. Click "Upgrade" button to start flash.
 4. <font color=#ff0000>If the upgrade fails, you can try to erase the Flash by pressing the "EraseFlash" button before upgrading. Be sure to erase and upgrade according to the [Flashing Notes](flashing-notes.html).</font>

**WARNING: If you flash firmware laoder different version of the original machine, please click "Erase Flash" before upgrading the firmware.**

![](img/upgrade_firmware4.png)

### Flash partition image
The partition of each firmware may be different. Please note the following two points:
* Use Androidtool_2.38 to upgrade ubuntu (MBR) and Android7.1 firmware using the default configuration;
* Use Androidtool_2.58 to upgrade ubuntu (GPT) to use the default configuration. To upgrade Android8.1 firmware, please do the following:

<font color=#ff0000>Switch to "Download Page"; right click on the form and select "Load Config"; select rk3399-Android81.cfg</font>  

Steps of flashing partition images:
 1. Switch to "Download Image" tab page.
 2. Check the partitions you want.
 3. Make sure the image file's path is correct. Click the rightmost empty table cell to select new path if needed.
 4. Click "Run" button to start flashing. Device will reboot automatically when finish.

![](img/upgrade_firmware3.png)

## Linux
There is no need to install a device driver under Linux. You can connect to the device by referring to the Windows chapter.
* Tool:[upgrade_tool_xxxx(version number)](http://en.t-firefly.com/doc/download/3.html#linux_12)  

**Note:** The version of the tool used by different firmware may be different. Please download the corresponding version according to the [Flashing Notes](flashing-notes).

### Upgrade_tool
Download [Linux_Upgrade_Tool](http://en.t-firefly.com/doc/download/3.html#linux_12)(Android8.1 need Linux_Upgrade_Tool_for_android8.1), and install it to host filesystem:
```
   unzip Linux_Upgrade_Tool_xxxx.zip
   cd Linux_UpgradeTool_xxxx
   sudo mv upgrade_tool /usr/local/bin
   sudo chown root:root /usr/local/bin/upgrade_tool
   sudo chmod a+x /usr/local/bin/upgrade_tool
```
<font color=#ff0000>NOTE: If you are prompted with the following error:</font>
```
upgrade_tool: error while loading shared libraries: libudev.so.1: cannot open shared object file: No such file or directory
```
You can use the following command to solve：
```
sudo ln -sf /lib/i386-linux-gnu/libudev.so.1 /lib/i386-linux-gnu/libudev.so.0
```
### Flashing firmware:
```
sudo upgrade_tool uf update.img #update.img:Firmware to be upgraded
```
If the upgrade fails, you can try to erase and then upgrade. Be sure to erase and upgrade according to the table of [Flashing Notes](flashing-notes.html).
```
# erase flash
sudo upgrade_tool ef update.img #update.img:Firmware to be upgraded
# flashing
sudo upgrade_tool uf update.img 
```
### Flashing partition images
Ubuntu(MBR)、Android7.1、Android8.1,Use the following way:
```
sudo upgrade_tool di -b /path/to/boot.img
sudo upgrade_tool di -k /path/to/kernel.img
sudo upgrade_tool di -s /path/to/system.img
sudo upgrade_tool di -r /path/to/recovery.img
sudo upgrade_tool di -m /path/to/misc.img
sudo upgrade_tool di resource /path/to/resource.img
sudo upgrade_tool di -p paramater   #flash parameter
sudo upgrade_tool ul bootloader.bin #flash bootloader
sudo upgrade_tool di trust /path/to/trust.img #flash trust
```
Ubuntu(GPT),Use the following way:
```
sudo upgrade_tool ul $LOADER
sudo upgrade_tool di -p $PARAMETER
sudo upgrade_tool di -uboot $UBOOT
sudo upgrade_tool di -trust $TRUST
sudo upgrade_tool di -b $BOOT
sudo upgrade_tool di -r $RECOVERY
sudo upgrade_tool di -m $MISC
sudo upgrade_tool di -oem $OEM
sudo upgrade_tool di -userdata $USERDATA
sudo upgrade_tool di -rootfs $ROOTFS
```
If errors occur due to flash problem, you can try to low format, or erase the flash:
```
upgrade_tool lf update.img # low format flash
upgrade_tool ef update.img # erase flash
```
## FAQS
### How to enter MaskRom mode
A1: If Loader mode is not available , you might need to enforce the device into MaskRom mode. Please check [here](maskrom.html).
