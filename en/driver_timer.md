# TIMER
## Introduction
Timer is a programmable timer peripheral. This component is an APB slave device. In RK3399 there are 12 Timers(timer0~timer11), 12 Secure Timers(stimer0~stimer11) in ALIVE and 2 Timers(pmutimer0~pmutimer1) in PMU. We use the Timers(timer0-timer11) clock is 24MHZ and  Timer mode selection have free-running and user-defined count
## Timer Usage Flow
![](img/timer1.png)
## Timer mode selection
User-defined count mode :Timer loads TIMERn_LOAD_COUNT3 and TIMERn_ LOAD_COUNT2 as initial value. When the timer counts up to the value in TIMERn_LOAD_COUNT1 and TIMERn_LOAD_COUNT0, it will not automatically reload the count register. User need to disable timer firstly and follow the programming sequence to make timer work again. 
 
Free-running mode Timer loads the TIMERn_LOAD_COUNT3 and TIMERn_LOAD_COUNT2 register as initial value. Timer will automatically reload the count register, when timer counts up to the value in TIMERn_LOAD_COUNT1 and TIMERn_LOAD_COUNT0.
## Software config
1. Defined the config in the dts file `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi`
```
	rktimer: rktimer@ff850000 {
		compatible = "rockchip,rk3399-timer";
		reg = <0x0 0xff850000 0x0 0x1000>;
		interrupts = <GIC_SPI 81 IRQ_TYPE_LEVEL_HIGH 0>;
		clocks = <&cru PCLK_TIMER0>, <&cru SCLK_TIMER00>;
		clock-names = "pclk", "timer";
	};
```
It is defined the register and Interrupt and clock so on.  

Other Timer interrupt num can see  follow picture.

![](img/timer2.png)  

2. The driver file `Kernel/drivers/clocksource/rockchip_timer.c`.

## Register and usage
1.Register picture

![](img/timer3.png)

2.Usage Check the register value
```
root@rk3399_firefly_box:/ # io -4 0xff85001c  //Control Register Value
ff85001c:  00000007  
root@rk3399_firefly_box:/ # io -4 0xff850000  //Register Count Value
ff850000:  0001639f
```
Control count register  
```
root@rk3399_firefly_box:/ # io -4 -w 0xff85001c 0x06  //Disable Timer
```
