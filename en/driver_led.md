# LED
## Introduction
There are 2 LEDs on the Firefly-RK3399 development board, as the following table shows:
![](img/led.png)
Both are programmable using ether LEDs class devices or GPIOs.

## LEDs as Class Devices
Linux has its own LED subsystem for LED devices. In Firefly-RK3399, 2 LEDs are configured as LED class devices.  

You can control them via `/sys/class/leds/`.  

The default status of the two on-board leds are:  
* Blue: on after the system powers on.
* Yellow: defined by user.  

You can change the behavior of each LED by using the echo command to write command to its brightness property:
```
root@rk3399_firefly_box:~ # echo 0 >/sys/class/leds/firefly:blue:power/brightness  //Blue led off
root@rk3399_firefly_box:~ # echo 1 >/sys/class/leds/firefly:blue:power/brightness  //Blue led on
```

## Using trigger control LED
Trigger contains a variety of ways to control the LED, here with two examples to illustrate

* Simple trigger LED
* Complex trigger LED

For more information, please read the document leds-class.txt .  

First of all, we need to know how many LED definition, while the corresponding property of the LED is.  

Define LED node in file  `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-port.dtsi`
```
&power_led {                             
    gpios = <&gpio2 27 GPIO_ACTIVE_HIGH>;
	};

&user_led {
    gpios = <&gpio0 13 GPIO_ACTIVE_HIGH>;
	};

leds {   
	compatible = "gpio-leds";   
	power {   
	label = "firefly:blue:power";   
	linux,default-trigger = "ir-power-click";   
	default-state = "on";   
	gpios = <&gpio2 27 GPIO_ACTIVE_HIGH>;   
	pinctrl-names = "default";   
	pinctrl-0 = <&led_power>;   
	};   
	user {   
	label = "firefly:yellow:user";   
	linux,default-trigger = "ir-user-click";   
	default-state = "off";   
	gpios = <&gpio0 13 GPIO_ACTIVE_HIGH>;   
	pinctrl-names = "default";   
	pinctrl-0 = <&led_user>;   
	};
};
```
Note: The value of .compatible must match the one in drivers/leds/leds-gpio.c.

## Simple trigger LED
By name is to see is a simple trigger mode control LED, as follows on the default open yellow LED and Firefly-RK3399 yellow LED light after the boot  
(1) Defined LED trigger In the kernel/drivers/leds/trigger/led-firefly-demo.c add the following
```
DEFINE_LED_TRIGGER(ledtrig_default_control);
```
(2) Register the trigger.
```
led_trigger_register_simple("ir-user-click", &ledtrig_default_control);
```
(3) Turn on the LED.
```
led_trigger_event(ledtrig_default_control, LED_FULL);     //yellow led on
```
（4）Enable LED demo  

led-firefly-demo is disabled in default,if you need to open the demo drive can use the following patch:
```
--- a/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi
+++ b/kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi
@@ -52,7 +52,7 @@
            led_demo: led_demo { 
-                status = "disabled";
+                status = "okay"; 
                 compatible = "firefly,rk3399-led"; 
                 };
```

## Complex trigger LED
The following is the trigger mode control LED complex example, timer trigger is to let the LED to achieve constant light off effect.  
We need to configure the timer trigger on the kernel.  

In the kernel path using make menuconfig, in accordance with the following method will timer trigger driver selected.
```
Device Drivers
--->LED Support
   --->LED Trigger support 
      --->LED Timer Trigger
```
Save the configuration and compile the kernel, the kernel.img burn Firefly-RK3399 board We can use the serial input command, you can see the blue light non-stop interval flashing
```
echo "timer" > sys/class/leds/firefly\:blue\:power/trigger
```
The user can also use the cat command to get the available values for the trigger:
```
root@rk3399_firefly_box:/ # cat sys/class/leds/firefly\:blue\:power/trigger    
none rc-feedback test_ac-online test_battery-charging-or-full test_battery-charging 
test_battery-full test_battery-charging-blink-full-solid test_usb-online mmc0 mmc1 
ir-user-click [timer] heartbeat backlight default-on rfkill0 mmc2 rfkill1 rfkill2
```
