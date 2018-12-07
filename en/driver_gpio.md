# GPIO
## Introduction
GPIO, short for General-Purpose Input/Output, is a flexible software-controlled digital signal. RK3399 have 5 banks：GPIO0，GPIO1, ..., GPIO4, Each bit of the bank is numbered with A0~A7, B0~B7, C0~C7, D0~D7 (Not all banks have full numbers, for example, GPIO4 only has C0~C7, D0~D2). All the GPIO in the initial state after power are input mode, and it can be set to pull-up or pull-down by software, it also can be set to interrupt pin. In addition, their drive strength is programmable. Besides the general-purpose input/output, GPIO may be multiplexed with other functions. For example, GPIO2_B2, has the following extra functions:
* SPI2_TXD
* CIF_CLKIN
* I2C6_SCL

The drive current, pull up/down and reset state of all the gpios are not neccessary the same. Please refer section titled "Chapter 10 GPIO" in RK3399 Datasheet. RK3399's GPIO driver is implemented in the following pinctrl file:
```
kernel/drivers/pinctrl/pinctrl-rockchip.c
```
The core logic is filling up methods and parameters of each GPIO bank before calling gpiochip_add to register into the kernel.

To facilitate the development of the user, Firefly-RK3399 designed a row of general-purpose GPIO port, the pins is as follows:
![](img/gpio1.png)
In this article, I used TP_RST(GPIO0_B4) and LCD_RST(GPIO4_D5) as an example to write a simple driver to operate GPIO. The example path in the SDK is
```
kernel/drivers/gpio/gpio-firefly.c
```
Next I will describe how to operate GPIO
## Input/Output
To begin with, you need to add resource description to the dts (Device Tree) file `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi`:
```
    gpio_demo: gpio_demo {
        status = "okay";
        compatible = "firefly,rk3399-gpio";
        firefly-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;          /* GPIO0_B4 */
        firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */                                                                                             
    };
```
Here, GPIO0_B4 is defined as the general output input port:
```
firefly-gpio GPIO0_B4
```
The DTS of Firefly-RK3399 is different from Firefly-RK3288, GPIO0_B4 is described as <&gpio0 12 GPIO_ACTIVE_HIGH>, Where 12 is derived from: 8 + 4 = 12, where 8 is because GPIO0_B4 is a group B belonging to GPIO0, 0 if it is a group A, and 16 if it is a group C, Group D is 24, with this recursive, and 4 because B4 behind 4. GPIO_ACTIVE_LOW means low level voltage is effective for light on. If high level voltage is effective, replace it with GPIO_ACTIVE_HIGH.

