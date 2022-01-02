---
layout: post
title: Как запустить MicroPython на ESP8266
date: 2017-04-21 21:38:47.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY и электроника
tags:
- DIY
- ESP8266
- MicroPython
- Python
- Интернет вещей
- Электроника
permalink: "/ru/diy-electronics-ru/how-to-run-micropython-on-esp8266.html"
---
Мне нравится идея Интернета вещей, и последнее время идея эта становится довольно популярной. У нас уже есть куча вещей, которые подключаются к интернету: телевизоры, принтеры, холодильники, автомобили и даже зубные щетки. Более того, у нас еже есть целые ботнеты, которые укомплектованы IoT устройствами, и которые успешно используются для массивных DDoS атак. Иногда&nbsp;я предпочитаю называть все&nbsp;это "Internet of Shit", потому что порой непонятно, зачем некоторые устройства пытаются выйти в интернеты. Кстати, есть интересный twitter, который так и называется "Internet of Shit". Очень рекомендую.

Использовать IoT устройства интересно, модно и современно. Более того, иногда это даже действительно полезно. Но куда более интересней принять более активное участие. Например, можно сделать свое собственное IoT устройство с блэкджеком и шлюхами. И огромное спасибо тем людям, которые создали ESP8266 контроллеры, который теперь позволяет абсолютно всем легко и просто создавать свои IoT устройства. Возможно вы уже знаете, что ESP8266 очень дешевые. А еще их относительно легко программировать, если вы дружите с Гуглом.

Я давно собирался что-нибудь сделать на базе ESP8266. Наконец-то у меня дошли руки, о чем бы хотелось поделиться с надеждой, что это будет кому-нибудь полезно. Перед тем как начать, я нашел довольно много статей про ESP8266 и NodeMCU прошивки, которые позволяют запускать Lua скрипты на ESP8266. Это, конечно, замечательно, но мне очень не хотелось изучать новый для меня язык Lua. Другая проблема это моя лень. Но я немного пишу на Python, и к счастью существует версия Python для микроконтроллеров, которая называется MicroPython. Работает этот MicroPython в том числе и на ESP8266.

Дальше идет рассказ о том, как же запустить этот MicroPython на ESP8266 и помигать светодиодиком.

Here is an English version -&nbsp;[Getting started with ESP8266 and MicroPython](/fun/diy-electronics/getting-started-with-esp8266-and-micropython.html)

![Как запустить MicroPython на ESP8266]({{ site.baseurl }}/assets/images/2017/04/all_parts.jpg)

# Железо: ESP8266 ESP-07&nbsp;и остальные мелкие детали

Если я правильно помню, то существует около двенадцати вариаций ESP8266 (а может быть и больше). Я использовал ту, что называется ESP-07, которая выглядит вот так:

![Микроконтроллер ESP8266 ESP-07]({{ site.baseurl }}/assets/images/2017/04/esp8266_esp_07.jpg)

Наверное проще и дешевле всего это заказать эту штуку на AliExpress или DX. Я бы еще посоветовал прикупить адаптер (как на картинке выше), которые облегчит работу с подключением ESP8266, если вы, как и я, не вытравляете печатные платы в домашних условиях, а пользуетесь всякими prototype boards.

Как-то давно я заказал парочку ESP8266 на AliExpress, и они долго пылились на полке. Вообще я их даже заказывал дважды у разных продавцов и заметил, что GPIO4 и GPIO5 пины могут меняться местами:

![GPIO4 and GPIO5 могут меняться на разных ESP8266 ESP07 контроллерах]({{ site.baseurl }}/assets/images/2017/04/two_esp8266_esp_07.jpg)

Похоже на опечатку. Интернеты говорят, что это опечатка, но я не проверял. Я просто использовал ту, у которой разметка совпадает с разметкой на адаптере.

А вот и схема включения:

![Как подключить ESP8266 ESP-07 микроконтроллер]({{ site.baseurl }}/assets/images/2017/04/esp8266_esp_07_circuit.jpg)

На почетном месте в центре видим ESP8266, а по бокам разную периферию:

1. Кнопки `Flash` и&nbsp;`Reset` buttons позволяют переключать устройство в разные режимы. Вообще их не так много, а именно&nbsp;два, о которых я знаю: режим bootloader и&nbsp;режим flash. В режиме&nbsp;bootloader можно прошивать&nbsp;ESP8266 через UART (пины TXD and RXD на картинке выше). Flash startup это обычные режим, когда устройство запускает прошивку, которую на него залили.
2. UART порт&nbsp;это&nbsp;пины TXD и RXD, через которые ESP8266 можно подключить к компьютеру с помощью USB-Serial конвертера.
3. LD1 это светодиод, который подключен к пину&nbsp;GPIO13 через сопротивление, которое ограничивает ток через порт.
4. Пара других GPIO пинов тоже используются, но об этом ниже.
5. Регулятор напряжения в 3.3 вольт на базе LM1117T.

