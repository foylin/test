# LCD
## Introduction
Firefly-RK3399 has two LCD screen interface, one is EDP, one is MIPI, the corresponding interface board position is as follows:
![](img/lcd1.jpg)

## Configuration
Take Android7.1，Android6.0 refer to Android6.0 LCD Usage.

The Firefly-RK3399 default configuration file `kernel/arch/arm64/configs/firefly_defconfig` have set up the LCD configuration file, if you want to modify, please note that the following configuration: 
```
CONFIG_LCD_MIPI=y
CONFIG_MIPI_DSI=y
CONFIG_RK32_MIPI_DSI=y
```
## Configure DTS
### LCD pins configuration
The Firefly-RK3399 SDK has an LCD DTS file in `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-mini-edp.dts`, From this file, we can see the following statement:  
```
   / {
	model = "Firefly-RK3399 Board edp (Android)";
	compatible = "rockchip,android", "rockchip,rk3399-firefly-edp", "rockchip,rk3399";
	edp_panel: edp-panel {
		compatible = "lg,lp079qx1-sp0v";
		bus-format = <MEDIA_BUS_FMT_RGB666_1X18>;
		backlight = <&backlight>;
		ports {
			panel_in_edp: endpoint {
				remote-endpoint = <&edp_out_panel>;
			};
		};
		power_ctr: power_ctr {
              rockchip,debug = <0>;
              lcd_en: lcd-en {
                      gpios = <&gpio1 1 GPIO_ACTIVE_HIGH>;
					   pinctrl-names = "default";
					   pinctrl-0 = <&lcd_panel_enable>;
                      rockchip,delay = <20>;
              };
              lcd_rst: lcd-rst {
                      gpios = <&gpio4 29 GPIO_ACTIVE_HIGH>;
					   pinctrl-names = "default";
					   pinctrl-0 = <&lcd_panel_reset>;
                      rockchip,delay = <20>;
               };
       };
	};
	test-power {
		status = "okay";
	};
};
...
   &pinctrl {
       lcd-panel {
               lcd_panel_reset: lcd-panel-reset {
                       rockchip,pins = <4 29 RK_FUNC_GPIO &pcfg_pull_up>;
               };
               lcd_panel_enable: lcd-panel-enable {
                       rockchip,pins = <1 1 RK_FUNC_GPIO &pcfg_pull_up>;
               };
       };
    };
```
The power control pins of the LCD are defined here: 
```
lcd_en(GPIO1_A1) GPIO_ACTIVE_HIGH
lcd_rst(GPIO4_D5) GPIO_ACTIVE_HIGH
```
They are active-high. For details on pin configuration, refer to the "[GPIO](driver_gpio.html)" section.

### Backlight Configuration
In DTS file `kernel/arch/arm64/boot/dts/rockchip/rk3399-firefly-core.dtsi` configuration of the backlight information.
```
/ {
    compatible = "rockchip,rk3399-firefly-core", "rockchip,rk3399";
    backlight: backlight {
        status = "disabled";
        compatible = "pwm-backlight";
        pwms = <&pwm0 0 25000 0>;
        brightness-levels = <
              0   1   2   3   4   5   6   7
              8   9  10  11  12  13  14  15
             16  17  18  19  20  21  22  23
             24  25  26  27  28  29  30  31
             32  33  34  35  36  37  38  39
             40  41  42  43  44  45  46  47   
             48  49  50  51  52  53  54  55
             56  57  58  59  60  61  62  63  
             64  65  66  67  68  69  70  71  
             72  73  74  75  76  77  78  79  
             80  81  82  83  84  85  86  87  
             88  89  90  91  92  93  94  95
             96  97  98  99 100 101 102 103  
            104 105 106 107 108 109 110 111  
            112 113 114 115 116 117 118 119  
            120 121 122 123 124 125 126 127  
            128 129 130 131 132 133 134 135
            136 137 138 139 140 141 142 143 
            144 145 146 147 148 149 150 151 
            152 153 154 155 156 157 158 159 
            160 161 162 163 164 165 166 167 
            168 169 170 171 172 173 174 175        
            176 177 178 179 180 181 182 183 
            184 185 186 187 188 189 190 191 
            192 193 194 195 196 197 198 199 
            200 201 202 203 204 205 206 207 
            208 209 210 211 212 213 214 215 
            216 217 218 219 220 221 222 223 
            224 225 226 227 228 229 230 231
            232 233 234 235 236 237 238 239  
            240 241 242 243 244 245 246 247  
            248 249 250 251 252 253 254 255>;
        default-brightness-level = <200>;
    };
......
};
```
Pwms: the attribute of PWM. The Firelfy-RK3399 use pwm0. 25000ns is the cycle of PWM(40 KHz). brightness-level: array for back light, the max value is 255. The configuration of dark and light area default-brightness-level: default brightness, the range of this value is 0~255. enable-gpios: the enable pin of back light. See the documentations in the follow path for more information: `Documentation/devicetree/bindings/video/backlight/pwm-backlight.txt`

