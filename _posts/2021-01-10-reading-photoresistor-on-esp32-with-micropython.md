---
layout: post
title: Reading a photoresistor on ESP32 with MicroPython
date: 2021-01-10 13:23:36.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY and electronics
tags:
- DIY
- Electronics
- ESP32
- LDR
- MicroPython
meta:
  _edit_last: '1'
  _yoast_wpseo_content_score: '60'
  _yoast_wpseo_primary_category: '155'
  _yoast_wpseo_focuskw: photoresistor
  _yoast_wpseo_metadesc: A photoresistor is a resistor that changes its resistance
    depending on light intensity. Let's see how we use a photoresistor with ESP32
    and MicroPython.
  _yoast_wpseo_linkdex: '73'
  rp4wp_auto_linked: '1'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617980191'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/en/diy-electronics/reading-photoresistor-on-esp32-with-micropython.html"
---
<!-- wp:paragraph -->

A photoresistor or a light-dependent resistor (LDR) is a resistor that changes its value (resistance) depending on light intensity. More precisely, when light falls upon it, the resistance decreases. It is normally used as a light or dark detector. For example, it may be used in a circuit that turns lights in a room on when it gets dark. Let's see how we use a photoresistor with ESP32 and MicroPython.

<!-- /wp:paragraph -->

<!-- wp:image {"id":3997,"sizeSlug":"large","linkDestination":"none"} -->

![ESP32 and photoresistor]({{ site.baseurl }}/assets/images/2021/01/e2-1024x768.jpg)  

_by Alena Penkova_

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## Circuit for photoresistor and ESP32

<!-- /wp:heading -->

<!-- wp:image {"id":3985,"sizeSlug":"large","linkDestination":"none"} -->

![A circuit for connecting a photoresistor (LDR) with ESP32 board]({{ site.baseurl }}/assets/images/2021/01/Schematic_ESP32-DevKit-v1-and-LDR_2021-01-03-2.png)

<!-- /wp:image -->

<!-- wp:paragraph -->

Here is a list of components we need:

<!-- /wp:paragraph -->

<!-- wp:list -->

- ESP32 development board
- A light dependent resistor (LDR)
- 10 KOhm resistor
- Breadboard
- Wires

<!-- /wp:list -->

<!-- wp:paragraph -->

The circuit is pretty simple. A photoresistor R1 is connected to the pin D34 with a pull-down resistor R2. We're going to measure the voltage on the pin is going to change when the light intensity changes. To do that, the pin should support analog reading. In other words, it needs to be an ADC pin (Analog to Digital Converter). The pin D34 is exactly this kind of pin, according to the datasheets.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The board is going to be powered via USB.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Reading a photoresistor with MicroPython

<!-- /wp:heading -->

<!-- wp:paragraph -->

MicroPyhon offers the class `ADC` that reads analog values from a pin. The class just take a pin number. Then, values can be read by calling the method `ADC.read()`. The method returns a number from 0 to 4095. In the default configuration, an instance of `ADC` expects voltages from 0V to 1V on the pin. If the voltage is above 1V, then the method will return 4095. However, it's possible to increase the supported voltage range by calling the method `ADC.atten()`.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

The code below reads a value from a photoresistor every 3 seconds:

<!-- /wp:paragraph -->

<!-- wp:html -->  
<script src="https://gist.github.com/artem-smotrakov/63bfcf5ebf7aa2cfc4effefe456c6cd4.js"></script>  
<!-- /wp:html -->

<!-- wp:paragraph -->

Let's take a look at the class `LDR`. It takes a pin number that is connected to the photoresistor, and a desired range `[min_value, max_value]` for returned values. The constructor checks the parameters and initializes a new `ADC`. The method `read()` returns a raw value from the `ADC` (from 0 to 4095). The method `value()` calls the method `read()` and then maps the returned raw value to the desired range `[min_value, max_value]`.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Uploading MicroPython and the code to ESP32

<!-- /wp:heading -->

<!-- wp:paragraph -->

The MicroPython firmware and the code are available on [GitHub](https://github.com/artem-smotrakov/esp32-ldr):

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
git clone https://github.com/artem-smotrakov/esp32-ldr
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

To flash the firmware and upload the code, we'll need several tools:

<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->

1. `esptool`&nbsp;for uploading MicroPython to ESP32
2. `mpfshell`&nbsp;for uploading files to ESP32
3. `minicom`&nbsp;for connecting to ESP32 for debugging purposes

<!-- /wp:list -->

<!-- wp:paragraph -->

The steps below work on Linux (I tested in on Ubuntu 18 and 20). On Windows or Mac, it may or may not work.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Note that some ESP32 development boards require pressing the&nbsp;`EN`&nbsp;button while holding the&nbsp;`Boot`&nbsp;button for switching the board to the mode for erasing or uploading firmware. Do it before erasing, writing or verifying the firmware.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

First, erase the old firmware on the ESP32:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
esptool.py --port /dev/ttyUSB0 erase_flash
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Next, upload MicroPython v1.13:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
esptool.py \
    --chip esp32 \
    --port /dev/ttyUSB0 \
    --baud 460800 \
    write_flash -z 0x1000 esp32-idf3-20200902-v1.13.bin
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Then, you may want to verify it:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
esptool.py \
    --chip esp32 \
    --port /dev/ttyUSB0 \
    --baud 460800 \
    verify_flash 0x1000 esp32-idf3-20200902-v1.13.bin
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Then, upload the code to the board:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
mpfshell \
    -n -c \
    "open ttyUSB0; \
    lcd src; mput .*\.py;"
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

By the way, the directory&nbsp;`scripts`&nbsp;contains scripts that also do the job.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Finally, you can connect to the board with&nbsp;`minicom`&nbsp;and see the measurements:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
minicom --device /dev/ttyUSB0 -b 115200
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

The output is going to look like something like this:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
value = 18.92552 
value = 18.87668 
value = 5.006105 
value = 18.63248
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Try covering the photoresistor with a hand or putting more light on it. See how the values change.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## References

<!-- /wp:heading -->

<!-- wp:list -->

- [ESP32 datasheet](https://www.espressif.com/sites/default/files/documentation/esp32_datasheet_en.pdf)
- [ESP32 Pinout Reference](https://randomnerdtutorials.com/esp32-pinout-reference-gpios/)
- [ADC on MicroPython](https://docs.micropython.org/en/latest/esp32/quickref.html#adc-analog-to-digital-conversion)

<!-- /wp:list -->

