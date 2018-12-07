# ADB Usage
## Introduction
Adb, short for Android Debug Bridge, is Android's commandline debugging utility, which can be used to trace system log, upload/download file, install applications, etc.
## Before Start
In Android system, go to "Settings" -> "Development options", check the "USB Debugging" option.

**Firefly-RK3399** Use a Type-C to connnect development board and host.

**AIO-3399J** Use a Dual male usb data cable connect host and Dual USB3.0 upper of Mainboard.(Engineering prototype connect Dual USB3.0 lower part)
* host/device mode switch.

Add the dts below，AIO-3399J default set host mode.

Turn on usb device/host mode.
**OTG ADB**
 * Settings->USB->Connect to PC

**Network ADB**
 * Setting->Developer Option->ADB over network

check the 3399 IP address, Use command under the PC  

example:

adb connect + IP  

adb shell

## Adb Installation under Windows
First, please reference [Install RK USB driver]() to get the driver ready.  

Then go to [http://adbshell.com/download/download-adb-for-windows.html](http://adbshell.com/download/download-adb-for-windows.html) to download adb.zip , uncompress it to `C:\adb` to ease later use.  

Open a cmd window, input:
```
cd C:\adb
adb shell
```
If everything works, you have entered `adb shell`, and can run all kinds of commands available in device.

## Adb Installation under Ubuntu
* Install adb:
```
sudo apt-get install android-tools-adb
```
* Add device identity:
```
mkdir -p ~/.android
vi ~/.android/adb_usb.ini
# append following line
0x2207
```
* Add udev rule:
```
sudo vi /etc/udev/rules.d/51-android.rules
# append following line:
SUBSYSTEM=="usb", ATTR{idVendor}=="2207", MODE="0666"
```
* plug the USB OTG cable out, then plug in, or run the following commands, to make udev rule take effect:
```
sudo udevadm control --reload-rules
sudo udevadm trigger
```
* Restart adb daemon:
```
sudo adb kill-server
adb start-server
```

## Common Adb Commands
### Connection Manage
List all the connected devices:
```
adb devices
```
If there are multiple devices connected, the device serial no is needed to distinguish:
```
export ANDROID_SERIAL=<device serial no>
adb shell ls
```
Adb can also use the TCP/IP network to connect to device:
```
# Restart adbd in device, which will be listened at TCP port 5555.
adb tcpip 5555
# Now, the Type-C cable can be disconnected.
# Connect to the device, whose IP is 192.168.1.100 here.
adb connect 192.168.1.100:5555
# Disconnect the device
adb disconnect 192.168.1.100:5555
```
### Debugging
**Show System Log (adb logcat)**
* Usage
```
adb logcat [Options] [Label]
```
* For example:
```
# Show all the log.
adb logcat
# Show only part of the log matching tags specified.
adb logcat -s WifiStateMachine StateMachine
```
**Running Commands (adb shell)**

**Get Details of System (adb bugreport)**  
`adb bugreport` is used for bug report, which contains lots of useful information about system.
* For example:
```
adb bugreport
# Save to host, then open it with text editor to view adb
adb bugreport >bugreport.txt
```

### Application Manage
**Install Application (adb install)**

* Usage：
```
adb install [Options] Application.apk
```
* Options are:
```
-l forward-lock
-r Reinstall application, keeping previous data
-s Install to SD card, instead of internal storage
```
* For example:
```
# Install facebook.apk
adb install facebook.apk
# Upgrade twitter.apk
adb install -r twitter.apk
```
If installation is successful, it will prompt "Success"; otherwise, it will fail with following messages:
* INSTALL_FAILED_ALREADY_EXISTS: "-r" parameter is needed to reinstall.
* INSTALL_FAILED_SIGNATURE_ERROR: The signatures do not match. It sometimes happens that the released application and the debug one have different signatures. If the signature of APK is confirmed to be valid, use adb uninstall to uninstall old application first, then try again to install.
* INSTALL_FAILED_INSUFFICIENT_STORAGE: The storage space is not enough, need to check. Install the SD card or uninstall some unused applications to free some space.

**Uninstall Application (adb uninstall)**
* Usage:
```
adb uninstall ApplicationPackageName
```
* For example:
```
adb uninstall com.android.chrome
```
The package name of application can be listed with:
```
adb shell pm list packages -f
```
Result:
```
...
package:/system/app/Bluetooth.apk=com.android.bluetooth
...
```
The former is apk filename, and the latter is the corresponding package name.

### Full Commandline Help (adb help)
```
Android Debug Bridge version 1.0.31

 -a                            - directs adb to listen on all interfaces for a connection
 -d                            - directs command to the only connected USB device
                                 returns an error if more than one USB device is present.
 -e                            - directs command to the only running emulator.
                                 returns an error if more than one emulator is running.
 -s <specific device>          - directs command to the device or emulator with the given
                                 serial number or qualifier. Overrides ANDROID_SERIAL
                                 environment variable.
 -p <product name or path>     - simple product name like 'sooner', or
                                 a relative/absolute path to a product
                                 out directory like 'out/target/product/sooner'.
                                 If -p is not specified, the ANDROID_PRODUCT_OUT
                                 environment variable is used, which must
                                 be an absolute path.
 -H                            - Name of adb server host (default: localhost)
 -P                            - Port of adb server (default: 5037)
 devices [-l]                  - list all connected devices
                                 ('-l' will also list device qualifiers)
 connect <host>[:<port>]       - connect to a device via TCP/IP
                                 Port 5555 is used by default if no port number is specified.
 disconnect [<host>[:<port>]]  - disconnect from a TCP/IP device.
                                 Port 5555 is used by default if no port number is specified.
                                 Using this command with no additional arguments
                                 will disconnect from all connected TCP/IP devices.

device commands:
  adb push [-p] <local> <remote>
                               - copy file/dir to device
                                 ('-p' to display the transfer progress)
  adb pull [-p] [-a] <remote> [<local>]
                               - copy file/dir from device
                                 ('-p' to display the transfer progress)
                                 ('-a' means copy timestamp and mode)
  adb sync [ <directory> ]     - copy host->device only if changed
                                 (-l means list but don't copy)
                                 (see 'adb help all')
  adb shell                    - run remote shell interactively
  adb shell <command>          - run remote shell command
  adb emu <command>            - run emulator console command
  adb logcat [ <filter-spec> ] - View device log
  adb forward --list           - list all forward socket connections.
                                 the format is a list of lines with the following format:
                                    <serial> " " <local> " " <remote> "\n"
  adb forward <local> <remote> - forward socket connections
                                 forward specs are one of: 
                                   tcp:<port>
                                   localabstract:<unix domain socket name>
                                   localreserved:<unix domain socket name>
                                   localfilesystem:<unix domain socket name>
                                   dev:<character device name>
                                   jdwp:<process pid> (remote only)
  adb forward --no-rebind <local> <remote>
                               - same as 'adb forward <local> <remote>' but fails
                                 if <local> is already forwarded
  adb forward --remove <local> - remove a specific forward socket connection
  adb forward --remove-all     - remove all forward socket connections
  adb jdwp                     - list PIDs of processes hosting a JDWP transport
  adb install [-l] [-r] [-d] [-s] [--algo <algorithm name> --key <hex-encoded key> --iv <hex-encoded iv>] <file>
                               - push this package file to the device and install it
                                 ('-l' means forward-lock the app)
                                 ('-r' means reinstall the app, keeping its data)
                                 ('-d' means allow version code downgrade)
                                 ('-s' means install on SD card instead of internal storage)
                                 ('--algo', '--key', and '--iv' mean the file is encrypted already)
  adb uninstall [-k] <package> - remove this app package from the device
                                 ('-k' means keep the data and cache directories)
  adb bugreport                - return all information from the device
                                 that should be included in a bug report.

adb backup [-f <file>] [-apk|-noapk] [-obb|-noobb] [-shared|-noshared] [-all] [-system|-nosystem] [<packages...>]
                               - write an archive of the device's data to <file>.
                                 If no -f option is supplied then the data is written
                                 to "backup.ab" in the current directory.
                                 (-apk|-noapk enable/disable backup of the .apks themselves
                                    in the archive; the default is noapk.)
                                 (-obb|-noobb enable/disable backup of any installed apk expansion
                                    (aka .obb) files associated with each application; the default
                                    is noobb.)
                                 (-shared|-noshared enable/disable backup of the device's
                                    shared storage / SD card contents; the default is noshared.)
                                 (-all means to back up all installed applications)
                                 (-system|-nosystem toggles whether -all automatically includes
                                    system applications; the default is to include system apps)
                                 (<packages...> is the list of applications to be backed up.  If
                                    the -all or -shared flags are passed, then the package
                                    list is optional.  Applications explicitly given on the
                                    command line will be included even if -nosystem would
                                    ordinarily cause them to be omitted.)

adb restore <file>           - restore device contents from the <file> backup archive

adb help                     - show this help message
  adb version                  - show version num

scripting:
  adb wait-for-device          - block until device is online
  adb start-server             - ensure that there is a server running
  adb kill-server              - kill the server if it is running
  adb get-state                - prints: offline | bootloader | device
  adb get-serialno             - prints: <serial-number>
  adb get-devpath              - prints: <device-path>
  adb status-window            - continuously print device status for a specified device
  adb remount                  - remounts the /system partition on the device read-write
  adb reboot [bootloader|recovery] - reboots the device, optionally into the bootloader or recovery program
  adb reboot-bootloader        - reboots the device into the bootloader
  adb root                     - restarts the adbd daemon with root permissions
  adb usb                      - restarts the adbd daemon listening on USB
  adb tcpip <port>             - restarts the adbd daemon listening on TCP on the specified port
networking:
  adb ppp <tty> [parameters]   - Run PPP over USB.
 Note: you should not automatically start a PPP connection.
 <tty> refers to the tty for PPP stream. Eg. dev:/dev/omap_csmi_tty1
 [parameters] - Eg. defaultroute debug dump local notty usepeerdns

adb sync notes: adb sync [ <directory> ]
  <localdir> can be interpreted in several ways:

- If <directory> is not specified, both /system and /data partitions will be updated.

- If it is "system" or "data", only the corresponding partition
    is updated.

environmental variables:
  ADB_TRACE                    - Print debug information. A comma separated list of the following values
                                 1 or all, adb, sockets, packets, rwx, usb, sync, sysdeps, transport, jdwp
  ANDROID_SERIAL               - The serial number to connect to. -s takes priority over this if given.
  ANDROID_LOG_TAGS             - When used with the logcat option, only these debug tags are printed.
```
