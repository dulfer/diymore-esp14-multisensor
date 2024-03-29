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

## Communicating with the sensors
DiyMore does not provide any example code or information on how to talk to the sensors. Various reviews show that DiyMore is not very responsive to buyers asking for pinouts or support, so we are on our own here. 
 
Luckily all sensors communicate using I2C so that must be the way forward. Required pins to the I2C interface are broken out on the board and the datasheets offer all the information about I2C communication with the individual sensors.

## Development environment
My weapon of choice for writing code is Visual Studio Code, for the purpose of programing an ESP extended with Platform.IO (and other extensions which are not relevant to this project). I found PlatformIO to save me lots of headache trying to track down and install missing libraries for projects and offer a convenient way of compiling and uploading my code to the ESP without having to leave the IDE.

[TODO: *information on how to setup VSCode and PlatformIO here / link to seperate markdown file?* ]

## Sensors on the board
### BMP280
Bosch BMP280 is an absolute barometric pressure sensor especially designed for mobile applications. The sensor module is housed in an extremely compact package. Its small dimensions and its low power consumption allow for the implementation in battery powered devices such as mobile phones, GPS modules or watches.

Communication with the sensor is either over SPI or I2C (I2C is default).  
The datasheet mentions the 7bit device address depends on whether SDO is connected to ground __0x76__ or to Vcc __0x77__. 

| SDO State | Device Address |
| --- | --- |
| HIGH | 0x77 |
| LOW  | 0x76 |

_[TODO: tracing the tracks under the silk screen to determine board config ]_


Source: [BMP280 Datasheet](https://ae-bst.resource.bosch.com/media/_tech/media/datasheets/BST-BMP280-DS001.pdf)

### HDC1080
The Texas Instruments HDC1080 is a digital humidity sensor with integrated temperature sensor that provides excellent measurement accuracy up to 14bit at very low power. The sensor comes in a 6-pin PWSON package and is __factory calibrated__.

 * Relative Humidity Accuracy ±2% (typical)
 * Temperature Accuracy ±0.2°C (typical)
 
Communication with the sensors is done using I2C.  
The 7-bits I2C device address is __0x40__ (1000000b)

The device does a temperature measurement, followed by a humidity measurement and writes them back . 
Reading actual temperature and humidity values from the device is done by 
1. writing the desired mode and resolution to register address (__0x02__)
2. triggering the actual measurement by executing the pointer write action with address pointer set to 0x00
3. wait for a few millis while the conversion is being done, time depends on the set resolution (11-bit= ~8ms, 14-bit= ~14ms)
4. read the values from the register

Using the Wire library, the code to read values from the sensor is
```c
Wire.beginTransmission(deviceAddress);
Wire.write(pointer);
Wire.endTransmission();
	
// waiting for conversion to finish
// for 11bit resolution the conversion time is ~8ms
delay(8);
 
Wire.requestFrom(deviceAddress, 0x02);

byte msb = Wire.read();
byte lsb = Wire.read();
```

#### Conversion Time
Number of millis to wait after triggering the measurement before reading the value from the sensor  

| Resolution | Humidity | Temperature |
|---|---|---|
| 8 bit | 2.50 ms | |
| 11 bit | 3.85 ms | 3.65 ms |
| 14 bit | 6.50 ms | 6.53 ms |

Source: [HDC1080 Datasheet](http://www.ti.com/lit/ds/symlink/hdc1080.pdf)

### BH1750
The BH1750 ambient light intensity sensor breakout board has a 16-bit A2D converter built-in that can directly output a digital signal. The output from the sensor is in Lux (Lx) and does not require advanced calculations in the sketch. It is possible to detect wide range at High resolution (1 - 65535 lx)

The BH1750 communicates using I2C.  
The datasheet mentions the device address depends on ADDR is high (above 0.7VCC) __0x5C__ or low (below 0.3VCC) __0x23__

| ADDR State | Device Address |
| --- | --- |
| HIGH | 0x5C |
| LOW  | 0x23 |

Source: [BH1750 Datasheet](https://www.mouser.com/ds/2/348/bh1750fvi-e-186247.pdf)

### Espressif ESP-14
The revision 14 in this module's name suggests this board uses a relatively recent iteration of the ESP8266 powered modules, a successor of the pretty awesome ESP-12. However, investigation shows that this module is actually a weird outlyer. 

> [Elliot Williams (Hackaday)](https://hackaday.com/2017/02/13/hacking-on-the-weirdest-esp-module/) 
> "Sometimes I see a component that’s bizarre enough that I buy it just to see if I can actually do something with it. That’s the case with today’s example, the ESP-14. At first glance, you’d ask yourself what AI Thinker, the maker of many of the more popular ESP8266 modules, was thinking. (...) Slaving the ESP8266 to an STM8S is like taking a Ferrari and wrapping it inside a VW Beetle."  

Nothing wrong with the ESP mcu, but the fact that they threw in a (dirt cheap, 8-bit) STM8003 makes me worry that programming this device will not be as straight forward as I hoped. What is the role of the STM in this module? Is it responsible for handling the I2C communication? How does it interface with the ESP?  

DiyMore must have chosen the ESP-14 for a reason. Perhaps I can use the STM to pull the ESP from a deep sleep, low power, state and have the module run on a battery for years. Or was it just because it's cheap?

I did not expect the ESP to block my progress... 

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to add or change so we can have an open discussion on this.

## References
BMP280 Datasheet - https://ae-bst.resource.bosch.com/media/_tech/media/datasheets/BST-BMP280-DS001.pdf  
HDC1080 Datasheet - http://www.ti.com/lit/ds/symlink/hdc1080.pdf  
BH1750 Datasheet - https://www.mouser.com/ds/2/348/bh1750fvi-e-186247.pdf  

This board was featured in Andreas Spiess' mailbox video on YouTube:  
[Mailbag incl. defective LiPo, Co2 sensor, ESP8266 Sensor module, RPi PoE, 120V inverter, OneThinx](https://www.youtube.com/watch?v=3M9biP1ilsE&t=770s) (9 June 2019)

I2C - https://en.wikipedia.org/wiki/I%C2%B2C | https://i2c.info/

ESP-14 - [Hacking on the weirdest ESP module](https://hackaday.com/2017/02/13/hacking-on-the-weirdest-esp-module/) | Hackaday
