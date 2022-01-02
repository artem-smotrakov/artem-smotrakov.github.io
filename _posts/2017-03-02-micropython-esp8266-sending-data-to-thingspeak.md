---
layout: post
title: 'MicroPython on ESP8266: sending data to ThingSpeak'
date: 2017-03-02 23:05:09.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY and electronics
tags:
- DHT22
- ESP8266
- Internet of Shit
- IoT
- MicroPython
- Python
permalink: "/en/diy-electronics/micropython-esp8266-sending-data-to-thingspeak.html"
---
When you play with new microcontroller, first thing you usually do is driving an LED. That's a classic "Hello World!" project for microcontrollers. That's what I did when I was [playing first time with ESP8266 and MicroPython](/fun/diy-electronics/getting-started-with-esp8266-and-micropython.html). Let's move on, and implement another classic project - measuring temperature and humidity with DHT22 sensor. But we don't want to be&nbsp;quiet, so we are going to&nbsp;share this so important data on the Internet. ThingSpeak will help us with it. Let's add a new warrior to the army of Internet of Shit!



## Hardware and circuit: ESP8266 (ESP-07) and DHT22

This project is based on&nbsp;my previous post -&nbsp;[Getting started with ESP8266 and MicroPython](/fun/diy-electronics/getting-started-with-esp8266-and-micropython.html). So, everything what I described there apply here.

Also note that it may be better to use ESP8266 which has more than 512K of flash. It looks like that limited versions of MicroPython may not work well (see&nbsp;[Problems with running MicroPython on ESP8266 with 512K](/fun/diy-electronics/problems-with-running-micropython-on-esp8266-with-512k.html)).

For this project we're going to use ESP-07 version of ESP8266. You can also use ESP-12 without any changes for the circuit below. But if you use other modifications of ESP8266, then you may need to modify a bit the circuit below.

Here is the circuit I used:

![Cirquit for ESP8266 ESP-07 and DHT22]({{ site.baseurl }}/assets/images/2017/03/circuit.jpg)

The circuit is similar to that one from my previous post. Here is what I added:

- DHT22 sensor and a resistor R6.
- A switch on GPIO5 which is used to turn on `configuration mode` which I'll explain below.
- A switch on GPIO4 which is not used. I added it just in case because I had a unit with two switches.

We use DHT22 sensor for measuring temperature and humidity. You can also use DHT11. That's probably the most popular sensor for DIY projects, so MicroPython team kindly added a support for this sensor out-of-the-box. In case you have never seen this sensor, DHT22 looks like this:

![DHT22 sensor]({{ site.baseurl }}/assets/images/2017/03/IMG_20170225_141850.jpg)

Here is how the device looks like:

![ESP8266 and DHT22 sensor]({{ site.baseurl }}/assets/images/2017/03/IMG_20170225_143653.jpg)

## Starting a Wi-Fi access point on ESP8266

