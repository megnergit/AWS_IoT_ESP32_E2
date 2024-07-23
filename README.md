# Connect ESP32 to AWS IOT

## Goal
We will use

* LDR [(light dependent sensor)](https://www.reichelt.com/de/en/arduino-photo-resistor-module-ldr--ard-sen-ldr-p282513.html?r=1)
* [ESP32](https://www.az-delivery.de/en/products/esp32-developmentboard)
* AWS IoT

to read the light brightness on AWS console.

I used AWS [official tutorial](https://aws.amazon.com/jp/blogs/compute/building-an-aws-iot-core-device-using-aws-serverless-and-an-esp32/) as a reference, but since 2020 when this tutorial was written, many things have been changed in AWS IoT. I will mostly concentrate on the updates below. 

There are at least [3
ways](https://qiita.com/moritalous/items/57c77819f23661cfe8d8) to
connect ESP32 to AWS IoT.

1. MQTT + AruduinoJson
2. ```arduino-aws-greengrass-iot```
3. FreeRTOS

'2.' and '3.' installs an operation system to ESP32 (Greengrass or FreeRTOS).
That should be the standard way to develop an IoT, but I would like to try
something else :) Therefore we will go with '1.'

/* --- */
## Screenshot

/* --- */
## Overview

1. Test ESP32
   - without AWS
   + Cable !!!!
   + WiFi

2. Set up [PlatformIO](https://platformio.org/) as a development environment
   + Create project
   + Install extentions

3. Write sketch (=code) for ESP32 and AWS IoT
   + main.cpp
   + secrets.h (to hold AWS certificates to connect to AWS IoT)

4. Create IoT device on AWS

/* --- */

## Test ESP32

### ESP32 and SE012

ESP32 is a microcomputer like Arduinos, but with WiFi chip on it.  A
web server runs on the microcomputer. When we connect sensors to
ESP32, we can read the sensor on air (no wired conenction necessary).

![The board is ESP32. The small one is SE012.](./images/IMG_0382.jpg)

NodeMCU ESP32S has 5V output. Any 3-pin sensors (5V, GND, signal) can be
attached to it.

Here we will use photoresistor (=LDR) SE012 by IDUINO. The signal pin-out will be 
* low voltage when bright (~0)
* high voltage when dark (~4095, for instance when you wrap the sensor by hand)

### Wiring

| SE012 (LDR)  |  ESP32 |
|--------------|--------|
| 'S' pin      | SVP    |
| 5V (middle)  | 5V     | 
| GND ('-' pin)| GND    |


![Wiring](./images/image_123923953.JPG)

We do not even need a breadboard.


### CABEL!!!!

ESP32 use USB mini to communicate to our laptop.

USB mini cable has two types

* one that transmit data (=signal =information) and power
* one that transmit only power

Make sure to use the cable that transmit data. It is hard to tell 
which one is which from the appearance.
Use the one that _worked_ with your other digital devices.


### Coding

The laptop-ESP32 solution without AWS will be shown in another repository soon.
One can read sensor output

* ```curl``` or
* web browser

------------------------
## PlatformIO

PlatformIO is an extention for VS code.

... I was happily using Arduino IDE, but could not upload the sketch
on ESP32.  It was because of the wrong USB mini cable (was charging
cable, not data cable), while doing trials and erros, I switched my
developemnt environment to VS code and PlatformIO. It works fine.

PlatformIO + VS code is the replacement of Arduino IDE, and it looks like
everything I can do with Arduino IDE, I can do with PlatformIO.
One big plus is

* (Almost) no need to install libraries one by one. PlatformIO will take care of it.


------------------------
## Write sketch

Create a new project following the official instruction of PlatformIO.
The development directory would look as follows.

```
.
├── include
│   └── README
├── lib
│   └── README
├── platformio.ini
├── src
│   ├── main.cpp
│   └── secrets.h
└── test
    └── README
```

We have to write two files

1. ```main.cpp```
2. ```secrets.h```

1. is the sketch that we will upload to ESP32.
2. is the file that contains certificates needed to connect to AWS IoT. ```secrets.h```
   is called from ```main.cpp```.

I created my ```main.cpp``` chaning the tutorial code here and there.

```secrets.h``` was a bit more complicated, and the procedure will be
described below a bit more in detail.

You will need two libraries:

1. ArduinoJson by Benoit Blanchon
2. MQTT by Joel Gaehwiler

![Library you need](./images/platformio-01.png)


Please look at the [official
tutorial](https://aws.amazon.com/jp/blogs/compute/building-an-aws-iot-core-device-using-aws-serverless-and-an-esp32/)
at the section "Installing and configuring the Arduino IDE".


## Certificates (```secrets.h```)






------------------------
## AWS IoT





---
# END







