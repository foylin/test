# SPI
## Introduction
SPI is a high-speed, full-duplex, synchronous serial communication interface for connection to a microcontroller, sensors, storage devices, etc.

There are one SPI(SPI1) on the Firefly-RK3399 development board, the following picture shows the position:  
![](img/spi2.jpg)

## SPI working mode
SPI work in master-slave mode, This usually has a master device and one or more slave devices. They need at least four lines, which are:
```
CS		Chip select signals
SCLK		Clock signal
MOSI		Master data output, and Slave data input
MISO		Master data input, and Slave data output
```
Linux kernel using a combination of CPOL and CPHA to represent the SPI's four working modes:
```
CPOL＝0，CPHA＝0		SPI_MODE_0
CPOL＝0，CPHA＝1		SPI_MODE_1
CPOL＝1，CPHA＝0		SPI_MODE_2
CPOL＝1，CPHA＝1		SPI_MODE_3
```
CPOL:Represents the initial level of the state of the clock signal,0 is low,1 is high.  

CPHA:Represents the sampling clock edge, 0 is leading edge, 1 is trailing edge.

SPI waveforms of four working modes as follows:
![](img/spi1.jpg)

## Driver
In this paper, let's take W25Q128FV Flash module for example to introduce the use of SPI.
### Hardware connections
the connections  between Firefly-RK3399 and W25Q128FV  like the following table：  
![](img/spi3.png)
### Writing Makefile/Kconfig
Add  SPI_FIREFLY in kernel/drivers/spi/Kconfig:
```
config SPI_FIREFLY
       tristate "Firefly SPI demo support "
       default y
        help
          Select this option if your Firefly board needs to run SPI demo.
```
Add the driver into the kernel/drivers/spi/Makefile such as:
```
obj-$(CONFIG_SPI_FIREFLY)              += spi-firefly-demo.o
```
Select the driver files to add to the kernel options using make menuconfig:
```
  │ Symbol: SPI_FIREFLY [=y] 
  │ Type  : tristate
  │ Prompt: Firefly SPI demo support
  │   Location:
  │     -> Device Drivers
  │       -> SPI support (SPI [=y])
  │   Defined at drivers/spi/Kconfig:704
  │   Depends on: SPI [=y] && SPI_MASTER [=y]
```

## Define  SPI DTS
Add SPI driver node description in DTS, such as:
```
/* Firefly SPI demo */
&spi1 {
	spi_demo: spi-demo@00{
		status = "okay";
		compatible = "firefly,rk3399-spi";
		reg = <0x00>;
		spi-max-frequency = <48000000>;
		/* rk3399 driver support SPI_CPOL | SPI_CPHA | SPI_CS_HIGH */
		//spi-cpha;		/* SPI mode: CPHA=1 */
		//spi-cpol;   	/* SPI mode: CPOL=1 */
		//spi-cs-high;
	};
};
 
&spidev0 {
	status = "disabled";
};
```
Status: If you want to enable SPI, then set "okay", if not enabled, set "disable".  

spi-demo@00:Because this example uses  CS0, so here is set to 00, if you use CS1, is set to 01.  

Compatible:This property must be consistent with the structure of_device_id members.  

Reg: Make this 0x00, consistent with spidev@00.  

spi-max-frequency: the highest frequency of SPI, Firefly-RK3399 support up to 48000000.  

spi-cpha，spi-cpol: Set the SPI work mode here, The SPI module in this example work in SPI_MODE_1, so we set spi-cpha = <1>. You should mask this two, if your SPI module work in SPI_MODE0. You should set spi-cpha = <1>;spi-cpol = <1>, if your SPI module work in SPI_MODE3.  

spidev0: need to close spidev0, because spi_demo and spidev0 use the same hardware resource.  
### Define SPI driver
Create a new file in the kernel source: `kernel/drivers/spi/spi-firefly-demo.c`.
   
Before defining the SPI driver, define the of_device_id first.  