Next, add requesting and controlling of GPIO to the device driver:
```
 static int firefly_gpio_probe(struct platform_device *pdev)
 {   
     int ret; int gpio; 
     enum of_gpio_flags flag; 
     struct firefly_gpio_info *gpio_info; 
     struct device_node *firefly_gpio_node = pdev->dev.of_node; 
     printk("Firefly GPIO Test Program Probe\n"); 
     gpio_info = devm_kzalloc(&pdev->dev,sizeof(struct firefly_gpio_info *), GFP_KERNEL); 
     if (!gpio_info) { return -ENOMEM; } 
     gpio = of_get_named_gpio_flags(firefly_gpio_node, "firefly-gpio", 0, &flag); 
     if (!gpio_is_valid(gpio)) { 
	printk("firefly-gpio: %d is invalid\n", gpio); 
	return -ENODEV;
     } 
     if (gpio_request(gpio, "firefly-gpio")) { 
	printk("gpio %d request failed!\n", gpio); 
	gpio_free(gpio); return -ENODEV; 
     } 
     gpio_info->firefly_gpio = gpio; 
     gpio_info->gpio_enable_value = (flag == OF_GPIO_ACTIVE_LOW) ? 0:1; 
     gpio_direction_output(gpio_info->firefly_gpio, gpio_info->gpio_enable_value); 
     printk("Firefly gpio putout\n"); 
     ...... 
}
```
First call of_get_named_gpio_flags to read number and flag of the gpio named "firefly-gpio" and "firefly-irq-gpio". Use gpio_is_valid to check whether this gpio number is valid, then request this gpio from kernel with gpio_request. If errors occur, call gpio_free to free the gpio which was previously requested successfully. Call gpio_direction_output to set gpio in output direction, with high or low output. Here the flag is GPIO_ACTIVE_LOW, in order to turn on the led, write 0 to it. If you want to read from gpio, you need to set it in input direction before reading the value:
```
int val;
gpio_direction_input(your_gpio);
val = gpio_get_value(your_gpio);
```
Here are the definition of some commonly used GPIO APIs:
```
 #include <linux/gpio.h>
 #include <linux/of_gpio.h>  
 enum of_gpio_flags {OF_GPIO_ACTIVE_LOW = 0x1,
 };  
 int of_get_named_gpio_flags(struct device_node *np, const char *propname,int index, enum of_gpio_flags *flags);  
 int gpio_is_valid(int gpio); 
 int gpio_request(unsigned gpio, const char *label); 
 void gpio_free(unsigned gpio); 
 int gpio_direction_input(int gpio); 
 int gpio_direction_output(int gpio, int v)
```
## Interruption
The Firefly example program also includes an interrupt pin, and the Interrupts of the GPIO port are analogous to GPIO Input/Output, you need to add resource description to the dts (Device Tree) file first.
```
kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-port.dtsi
  gpio {
      compatible = "firefly-gpio";
      firefly-irq-gpio = <&gpio4 29 IRQ_TYPE_EDGE_RISING>;  /* GPIO4_D5 */
  };
```
IRQ_TYPE_EDGE_RISING indicates that the interrupt is triggered by a rising edge, the interrupt function can be triggered when a rising edge signal is received on this pin. Here, it may be configured as follows:
```
IRQ_TYPE_NONE         //Default, no defined interrupt trigger type
IRQ_TYPE_EDGE_RISING  //Rising edge triggers 
IRQ_TYPE_EDGE_FALLING //Falling edge trigger
IRQ_TYPE_EDGE_BOTH    //Both rising and falling edges trigger
IRQ_TYPE_LEVEL_HIGH   //High level trigger
IRQ_TYPE_LEVEL_LOW    //Low level trigger
```
Then the probe function to resolve the DTS, and then apply for interrupt registration, the code is as follows:
```
 static int firefly_gpio_probe(struct platform_device *pdev)
 {   
     int ret; int gpio; enum of_gpio_flags flag; 
     struct firefly_gpio_info *gpio_info; 
     struct device_node *firefly_gpio_node = pdev->dev.of_node; 
     ......     
     gpio_info->firefly_irq_gpio = gpio; 
     gpio_info->firefly_irq_mode = flag; 
     gpio_info->firefly_irq = gpio_to_irq(gpio_info->firefly_irq_gpio); 
     if (gpio_info->firefly_irq) { 
	 if (gpio_request(gpio, "firefly-irq-gpio")) { 
		 printk("gpio %d request failed!\n", gpio); 
		 gpio_free(gpio); return IRQ_NONE; 
	 } 
         ret = request_irq(gpio_info->firefly_irq, firefly_gpio_irq, flag,  "firefly-gpio", gpio_info); 
	 if (ret != 0) free_irq(gpio_info->firefly_irq, gpio_info); 
	 dev_err(&pdev->dev, "Failed to request IRQ: %d\n", ret); 
      } 
     return 0;
 }
 static irqreturn_t firefly_gpio_irq(int irq, void *dev_id) //Interrupt function
 { 
     printk("Enter firefly gpio irq test program!\n"); 
     return IRQ_HANDLED;
 }
```
Call gpio_to_irq to convert the PIN value of the GPIO to the corresponding IRQ value, and call gpio_request request to use the IO port, then call request_irq to request an interrupt, if it fails to call free_irq release. gpio_info-firefly_irq request the hardware interrupt number, firefly_gpio_irq is the interrupt function, gpio_info->firefly_irq_mode is the interrupt handling properties, "firefly-gpio" is the device driver name, gpio_info is the device's device structure
## Multiplexing
The gpio has multiplexing functions. How to declare it, and how to switch it at the runtime? We take I2C4 for a brief descrition.

Here is what we find in the data sheet:
```
Pad# 	                func0 	        func1
I2C4_SDA/GPIO1_B3 	gpio1b3 	i2c4_sda
I2C4_SCL/GPIO1_B4 	gpio1b4 	i2c4_scl
```
In `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi` i2c4 is declared as below:
```
     i2c4: i2c@ff3d0000 { 
		compatible = "rockchip,rk3399-i2c"; 
		reg = <0x0 0xff3d0000 0x0 0x1000>; 
		clocks = <&pmucru SCLK_I2C4_PMU>, <&pmucru PCLK_I2C4_PMU>; 
		clock-names = "i2c", "pclk"; 
		interrupts = <GIC_SPI 56 IRQ_TYPE_LEVEL_HIGH 0>; 
		pinctrl-names = "default", "gpio"; 
		pinctrl-0 = <&i2c4_xfer>; pinctrl-1 = <&i2c4_gpio>;   //The source code is not added here 
		#address-cells = <1>;  
		#size-cells = <0>;  
		status = "disabled"; 
    };
```
Properties prefixed with "pinctrl-" are relative with mutiplexing:

* pinctrl-names defines a state name list: default (i2c) and gpio. 
* pinctrl-0 defines the pinctrls which needs to apply in state 0: &i2c4_xfer 
* pinctrl-1 defines the pinctrls which needs to apply in state 1: &i2c4_gpio

