
# 入手指南
## 配件

Firefly-RK3399 的标准套装包含以下配件：

* Firefly-RK3399 主板一块
* 12V-2A电源适配器一个

另外可以选购的配件有：

* Firefly串口模块

另外，在使用过程中，你可能需要以下配件：

*    显示设备
     * 带 HDMI 接口的显示器或电视，及 HDMI 连接线
*    网络
     *   100M/1000M 以太网线缆，及有线路由器
     *   WiFi 路由器
*    输入设备
     *   USB 无线/有线的鼠标/键盘
     *   红外遥控器(需要接上红外接收器)
*    升级固件，调试
     *   Type-C数据线
     *   串口转 USB 适配器.
*    发货清单参考

![](img/started1.jpg)

* 安装方法

![](img/started2.jpg)

Firefly-RK3399 支持从以下存储设备启动：

* SD 卡
* eMMC

我们需要将系统固件烧写到 SD 卡或 eMMC 里，这样开发板上电后才能正常启动进入操作系统。

 <a id="firmware-format"></a>
## 固件类型

固件有两种格式：

* 原始固件(raw firmware)
* RK固件(Rockchip firmware)

<a id="raw-firmware-format"></a>
[原始固件]是一种能以逐位复制的方式烧写到存储设备的固件，是存储设备的原始映像。原始固件一般烧写到SD卡中，但也可以烧写到eMMC中。烧写原始固件有许多工具可以选用：

<a id="rk-firmware-format"></a>
[RK固件]是以Rockchip专有格式打包的固件，使用Rockchip提供的upgrade_tool(Linux)或AndroidTool(Windows)工具烧写到eMMC闪存中。RK固件是Rockchip的传统固件打包格式，常用于Android设备上。另外，Android的RK固件也可以使用SD Firmware Tool工具烧写到SD卡中。

<a id="partition-image"></a>
[分区映像]是分区的映像数据，用于存储设备对应分区的烧写。例如，编译Android SDK会构建出`boot.img`、`kernel.img`和`system.img`等分区映像文件，`kernel.img`会被写到eMMC或SD卡的“kernel”分区。

## 下载和烧写固件

以下是支持的系统列表：
* Android 7.1
* Android 8.1
* Ubuntu 16.04
* Ubuntu 18.04
* Debian 9

根据所使用的操作系统来选择合适的工具去烧写固件：

- 烧写SD卡
  + 图形界面烧写工具：
	* [Etcher] (windows/linux/Mac)
  + 命令行烧写工具
	* [dd] (Linux)
- 烧写eMMC
  + 图形界面烧写工具：
	* [AndroidTool] (Windows)
  + 命令行烧写工具：
	* [upgrade_tool] (Linux)

## 开机
确认主板配件连接无误后，将电源适配器插入带电的插座上，电源线接口插入开发板，开发板第一次加电会自动开机。 在 Android 系统选择关机后，维持开发板供电，此时 Firefly-RK3399 有两种开机方式：

*    长按电源键三秒

*    按红外遥控器上的开机按钮

开机时，蓝色的电源指示灯会亮起。如果板子接了HDMI显示器，可以看到Firefly 官方logo.

## FAQs
### Q1：如何调整HDMI输出分辨率？

A1:Firefly-RK3399 的 HDMI 能自动识别显示的分辨率。假如无法读取显示器的 EDID (Extended Display Identification Data, 扩展显示器标识数据) ，HDMI 会默认设置为 1080P 的分辨率。你也可以进入系统设置对 HDMI 分辨率进行手动调整。
 
 ![](img/started3.jpg)
 
### Q2:开机异常并循环重启怎么办？

A2:有可能是电源电流不够，请使用电压为12V，电流为2.5A~3A的电源。
### Q3:ubuntu默认的用户名和密码是什么？
A3:用户名 firefly  密码 firefly 

切换超级用户  sudo -s
### Q4:Git链接地址？

A4:[https://gitlab.com/TeeFirefly/FireNow-Nougat](https://gitlab.com/TeeFirefly/FireNow-Nougat)
### Q5:哪里有RK3399 芯片技术手册？
A5:RK3399 芯片技术手册链接：[Brief](http://www.t-firefly.com/download/Firefly-RK3399/docs/Chip%20Specifications/Rockchip_RK3399_Datasheet_V0.7_20160219.pdf) [Part1](http://www.t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part1.pdf) [Part2](http://www.t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part2.pdf)
### Q6:怎么烧写MAC地址？
A6:MAC地址可以让用户自己更改，请先进入升级模式（参照升级固件中连接设备的方法），后使用工具 [WNpctool](https://pan.baidu.com/s/1kU727kF#list/path=%2F) 烧写MAC地址。
### Q7:Ubuntu系统，如果插入耳机后，没有声音，该如何处理？
A7:Menu->Multimedia->PulseAudio Volume Control->Configuration->选择正在工作的声卡，关闭另一个声卡

[RK固件]:started.html#rk-firmware-format
[原始固件]:started.html#raw-firmware-format
[分区映像]:started.html#partition-image
[固件类型]:started.html#firmware-format
[SD Firmware Tool]:upgrade_firmware_rk.html#SD_Firmware_Tool
[Etcher]:upgrade_firmware_sd.html#Etcher
[dd]:upgrade_firmware_sd.html#dd
[AndroidTool]:upgrade_firmware.html#Androidtool
[upgrade_tool]:upgrade_firmware.html#upgrade_and_upgrade_tool

## [《FAQs 常见问题》](faqs.html)

