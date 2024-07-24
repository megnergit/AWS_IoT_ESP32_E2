# Connect ESP32 to AWS IoT

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
something else. Therefore we will go with '1.'

<!-- ================================================== -->     
## Screenshot

![MQTTtest](./images/MQTTtest.gif)

<!-- ================================================== -->     
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
   + compile main.cpp with secrets.h and write the code to ESP32

4. Create IoT device on AWS


<!-- ================================================== -->     
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


### CABLE!!!!

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

<!-- ================================================== -->     
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


<!-- ================================================== -->     
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

'1.' is the sketch that we will upload to ESP32.
'2.' is the file that contains certificates needed to connect to AWS IoT. ```secrets.h```
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


<!-- ================================================== -->     
## Certificates

This section is about ```secrets.h```.

In order for our device (ESP32) to talk to AWS IoT we need the following keys and certificate.

1. IoT device private key
2. IoT device public key
3. IoT device certificate
4. AWS root CA


### IoT device

A big update since the official tutorial is that now we can download
1-3 in one go. At the time 2020, we had to download one by one (we
still have to get AWS root CA separately, though).


On AWS IoT console (we will go through it again in the next section)
click

and you will find ```connect_device_packages.zip``` on your Downloads folder.

When you unzip ```connect_device_packages.zip``` you will get

```
> unzip connect_device_package.zip
Archive:  connect_device_package.zip
 extracting: ESP32-LDR.cert.pem
 extracting: ESP32-LDR.public.key
 extracting: ESP32-LDR.private.key
 extracting: ESP32-LDR-Policy
 extracting: start.sh
```

(Here ```ESP32-LDR``` is the arbitrary name that you put when you
created IoT device on AWS console.)

### AWS Root CA
I took it from [here](https://www.amazontrust.com/repository/).
Pick "CA 1" and "PEM". 


### Put them in ```secrets.h```

We will put 

- private key
- device certificate
- AWS root CA

in ``secrets.h```.

The template file of ```secrets.h``` looks like this.

```
#include <pgmspace.h>

#define SECRET
#define THINGNAME "ESP32-LDR"          // <==== This is your device name

const char WIFI_SSID[] = "";           // <==== Your local WiFi network name
const char WIFI_PASSWORD[] = "";       // <==== Your WiFi password
const char AWS_IOT_ENDPOINT[] = "";    // <==== IoT device name. Something like "XXXXzXXXX.iot.eu-central-1.amazonaws.com"

// Amazon Root CA 1
static const char AWS_CERT_CA[] PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
)EOF";

// Device Certificate
static const char AWS_CERT_CRT[] PROGMEM = R"KEY(
-----BEGIN CERTIFICATE-----
-----END CERTIFICATE-----
)KEY";

// Device Private Key
static const char AWS_CERT_PRIVATE[] PROGMEM = R"KEY(
-----BEGIN RSA PRIVATE KEY-----
-----END RSA PRIVATE KEY-----
)KEY";
```


Set each of the content of the files you extracted (for device) or
downloaded from AWS (for CA) in ```secrets.h```

**Note**

* Just do

```
> cat ESP32-LDR.private.key
[meg@elias ~/aws/iot/secrets]$ \cat ESP32-LDR.private.key
-----BEGIN RSA PRIVATE KEY-----
....
-----END RSA PRIVATE KEY-----
```

and cut and paste the whole output to the suited part of ```secrets.h``` __including__

* "-----BEGIN RSA PRIVATE KEY-----" and
* "-----END RSA PRIVATE KEY-----".

You do not need to touch up the text with ```echo -n``` or anything. 


<!-- ================================================== -->     
## AWS IoT

### Create IoT device

Okay, now let us start working with AWS IoT. Again (a bit outdated) [tutorial](https://aws.amazon.com/jp/blogs/compute/building-an-aws-iot-core-device-using-aws-serverless-and-an-esp32/) is our guide. 


1. Go to https://aws.amazon.com -> AWS IoT

![Library you need](./images/platformio-01.png)

2. 'Connect device'

![Connect evice](./images/create_device.png)

3. 'Copy' device name ('XXXXXXXX.iot.eu-central-1.amazonaws.com' or similar)

![Copy device name](./images/copy_device_name.png)

4. 'Next'

5. Put the name of your device (here I set 'ESP32-LDR') and 'Next'.

![Name of your device](./images/name_of_your_device.png)

6. Pick
   * Linux / macOS
   * Python
   and 'Next'

![Linux platform](./images/linux_platform.png)


7. __Download connection kit__
   Press 'Download conenction kit' button. You got ```connect_device_packages.zip``` here.
   Use it to fill ```secrets.h``` in the previous section
   => 'Next'

![Download connection kit](./images/download_connection_kit.png)

8. 'Continue' (we do not use ```start.sh``` as we do not have linux on our ESP32)

9. 'Device is connected' (actually not yet) => 'View thing'
    You see a page for your device.

![Device is connected](./images/device_is_connected.png)


10. All right. From the left column of that device page, go to 'Security' then => 'Policies'.

![Security to Policies](./images/security_to_policies.png)


11. 'Create policy'

![Create policy](./images/create_policy.png)

12. Let us create a policy names 'ESP32Policy' (or of your choice). 

![Policy name](./images/policy_name.png)

13. Click on 'JSON'.

![We are creating JSON policy](./images/create_json_policy.png)

14. [__Paste the policy from the tutorial__](https://aws.amazon.com/jp/blogs/compute/building-an-aws-iot-core-device-using-aws-serverless-and-an-esp32/). The one from 'Creating an AWS IoT device'

15. Change _3_ parts. 'REGION' (e.g. `eu-central-1`), 'ACCOUNT_ID'
(usual 12-digit numbers) _and_ 'MyNewESP32' (the last one to the name
of your device. In my case 'ESP32-LDR'). Like this. => 'Create'

16. Go back to your device from the left column 'All devices' -> 'Things'.


![Go back to your device](./images/back_to_device.png)


17. Click on 'Certificates' in the tabs on the bottom. Click on the Certificate ID (long random text).

![Click on certificates](./images/click_on_certificate.png)

![Click on poliicy](./images/click_on_policy.png)

18. Click on 'Attach policy'. Attach the policy we creatd on 14.

![Attach poliicy](./images/attach_policy.png)


### Test MQTT message

19. Form the left column of IoT console, click on 'MQTT test client'.

![MQTT test client](./images/mqtt_client.png)


20. 'Subscribe to a topic'. Type in 'esp32/pub' (like we put in 'ESP32Policy'),
and click on 'Subscribe'. You should see the reading of the LDR like the screenshot at the begining of this README.


<!-- ================================================== -->     
## Troubleshooting

* ```[  2194][E][WiFiClientSecure.cpp:144] connect(): start_ssl_client: -29312
.[  3886][E][ssl_client.cpp:37] _handle_error(): [start_ssl_client():273]: (-29312) SSL - The connection indicated an EOF```
=> problem in ```secrets.h```. Redo the editing again.


* ```<Arduino.h> is missing``` => do not use '->' button of VS code, but select 'Build' from the drop-down menu.

------
## END






