# FAQS
## Q1: How to adjust the HDMI output resolution?
A1: Firefly-RK3399 HDMI can automatically identify the display resolution. If you can not read the monitor's EDID (Extended Display Identification Data, extended display identification data), HDMI will default to 1080P resolution. You can also enter the system settings to adjust the HDMI resolution manually .

![](img/started3.jpg)

## Q2: What should I do when my machine is abnormally and cycled?
A2: It probably not enough supply current, please use the power adapter of 12V voltage, current is 2.5A ~ 3A .
## Q3: What is ubuntu's default username and password?
A2: Username: root  Password: firefly  
    Username: firefly  Password: firefly
## Q3: What is wiki address?
A3: http://wiki.t-firefly.com/index.php/Firefly-RK3399/en
## Q4: What is the Git link address
A4: https://gitlab.com/TeeFirefly/FireNow-Marshmallow
## Q5: Where is RK3399 datasheet?
A5: [Brief](http://www.t-firefly.com/download/Firefly-RK3399/docs/Chip%20Specifications/Rockchip_RK3399_Datasheet_V0.7_20160219.pdf) [Part1](http://www.t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part1.pdf) [Part2](http://www.t-firefly.com/download/Firefly-RK3399/docs/TRM/Rockchip%20RK3399TRM%20V1.3%20Part2.pdf)
## Q6: How to burn the MAC address?
A6: You can change MAC address of the Firefly-RK3399 by yourself, and you can modify MAC address by the tool -- WNpctool before Connect Device.
## Q7: How to catch log from the board?
 A7: Settings->About phone->click 5 times Build number->Developer options->Enable logging to save, After that, the directory .LOGSAVE will show up on the /sdcard/ director