## The configuration of display sequence
*  Kernel  
Timing writed to panel-simple.c ，using short character  to match directly.

The following statement is in the drivers/gpu/drm/panel/panel-simple.c file：
```
static const struct drm_display_mode lg_lp079qx1_sp0v_mode = {
	.clock = 200000,
	.hdisplay = 1536,
	.hsync_start = 1536 + 12,
	.hsync_end = 1536 + 12 + 16,
	.htotal = 1536 + 12 + 16 + 48,
	.vdisplay = 2048,
	.vsync_start = 2048 + 8,
	.vsync_end = 2048 + 8 + 4,
	.vtotal = 2048 + 8 + 4 + 8,
	.vrefresh = 60,
	.flags = DRM_MODE_FLAG_NVSYNC | DRM_MODE_FLAG_NHSYNC,
};
static const struct panel_desc lg_lp097qx1_spa1 = {
	.modes = &lg_lp097qx1_spa1_mode,
	.num_modes = 1,
	.size = {
		.width = 320,
		.height = 187,
	},
};
... ...
   static const struct of_device_id platform_of_match[] = {
	{
		.compatible = "simple-panel",
		.data = NULL,
	},{
......
	}, {
		.compatible = "lg,lp079qx1-sp0v",
		.data = &lg_lp079qx1_sp0v,
	}, {
......
	}, {
		/* sentinel */
	}
 };
```
MODULE_DEVICE_TABLE(of, platform_of_match); This  structure ”lg_lp079qx1_sp0v_mode“ configures the LCD parametere.
* U-boot  
Timing writed to rockchip_panel.c ，using short character  to match directly. The following statement is in thedrivers/video/rockchip_panel.c file：
```
static const struct drm_display_mode lg_lp079qx1_sp0v_mode = {
	.clock = 200000,
	.hdisplay = 1536,
	.hsync_start = 1536 + 12,
	.hsync_end = 1536 + 12 + 16,
	.htotal = 1536 + 12 + 16 + 48,
	.vdisplay = 2048,
	.vsync_start = 2048 + 8,
	.vsync_end = 2048 + 8 + 4,
	.vtotal = 2048 + 8 + 4 + 8,
	.vrefresh = 60,
	.flags = DRM_MODE_FLAG_NVSYNC | DRM_MODE_FLAG_NHSYNC,
};
static const struct rockchip_panel g_panel[] = {
	{
		.compatible = "lg,lp079qx1-sp0v",
		.mode = &lg_lp079qx1_sp0v_mode,
	}, 
    {
		.compatible = "auo,b125han03",
		.mode = &auo_b125han03_mode,
	},
};
```
This  structure ”lg_lp079qx1_sp0v_mode“ configures the LCD parameters.

As to timing’s attribute, you can take a look at this picture.  
![](img/lcd2.png)
