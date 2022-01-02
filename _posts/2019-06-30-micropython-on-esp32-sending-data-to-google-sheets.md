---
layout: post
title: 'MicroPython on ESP32: sending data to Google Sheets'
date: 2019-06-30 21:26:15.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY and electronics
tags:
- DHT22
- ESP32
- ESP8266
- Google
- Internet of Shit
- IoT
- JWT
- MicroPython
- OAuth2
- Open Source
permalink: "/en/diy-electronics/micropython-on-esp32-sending-data-to-google-sheets.html"
---


On a wonderful weekend in summer time, instead of going out to a beach or somewhere else, I was staying at home and wondering if it's possible to send data from an ESP board to a Google sheet using my favorite MicroPython. Let's say it can send temperature and humidity measured by a DHT22 sensor. That's how the project started.





(this post contains a brief description of the project, more technical details can be found in the [next post](/en/diy-electronics/weather-station-based-on-esp32-and-micropython.html))





![ESP32 development board and DHT22 sensor: sending data to a Google sheet.]({{ site.baseurl }}/assets/images/2019/06/esp32_dev_board_and_dht22.jpg)



  
  




A quick search in Google showed that isn't not a new idea, and there are already people who implemented sending data to a Google sheet from ESP boards. I read several articles about this topic, and I noticed two things about all projects I found: either the projects require a Google sheet to be publicly available on the Internet, or the projects use a middleman such as PushingBox or IFTTT to put data to a sheet.





But Google Sheets provide awesome HTTP API, why can't we just send a POST request to insert data to a sheet? The reason is that Google Sheets API care about security and require authentication. In particular, Google Sheet API support two authentication methods: API keys and OAuth2 tokens. Using an API key sounds like an easy way: we can just create an API key in Google IAM console, hardcode it in our ESP board, successfully forget about rotating the key and security at all, and use the key forever. Unfortunately (or fortunately) it's not going to work. Google Sheet API allow API keys only for read operations. If you want to write to a sheet, you'll have to use OAuth2.





Here the problems come. First, you need to create a service account in Google IAM console, then create a private RSA key for the service account. Okay, that's not that hard. Then, your ESP board has to build a JWT request, put a current timestamp in to it, and sign the request with the key using RSA-SHA256 signature algorithm. Next, the board has to send the signed JWT request to Google OAuth2 service which finally returns an OAuth2 token. Finally, the board can use the token to make a call to Google Sheets API.





Let me quickly summarize what a ESP board needs to be able to insert a row to a Google sheet: an HTTP client, a JSON parser, a real-time clock with correct time or an NTP client, and an implementation of RSA-SHA256 signature algorithm. Luckily MicroPython provides most of the stuff except RSA-SHA256 signatures. To be precise, MicroPython can calculate a SHA256 hash but it can't sign data with RSA algorithm. Furthermore, RSA signing is an expensive operation which may take quite a lot of time and memory but ESP boards are not that powerful as PCs and don't have too much memory.





I guess the complexity which Google authentication brings together with absence of RSA implementation for MicroPython made people use a publicly available sheets or a middleman services which can implement the authentication process. Of course, as a security engineer, I could not accept a solution with a publicly available sheet which contains that sensitive data such as temperature and humidity in my room. And again since I am a security engineer, I don't like middlemans. In the end, the only option was to implement somehow the authentication process described above on a ESP board.





As I mentioned earlier, MicroPython already provides most of the required stuff:





- [ujson](https://docs.micropython.org/en/latest/library/ujson.html) allows parsing JSON
- [ntptime.py](https://github.com/micropython/micropython/blob/master/ports/esp8266/modules/ntptime.py) provides an NTP client
- [uhashlib](https://docs.micropython.org/en/latest/library/uhashlib.html) can compute SHA256 hashes
- [http.client](https://github.com/micropython/micropython-lib/tree/master/http.client) provides an HTTP client





The only missing part is RSA signatures. One of the main rules in cryptography says: don't implement any cryptographic algorithms by yourself, and use existing implementations instead. I knew that rule and found [python-rsa](https://github.com/sybrenstuvel/python-rsa/) package which implements RSA algorithm in pure Python. Besides signing with RSA, the library supports other operations such as verifying signatures, encryption and decryption, loading and storing keys and so on. That makes the library too heavy for a tiny ESP board, and I decided to keep only the implementation of RSA signing and remove the rest of the code. I even removed the code which loads RSA keys in PKCS1 format because it required porting [pyasn1](https://github.com/etingof/pyasn1) package which is not a small one. I also had to implement modular exponentiation based on [the right-to-left binary method](https://en.wikipedia.org/wiki/Modular_exponentiation#Right-to-left_binary_method) and several other operations because it turned out that they are not provided by MicroPython. It resulted to a new [micropython-rsa-signing](https://github.com/artem-smotrakov/micropython-rsa-signing) library.





I used [one of my previous projects with ESP8266 and MicroPython](/diy-electronics/getting-started-with-esp8266-and-micropython.html) as a starting point. After implementing the authentication process and testing it on my laptop, I finally picked up a ESP8266 board and ran the code there. Unfortunately I found out the the code runs very slow. But the main problem was that the ESP8266 board didn't have enough memory to finish RSA signing. I tried some optimizations and even tried to embed the application code in to MicroPython firmware but nothing helped.





The last hope was to run the code on ESP32. I had not played with ESP32 before, and ordered my first development board. And luckily it worked out! The application no longer complained about low memory, and in fact it ran much faster.





[The code is available on GitHub](https://github.com/artem-smotrakov/esp32-weather-google-sheets). The README briefly describes how the project can be built. [I also created a project on Hackaday.io](https://hackaday.io/project/166197-esp32-weather-station-and-google-sheets). I hope I can find some time to provide more details in a follow-up post.



