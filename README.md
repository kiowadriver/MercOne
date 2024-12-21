# Klipper 3D printer configuration backup
## _ZeroG MercuryOne.1 w/ Hydra1.1_

[![N|S](https://avatars.githubusercontent.com/u/78506178?v=4)](https://docs.zerog.one/manual/build/mercury_eva)



This repo is my online backup used to restore the klipper configuration.
Build your own MercuryOne.1 
Docs: https://docs.zerog.one/

## Components

The following is a list of current components:
- MCU: [Duet3D Mini 5+](https://www.duet3d.com/duet3mini5plus) - The hydra1.1 mod requires a total of five (5) stepper motors. Requires firmware flashed for klipper 
- MCU Power Supply: [Meanwell LRS-350-24](https://www.meanwell.com/productPdf.aspx?i=459) - 350W 24VDC Power Supply
- SBC: [Raspberry Pi 3 model B+](https://www.raspberrypi.com/products/raspberry-pi-3-model-b-plus/) - Running Klipper Mainsail
- SBC Power Supply: [GeeekPi 27W Power Supply for Raspberry Pi 5](www.amazon.com) - Must output at least 5A to power MCU 5vdc, RPI, NeoPixels and accessories
- SBC Soft Power Control: [PetRock PowerBloc](https://www.petrockblock.com/powerblock-raspberry-pi-power-button/) - Can output up to 7A depending on source USB-C supply. Provides soft power control
- Toolboard: [HUVUD v62](https://www.lukeslabonline.com/products/huvud?srsltid=AfmBOopEAh4QOpoMUqAeWhaTBG-vQzWRpcosn5MS7Hgkb8bIckftSSv-) - Simplifies wiring to the toolhead by using CAN bus
- Probe: [Beacon D](https://beacon3d.com/product/beacon/) - Eddy current sensor for mesh bed compensation
- Extruder: [Bondtech LGX Lite v1](https://www.bondtech.se/product/lgx-lite-large-gears-extruder/?srsltid=AfmBOoqgWDClp8ee0Q8WN4uUiA37GEp0CGhkhtcDBhZUndZCOz_L350P) - Extruder driven by toolboard
- Hotend: [Phaetus Papido HF 1.75"](https://www.phaetus.com/products/rapido-hotend) - High flow hotend
- Bed & Heater: [FYSETC 255mm](https://www.fysetc.com/collections/build-plate?srsltid=AfmBOoqMgZ_5Tl_YJqbWfPQtra7XZMAf-IZySl2OC9-75eakVQTpeYrE) - 110VAC AC heater controlled with SRR, JANUS BPS-PET Steel Sheet with Mic6 Metal Plate Silicone Heat Bed Andmagnetic Sticker for ZEROG 
- X-Y Steppers: [Ldo 42STH48-2504AH](https://www.fabreeko.com/products/ldo-42sth48-2504ah-high-temp?srsltid=AfmBOoodsxMzFuTsef1rY_hdTcE-_lMs57thaipDaFKRzZfVEwxLG4EP) - X and Y Steppers
- Z1 & Z2 Steppers: [Creality 42-34](https://www.creality3dofficial.com/products/2pcs-ce-certification-2phase-reprap-stepper-motor-42-motor-42-40-3d-printer?srsltid=AfmBOornuw2jv8F69XxVZhaaLflQldGlQWgPeZK1NleKhOC-i5QyUVHl) - Reuse from Ender 5
- Z Stepper: [Creality 42-40(S)](https://www.creality3dofficial.com/products/2pcs-ce-certification-2phase-reprap-stepper-motor-42-motor-42-40-3d-printer?srsltid=AfmBOornuw2jv8F69XxVZhaaLflQldGlQWgPeZK1NleKhOC-i5QyUVHl) - Reuse from Ender 5


## Firmware Installation


### Update Firmware on Duet Mini 5+:
https://www.klipper3d.org/Installation.html

```sh
Double tap the reset button so you see flashing red LED

Get the Device ID using the following:
ls /dev/serial/by-id/*

Note: By default the can0, mini5, and huvud are all set to 250K speed need to first flash huvud to 1000k, then mini5, then change can0 to be the same

cd ~/klipper
make menuconfig

Firmware Settings:
# Enable extra low-level config
# Micro Controller: SAME54P20 (SAME5x)
# Processor Model: SAME54P20
# Bootloader offset: 16KiB bootloader (Duet3)
# Clock Reference: 25Mhz crystal
# Processor Speed: 120 MHz (standard)
# Communication: (USB to CAN bus bridge)
# CAN bus interface: CAN bus on PB15/PB14
# CAN speed (250k initially but then set to 1000k)

make clean
make
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/{the id you found above}
sudo service klipper start
```


### HUVUD Firmware:
https://www.klipper3d.org/Bootloaders.html#stm32f103stm32f0x2-with-canboot-bootloader

```sh
cd ~/klipper
make menuconfig

Firmware Settings: (note, the canboot firmware is still set for 250K speed, operational is 1M)
# Enable extra low-level config
# Micro Controller: STM32
# Processor Model: STM32F103
# Bootloader offset: 8 KiB bootloader
# Clock Reference: 8Mhz crystal
# Communication: (CAN bus (on PB8/PB9))
# CAN speed (1000k)

make clean
make
cd lib/canboot
python3 flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u {Device ID}
```

## Pins For Reference

### Duet Mini 5+
Note: "!" is for inverted select pin
| Description | Pins |
| ------ | ------ |
| Driver Step Pins | 0:PC26, 1:PC25, 2:PC24, 3:PC19, 4:PC16, 5:PC30, 6:PC18 |
| Driver Dir pins | 0:PB3, 1:PB29, 2:PB28, 3:PD20, 4:PD21, 5:PB0, 6:PA27 |
| Driver Enable | !PC28 |
| Uart addresses | 0:0 1:1 2:2 3:3 4:!0 5:!1 6:!2 |
| Thermistor Pins | T0:PC0, T1:PC1, T2:PC2 |
| Vssa Sense | PB4 |
| Vref Sense | PB5 |
| Current Sense resistor for drivers | .076ohm|
| SPI lines | {PD11, PC7} -> Shared SerCom#7, SPIMosi:PC12, SPIMiso:PC15, SPISCLK:PC13|
| Vin Monitor | PC3, uses 11:1 voltage divider|
| LED's | Diag:PA31, Act:PA30|
| 12864 LCD | LCDCSPIN:PC6, ENCA:PC11, ENCB:PD1, ENCSW:PB9, LCD A0:PA2, LCDBeep:PA9, LCD Neopixel Out:PB12 (shared with IO3.out)|
| Neopixel Out | PA8, Requires external 5VDC from PowerBloc|
| Serial0 | TX:PB25, RX:PB24 (USB)|
| Serial1 | TX:PB31, RX:PB30|
| SBC SPISS | pin:PA6, SBCTfrReady:PA3, SerComPins:{PA4, PA5, PA6, PA7}|
| CAN Pins | TX:PB14 RX:PB15|
| Heaters, Fan outputs | {Out0:PB17 Out1:PC10 Out2:PB13 Out3:PB11 Out4:PA11, Out5:PB2, Out6:PB1} - Out6 is shared with VFD_Out|
| Tach Pins for Fans | {Out3.Tach:PB27 Out4.Tach:PB26}|
| GPIO_out  | {IO1:PB31 IO2:PD9 IO3:PB12 IO4:PD10} IO4 is shared with PSON|
| GPIO_in  | {IO1:PB30 IO2:PD8 IO3:PB7 IO4:PC5 IO5:PC4 IO6:PC31}|
| Driver Diag   | {D0:PA10, D1:PB8, D2:PA22, D3:PA23, D4:PC21, D5:PB10, D6:PA27}|
| Mux Pin  | PD0|

#### Z-Axis Lead Screw Positions
Note: Z1 and Z2 are the front of the printer, z is in the center and rear
| Lead Screw | Nozzle Position (X,Y) |
| ------ | ------ |
| Z [Center Rear]| 124.5,224 |
| Z1 [Front Left] | 7,0|
| Z2 [Front Right]| 245,0 |

## License

MIT
