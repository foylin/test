# Ubuntu application layer support

## Video hardware codec support

Mpp is a set of video codec APIs provided by Rockchip for RK3399, and based on mpp, Rockchip provides a set of gstreamer codec plugins. Users can use the gstreamer to do video codec application according to their own needs, or directly call mpp to realize hardware codec acceleration.

The Ubuntu system released by Firefly has provided complete gstremaer and mpp support, and provides a corresponding demo for users to develop reference.

### Gstreamer

*   Ubuntu 16.04 下，gstreamer 1.12 Already installed in the /opt/ directory
*   Ubuntu 18.04下， gstreamer 1.12 Already installed in the system

/usr/local/bin/h264dec.sh  Test H264 decoding

/usr/local/bin/h264enc.sh  Test H264 coding



Users can refer to these two scripts to configure their own gstreamer application.

### Mpp

*   The mpp-related dev packages are already installed on the system.

    /opt/mpp/下分别是mpp 编解码的相关demo 和 源文件。

    /opt/mpp/ files are the demo and source files for the mpp codec.

## OpenGL-ES

RK3399 support OpenGL ES1.1/2.0/3.0/3.1。 

The Ubuntu system released by Firefly already provides full OpenGL-ES support. Run `glmark2-es2` to test openGL-ES support. If you want to avoid the impact of the screen refresh rate on the test results, you can use the following command test on the serial terminal.

```shell
# systemctl stop lightdm
# export DISPLAY=:0
# Xorg &
# glmark2-es2 –off-screen
```

In the Chromium browser, type: `chrome://gpu` in the address bar to see hardware acceleration support.

Note:  

1.  EGL is an extension of the OpenGL for the x window system on the arm platform, which is equivalent to the glx library under x86.

2.  Since the driver mode setting used by Xorg will load libglx.so by default (disabling glx will cause some applications failing to detect the glx environment), libglx.so will search the dri implementation library in the system. However, rk3399 Xorg 2D acceleration is directly based on DRM, and does not implement dri library, so libglx.so will report the following error during startup.

    ```
    (EE) AIGLX error: dlopen of /usr/lib/aarch64-linux-gnu/dri/rockchip_dri.so failed
    ```

   This has no effect on system operation and does not need to be processed.

3.  Based on the same reason, some applications will report the following errors during startup, without processing, and will not affect the operation of the application.

    ```
    libGL error: unable to load driver: rockchip_dri.so
    libGL error: driver pointer missing
    libGL error: failed to load driver: rockchip
    ```

4.  Some versions of Ubuntu released by Firefly have turned off loading libglx.so by default. In some cases, some applications will run the following error.
 
    `GdkGLExt-WARNING **: Window system doesn't support OpenGL.`

    The method of correction is as follows：

    Delete  `/etc/X11/xorg.conf.d/20-modesetting.conf`  three-line configuration。

    ```shell
    Section "Module"
         Disable     "glx"
    EndSection
    ```

    

## OpenCL

The Ubuntu system released by Firefly has added opencl1.2 support, and can run the system's built-in `clinfo` to get the platform opencl related parameters.

```
firefly@firefly:~$ clinfo 
Platform #0
 Name:                                  ARM Platform
 Version:                               OpenCL 1.2 v1.r14p0-01rel0-git(966ed26).f44c85cb3d2ceb87e8be88e7592755c3

 Device #0
   Name:                                Mali-T860
   Type:                                GPU
   Version:                             OpenCL 1.2 v1.r14p0-01rel0-git(966ed26).f44c85cb3d2ceb87e8be88e7592755c3
   Global memory size:                  1 GB 935 MB 460 kB 
   Local memory size:                   32 kB 
   Max work group size:                 256
   Max work item sizes:                 (256, 256, 256)
…
```



## TensorFlow Lite

RK3399 support for neural network GPU acceleration scheme LinuxNN, Firefly released Ubuntu system, has added support for LinuxNN.


Under opt/tensorflowbin/, run test.sh to test the Demo of the MobileNet model image classifier and the target detection of the MobileNet-SSD model.

```
firefly@firefly:/opt/tensorflowbin$ ./test.sh 
Loaded model mobilenet_ssd.tflite
resolved reporter
nn version: 1.0.0
findAvailableDevices
filename:libarmnn-driver.so     d_info:40432     d_reclen:40s
[D][ArmnnDriver]: Register Service: armnn (version: 1.0.0)!
first invoked time: 1919.17 ms 
invoked 
average time: 108.4 ms 
validCount: 26
car     @ (546, 501) (661, 586)
car     @ (1, 549) (51, 618)
person  @ (56, 501) (239, 854)
person  @ (332, 530) (368, 627)
person  @ (391, 541) (434, 652)
person  @ (418, 477) (538, 767)
person  @ (456, 487) (602, 764)
car     @ (589, 523) (858, 687)
person  @ (826, 463) (1034, 873)
bicycle @ (698, 644) (1128, 925)
write out.jpg succ!
```



## Screen rotation

The Ubuntu system released by Firefly, if you need to rotate the display direction of the system by default, you can modify the direction of the corresponding display device in /etc/default/xrandr.
```shell
firefly@firefly:~$ cat /etc/default/xrandr 
#!/bin/sh

# Rotation can be one of 'normal', 'left', 'right' or 'inverted'.

# xrandr --output HDMI-1 --rotate normal
# xrandr --output LVDS-1 --rotate normal
# xrandr --output EDP-1 --rotate normal
# xrandr --output MIPI-1 --rotate normal
# xrandr --output VGA-1 --rotate normal
# xrandr --output DP-1 --rotate normal
```

For platforms with a touch screen, if you need to rotate the direction of the touch screen, you can modify the SwapAxes / InvertX / InvertY values in /etc/X11/xorg.conf.d/05-gslX680.conf.

```shell
firefly@firefly:~$ cat /etc/X11/xorg.conf.d/05-gslX680.conf 
Section "InputClass"
        Identifier "gslX680"
        MatchIsTouchscreen "on"
        MatchProduct  "gslX680"
        Driver "evdev"
        Option "SwapAxes" "off"
       # Invert the respective axis.
        Option "InvertX" "off"
        Option "InvertY" "off"
EndSection
```