Как я уже говорил, у нас есть два режима: bootloader и flash startup. Чтобы переключиться в bootloader режим, надо подать&nbsp;низкий уровень на GPIO0 и GPIO15, а также высокий уровень на GPIO2, а потом перезагрузить ESP8266. Иными словами, нужно зажать кнопку `Flash`,&nbsp;в это время нажать&nbsp;`Reset`, а потом отпустить `Flash`.

В начале я нашел несколько более простых схем:

1. Нет резистора R4
2. GPIO15 пин подсоединен к кнопке&nbsp;`Flash`
3. Нет резистора R3, GPIO2 пин не используется&nbsp;(высокий уровень не подается на этот пин).

Но все это не работало. Получалось залить MicroPython прошивку (о чем ниже), но при подключении к прошитой ESP8266 через USB-Serial конвертер Python консоль не появлялась. Попытки перезагрузить устройство приводили к появлению неведомых символов в окошке терминала. Поиски в интернетах показали, что это может означать, что устройство не переключилось в flash startup режим, и до сих пор находится в booloader режиме. Подключение GPIO2 к VCC через R3 резистор, добавление резистора R4 и подача низкого уровня на GPIO15 в результате помогли.

Можно заметить, что REST и GPIO16 пины подсоединены к конденсатору C1 и резистору R1. Поиски в интернетах показали, что это позволяет ESP8266 перезагрузить саму себя после переключения в sleep mode. К сожалению, не нашлось времени изучить это подробнее, но на всякий случай C1 и R1 были добавлены в схему.

Кому интересно - все ссылки на источники в конце поста.

