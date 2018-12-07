# Maskrom
## Introduction
MaskRom mode is the bottom line preventing the device from bricking. Enforcing device into MaskRom mode involves hardware operation, which is risky. Therefore, please try to put the device into Loader mode, or boot the device with sd-card, before risking MaskRom mode.  

<font color=#ff0000>Please read and operate with great care!</font>
## Firefly-RK3399 enter MaskRom mode
principle：
Artificial to the Flash data pin connected to ground, the system will think Flash data error, so clearing the Flash data.
1. Locate test point TP31 and TP32, which are on the back, as following images show:
![](img/maskrom1.jpg)
2. Power down the device.
3. Plug out SD card.
4. Use a micro USB Type-C cable to connnect device and host pc.
5. Use metal tweezers to keep TP31 and TP32 connected.
6. Power on the board.
7. Wait a moment, then release the metal tweezers.

Device should enter MaskRom mode:

![](img/maskrom2.jpg)

Please check [Boot Mode](boot-mode.html) for more detail of MaskRom mode.