These pinctrls are defined in `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi` as follows:
```
     ......     
     pinctrl: pinctrl { 
		compatible = "rockchip,rk3399-pinctrl"; 
		rockchip,grf = <&grf>; 
		rockchip,pmu = <&pmugrf>; 
		#address-cells = <0x2>; 
		#size-cells = <0x2>; 
		ranges; 
		i2c4 {
             i2c4_xfer: i2c4-xfer { 
				rockchip,pins = <1 12 RK_FUNC_1 &pcfg_pull_none>, <1 11 RK_FUNC_1 &pcfg_pull_none>; 
		    }; 
		    i2c4_gpio: i2c4-gpio { 
				rockchip,pins = <1 12 RK_FUNC_GPIO &pcfg_pull_none>, <1 11 RK_FUNC_GPIO &pcfg_pull_none>; 
		    };          
		}; 
     };
```
RK_FUNC_1,RK_FUNC_GPIO is defined in `kernel/include/dt-bindings/pinctrl/rk.h`
```
 #define RK_FUNC_GPIO    0
 #define RK_FUNC_1   1
 #define RK_FUNC_2   2
 #define RK_FUNC_3   3
 #define RK_FUNC_4   4
 #define RK_FUNC_5   5 
 #define RK_FUNC_6   6
 #define RK_FUNC_7   7
```
In addition, values such as "1 11" and "1 12" have the same encoding rules as described in the previous section "Input Outputs", where "1 11" represents GPIO1_B3 and "1 12" represents GPIO1_B4.

Therefore, when using multiplexing, if you select "default" (I2C function here) state， the kernel will apply pinctrls of i2c4_xfer, which results in switching GPIO1_B3 pin and GPIO1_B4 pin to I2C functions; if you select "gpio" state, the kernel will apply the i2c4_gpio pinctrl, restoring the two pins back to general-purpose input/output function.

We look at the i2c driver `kernel/drivers/i2c/busses/i2c-rockchip.c` is how to switch multiplexing function:
```
 static int rockchip_i2c_probe(struct platform_device *pdev)
 {   
     struct rockchip_i2c *i2c = NULL; 
     struct resource *res; struct device_node *np = pdev->dev.of_node; 
     int ret;
     // ...
     i2c->sda_gpio = of_get_gpio(np, 0);
     if (!gpio_is_valid(i2c->sda_gpio)) {
			dev_err(&pdev->dev, "sda gpio is invalid\n");
            return -EINVAL;
            }
     ret = devm_gpio_request(&pdev->dev, i2c->sda_gpio, dev_name(&i2c->adap.dev));
     if (ret) {
			dev_err(&pdev->dev, "failed to request sda gpio\n");
            return ret;
            }
	i2c->scl_gpio = of_get_gpio(np, 1);
    if (!gpio_is_valid(i2c->scl_gpio)) {
			dev_err(&pdev->dev, "scl gpio is invalid\n");
            return -EINVAL;
            }
	ret = devm_gpio_request(&pdev->dev, i2c->scl_gpio, dev_name(&i2c->adap.dev));
    if (ret) {
			dev_err(&pdev->dev, "failed to request scl gpio\n");
            return ret;
            }
	i2c->gpio_state = pinctrl_lookup_state(i2c->dev->pins->p, "gpio");
    if (IS_ERR(i2c->gpio_state)) {
			dev_err(&pdev->dev, "no gpio pinctrl state\n");
            return PTR_ERR(i2c->gpio_state);
            }
	pinctrl_select_state(i2c->dev->pins->p, i2c->gpio_state);
	gpio_direction_input(i2c->sda_gpio);
	gpio_direction_input(i2c->scl_gpio);
	pinctrl_select_state(i2c->dev->pins->p, i2c->dev->pins->default_state);
    // ...
}
```
First call of_get_gpio to get the gpio pins from the "gpios" list property in the i2c4 node in device tree:
```
gpios = <&gpio1 GPIO_B3 GPIO_ACTIVE_LOW>, <&gpio1 GPIO_B4 GPIO_ACTIVE_LOW>;
```
Then call devm_gpio_request request gpio, followed by calling pinctrl_lookup_state to look up the “gpio” state. The "default" state is already saved in i2c->dev-pins->default_state by the kernel device tree parsing.

Finally, call pinctrl_select_state to select between "default" or "gpio" state, which results in the correspoding multiplexing function.

Here are some commonly used multiplexing APIs:
```
#include <linux/pinctrl/consumer.h> 
struct device {
		//...#ifdef 
		CONFIG_PINCTRLstruct dev_pin_info	
		*pins;#endif
		//...
}; 
struct dev_pin_info {
		struct pinctrl *p;
		struct pinctrl_state *default_state;
		#ifdef CONFIG_PMstruct pinctrl_state *sleep_state;
		struct pinctrl_state *idle_state;#endif
}; 
struct pinctrl_state * pinctrl_lookup_state(struct pinctrl *p, const char *name); 
int pinctrl_select_state(struct pinctrl *p, struct pinctrl_state *s);
```
## IO-Domain

In complex SOC systems, the designer typically divides the system's power supply into multiple independent blocks, known as the Power Domain, this has many advantages, such as:

* In the IO-Domain DTS node unified configuration voltage domain, do not need to configure each drive once, easy to manage;
* In accordance with the practice of Upstream, Upstream after more convenient if necessary;
* The IO-Domain driver supports dynamic adjustment of the voltage domain during operation. For example, a Regulator of the PMIC can switch between 1.8v and 3.3v dynamically. Once the Regulator voltage changes, the IO-Domain driver is notified to reset the voltage domain.

The Power Domain Map table and configuration on the Firefly-RK3399 schematic are shown in the following table.
![](img/gpio2.png)