Еще на схеме можно найти регулятор напряжения. Интернеты советуют, что лучше использовать отдельный источник питания для ESP8266, потому что потребляемый ток довольно высок. Поэтому был добавлен [простой регулятор напряжения на базе&nbsp;LM1117T](http://blog.gypsyengineer.com/fun/how-to-make-breadboard-power-supply.html).

Еще понадобится USB-Serial конвертер, который уже упоминался пару раз, чтобы подключить ESP8266 к компьютеру. Конвертеры эти дешевые и могут быть найдены на том же AliExpress.

Вот как все выглядело в результате:

![ESP8266 ESP-07 и USB-Serial конвертер]({{ site.baseurl }}/assets/images/2017/04/esp8266_esp_07_example.jpg)

# Заливка MicroPython на ESP8266

Как уже говорилось ранее, будем заливать MicroPython, который можно скачать здесь:

[http://micropython.org/download/#esp8266](http://micropython.org/download/#esp8266)

Там есть несколько типов прошивок:

- стабильные, которые весят больше 512K
- ежедневные, которые весят больше 512K
- ежедневные, которые весят меньше 512K (это урезанные версии MicroPython)

Выбор прошивки зависит от объема памяти вашего ESP8266 устройства. Когда я покупал свои ESP8266, то не обратил внимания на объем памяти, а следовало бы. В последствии выяснилось, что несколько моих ESP8266&nbsp;имело лишь 512K памяти, поэтому пришлось использовать урезанные версии прошивок. Так же в последствии выяснилось, что [с урезанными версиями MicroPython могут возникнуть некоторые проблемы](/fun/diy-electronics/problems-with-running-micropython-on-esp8266-with-512k.html). В общем, лучше использовать устройства, у которых памяти больше 512K, чтобы можно было залить обычный (неурезанный) MicroPython. Если вдруг вы не знаете, сколько памяти на вашем ESP8266, то просто попробуйте залить полную прошивку. Если памяти на устройстве не достаточно, то вы увидите соответствующую ошибку.

Для заливки прошивки можно использовать esptool, что на мой взгляд является самым популярным методом. Esptool написан на Python, причем похоже, что нужен именно Python 2, а не 3. Esptool можно установить с помощью pip или просто склонировать&nbsp;[https://github.com/espressif/esptool](https://github.com/espressif/esptool)

Прошьем уже&nbsp;ESP8266 наконец-то. Для начала надо переключить в bootloader режим, о котором было сказано выше. Потом было бы неплохо удалить все с нашей&nbsp;ESP8266:

```
sudo esptool --port /dev/ttyUSB0 erase_flash
```

Дальше надо опять переключиться в&nbsp;bootloader режим. В этот &nbsp;режим надо переключаться каждый раз, когда нам нужно удалить или залить прошивку, потому что после перезагрузки ESP8266 будет возвращаться в flash startup режим. Примерно так заливаем прошивку:

```
sudo esptool --port /dev/ttyUSB0 --baud 460800 \
    write_flash --flash_size=detect 0 \
    esp8266-512k-20170212-v1.8.7-213-gaac2db9.bin
```

Потом можно проверить, что прошивка прошла успешно (нужно опять переключиться в bootloader режим):

```
sudo esptool --port /dev/ttyUSB0 --baud 460800 \
    verify_flash --flash_size=detect 0 \
    esp8266-512k-20170108-v1.8.7.bin
```

Вот как это все выглядит:

```
$ sudo esptool --port /dev/ttyUSB0 erase_flash
esptool.py v2.0-beta1
Connecting.....
Detecting chip type... ESP8266
Uploading stub...
Running stub...
Stub running...
Erasing flash (this may take a while)...
Chip erase completed successfully in 1.3s
Hard resetting...
$ sudo esptool --port /dev/ttyUSB0 --baud 460800 write_flash --flash_size=detect 0 esp8266-512k-20170108-v1.8.7.bin
esptool.py v2.0-beta1
Connecting....
Detecting chip type... ESP8266
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Attaching SPI flash...
Configuring flash size...
Auto-detected Flash size: 512KB
Flash params set to 0x0000
Compressed 500608 bytes to 330679...
Wrote 500608 bytes (330679 compressed) at 0x00000000 in 7.5 seconds (effective 536.9 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting...
$ sudo esptool --port /dev/ttyUSB0 --baud 460800 verify_flash --flash_size=detect 0 esp8266-512k-20170108-v1.8.7.bin
esptool.py v2.0-beta1
Connecting....
Detecting chip type... ESP8266
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 460800
Changed.
Attaching SPI flash...
Configuring flash size...
Auto-detected Flash size: 512KB
Flash params set to 0x0000
Verifying 0x7a380 (500608) bytes @ 0x00000000 in flash against esp8266-512k-20170108-v1.8.7.bin...
-- verify OK (digest matched)
Hard resetting...
```

Если все прошло гладко, то на нашей ESP8266 должен оказаться желанный MicroPython. Теперь можно подключиться к нашей ESP8266 через терминал. На Linux можно использовать, например, minicon:

```
sudo minicom --device /dev/ttyUSB0 -b 115200
```

Здесь нужно использовать 115200 baud rate.&nbsp;Если вы используете minicon, то вам наверное будет интересно, как из него выйти. Это не совсем очевидно. Надо нажать&nbsp;Ctrl-A, а потом Q.

В окошке терминала должна появится Python консоль:

```
MicroPython v1.8.7 on 2017-01-08; ESP module with ESP8266
Type "help()" for more information.
>>>
```

# Мигание светодиодом на ESP8266 с&nbsp;MicroPython

В завершение помигаем светодиодом:

```
MicroPython v1.8.7 on 2017-01-08; ESP module with ESP8266
Type "help()" for more information.
>>> from machine import Pin
>>> pin = Pin(13, Pin.OUT)
>>> pin.high()
>>> pin.low()
>>>
```

Удачи!

- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170212_121132.jpg)](/wp-content/uploads/2017/04/IMG_20170212_121132.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170212_121041.jpg)](/wp-content/uploads/2017/04/IMG_20170212_121041.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170212_121023.jpg)](/wp-content/uploads/2017/04/IMG_20170212_121023.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_212020.jpg)](/wp-content/uploads/2017/04/IMG_20170211_212020.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_210602.jpg)](/wp-content/uploads/2017/04/IMG_20170211_210602.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_191452.jpg)](/wp-content/uploads/2017/04/IMG_20170211_191452.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_191244.jpg)](/wp-content/uploads/2017/04/IMG_20170211_191244.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_191218.jpg)](/wp-content/uploads/2017/04/IMG_20170211_191218.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_190316.jpg)](/wp-content/uploads/2017/04/IMG_20170211_190316.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_190204.jpg)](/wp-content/uploads/2017/04/IMG_20170211_190204.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_185328.jpg)](/wp-content/uploads/2017/04/IMG_20170211_185328.jpg)
- 
[![]({{ site.baseurl }}/assets/images/2017/04/IMG_20170211_184318.jpg)](/wp-content/uploads/2017/04/IMG_20170211_184318.jpg)

Ссылки:

- [http://www.esp8266.com/wiki/doku.php?id=esp8266-module-family#esp-07](http://www.esp8266.com/wiki/doku.php?id=esp8266-module-family#esp-07)
- [https://docs.micropython.org/en/latest/esp8266/esp8266/tutorial/intro.html](https://docs.micropython.org/en/latest/esp8266/esp8266/tutorial/intro.html)
- [https://learn.adafruit.com/micropython-basics-how-to-load-micropython-on-a-board/esp8266](https://learn.adafruit.com/micropython-basics-how-to-load-micropython-on-a-board/esp8266)
- [http://www.agcross.com/2015/09/the-esp8266-wifi-chip-part-3-flashing-custom-firmware/](http://www.agcross.com/2015/09/the-esp8266-wifi-chip-part-3-flashing-custom-firmware/)
- [https://forum.micropython.org/viewtopic.php?t=2277](https://forum.micropython.org/viewtopic.php?t=2277)
- [https://zoetrope.io/tech-blog/esp8266-bootloader-modes-and-gpio-state-startup](https://zoetrope.io/tech-blog/esp8266-bootloader-modes-and-gpio-state-startup)
- [http://docs.micropython.org/en/latest/wipy/library/machine.Pin.html](http://docs.micropython.org/en/latest/wipy/library/machine.Pin.html)

