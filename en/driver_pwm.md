# PWM
## Introduction
Firefly-RK3399 development board has 4 PWMs, namely PWM0 ~ PWM3. 4 PWMs use for eDP backlight, MIPI backlight, VDD logic , IR . This article describes how to configure and use PWM.
## Configure DTS
Add code below to turn on pwm demo .
```
      pwm_demo: pwm_demo {
              status = "okay";
              compatible = "firefly,rk3399-pwm";
              pwm_id = <1>;
              min_period = <0>;
              max_period = <10000>;
              duty_ns = <5000>;
      };
```

## Configure pwm driver
The PWM driver of RK3399 is in file `kernel/drivers/pwm/pwm-rockchip.c`.  

Modify the code as shown below:  

(1)、Include the pwm.h file  
```
#include <linux/pwm.h>
```
This file includes the API functions of PWM.    

(2)、 Request PWM.  

You can use the pwm_request function to request the PWM device. For example:
```
struct pwm_device * pwm1 = NULL;pwm1 = pwm_request(1, “firefly-pwm”);
```
See The API of PWM device for more detail of this function.    

(3)、Set PWM configuration  

You can use the function pwm_config to set the PWM configuration. For example:
```
int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);
```
For example:
```
pwm_config(pwm1, 500000, 1000000)；
```
See The API of PWM device for more detail of this function.    

(4)、Enable PWM Function
```
int pwm_enable(struct pwm_device *pwm);
```
enable PWM，For example：  
```
pwm_enable(pwm0);
```
See The API of PWM device for more detail of this function.  

== The API of PWM device ==
```
 /**
  * pwm_request() - request a PWM device
  * @pwm_id: global PWM device index
  * @label: PWM device label
  *
  * This function is deprecated, use pwm_get() instead.
  */struct pwm_device *pwm_request(int pwm_id, const char *label);

/**
 * pwm_free() - free a PWM device
 * @pwm: PWM device
 *
 * This function is deprecated, use pwm_put() instead.
 */void pwm_free(struct pwm_device *pwm);

/**
 * pwm_config() - change a PWM device configuration
 * @pwm: PWM device
 * @duty_ns: "on" time (in nanoseconds)
 * @period_ns: duration (in nanoseconds) of one cycle
 */int pwm_config(struct pwm_device *pwm, int duty_ns, int period_ns);

/**
 * pwm_enable() - start a PWM output toggling
 * @pwm: PWM device
 */int pwm_enable(struct pwm_device *pwm);

/**
 * pwm_disable() - stop a PWM output toggling
 * @pwm: PWM device
 */void pwm_disable(struct pwm_device *pwm);
```
Sample codeː `kernel/drivers/pwm/pwm-firefly.c`
## Debug
cat  /sys/kernel/debug/pwm  ---success or not，check the address of register

## FAQ
### Cannot register pwm：
1. Set dts pwm configuration is "okay"
2. Check the io of pwm if it was used by other source .According to the return to figure out the error.