of_device_id is defined to match the corresponding node in the dts file:
```
static struct of_device_id firefly_match_table[] = {{ .compatible = "firefly,rk3399-spi",},	{},};
```
the value of compatible is the same as DTS's.  

Define the spi_driver:
```
static struct spi_driver firefly_spi_driver = {
	.driver = {
		.name = "firefly-spi",
		.owner = THIS_MODULE,
		.of_match_table = firefly_match_table,},
	.probe = firefly_spi_probe,};
```
### Register SPI driver

Register SPI driver to the kernel in firefly_spi_init:spi_register_driver(&firefly_spi_driver).

If it match Successful when kernel start up, it will  config the mode , speed and so on, call the driver's probe function.   

Probe function as following: static int firefly_spi_probe(struct spi_device *spi)
### SPI transfer
there are two transfer interface in firefly_spi_probe.
1. firefly_spi_read_w25x_id_0: using spi_transfer and spi_message to read ID.
2. firefly_spi_read_w25x_id_1: using spi_write_then_read to read ID.

if everything is ok, you will see the information in console:
```
root@rk3399_firefly_box:/ # dmesg | grep firefly-spi             
[    1.006235] firefly-spi spi0.0: Firefly SPI demo program
[    1.006246] firefly-spi spi0.0: firefly_spi_probe: setup mode 0, 8 bits/w, 48000000 Hz max
[    1.006298] firefly-spi spi0.0: firefly_spi_read_w25x_id_0: ID = ef 40 18 00 00
[    1.006361] firefly-spi spi0.0: firefly_spi_read_w25x_id_1: ID = ef 40 18 00 00
```
### Enable SPI demo
spi-firefly-demo is disabled in default, enable it in `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-demo.dtsi` like the following steps:
1. set "okay" in spi1 status;
2. set "disabled" in spidev0 status.

### Commonly used SPI APIs
Here are the definition of some commonly used GPIO APIs:
```
static void spi_message_init(struct spi_message *m); 
static void spi_message_add_tail(struct spi_transfer *t, struct spi_message *m); 
int spi_sync(struct spi_device *spi, struct spi_message *message) ; 
static int spi_write(struct spi_device *spi, const void *buf, size_t len); 
static int spi_read(struct spi_device *spi, void *buf, size_t len); 
static ssize_t spi_w8r8(struct spi_device *spi, u8 cmd); 
static ssize_t spi_w8r16(struct spi_device *spi, u8 cmd); 
static ssize_t spi_w8r16be(struct spi_device *spi, u8 cmd); 
int spi_write_then_read(struct spi_device *spi, const void *txbuf, unsigned n_tx, void *rxbuf, unsigned n_rx);
```
## Interface
SPI devices have a limited userspace API, you can use it except  kernel space interface such as IRQ handlers or other layers of the driver stack.

the path of spidev device in Firefly-RK3399: `/dev/spidev0.0`.
  
and the source path:`kernel/drivers/spi/spidev.c`.

enable SPI_SPIDEV in config：
```
 │ Symbol: SPI_SPIDEV [=y]
 │ Type  : tristate
 │ Prompt: User mode SPI device driver support 
 │   Location:
 │     -> Device Drivers
 │       -> SPI support (SPI [=y])
 │   Defined at drivers/spi/Kconfig:684
 │   Depends on: SPI [=y] && SPI_MASTER [=y]
```
the DTS  node:
```
&spi1 {
	status = "okay";
	max-freq = <48000000>;
	dev-port = <0>;
 
	spidev0: spidev@00 {
		status = "okay";
		compatible = "linux,spidev";
		reg = <0x00>;
		spi-max-frequency = <48000000>;
	};
};
```
Please refer to spidev  for detailed instructions.

## FAQs
### Q1: SPI data transfer error
A1:  make sure  pins iomux is SPI, TX/RX/CLK  signal accurately， and set the correct mode with device。
