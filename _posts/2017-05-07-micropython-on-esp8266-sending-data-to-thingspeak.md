---
layout: post
title: 'MicroPython на ESP8266: отправка данных на ThingSpeak'
date: 2017-05-07 16:35:16.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY и электроника
tags:
- ESP8266
- MicroPython
- Python
- Интернет вещей
meta:
  mask_links: default
  _edit_last: '1'
  _wpgo_column_layout_save: ''
  _wpgo_column_layout: default
  _syntaxhighlighter_encoded: '1'
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _aioseop_description: 'MicroPython на ESP8266: Wi-Fi точка доступа с web-сервером,
    подключение к Wi-Fi, измерение температуры и влажности с DHT22, а также отправка
    данных в ThingSpeak.'
  _aioseop_title: 'MicroPython на ESP8366: отправка данных на ThingSpeak'
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '153'
  _thumbnail_id: '2520'
  _yoast_wpseo_content_score: '30'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:1;s:7:"replies";i:1;s:7:"authors";i:2;s:14:"recent_authors";a:2:{i:0;O:8:"stdClass":3:{s:20:"comment_author_email";s:25:"artem.smotrakov@gmail.com";s:14:"comment_author";s:5:"artem";s:7:"user_id";s:1:"1";}i:1;O:8:"stdClass":3:{s:20:"comment_author_email";s:22:"ss.resurs.ru@gmail.com";s:14:"comment_author";s:12:"Сергей";s:7:"user_id";s:1:"0";}}}
  _yoast_wpseo_focuskw: Micropython
  _yoast_wpseo_metadesc: Про второй традиционный проект для микроконтроллера - измерение
    температуры и влажности со всеми любимым датчиком DHT22.
  _yoast_wpseo_linkdex: '62'
  _wp_old_slug: micropython-on-esp8366-sending-data-to-thingspeak
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617983501'
permalink: "/ru/diy-electronics-ru/micropython-on-esp8266-sending-data-to-thingspeak.html"
---
Традиционным "Hello World!"&nbsp;в мире микроконтроллеров&nbsp;можно по праву назвать мигание светодиодом. Это и было сделано после [установки MicroPython на ESP8266](/fun-ru/diy-electronics-ru/how-to-run-micropython-on-esp8266.html). Но время идти двигаться дальше и реализовать второй традиционный проект для микроконтроллера - измерение температуры и влажности со всеми любимым датчиком DHT22. Но мы не ограничимся тихими измерениями, а вместе с этим будем отсылать эти важные данные в интернеты. Для этого у нас есть, например, прекрасный сервис ThingSpeak. Пополним армию Internet of Shit!

English version -&nbsp;[MicroPython on ESP8266: sending data to ThingSpeak](/fun/diy-electronics/micropython-esp8266-sending-data-to-thingspeak.html)



# Железо и схема: ESP8266 (ESP-07) и DHT22

За основу возьмем предыдущий&nbsp;пост&nbsp;-&nbsp;[как запустить MicroPython на ESP8266](/fun-ru/diy-electronics-ru/how-to-run-micropython-on-esp8266.html). Все описанное в том&nbsp;посте вполне применимо и к этому.

Так же отметим, что лучше использовать ESP8266, у которых объем памяти превышает 512K, потому что можно использовать полные версии MicroPython. Похоже, что [урезанные версии MicroPython могут не всегда хорошо работать на ESP8266](/fun-ru/diy-electronics-ru/problems-with-running-micropython-on-esp8266-with-512k-2).

Будем опять использовать ESP-07, хотя можно использовать и ESP-12 без внесения изменений в схему ниже. С другими версиями ESP8266 возможно придется что-то поменять.

![Схема включения ESP8266 ESP-07 и DHT22]({{ site.baseurl }}/assets/images/2017/05/circuit.jpg)

Схема похожа на ту, что была в предыдущем посте. Вот, что было добавлено

- Датчик DHT22 и сопротивление R6.
- Переключатель на GPIO5, который используется для перевода устройства в&nbsp;конфигурационный режим, о котором ниже.
- Переключатель на&nbsp;GPIO4, который в результате не использовался. Он был добавлен просто на всякий случай, потому что у меня были лишь сдвоенные переключатели.

