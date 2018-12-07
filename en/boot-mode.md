# Boot mode description
## Introduction
RK3399 has flexible boot modes. Under normal circumstances, unless hardware is damaged, Firefly-RK3399 development board will never become brick (brick means not able to boot or flash).    

If accident happens during upgrading firmware, bootloader is broken, making it impossible to upgrade again. The device can enter into MaskRom mode as the last resort.
## Load Methods
RK3399 has 32KB BootRom and 200KB internal SRAM, which supports booting from following devices:
  * SPI
  * eMMC
  * SDMMC

That is to say, besides Nand, SPI and eMMC flash, RK3288 also supports booting from SD card.  

Moreover, RK3399 can download system code from USB Type-C.
## Boot Sequence
The boot sequence is:  
 1. Soc powers up and initializes.
 2. BootRom code runs in SRAM, loads and verifies bootloader's bootstrap code from storage device.
 3. If the verification passes, run the bootloader bootstrap code.
 4. The bootloader initializes DDR RAM, loads the complete bootloader into DDR RAM and runs it.
 5. Bootloader loads Linux kernel from storage device, then executes it.
 6. Linux kernel takes control of everything now.

## Boot Mode
RK3399 has three boot modes:
  * Normal mode
  * Loader mode
  * MaskRom mode

### Normal Mode
Normal mode, that is the normal boot procedure, load every components subsequently, and boot into operation system normally.
### Loader Mode
In Loader mode, bootloader will enter into upgrade state, waiting for commands from host, which is used in firmware upgrading.    

To enter Loader mode, make the bootloader aware that the RECOVERY key is pressed and USB cable is connected:
 1. Keep device power on.
 2. Use micro USB Type-C cable to connect host and device together.
 3. Press and hold RECOVERY key.
 4. Shortly press RESET key.
 5. Release RECOVERY key.

**Note:** If device still can not be found after pressing "RESET", then try this: long press "PWRKEY" after short pressing of "RESET", before finally releasing "RECOVERY".
### MaskRom Mode
MaskRom mode is used to fix system when bootloader is broken.  
 
In most cases, there are no need to enter MaskRom mode. When the BootRom code fails to verify bootloader (cannot read IDR block, or bootloader is broken), it will boot into MaskRom mode, which waits for host to send bootloader code through USB, then runs it, so that bootload can take control again.     

To enforce device into MaskRom mode, please check [this](maskrom.html).
