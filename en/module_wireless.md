# Wireless Module
## SLM630B 4G module
### Product Parameter
* Model
* SLM630B
* Power supply
* 3.3V～4.2V (Typical 3.3V)
* Band
* TD-LTE: Band38/39/40/41
* FDD-LTE： B1/3
* WCDMA: B1
* TD-SCDMA: B34/39
* EVDO: BC0
* CDMA 1x: BC0
* GSM: Band 8/3
* Data
* TD-LTE： Rel 9 Cat3 TD-LTE 61Mbps/18Mbps
* FDD-LTE： Rel 9 Cat3 FDD-LTE 100Mbps/50Mbps
* DC-HSPA+： 42Mbps/5.76Mbps
* HSPA： 7.2Mbps/5.76Mbps
* WCDMA： 384kbps/384kbps
* EVDO RevB： 14.7Mbps/5.4Mbps
* EVDO RevA： 3.1Mbps/1.8Mbps
* TD-HSPA： 4.2Mbps/2.2Mbps
* TD-SCDMA： 384kbps/384kbps
* CDMA： 1x 153.6kbps/153.6kbps
* EDGE： 236.8kbps/236.8kbps
* GPRS： 85.6kbps/85.6kbps
* CSD： GSM CSD: 14.4kbps
* TCP/IP：Embedded TCP/IP protocol stack
* Connector
* PCI express Mini Card connector
* HRS U.FL-R-SMT-1 RF connector ×3
* Dimensions
* 51.0×30.0×4.8mm
* Weight
* Approvals
* RoHS  CCC
* Related technical information：SLM630B

### Firmware follow
<font color=#ff0000>SLM630B USB 4G module firmware link:</font>[firmware](https://drive.google.com/drive/folders/0B7HO8lbGgAqAWEZYVDZuNjBrU00)  
<font color=#ff0000>SLM630B USB 4G module patch link:</font>[patch](https://drive.google.com/drive/folders/0B7HO8lbGgAqAWEZYVDZuNjBrU00)
### Picture
![](img/wireless_en.jpg)
### Connection Method
* Connection Method１（PCIe）
![](img/4G_connetion_en.jpg)
* Connection Method２（USB）
![](img/wireless_connetion_antenna.png)
## EC20 4G module
### Product Parameter
* Model
* EC20-C R2.0 Mini PCIe-C
* Power supply
* 3.3V～3.6V (Typical 3.3V)
* Band
* TDD-LTE: B38/B39/B40/B41
* FDD-LTE：B1/B3/B8
* WCDMA: B1/B8
* TD-SCDMA: B34/B39
* GSM: 900/1800
* Data
* TDD-LTE： Max 130Mbps (DL) Max 35Mbps (UL)
* FDD-LTE： Max 150Mbps (DL) Max 50Mbps (UL)
* DC-HSPA+： Max 42Mbps (DL) Max 5.76Mbps (UL)
* UMTS： Max 384Kbps (DL) Max 384Kbps (UL)
* CDMA： Max 3.1Mbps (DL) Max 1.8Mbps (UL)
* TD-SCDMA： Max 4.2Mbps (DL) Max 2.2Mbps (UL)
* EDGE： Max 236.8Kbps (DL) Max 236.8Kbps (UL)
* GPRS： Max 85.6Kbps (DL) Max 85.6Kbps (UL)
* Connector
* USB: USB 2.0 high-speed interface, 480Mbps
* Digital voice: 1 digital voice interface (optional)
* USIM: 1.8 V / 3 V
* Network instructions: * 2, NET_STATUS, and NET_MODE
* UART: x 1 UART
* Reset: low level
* PWRKEY: low level
* Antenna interface: x 3 (main antenna, split antenna and GNSS antenna interface)
* ADC: x 2
* Dimensions
* 51.0×30.0×4.9mm
* Weight
* About 10.5 g
* Approvals
* CCC/ NAL*/ TA*
* Related technical information：[EC20](http://0.1.225.78/)

### Firmware follow
<font color=#ff0000>The official website of the public version of the default firmware support EC20 4Gdongle module, Baidu Pan link：</font>[public version of the default firmware](http://www.t-firefly.com/share/index/listpath/id/ab0b19d49d35105ec574a2caf307103e.html)  

<font color=#ff0000>The EC20 USB 4G module patch link:</font>[patch](https://drive.google.com/drive/folders/0B7HO8lbGgAqAWEZYVDZuNjBrU00)
## GPS Module
### Product Parameter
![](img/GPS_parameter_en.jpg)
### 1.Firmware download
The firmware defaults to open the GPS :[Download Link]()
 
### 2.Modification method
GPS is available by default in firmware above, but the public version firmware defaults to close it. You can modify "<font color=#ff0000>ro.factory.hasGPS</font>" in "<font color=#ff0000>/system/build.prop</font>" to close or open GPS. Then, please restart your board after completing the modification. 
### 3.Note
The GPS function will occupy uart4, so please follow above modification method to close the GPS before using uart4 to do other things.
