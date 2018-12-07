# I2C
## Introduction
Firefly-RK3399 has 9 on-chip I2C controllers, The usage of each I2C is shown in the table below:

![](img/i2c_table.png)
In order to config I2C, there are mainly two steps:
1. Define and register I2C device.
2. Define and register I2C driver.

Now, take GSL3680 as an example.
## Define and Register I2C Device

You need an i2c_client struct to describe the i2c device. However this struct do not need to defined directly. The only thing you need to do is providing the device information and the kernel will build one from it.

You can write the information of the I2C device as a node in the dts file, which is shown below:
```
kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-mini-edp.dts
&i2c4 {
    status = "okay";
    gsl3680: gsl3680@41 {
              compatible = "gslX680";
              reg = <0x41>;
              screen_max_x = <1536>;
              screen_max_y = <2048>;
              touch-gpio = <&gpio1 20 IRQ_TYPE_LEVEL_LOW>;
              reset-gpio = <&gpio0 12 GPIO_ACTIVE_HIGH>;
      };  
};
```
## Define and Register I2C Driver
### Define I2C Driver
Before defining i2_driver, you need to define of_device_id and i2c_device_id first.

of_device_id is defined to match the corresponding node in the dts file:
```
 static struct of_device_id gsl_ts_ids[] = {{.compatible = "gslX680"},{}   
 };
```
define i2c_device_id:
```
 static const struct i2c_device_id gsl_ts_id[] = { {GSLX680_I2C_NAME, 0}, {}   
 };
 MODULE_DEVICE_TABLE(i2c, gsl_ts_id);
```
Define i2c_driver as following:
```
 static struct i2c_driver gsl_ts_driver = { 
	.driver = { 
    		.name = GSLX680_I2C_NAME, 
			.owner = THIS_MODULE, 
			.of_match_table = of_match_ptr(gsl_ts_ids), 
	},   
	#ifndef CONFIG_HAS_EARLYSUSPEND 
	//.suspend  = gsl_ts_suspend, 
	//.resume   = gsl_ts_resume,
	#endif
	.probe      = gsl_ts_probe, 
	.remove     = gsl_ts_remove, 
	.id_table   = gsl_ts_id,
 };
```
NOTE: id_table shows which device the driver support.
### Register I2C Driver
Use i2c_add_driver to register I2C driver.
```
i2c_add_driver(&gsl_ts_driver);
```
During the call to i2c_add_driver to register I2C driver, all the I2C devices will be traversed. Once matched, the probe function of the driver will be executed.
### Communicate with I2C Client
After registering I2C driver, you can communicate with I2C client from now on.
* Write message to the client:
```
 int i2c_master_send(const struct i2c_client *client, const char *buf, int count)
 {   
     int ret; 
     struct i2c_adapter *adap = client->adapter; struct i2c_msg msg; 
     msg.addr = client->addr; msg.flags = client->flags & I2C_M_TEN; msg.len = count; msg.buf = (char *)buf; 
     ret = i2c_transfer(adap, &msg, 1);      /*
      * If everything went ok (i.e. 1 msg transmitted), return #bytes
      * transmitted, else error code.
      */ return (ret == 1) ? count : ret;
 }
```
* Read message from the client:
```
 int i2c_master_recv(const struct i2c_client *client, char *buf, int count)
 { 
     struct i2c_adapter *adap = client->adapter; struct i2c_msg msg; 
     int ret; 
     msg.addr = client->addr; 
     msg.flags = client->flags & I2C_M_TEN; msg.flags |= I2C_M_RD; 
     msg.len = count; 
     msg.buf = buf; 
     ret = i2c_transfer(adap, &msg, 1);      /* 
     * If everything went ok (i.e. 1 msg received), return #bytes received,
     * else error code.
     */ 
     return (ret == 1) ? count : ret;
 }   
 EXPORT_SYMBOL(i2c_master_recv);
```
## FAQs
### Q1: If such a log: "timeout, ipd: 0x00, state: 1" How to debug?
A1: When this log: "timeout, ipd: 0x00, state: 1" appears, please check whether the hardware pull-up power;
### Q2: The call to i2c_transfer returns a value of -6?
A2: If the call return i2c_transfer -6, said NACK error, that is, no response to the other side equipment, which is generally the problem of peripherals, the common are the following:

* I2C address error, the solution is to measure I2</ sup>C waveform, to confirm whether I2C device address error;
* I2C slave device is not in the normal working state, such as no power, wrong power-up sequence;
* The timing does not meet the requirements of I2C slave devices will also produce Nack signal, such as the following third point;
### Q3: When the peripherals for the middle of the time sequence is read stop signal is not a repeat start signal, how to deal with?
A3: When the peripheral is reading the timing requirements for the middle of the stop signal is not a repeat start signal when the need to call twice i2c_transfer, I2C read split into two, modified as follows:
```
static int i2c_read_bytes(struct i2c_client *client,u8 cmd, u8 *data, u8 data_len) { 
               struct i2c_msg msgs[2]; 
               int ret; 
               u8 *buffer;  
               buffer = kzalloc(data_len, GFP_KERNEL); 
               if (!buffer) return -ENOMEM;  
               msgs[0].addr = client->addr; 
               msgs[0].flags = client->flags; 
               msgs[0].len = 1; 
               msgs[0].buf = &cmd; 
               ret = i2c_transfer(client->adapter, msgs, 1); 
               if (ret < 0) { 
			dev_err(&client->adapter->dev, "i2c read failed\n"); 
        		kfree(buffer); 
                  	return ret; 
	       }  
               msgs[1].addr = client->addr; 
               msgs[1].flags = client->flags | I2C_M_RD; 
               msgs[1].len = data_len; 
               msgs[1].buf = buffer;  
               ret = i2c_transfer(client->adapter, &msgs[1], 1); 
               if (ret < 0) 
			   			dev_err(&client->adapter->dev, "i2c read failed\n"); 
               else
               			memcpy(data, buffer, data_len);  
               kfree(buffer);
               return ret; 
}
```
