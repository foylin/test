# Camera
## Board Resource

There is a MIPI camera interface on the Firefly-RK3288 development board, with maximum input resolution of 14M (4416x3312) pixels. You can also connect USB cameras to the board.

This article will introduce how to make the camera work properly, using OV13850/OV5640 as an example.

* Board Interface
![](img/Rk3399-camera-board-en.jpg)
## DTS Configuration
```
 isp0: isp@ff910000 {
	 …
	 status = "okay";
 }
 isp1: isp@ff920000 {
	 …
	 status = "okay";
 }
```
## Driver

The code directories of camera are shown as followed:
```
Android：
`- hardware/rk29/camera
   |- Config
   |  `- cam_board.xml      // Parameter Configuration
   |- CameraHal             // HAL source code
   `- SiliconImage          // ISP library, including driver source code of all supported modules.
      `- isi/drv/OV13850    // Driver source code of OV13850 module.
         `- calib/OV13850.xml // Calibration parameter of OV13850 module.

  `- hardware/rockchip/camera
   |- Config
   |  `- cam_board.xml      // Parameter Configuration
   |- CameraHal             // HAL source code
   `- SiliconImage          // ISP library, including driver source code of all supported modules.
      `- isi/drv/OV5640     // Driver source code of OV5640 module.
         `- calib/OV5640.xml // Calibration parameter of OV5640 module. 


 Kernel：
 |- kernel/drivers/media/video/rk_camsys  // CamSys driver source code
 `- kernel/include/media/camsys_head.h