We're done with hardware, let's talk about software. I put sources to&nbsp;a repo on [https://github.com/artem-smotrakov/yellow-duck](https://github.com/artem-smotrakov/yellow-duck)

```
$ git clone https://github.com/artem-smotrakov/yellow-duck
```

Since we're going to send temperature and humidity to ThingSpeak, we need to connect our ESP8266 board to Wi-Fi. To do that, ESP8266 needs SSID and password. The most obvious way is to hardcode SSID and password. Here is how you can connect to Wi-Fi&nbsp;with MicroPython:

<script src="https://gist.github.com/artem-smotrakov/b8e31cc3c34214957b518518f1fecf20.js"></script>

But it's not convenient to have hardcoded Wi-Fi settings. If SSID/password change, we'll need to change sources and upload it to the board again. Instead of hardcoding SSID/password, we can do the following:

- Add a `config mode` switch on GPIO5 (see on the circuit above) which turns on configuration mode.
- When the board starts, it check if `config mode` switch is on (see `is_config_mode` function).
- In case configuration mode is on, the board sets up a Wi-Fi access point, and starts a local web server (see `start_access_point` and `start_local_server` functions).
- When a user connects to this Wi-Fi access point, the web server offers to set SSID and password for Wi-Fi network which should be used then.
- When a user provides SSID/password, the web server stores it to a configuration file on ESP8266, and reboots the board (see `write_wifi_config`and `reboot` functions).
- At this point, a user should also turn off `config mode`.
- After reboot, the board connects to specified Wi-Fi network (see `connect_to_wifi` function).
- In case of successful connection, it turns on LD1 (see `turn_on_wifi_led` function).

So, if we want to use the device in another Wi-Fi network, or we just changed Wi-Fi password, we don't need to flash the device again.

After the device successfully connected to Wi-Fi, it starts measuring temperature and humidity with specified interval. As I mentioned earlier, MicroPython supports both DHT11 and DHT22 sensors out-of-the-box. We just use `dht.DHT22` function, see `mesure_temperature_and_humidity`:

<script src="https://gist.github.com/artem-smotrakov/b864a0ebadfff0e50ec227bc6fea5c47.js"></script>

That's pretty simple. Then, we need to send data to the Internet. We're going to use ThingSpeak which is a nice and free platform for gathering, analyzing, and visualizing data from IoT devices. ThingSpeak provides a RestFUL API for IoT devices. First, you need to create an account on ThingSpeak. Next, you'll need to create a channel. After that, you'll get an API key for posting data to your channel. This API key then should be used for sending data to ThingSpeak. Don't forget to enable two fields in your channel: one for temperature, and another one for humidity.

![Channel settings on ThingSpeak]({{ site.baseurl }}/assets/images/2017/03/thingspeak.png)

Here is how we send data to ThingSpeak:

<script src="https://gist.github.com/artem-smotrakov/80cac14c87ad443da5abe416c7026d60.js"></script>

We have the same problem here like we had with SSID/password - we need to provide API key, but we don't want to hardcode it. The solution is the same - the device offers a user to specify API key in `config mode`.

As you might notice, we use TLS here - see `ssl.wrap_socket` call above which wraps a usual socket with an TLS socket. To be honest, I was surprised a bit when learned that MicroPython for ESP8266 supports TLS. But there is one thing about it. Even if TLS is used here, MicroPython for ESP8266 doesn't support validation of server certificate (at least, v1.8.7 doesn't support that). So, we're relatively safe if an adversary can just eavesdrop the connection because all data will be encrypted. But if an adversary can modify the traffic, then we're in a trouble because server certificate may be replaced with a malicious one, so that an adversary can decrypt and modify the data.

You might notice a number of `print` calls in the code. It's just for debugging. You can connect to ESP8266 via USB-Serial adapter, and see those debug messages.

## Visualizations on ThingSpeak

If everything went smoothly, the device will start sending data to ThingSpeak. Then, ThingSpeak allows you to visualize the data, and analyze it with MatLab. The simplest way is just to draw a chart like these ones:

<iframe style="border: 1px solid #cccccc;" src="https://thingspeak.com/channels/230189/charts/1?bgcolor=%23ffffff&amp;color=%23d62020&amp;dynamic=true&amp;results=60&amp;type=line&amp;update=15" width="450" height="260"></iframe>

<iframe style="border: 1px solid #cccccc;" src="https://thingspeak.com/channels/230189/charts/2?bgcolor=%23ffffff&amp;color=%23d62020&amp;dynamic=true&amp;results=60&amp;type=line&amp;update=15" width="450" height="260"></iframe>

<iframe style="border: 1px solid #cccccc;" src="https://thingspeak.com/apps/matlab_visualizations/131354" width="450" height="260"></iframe>

These charts show temperature and humidity in my room. Here is a public channel for that:

[https://thingspeak.com/channels/230189](https://thingspeak.com/channels/230189)

## Links

- [Quick reference for the ESP8266](https://docs.micropython.org/en/latest/esp8266/esp8266/quickref.html)
- [MicroPython - network configuration](https://docs.micropython.org/en/latest/esp8266/library/network.html)
- [MicroPython - Temperature and Humidity](https://docs.micropython.org/en/latest/esp8266/esp8266/tutorial/dht.html)
- [MicroPython - socket module](https://docs.micropython.org/en/latest/esp8266/library/usocket.html)
- [Temperature and humidity module AM2302 Product Manual](http://akizukidenshi.com/download/ds/aosong/AM2302.pdf)

Have fun!

