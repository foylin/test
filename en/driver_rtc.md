# RTC
## Introduction
There are one RTC(Real Time Clock) integrated in RK808 on the Firefly-RK3399 .The main capabilities are clock, calendar，alarm, timer, and two 32KHz clock ouput.  
Connect CR2032  to J2 for providing rtc power if want to keep rtc running when main power goes off. connect like the following picture:  
![](img/Rk3399-rtcbat-pos-en.jpg)

## Driver
RTC's DTS information integrated in rk808 note  

driver path: `drivers/rtc/rtc-rk808.c`

## Interface
Linux has three main userspace RTC API:  
* SYSFS：/sys/class/rtc/rtc0/
* PROCFS： /proc/driver/rtc
* IOCTL： /dev/rtc0

### SYSFS
you can use cat/echo to operate the interfaces in `/sys/class/rtc/rtc0/`.  

for example,you can check date/time like this:
```
# cat /sys/class/rtc/rtc0/date
2013-01-18                                                              
#cat /sys/class/rtc/rtc0/time                                            
09:36:10
```
and set the boot time, such as boot after 120s:
```
echo +120 >  /sys/class/rtc/rtc0/wakealarm
cat /sys/class/rtc/rtc0/wakealarm
reboot -p
```
### PROCFS
List RTC information:
```
# cat /proc/driver/rtc                  
rtc_time        : 09:34:59                                   
rtc_date        : 2013-01-18                              
alrm_time       : 08:52:45                            
alrm_date       : 2013-01-18                            
alarm_IRQ       : no                             
alrm_pending    : no                                     
update IRQ enabled      : no                            
periodic IRQ enabled    : no                             
periodic IRQ frequency  : 1                           
max user IRQ frequency  : 64                             
24hr            : yes
```
### IOCTL
The ioctl() calls supported by `/dev/rtc0` are also supported by the RTC class framework.

Please refer to rtc.txt  for detailed instructions.

## FAQs
### Q1: RTC clock does not  synchronize after power on
A1: check whether the battery connect to the J2.
