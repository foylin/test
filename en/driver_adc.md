# ADC
## Introduction

Firefly-RK3399 development board has two types of AD interfaces: Temperature Sensor(TS-ADC) and SAR-ADC(Successive Approximation Register):

* TS-ADC(Temperature Sensor)：Embedded 2 channel TS-ADC in RK3399， TS-ADC clock must be less than 800KHZ
* SAR-ADC(Successive Approximation Register)：6-channel single-ended 10-bit SAR analog-to-digital converter，SAR-ADC clock must be less than 13MHZ

Linux kernel controls the ADC via industrial I/O subsystem, which is intended to provide support for ADC or DAC devices. Take the SAR-ADC as an example to introduce the basic configuration of the ADC method.
## DTS Config
### Config DTS Node

The DTS device node of the ADC is already defined in file `kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi`, as follows:
```
saradc: saradc@ff100000 {
               compatible = "rockchip,rk3399-saradc";
               reg = <0x0 0xff100000 0x0 0x100>;
               interrupts = <GIC_SPI 62 IRQ_TYPE_LEVEL_HIGH 0>;
               #io-channel-cells = <1>;
               clocks = <&cru SCLK_SARADC>, <&cru PCLK_SARADC>;
               clock-names = "saradc", "apb_pclk";
               status = "disabled";
       };
```
To begin with, you need to add resource description to the dts (Device Tree) file:
```
kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi :
   adc_demo: adc_demo{
       status = "disabled";
       compatible = "firefly,rk3399-adc";
       io-channels = <&saradc 3>;
   };
```
Here to apply for the SARADC third channel.
### Match the DTS node in the driver file

You can write the driver refer to Firefly adc demo :kernel/drivers/adc/adc-firefly-demo.c. This is a drive to detect the status of the Firefly-rk3399 fan. Firstly, define an array of struct of_device_id:
```
static const struct of_device_id firefly_adc_match[] = { { .compatible = "firefly,rk3399-adc" },{},};
```
Next, fill the .of_match_table member of platform_driver with  firefly_adc_match:
```
static struct platform_driver firefly_adc_driver = { 
    .probe      = firefly_adc_probe,
    .remove     = firefly_adc_remove,
    .driver     = { 
    			.name   = "firefly_adc",.owner  = THIS_MODULE,
    			.of_match_table = firefly_adc_match,
                },
};
```
And then, add requesting and controlling of SAR-ADC to the device driver:
```
static int firefly_adc_probe(struct platform_device *pdev)
{
	printk("firefly_adc_probe!\n");
    chan = iio_channel_get(&(pdev->dev), NULL);
    if (IS_ERR(chan)){
			chan = NULL;
			printk("%s() have not set adc chan\n", __FUNCTION__);
            return -1;
	}
    fan_insert = false;
    if (chan) {
			INIT_DELAYED_WORK(&adc_poll_work, firefly_demo_adc_poll);
			schedule_delayed_work(&adc_poll_work,1000);}return 0;
   }
```
## Driving instructions
### Get the AD channel
```
struct iio_channel *chan; //Define IIO channel structurechan = iio_channel_get(&pdev->dev, NULL); //Get IIO channel structure
```
Note: iio_channel_get gets the channel from the pdev which is paramter of the probe function shown below:
```
static int XXX_probe(struct platform_device *pdev);
```
### Read Raw Data from Channel
```
int val,ret;ret = iio_read_channel_raw(chan, &val);
```
Call iio_read_channel_raw to read the raw data from the channel and store the result in val parameter.
### Convert the Raw Data to Voltage
Use the standard voltage to convert the raw data to the result voltage. The equation is shown as below:
```
Vref / (2^n-1) = Vresult / raw
```
Note:
* Vref is the standard voltage
* n is the AD conversion accuracy, such as 10 bits or 12 bits.
* Vresult is the conversion result of the voltage.
* raw is the raw data read from function iio_read_channel_raw.

For example, the standard voltage is 1.8V, n is 10 bits, raw data is 568. The conversion expression is:
```
Vresult = (1800mv * 568) / 1023;
```
Interface specification
```
/**
 * iio_channel_get() - get description of all that is needed to access channel.
 * @dev:        Pointer to consumer device. Device name must match
 *          the name of the device as provided in the iio_map
 *          with which the desired provider to consumer mapping
 *          was registered.
 * @consumer_channel:   Unique name to identify the channel on the consumer
 *          side. This typically describes the channels use within
 *          the consumer. E.g. 'battery_voltage'
 */struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```
```
/**
 * iio_channel_release() - release channels obtained via iio_channel_get
 * @chan:       The channel to be released.
 */void iio_channel_release(struct iio_channel *chan);
```
```
/**
 * iio_read_channel_raw() - read from a given channel
 * @chan:       The channel being queried.
 * @val:        Value read back.
 *
 * Note raw reads from iio channels are in adc counts and hence
 * scale will need to be applied if standard units required.
 */int iio_read_channel_raw(struct iio_channel *chan, int *val);
```
## Debugging method
### Use the ADC demo
Change "disabled" to "okay" in file `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi`,to enable the adc_demo:
```
adc_demo: adc_demo{
        status = "okay";
        compatible = "firefly,rk3399-adc";
        io-channels = <&saradc 3>;
    };
```
Compile the kernel，and burn the kernel to the Firefly-RK3399 development board，then insert or remove the fan，it will Print the kernel logs as follows：
```
[   85.158104] Fan insert! raw= 135 Voltage= 237mV
[   88.422124] Fan out! raw= 709 Voltage=1247mV
```
### Gets each ADC value

There is a convenient way to query the value of each SARADC：
```
cat /sys/bus/iio/devices/iio\:device0/in_voltage*_raw
```
## FAQs
### Q1: Why use the above steps to apply for SARADC, there will be an application error situation ?

A1： When the drive to obtain access to the ADC channel to use, you need to control the load time of the driver, which should be later than the saradc's initialization.Saradc is registred for platform device drivers by using module_platform_driver()，which finally call module_init().So the user's driver loader function only needs to use a lower priority than module_init (),such as late_initcall (),it can guarantee the driver loaded time later than saradc's initialization time, and avoid making mistakes.making mistakes.