```
## Configuration Theory

What you need to configure the camera is to make the pins and clock work properly.

According to the schematic diagram below, you need to provide: AF_VDD28、DOVDD18、AVDD28、DVDD12、PWDN1、RST and MCLK.

* MIPI interface

![](img/camera2.jpg)

* DVP interface

![](img/camera3.jpg)

* AF_VDD28 is provided by hardware connection. No configuration is needed.

* DOVDD18、AVDD28 The DOVDD18 and AVDD28 are controlled by DVP_PWR:

![](img/camera4.jpg)
 
* DVP_PWR is controled by GPIO1_C7 .
* DVDD12 is controlled by CIF_PWER：

![](img/camera5.jpg)

* CIF_PWER is connected to GPIO7_B4 .
* PWDN1 and RST are connected to GPIO2_B6 and GPIO2_B7 respectively:

![](img/camera6.jpg)

All the pins are configured in cam_board.xml, with the exception of DVDD12 (CIF_POWER), which is configured in DTS and driver.
## Configuration Steps
### Android Layer Configuration

Modify hardware/rockchip/camera/Config/cam_board_rk3399.xml   or 

device/rockchip/rk3399/$(TARGET_PRODUCT)/cam_board.xml to configure camera：
```
<?xml version="1.0" ?>
<BoardFile>        
<BoardXmlVersion version="v0.0xf.0">        
</BoardXmlVersion>        
<CamDevie>            
<HardWareInfo>                
<Sensor>                    
<SensorName name="OV13850" ></SensorName>                   
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_1A"></SensorDevID>                   
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>                    <SensorI2cBusNum busnum="1></SensorI2cBusNum>                    
<SensorI2cAddrByte byte="2"></SensorI2cAddrByte>                    
<SensorI2cRate rate="100000"></SensorI2cRate> 
<SensorAvdd name="NC" min="28000000" max="28000000" delay="0"></SensorAvdd> 
<SensorDvdd name="NC" min="12000000" max="12000000" delay="0"></SensorDvdd> 
<SensorDovdd name="NC" min="18000000" max="18000000" delay="5000"></SensorDovdd> 
<SensorMclk mclk="24000000" delay="1000"></SensorMclk> 
<SensorGpioPwen ioname="RK30_PIN1_PC7" active="1" delay="1000"></SensorGpioPwen>     
<SensorGpioRst ioname="RK30_PIN0_PB0" active="1" delay="1000"></SensorGpioRst> 
<SensorGpioPwdn ioname="NC" active="0" delay="0"></SensorGpioPwdn> 
<SensorFacing facing="back"></SensorFacing> 
<SensorInterface interface="MIPI"></SensorInterface>                    
<SensorMirrorFlip mirror="0"></SensorMirrorFlip>                    
<SensorOrientation orientation="0"></SensorOrientation>                    
<SensorPowerupSequence seq="1234"></SensorPowerupSequence>                    
<SensorFovParemeter h="60.0" v="60.0"></SensorFovParemeter>                    
<SensorAWB_Frame_Skip fps="15"></SensorAWB_Frame_Skip>                    
<SensorPhy phyMode="CamSys_Phy_Mipi" lane="2"phyIndex="1"sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>       
</Sensor>                
<VCM>                    
<VCMDrvName name="BuiltInSensor"></VCMDrvName>                    
<VCMName name="NC"></VCMName>                    
<VCMI2cBusNum busnum="3"></VCMI2cBusNum>                    
<VCMI2cAddrByte byte="0"></VCMI2cAddrByte>                    
<VCMI2cRate rate="0"></VCMI2cRate>                    
<VCMVdd name="NC" min="0" max="0"></VCMVdd>                    
<VCMGpioPwdn ioname="NC" active="0"></VCMGpioPwdn>                    
<VCMGpioPower ioname="NC" active="0"></VCMGpioPower>                    
<VCMCurrent start="20" rated="80" vcmmax="100" stepmode="13"  drivermax="100"></VCMCurrent>                
</VCM><Flash>      
<FlashName name="Internal"></FlashName>      
<FlashI2cBusNum busnum="0"></FlashI2cBusNum>      
<FlashI2cAddrByte byte="0"></FlashI2cAddrByte>      
<FlashI2cRate rate="0"></FlashI2cRate>      
<FlashTrigger ioname="NC" active="0"></FlashTrigger>      
<FlashEn ioname="NC" active="0"></FlashEn>      
<FlashModeType mode="1"></FlashModeType>      
<FlashLuminance luminance="0"></FlashLuminance>      
<FlashColorTemp colortemp="0"></FlashColorTemp>   
</Flash></HardWareInfo> <SoftWareInfo>      
<AWB>               
<AWB_Auto support="1"></AWB_Auto>               
<AWB_Incandescent support="1"></AWB_Incandescent>               
<AWB_Fluorescent support="1"></AWB_Fluorescent>                
<AWB_Warm_Fluorescent support="1"></AWB_Warm_Fluorescent>                
<AWB_Daylight support="1"></AWB_Daylight>                
<AWB_Cloudy_Daylight support="1"></AWB_Cloudy_Daylight>                
<AWB_Twilight support="1"></AWB_Twilight>                
<AWB_Shade support="1"></AWB_Shade>         
</AWB>         
<Sence>                
<Sence_Mode_Auto support="1"></Sence_Mode_Auto>                
<Sence_Mode_Action support="1"></Sence_Mode_Action>               
<Sence_Mode_Portrait support="1"></Sence_Mode_Portrait>                
<Sence_Mode_Landscape support="1"></Sence_Mode_Landscape>                
<Sence_Mode_Night support="1"></Sence_Mode_Night>               
<Sence_Mode_Night_Portrait support="1"></Sence_Mode_Night_Portrait>                
<Sence_Mode_Theatre support="1"></Sence_Mode_Theatre>               
<Sence_Mode_Beach support="1"></Sence_Mode_Beach>                
<Sence_Mode_Snow support="1"></Sence_Mode_Snow>                
<Sence_Mode_Sunset support="1"></Sence_Mode_Sunset>                
<Sence_Mode_Steayphoto support="1"></Sence_Mode_Steayphoto>                
<Sence_Mode_Pireworks support="1"></Sence_Mode_Pireworks>                
<Sence_Mode_Sports support="1"></Sence_Mode_Sports>                
<Sence_Mode_Party support="1"></Sence_Mode_Party>               
<Sence_Mode_Candlelight support="1"></Sence_Mode_Candlelight>                
<Sence_Mode_Barcode support="1"></Sence_Mode_Barcode>                
<Sence_Mode_HDR support="1"></Sence_Mode_HDR>         
</Sence>         
<Effect>                
<Effect_None support="1"></Effect_None>                
<Effect_Mono support="1"></Effect_Mono>                
<Effect_Solarize support="1"></Effect_Solarize>                
<Effect_Negative support="1"></Effect_Negative>                
<Effect_Sepia support="1"></Effect_Sepia>                
<Effect_Posterize support="1"></Effect_Posterize>                
<Effect_Whiteboard support="1"></Effect_Whiteboard>               
<Effect_Blackboard support="1"></Effect_Blackboard>                
<Effect_Aqua support="1"></Effect_Aqua>          
</Effect>                
<FocusMode>                    
<Focus_Mode_Auto support="1"></Focus_Mode_Auto>                    
<Focus_Mode_Infinity support="1"></Focus_Mode_Infinity>                    
<Focus_Mode_Marco support="1"></Focus_Mode_Marco>                    
<Focus_Mode_Fixed support="1"></Focus_Mode_Fixed>                    
<Focus_Mode_Edof support="1"></Focus_Mode_Edof>         
<Focus_Mode_Continuous_Video support="0"></Focus_Mode_Continuous_Video>         
<Focus_Mode_Continuous_Picture support="1"></Focus_Mode_Continuous_Picture>           </FocusMode>                
<FlashMode>                    
<Flash_Mode_Off support="1"></Flash_Mode_Off>                    
<Flash_Mode_On support="1"></Flash_Mode_On>                    
<Flash_Mode_Torch support="1"></Flash_Mode_Torch>                    
<Flash_Mode_Auto support="1"></Flash_Mode_Auto>                    
<Flash_Mode_Red_Eye support="1"></Flash_Mode_Red_Eye>                
</FlashMode>                
<AntiBanding>                    
<Anti_Banding_Auto support="1"></Anti_Banding_Auto>                    
<Anti_Banding_50HZ support="1"></Anti_Banding_50HZ>                    
<Anti_Banding_60HZ support="1"></Anti_Banding_60HZ>                    
<Anti_Banding_Off support="1"></Anti_Banding_Off>                
</AntiBanding>                
<HDR support="1"></HDR>                
<ZSL support="1"></ZSL>                
<DigitalZoom support="1"></DigitalZoom>                
<Continue_SnapShot support="1"></Continue_SnapShot>                
<PreviewSize width="800" height="600"></PreviewSize>                
<DV>
<DV_QCIF name="qcif" width="176" height="144" fps="10" support="1"></DV_QCIF>
<DV_QVGA name="qvga" width="320" height="240" fps="10" support="1"></DV_QVGA>
<DV_CIF name="cif" width="352" height="288" fps="10" support="1"></DV_CIF>
<DV_VGA name="480p" width="640" height="480" fps="10" support="0"></DV_VGA>
<DV_480P name="480p" width="720" height="480" fps="10" support="0"></DV_480P>
<DV_720P name="720p" width="1280" height="720" fps="10" support="1"></DV_720P>
<DV_1080P name="1080p" width="1920" height="1080" fps="10" support="1"></DV_1080P>    </DV>            
</SoftWareInfo>        
</CamDevie>
//DVP       
<CamDevie>
<HardWareInfo>
<Sensor>
<SensorName name="OV5640" ></SensorName>
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_2"></SensorDevID>
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
<SensorI2cBusNum busnum="4"></SensorI2cBusNum>
<SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
<SensorI2cRate rate="100000"></SensorI2cRate>
<SensorMclk mclk="24000000"></SensorMclk>
<SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
<SensorDovdd name="NC" min="18000000" max="18000000"></SensorDovdd>
<SensorDvdd name="NC" min="0" max="0"></SensorDvdd>
<SensorGpioPwen ioname="NC" active="1"></SensorGpioPwen>
<SensorGpioRst ioname="NC" active="0"></SensorGpioRst>
<SensorGpioPwdn ioname="RK30_PIN2_PB4" active="1"></SensorGpioPwdn>
<SensorFacing facing="front"></SensorFacing>
<SensorInterface mode="CCIR601"></SensorInterface>
<SensorMirrorFlip mirror="0"></SensorMirrorFlip>
<SensorOrientation orientation="0"></SensorOrientation>
<SensorPowerupSequence seq="1234">
<SensorFovParemeter h="60.0" v="60.0"></SensorFovParemeter>
<SensorAWB_Frame_Skip fps="15"></SensorAWB_Frame_Skip>
<SensorPhy phyMode="CamSys_Phy_Cif" sensor_d0_to_cif_d="0" cif_num="0" sensorFmt="CamSys_Fmt_Raw_10b"></SensorPhy>
</Sensor><VCM><VCMDrvName name="NC"></VCMDrvName>                
<VCMName name="NC"></VCMName><VCMI2cBusNum busnum="0"></VCMI2cBusNum>
<VCMI2cAddrByte byte="0"></VCMI2cAddrByte><VCMI2cRate rate="0"></VCMI2cRate>
<VCMVdd name="NC" min="0" max="0"></VCMVdd>
<VCMGpioPwdn ioname="NC" active="0"></VCMGpioPwdn>
<VCMGpioPower ioname="NC" active="0"></VCMGpioPower>
<VCMCurrent start="20" rated="80" vcmmax="100" stepmode="13"  drivermax="100"></VCMCurrent></VCM>
<Flash>        
<FlashName name="NC"></FlashName>        
<FlashI2cBusNum busnum="0"></FlashI2cBusNum>        
<FlashI2cAddrByte byte="0"></FlashI2cAddrByte>        
<FlashI2cRate rate="0"></FlashI2cRate>        
<FlashTrigger ioname="NC" active="1"></FlashTrigger>        
<FlashEn ioname="NC" active="1"></FlashEn>        
<FlashModeType mode="0"></FlashModeType>                
<FlashLuminance luminance="0"></FlashLuminance>        
<FlashColorTemp colortemp="0"></FlashColorTemp></Flash>
</HardWareInfo>
<SoftWareInfo>
<AWB>
<AWB_Auto support="1"></AWB_Auto>        
<AWB_Incandescent support="1"></AWB_Incandescent>
<AWB_Fluorescent support="1"></AWB_Fluorescent>
<AWB_Warm_Fluorescent support="1"></AWB_Warm_Fluorescent>
<AWB_Daylight support="1"></AWB_Daylight>
<AWB_Cloudy_Daylight support="1"></AWB_Cloudy_Daylight>        
<AWB_Twilight support="1"></AWB_Twilight>
<AWB_Shade support="1"></AWB_Shade></AWB>
<Sence>
<Sence_Mode_Auto support="1"></Sence_Mode_Auto>
<Sence_Mode_Action support="1"></Sence_Mode_Action>
<Sence_Mode_Portrait support="1"></Sence_Mode_Portrait
<Sence_Mode_Landscape support="1"></Sence_Mode_Landscape>
<Sence_Mode_Night support="1"></Sence_Mode_Night>
<Sence_Mode_Night_Portrait support="1"></Sence_Mode_Night_Portrait>
<Sence_Mode_Theatre support="1"></Sence_Mode_Theatre>
<Sence_Mode_Beach support="1"></Sence_Mode_Beach>
<Sence_Mode_Snow support="1"></Sence_Mode_Snow>
<Sence_Mode_Sunset support="1"></Sence_Mode_Sunset>
<Sence_Mode_Steayphoto support="1"></Sence_Mode_Steayphoto>
<Sence_Mode_Pireworks support="1"></Sence_Mode_Pireworks>
<Sence_Mode_Sports support="1"></Sence_Mode_Sports>
<Sence_Mode_Party support="1"></Sence_Mode_Party>
<Sence_Mode_Candlelight support="1"></Sence_Mode_Candlelight>
<Sence_Mode_Barcode support="1"></Sence_Mode_Barcode>
<Sence_Mode_HDR support="1"></Sence_Mode_HDR>
</Sence>
<Effect>
<Effect_None support="1"></Effect_None>
<Effect_Mono support="1"></Effect_Mono>
<Effect_Solarize support="1"></Effect_Solarize>        
<Effect_Negative support="1"></Effect_Negative>
<Effect_Sepia support="1"></Effect_Sepia>
<Effect_Posterize support="1"></Effect_Posterize>        
<Effect_Whiteboard support="1"></Effect_Whiteboard>
<Effect_Blackboard support="1"></Effect_Blackboard>
<Effect_Aqua support="1"></Effect_Aqua>
</Effect><FocusMode><Focus_Mode_Auto support="0"></Focus_Mode_Auto>
<Focus_Mode_Infinity support="0"></Focus_Mode_Infinity>
<Focus_Mode_Marco support="0"></Focus_Mode_Marco>
<Focus_Mode_Fixed support="0"></Focus_Mode_Fixed>
<Focus_Mode_Edof support="0"></Focus_Mode_Edof>
<Focus_Mode_Continuous_Video support="0"></Focus_Mode_Continuous_Video>
<Focus_Mode_Continuous_Picture support="0"></Focus_Mode_Continuous_Picture>
</FocusMode>
<FlashMode>
<Flash_Mode_Off support="1"></Flash_Mode_Off>
<Flash_Mode_On support="1"></Flash_Mode_On
><Flash_Mode_Torch support="1"></Flash_Mode_Torch>
<Flash_Mode_Auto support="1"></Flash_Mode_Auto>
<Flash_Mode_Red_Eye support="1"></Flash_Mode_Red_Eye>
</FlashMode>
<AntiBanding>
<Anti_Banding_Auto support="1"></Anti_Banding_Auto>
<Anti_Banding_50HZ support="1"></Anti_Banding_50HZ>
<Anti_Banding_60HZ support="1"></Anti_Banding_60HZ>
<Anti_Banding_Off support="1"></Anti_Banding_Off></AntiBanding>        
<HDR support="1"></HDR>
<ZSL support="1"></ZSL>
<DigitalZoom support="1"></DigitalZoom>
<Continue_SnapShot support="1"></Continue_SnapShot>
<PreviewSize width="800" height="600"></PreviewSize>
<DV>
<DV_QCIF name="qcif" width="176" height="144" fps="10" support="1"></DV_QCIF>
<DV_QVGA name="qvga" width="320" height="240" fps="10" support="1"></DV_QVGA>
<DV_CIF name="cif" width="352" height="288" fps="10" support="1"></DV_CIF>
<DV_VGA name="480p" width="640" height="480" fps="10" support="1"></DV_VGA>
<DV_480P name="480p" width="720" height="480" fps="10" support="0"></DV_480P>
<DV_720P name="720p" width="1280" height="720" fps="10" support="0"></DV_720P>        <DV_1080P name="1080p" width="1920" height="1080" fps="10" support="0"></DV_1080P>
</DV>
</SoftWareInfo>
</CamDevie>
</BoardFile>
```
You can modify these contents:

* Sensor name
```
<SensorName name="OV13850" ></SensorName>
```
This name must match the name of the sensor driver, which has the following format:
```
libisp_isi_drv_OV13850.so
libisp_isi_drv_OV5640.so
```
This driver file will be created in directory " out/target/product/rk3288/system/lib/hw/" after  an Android build.
```
<SensorDevID IDname="CAMSYS_DEVID_SENSOR_1A"></SensorDevID>
```
The ID must be unique among the camera drivers.You can fill with the following values:
```
 CAMSYS_DEVID_SENSOR_1A
 CAMSYS_DEVID_SENSOR_1B
 CAMSYS_DEVID_SENSOR_2
