# GPIO 使用
<h4>1. 补光灯控制开关</h4>
```java
绿灯：echo X > /sys/class/leds/face:green:user/brightness 
红灯：echo X > /sys/class/leds/face:red:user/brightness
白灯：echo X > /sys/class/leds/face:white:user/brightness
    
有效值X：0为关闭，1为打开
```
<h4>2. 控制屏幕亮度</h4>
```java
屏幕亮度：echo X > /sys/class/backlight/backlight/brightness

有效值X ：0～255
```
<h4>3. 背光控制开关</h4>
```java
屏幕开关：echo X > /sys/class/backlight/backlight/bl_power
  
有效值X：0为打开，1为关闭
```
<h4>4. 继电器、韦根、485控制</h4>
```java
继电器信号 ：
	echo 0 > /sys/devices/platform/wiegand-gpio/mode_switch		切换至普通GPIO
	echo 1 >  /sys/devices/platform/wiegand-gpio/D0			拉高D0
	echo 1 >  /sys/devices/platform/wiegand-gpio/D1			拉高D1
    
韦根信号 ：
	echo 0 > /sys/devices/platform/wiegand-gpio/mode_switch		切换至韦根模式
	echo 卡号 > /sys/devices/platform/wiegand-gpio/wiegand26
        
485 ：
	echo 1 > /sys/devices/platform/wiegand-gpio/mode_switch		切换至485
	echo 内容 > /dev/ttyS4  
```
