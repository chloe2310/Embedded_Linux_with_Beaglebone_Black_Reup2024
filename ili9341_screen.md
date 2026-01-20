I. Pincontrol Subsystem
Ex: Cấu hình chân pwm trên BBB, viết một shell scrip để tăng giảm độ sáng con LED.

P9_14	GPIO		0x848		EHRPWM1A
P9_16	GPIO		0x84c		EHRPWM1B

&am33xx_pinmux {
	ehrpwm1_pins: ehrpwm1_pins {
		pinctrl-single,pins = <
			AM33XX_IOPAD(0x848, PIN_OUTPUT | MUX_MODE6) /* P9.14, EHRPWM1A */
			AM33XX_IOPAD(0x84c, PIN_OUTPUT | MUX_MODE6) /* P9.16, EHRPWM1B */
		>;
	};
};

&epwmss1 {
	status = "okay";
};

&ehrpwm1 {
	pinctrl-names = "default";
	pinctrl-0 = <&ehrpwm1_pins>;
	status = "okay";
};


1. Chắc chắn phải có driver cho linux
- search google hoặc liên hệ hãng
- mã màn: ILI9341

=> Tìm được device tree + driver

CONFIG_FB_TFT_ILI9341

2. Xác định các thành phần để bring-up màn hình
- VCC: 3.3V -> 5V										P9_06 (5V)
- GND													P9_02 (GND)

- Reset: low level enable (0V)							P9_12 (gpio1_28)
- DC/RS:												P9_13 (gpio0_31)
	1 -> gửi nhận bản tin điều khiển
	0 -> gửi nhận data
- LED: backlight tăng giảm độ sáng màn hình				P9_16 (ehrpwm1)
=> P9_03 (gpio0_3)

- CS: low level enable (0V)								P9_17 (spi0_cs0)
- SCK: clock 											P9_22 (spi0_sclk)
- MOSI: data từ bbb -> ili0341							P9_21 (spi0_d0)
- MISO: data từ ili9341 -> bbb							P9_18 (spi0_d1)

P9_21	0x954/154	spi0_d0		mode0	MISO (d0)

P9_16	0x84c/04c	ehrpwm1		mode6	LED

P9_22	0x950/150	spi0_sclk	mode0	SCK

P9_18	0x958/158	spi0_d1		mode0	MOSI (d1)

P9_13	0x874/074	gpio0_31	mode7	DC/RS

P9_12	0x878/078	gpio1_28	mode7	Reset

P9_17	0x95c/15c	spi0_cs0	mode0	CS

P9_02						GND					

P9_06						5V		


#define AM335X_PIN_SPI0_SCLK                    0x950

#define AM335X_PIN_SPI0_D0                      0x954

#define AM335X_PIN_SPI0_D1                      0x958

#define AM335X_PIN_SPI0_CS0                     0x95c

#define AM335X_PIN_SPI0_CS1                     0x960



Hai driver của màn ili9341

CONFIG_FB_TFT

CONFIG_FB_TFT_ILI9341

Cách điều khiển chân pwm

echo 0 > /sys/class/pwm/pwmchip0/export

echo 100 > /sys/class/pwm/pwmchip0/pwm0/period

echo 50 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle

echo 1 > /sys/class/pwm/pwmchip0/pwm0/enable
