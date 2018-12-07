# Senial Debug

## Buy Adapter
In online shops, there are many USB to serial adapters, which are categorized by the following chips:
* PL2303
* CH340
* CP210X

<font color=#ff0000>Note：The default baud rate for RK3399 is 1500000,Some USB to serial chip baud rate can not support 1500000，So before you buy it, make sure it supports 1500000.</font>

## Hardware Connection
There are four wires with different colors:
* Red: 3.3V Power, no need to connect.
* Black: GND，Ground, connect to GND pin of the board.
* White: TXD，Transmit，connect to TX pin of the board.
* Green: RXD，Receive, connect to RX pin of the board.
Connect as shown below
![](img/debug1.jpg)

## Parameter Setting
Firefly-RK3399 use the following serial parameters:
* Baud rate: 1500000
* Data bit: 8
* Stop bit: 1
* Parity check: none
* Flow control: none

## Serial debugging under Windows
### Install Driver
Download driver and install:
   * [CH340](https://sparks.gogo.co.nz/ch340.html)
   * [PL2303](http://www.prolific.com.tw/US/ShowProduct.aspx?pcid=41)
   * [CP210X](https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers)

If PL2303 does not works under Win8，please click here[Document](https://blog.csdn.net/ropai/article/details/19619951), please find drivers with version 3.3.5.122 or before.Plug in the adapter. OS will prompt that new hardware is found and being initialized. When it finish, you can find the new COM port in the Device Manager:
![](img/debug2.png)
### Install Software
Under Windows, the common softwares with serial port support  are putty and SecureCRT. Putty is open source software. We give a brief introduction here. The use of SecureCRT is similar.  

Download putty from [4](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). We recommend to download `putty.zip`, which contains other useful utilities.  

Extract and run `PUTTY.exe`.
  * Select "Connection type" to "Serial".
  * Modify "Serial line" to the COM port found in the device manager.
  * Set "Speed" to 1500000.
  * Click "Open" button.

![](img/debug3.png)

## Serial debugging under Ubuntu
In the Ubuntu can support 1500000 baud rate serial debugging tools are:
* minicom

The following shows how to use minicom.

Install：
```
sudo apt-get install minicom
```
Connect the serial adapter, and check the corresponding serial device file by checking:`/dev/ttyUSB0`
```
$ ls /dev/ttyUSB*
/dev/ttyUSB0
```
run:
```
$ sudo minicom
Welcome to minicom 2.7
OPTIONS: I18n                 
Compiled on Jan  1 2014, 17:13:19.                           
Port /dev/ttyUSB0, 15:57:00     
Press CTRL-A Z for help on special keys
```
Based on the above tips：Press `Ctrl-a` and then press Z again to bring up the Help menu.
```
+-------------------------------------------------------------------+
|                      Minicom Command Summary                      |
|                                                                   |
|              Commands can be called by CTRL-A                |
|                                                                   |
|               Main Functions                  Other Functions     |
|                                                                   |
| Dialing directory..D  run script (Go)....G | Clear Screen.......C |
| Send files.........S  Receive files......R | cOnfigure Minicom..O |
| comm Parameters....P  Add linefeed.......A | Suspend minicom....J |
| Capture on/off.....L  Hangup.............H | eXit and reset.....X |
| send break.........F  initialize Modem...M | Quit with no reset.Q |
| Terminal settings..T  run Kermit.........K | Cursor key mode....I |
| lineWrap on/off....W  local Echo on/off..E | Help screen........Z |
| Paste file.........Y  Timestamp toggle...N | scroll Back........B |
| Add Carriage Ret...U                                              |
|                                                                   |
|             Select function or press Enter for none.              |
+-------------------------------------------------------------------+
```
Press O as prompted to enter the setup screen：
```
           +-----[configuration]------+                                                                               
           | Filenames and paths      |                                                                               
           | File transfer protocols  |                                                                               
           | Serial port setup        |                                                                               
           | Modem and dialing        |                                                                               
           | Screen and keyboard      |                                                                               
           | Save setup as dfl        |                                                                               
           | Save setup as..          |                                                                               
           | Exit                     |                                                                               
           +--------------------------+
```
Move the cursor to "Serial port setup", Press enter to enter the serial interface settings, and then enter the prompt in front of the letters, select the corresponding option, set as follows:
```
  +-----------------------------------------------------------------------+                                          
   | A -    Serial Device      : /dev/ttyUSB0                              |                                          
   | B - Lockfile Location     : /var/lock                                 |                                          
   | C -   Callin Program      :                                           |                                          
   | D -  Callout Program      :                                           |                                          
   | E -    Bps/Par/Bits       : 1500000 8N1                               |                                          
   | F - Hardware Flow Control : No                                        |                                          
   | G - Software Flow Control : No                                        |                                          
   |                                                                       |                                          
   |    Change which setting?                                              |                                          
   +-----------------------------------------------------------------------+
```
<font color=#ff0000>Note: Hardware Flow Control and Software Flow Control should be set to No, otherwise it may not be input.</font>    

After finishing the setting, go back to the previous menu and select "Save setup as dfl" to save as the default configuration, which will be used by default.
