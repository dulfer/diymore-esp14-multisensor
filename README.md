# diymore-esp14-multisensor
Tinkering with the DiyMore sensor/communication module holding HDC1080, BMP280, BH1750 and ESP8266 ESP-14 WIFI module. 

> This repository will hold my notes, code and references during my attempts getting this thing to work and adding it as a multi-sensor to my home automation. At this point it is a work in process, so the content might disappoint you. However, you are invited to comment and contribute. 

![DiyMore](https://cdn.shopify.com/s/files/1/0122/7558/0986/products/hdc1080-temperature-and-humidity-bmp280-pressure-sensor-esp8266-wifi-module-diymore_2_474_1024x1024.jpg?v=1539768549 "DiyMore HDC1080 Multi Sensor")

## Description
During a search for indoor illumination sensors for my home automation projects, I stumbled across a very promising multi-sensor manufactured by DiyMore. The sensor is built around an Espressif ESP-14 MCU and has the following sensors on board:
 * __BMP280__ barometric pressure sensor, 
 * __HDC1080__ Low Power, High Accuracy Digital Humidity Sensor with Temperature Sensor
 * __BH1750__ ambient light sensor

## The module
This small nifty module is sold on [DIY MORE](https://www.diymore.cc/)'s website and [AliExpress](https://nl.aliexpress.com/item/ESP12F-ESP12-BMP280-HDC1080-BH1750FVI-Temperature-And-Humidity-Light-Pressure-Sensor-WIFI-Transmission-Module-For-LAN/32885147039.html?spm=a2g0z.10010108.1000016.1.1c9b653dQ3U9Sj&isOrigTitle=true) as _HDC1080 Temperature And Humidity BMP280 Pressure Sensor ESP8266 WIFI Module_. Priced at around $15 (May 2019) this module offers 3 very useful sensors in a small form factor.

### BMP280
Bosch BMP280 is an absolute barometric pressure sensor especially designed for mobile applications. The sensor module is housed in an extremely compact package. Its small dimensions and its low power consumption allow for the implementation in battery powered devices such as mobile phones, GPS modules or watches.

Communication with the sensor is either over SPI or I2C (I2C is default).  
The datasheet mentions the 7bit device address depends on whether SDO is connected to ground __0x76__ or to Vcc __0x77__. 

| SDO State | Device Address |
| --- | --- |
| HIGH | 0x77 |
| LOW  | 0x76 |

_[TODO: tracing the tracks under the silks screen to determine board config ]_


[BMP280 Datasheet](https://ae-bst.resource.bosch.com/media/_tech/media/datasheets/BST-BMP280-DS001.pdf)

### HDC1080
The Texas Instruments HDC1080 is a digital humidity sensor with integrated temperature sensor that provides excellent measurement accuracy up to 14bit at very low power. The sensor comes in a 6-pin PWSON package and is __factory calibrated__.

 * Relative Humidity Accuracy ±2% (typical)
 * Temperature Accuracy ±0.2°C (typical)
 
 Communication with the sensors is done using I2C.

[HDC1080 Datasheet](http://www.ti.com/lit/ds/symlink/hdc1080.pdf)

### BH1750
The BH1750 ambient light intensity sensor breakout board has a 16-bit A2D converter built-in that can directly output a digital signal. The output from the sensor is in Lux (Lx) and does not require advanced calculations in the sketch. It is possible to detect wide range at High resolution (1 - 65535 lx)

The BH1750 communicates using I2C.  
The datasheet mentions the device address depends on ADDR is high (above 0.7VCC) __0x5C__ or low (below 0.3VCC) __0x23__

| ADDR State | Device Address |
| --- | --- |
| HIGH | 0x5C |
| LOW  | 0x23 |

[BH1750 Datasheet](https://www.mouser.com/ds/2/348/bh1750fvi-e-186247.pdf)

### Espressif ESP-14
[ToDo]

## Communicating with the sensors
DiyMore does not provide any example code or information on how to talk to the sensors. Various reviews show that DiyMore is not very responsive to buyers asking for pinouts or support, so we are on our own here. 
 
Luckily all sensors communicate using I2C so that must be the way forward. Required pins to the I2C interface are broken out on the board and the datasheets offer all the information about I2C communication with the individual sensors.

## Development environment
My weapon of choice for writing code is Visual Studio Code, for the purpose of programing an ESP extended with Platform.IO (and other extensions which are not relevant to this project). I found PlatformIO to save me lots of headache trying to track down and install missing libraries for projects and offer a convenient way of compiling and uploading my code to the ESP without having to leave the IDE.

[TODO: *information on how to setup VSCode and PlatformIO here / link to seperate markdown file?* ]

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to add or change so we can have an open discussion on this.

## References
This board was featured in Andreas Spiess' mailbox video on YouTube:  
[Mailbag incl. defective LiPo, Co2 sensor, ESP8266 Sensor module, RPi PoE, 120V inverter, OneThinx](https://www.youtube.com/watch?v=3M9biP1ilsE&t=770s) (9 June 2019)

I2C - https://en.wikipedia.org/wiki/I%C2%B2C | https://i2c.info/
