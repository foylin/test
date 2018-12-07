# Screen Module
## 7.85-Inches EDP Display Module
### Product Parameter
* Model: LP079QX1
* Size: 7.85-inches
* Resolution: 2048x1563
* Interface: EDP
* Panel Type: IPS
* Viewing Angle: 160°
* Touch Screen: Capacitive multi-touch

### Firmware follow
<font color=#ff0000>The name of official firmware to support the EDP7.85 Display Module with the words "EDP785", the firmware link as followː</font>
[1]
### Compilation command
Using official SDK to compile firmware that support 7.85 inch screen firmware need the following command：
* Android 6.0
```
 ./FFTools/make.sh -j8 -d rk3399-firefly-edp
 ./FFTools/mkupdate/mkupdate.sh
```
* Android 7.1
```
 ./FFTools/make.sh -j8 -d rk3399-firefly-edp -l rk3399_firefly_edp_box-userdebug
 ./FFTools/mkupdate/mkupdate.sh -l rk3399_firefly_edp_box-userdebug
```

### Datasheet
[Datasheet and schematic of 7.85 inch edp lcd module](https://pan.baidu.com/s/1mhXH4fu#list/path=%2F)
### Picture
![](img/EDP785_en.jpg)
### Connection Method
![](img/EDP785_connect_en.jpg)
