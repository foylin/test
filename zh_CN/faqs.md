# FAQs

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

### Q8:Android下如何让系统抓取LOG？
A8:Settings(设置)->About phone(关于手机)->点击5下Build number(版本号)->Developer options(开发者选项)->Enable logging to save(启用日志保存)
打开功能后，系统的storage根目录下就会生成.LOGSAVE文件夹，里面包括系统logcat和内核kmsg。