```
* Collector Name
```
<SensorHostDevID busnum="CAMSYS_DEVID_MARVIN" ></SensorHostDevID>
```
Currently only support:
```
CAMSYS_DEVID_MARVIN
```
* I2C channel number
```
<SensorI2cBusNum busnum="1"></SensorI2cBusNum>
```
You can get the number via schematic or datasheet of RK3399.

* Length of Register Address (in bytes)
```
<SensorI2cAddrByte byte="2"></SensorI2cAddrByte>
```
* Clock of I2C (in Hz)
```
<SensorI2cRate rate="100000"></SensorI2cRate>
```
* MCLK of Camera (in Hz)
```
<SensorMclk mclk="24000000"></SensorMclk>
```
* PMU LDO Name Connected to AVDD. Set to NC if not connected.
```
<SensorAvdd name="NC" min="0" max="0"></SensorAvdd>
```
* PMU LDO Name Connected t

## DEBUG

Modify /system/etc/cam_board.xml with adb shell and reboot for active it
## FAQs

1. Cannot open the camera, check the sensor I2C first. If not, check （Power/PowerDown/Reset/Mclk/I2cBus）
2. support listː
* 13Mː OV13850/IMX214-0AQH5 
* 8Mː OV8825/OV8820/OV8858-Z(R1A)/OV8858-R2A
* 5Mː OV5648/OV5640
* 2Mː OV2680
* Read SDK/RKDocs for detail.