Как уже говорилось ранее, для измерения температуры и влажности будем использовать датчик&nbsp;DHT22. Можно также использовать&nbsp;DHT11. Это довольно популярные датчики, и MicroPython любезно умеет работать с ними. Вот так выглядит&nbsp;DHT22, если кто-то еще не видел:

![DHT22 датчик]({{ site.baseurl }}/assets/images/2017/05/IMG_20170225_141850.jpg)

Так все выглядело в результате:

![ESP8266 м датчик DHT22]({{ site.baseurl }}/assets/images/2017/05/IMG_20170225_143653.jpg)

# Запуск&nbsp;Wi-Fi точки доступа на ESP8266

С железной частью мы разобрались, перейдем к программной. Весь код лежит на GitHub:

[https://github.com/artem-smotrakov/yellow-duck](https://github.com/artem-smotrakov/yellow-duck)

```
git clone https://github.com/artem-smotrakov/yellow-duck
```

Так как мы собираемся отсылать температуру и влажность на ThingSpeak, нам надо подключить нашу ESP8266 к существующей (скажем, домашней) Wi-Fi сети для доступа в интернеты. Для этого&nbsp;нашей ESP8266 надо знать SSID и пароль к Wi-Fi сети. Вот так можно подключиться к Wi-Fi сети с помощью MicroPython:

```
import network
nic = network.WLAN(network.STA_IF)
nic.active(True)
nic.connect(ssid, password)
```

Но это не совсем хорошо и удобно, когда мы прописали пароли-явки прямо в код. Если SSID и пароль изменятся, то надо надо будет поправить исходники, а потом залить их опять на устройство. Вместо прописывания паролей и явок прямо в коде мы может сделать следующее:

- Добавить на пин GPIO5 переключатель в конфигурационный режим.
- При старте устройство проверяет, находится ли оно в конфигурационном режиме (функция&nbsp;`is_config_mode`).
- Если включен конфигурационный режим, то устройство поднимает Wi-Fi точку доступа и стартует локальный web-сервер (функции&nbsp;`start_access_point` и `start_local_server`).
- Когда пользователь подключается к этой точке доступа, web-сервер предлагает ему ввести SSID и пароль от Wi-Fi сети, которая должна использоваться в обычном режиме.
- После того, как пользователь введет SSID и пароль, web-сервер сохраняет их в конфигурационном файле и перезагружает устройство (функции&nbsp;`write_wifi_config` и `reboot`).
- До перезагрузки, пользователь должен переключить устройство из конфигурационного режима в обычный.
- После перезагрузки устройство подключается к Wi-Fi сети, SSID и пароль которой был указан (функция `connect_to_wifi`).
- В случае успешного подключения зажигается светодиод LD1 (функция&nbsp;`turn_on_wifi_led`).

Таким образом, если мы захотим использовать устройство с другими Wi-Fi сетями, или просто поменялся пароль, нам не надо будет заливать поправленные исходники.

После успешного подключения к Wi-Fi устройство начинает героически измерять температуру и влажность с определенным интервалом:

```
while True:
    current_time = time.time()
    if current_time - last_mesurement_time > MESUREMENT_INTERVAL:
        mesure_temperature_and_humidity()
        last_mesurement_time = current_time
    time.sleep(DELAY)
```

Как уже упоминалось ранее, MicroPython поддерживает как DHT22, так и DHT11. Вот как мы используем функцию&nbsp;`dht.DHT22` для измерения нужных нам характеристик агрессивной окружающей среды:

```
def mesure_temperature_and_humidity():
    import dht
    import machine
    d = dht.DHT22(machine.Pin(DHT22_PIN))
    d.measure()
    t = d.temperature()
    h = d.humidity()
    print('temperature = %.2f' % t)
    print('humidity = %.2f' % h)
```

Пока вроде бы довольно просто. Дальше нам надо отправить ценные данные в интернеты. Данные эти будем отправлять на ThingSpeak, который любезно и безвозмездно предлагает&nbsp;зарегистрироваться, чтобы потом можно было туда отсылать всякие важные данные вроде температуры и влажности воздуха, и более того, анализировать эти самые данные и даже рисовать по ним картинки. ThingSpeak для этих целей предоставляет RestFUL API для IoT устройств. Сначала регистрируемся. Дальше надо создать канал. Потом для этого канала получаем API-ключ, который будет использоваться устройством для отправки данных в этот канал. Не забываем включить два поля в нашем канале: одно для температуры, другое для влажности.

![Настройки канала в ThingSpeak]({{ site.baseurl }}/assets/images/2017/05/thingspeak.png)

Вот так мы отправляем данные в&nbsp;ThingSpeak:

```
global THINGSPEAK_WRITE_KEY
    if not THINGSPEAK_WRITE_KEY:
        print('not ThingSpeak key specified, skip sending data')
        return

    print('send data to ThingSpeak')
    s = socket.socket()
    ai = socket.getaddrinfo(API_THINGSPEAK_HOST, API_THINGSPEAK_PORT)
    addr = ai[0][-1]
    s.connect(addr)
    s = ssl.wrap_socket(s)
    data = 'field1=%.2f&field2=%.2f' % (t, h)
    http_data = THINGSPEAK_POST_TEMPLATE % (THINGSPEAK_WRITE_KEY, len(data), data)
    s.write(http_data.encode())
    s.close()
```

Здесь у нас возникает та же самая проблема, что и с SSID и паролем от Wi-Fi. Нам надо как-то рассказать устройству о API ключе, который оно должно использовать. Решение точно такое же: устройство предлагает указать API-ключ в конфигурационном режиме.

Как можно заметить, мы используем TLS, о чем говорит вызов функции&nbsp;`ssl.wrap_socket`, который заворачивает обычный socket&nbsp;в TLS socket. Признаться, я был немножко, но приятно удивлен, когда обнаружил, что MicroPython для ESP8266 умеет работать с TLS. Но есть один нюанс. Даже если TLS и используется, MicroPython на ESP8266 не поддерживает проверку серверного сертификата. (во всяком случае, версия 1.8.7 это не поддерживает). Это означает, что мы в относительной безопасности, пока злобные злоумышленники могут просто читать наши зашифрованные сообщения в рамках TLS-соединения. Расшифровать эти сообщения будет немного проблематично (но если во благо времени, то в общем случае это все равно возможно). Но вот если хитрые злоумышленники могут модифицировать передаваемые данные, то это уже беда, потому что они могут запросто подменить сертификат сервера с заранее известными ключами, и в результате они смогут спокойно расшифровать все наши сообщения.

Внимательный читатель кода мог заметить разные вызовы функции `print`. Это просто отладочная информация. Можно подсоединиться к ESP8266 через USB-Serial адаптер и увидеть эти самые отладочные сообщения.

# Визуализация данных на&nbsp;ThingSpeak

Если все прошло гладко, то устройство начнет отсылать данные на ThingSpeak. Дальше ThingSpeak великодушно предоставляет нам разные способы визуализации наших важных данных. И даже более того, ThingSpeak щедро позволяет нам анализировать эти данные с помощью MatLab. Самое простое, что приходит в голову, это нарисовать график:

<iframe style="border: 1px solid #cccccc;" src="https://thingspeak.com/channels/230189/charts/1?bgcolor=%23ffffff&amp;color=%23d62020&amp;dynamic=true&amp;results=60&amp;type=line&amp;update=15" width="450" height="260"></iframe>

<iframe style="border: 1px solid #cccccc;" src="https://thingspeak.com/channels/230189/charts/2?bgcolor=%23ffffff&amp;color=%23d62020&amp;dynamic=true&amp;results=60&amp;type=line&amp;update=15" width="450" height="260"></iframe>

<iframe style="border: 1px solid #cccccc;" src="https://thingspeak.com/apps/matlab_visualizations/131354" width="450" height="260"></iframe>

Эти графики показывают температуру и влажность в моей комнате. Вот и канал на ThingSpeak, который целиком и полностью посвящен этим важным показателям:

[https://thingspeak.com/channels/230189](https://thingspeak.com/channels/230189)

## Ссылки

- [Quick reference for the ESP8266](https://docs.micropython.org/en/latest/esp8266/esp8266/quickref.html)
- [MicroPython - network configuration](https://docs.micropython.org/en/latest/esp8266/library/network.html)
- [MicroPython - Temperature and Humidity](https://docs.micropython.org/en/latest/esp8266/esp8266/tutorial/dht.html)
- [MicroPython - socket module](https://docs.micropython.org/en/latest/esp8266/library/usocket.html)
- [Temperature and humidity module AM2302 Product Manual](http://akizukidenshi.com/download/ds/aosong/AM2302.pdf)
