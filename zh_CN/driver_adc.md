# ADC 使用

## 简介

Firefly-RK3399开发板上的 AD 接口有两种，分别为：温度传感器 (Temperature Sensor)、逐次逼近ADC (Successive Approximation Register)。其中：

*    TS-ADC(Temperature Sensor)：支持两通道，时钟频率必须低于800KHZ

*    SAR-ADC(Successive Approximation Register)：支持六通道单端10位的SAR-ADC，时钟频率必须小于13MHZ。

内核采用工业 I/O 子系统来控制 ADC，该子系统主要为 AD 转换或者 DA 转换的传感器设计。 下面以SAR-ADC为例子，介绍 ADC 的基本配置方法。
## DTS配置
### 配置DTS节点

Firefly-RK3399 SAR-ADC 的 DTS 节点在 kernel/arch/arm64/boot/dts/rockchip/rk3399.dtsi 文件中定义，如下所示：
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
用户首先需在DTS文件中添加ADC的资源描述：
```
kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi :
   adc_demo: adc_demo{
       status = "disabled";
       compatible = "firefly,rk3399-adc";
       io-channels = <&saradc 3>;
   };
```
这里申请的是SARADC通道3.
### 在驱动文件中匹配 DTS 节点

用户驱动可参考Firefly adc demo :kernel/drivers/adc/adc-firefly-demo.c，这是一个侦测Firefly-rk3399风扇状态的驱动。 首先在驱动文件中定义 of_device_id 结构体数组：
```
static const struct of_device_id firefly_adc_match[] = { 
     { .compatible = "firefly,rk3399-adc" },
     {},
};
```
然后将该结构体数组填充到要使用 ADC 的 platform_driver 中：
```
static struct platform_driver firefly_adc_driver = { 
    .probe      = firefly_adc_probe,
    .remove     = firefly_adc_remove,
    .driver     = { 
        .name   = "firefly_adc",
        .owner  = THIS_MODULE,
        .of_match_table = firefly_adc_match,
        },  
};
```
接着在firefly_adc_probe中对DTS所添加的资源进行解析：
```
static int firefly_adc_probe(struct platform_device *pdev)
{
     printk("firefly_adc_probe!\n");
     chan = iio_channel_get(&(pdev->dev), NULL);
     if (IS_ERR(chan))
     {
	chan = NULL;
        printk("%s() have not set adc chan\n", __FUNCTION__);
        return -1;
        }
        fan_insert = false;
        if (chan) {
		INIT_DELAYED_WORK(&adc_poll_work, firefly_demo_adc_poll);
		schedule_delayed_work(&adc_poll_work,1000);}return 0;}
```
## 驱动说明
### 获取 AD 通道
```
struct iio_channel *chan; //定义 IIO 通道结构体chan = iio_channel_get(&pdev->dev, NULL); //获取 IIO 通道结构体
```
注：iio_channel_get 通过 probe 函数传进来的参数 pdev 获取 IIO 通道结构体，probe 函数如下：
```
static int XXX_probe(struct platform_device *pdev);
```
### 读取 AD 采集到的原始数据
```
int val,ret;
ret = iio_read_channel_raw(chan, &val);
```
调用 iio_read_channel_raw 函数读取 AD 采集的原始数据并存入 val 中。
### 计算采集到的电压

使用标准电压将 AD 转换的值转换为用户所需要的电压值。其计算公式如下：
```
Vref / (2^n-1) = Vresult / raw
```
注：

*    Vref 为标准电压
*    n 为 AD 转换的位数
*    Vresult 为用户所需要的采集电压
*    raw 为 AD 采集的原始数据

例如，标准电压为 1.8V，AD 采集位数为 10 位，AD 采集到的原始数据为 568，则：
```
Vresult = (1800mv * 568) / 1023;
```
## 接口说明
```
struct iio_channel *iio_channel_get(struct device *dev, const char *consumer_channel);
```

*    功能：获取 iio 通道描述
*    参数：
     * dev: 使用该通道的设备描述指针
     * consumer_channel: 该设备所使用的 IIO 通道描述指针

```
void iio_channel_release(struct iio_channel *chan);
```

*    功能：释放 iio_channel_get 函数获取到的通道

*    参数：
     * chan：要被释放的通道描述指针

```
int iio_read_channel_raw(struct iio_channel *chan, int *val);
```

*    功能：读取 chan 通道 AD 采集的原始数据。

*    参数：
     * chan：要读取的采集通道指针
     * val：存放读取结果的指针

## 调试方法
### Demo程序使用

在kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi中使能adc_demo，将"disabled" 改为 "okay":
```
adc_demo: adc_demo{
        status = "okay";
        compatible = "firefly,rk3399-adc";
        io-channels = <&saradc 3>;
    };
```
编译内核，烧录内核到Firefly-RK3399 开发板上，然后插拔风扇时，会打印内核log信息如下：
```
[   85.158104] Fan insert! raw= 135 Voltage= 237mV
[   88.422124] Fan out! raw= 709 Voltage=1247mV
```
### 获取所有ADC值

有个便捷的方法可以查询到每个SARADC的值：
```
cat /sys/bus/iio/devices/iio\:device0/in_voltage*_raw
```
## FAQs
### 为何按上面的步骤申请SARADC，会出现申请报错的情况？
驱动需要获取ADC通道来使用时，需要对驱动的加载时间进行控制，必须要在saradc初始化之后。saradc是使用module_platform_driver()进行平台设备驱动注册，最终调用的是module_init()。所以用户的驱动加载函数只需使用比module_init()优先级低的，例如：late_initcall()，就能保证驱动的加载的时间比saradc初始化时间晚，可避免出错。