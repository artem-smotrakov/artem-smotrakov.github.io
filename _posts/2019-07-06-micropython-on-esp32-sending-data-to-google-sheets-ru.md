---
layout: post
title: 'MicroPython на ESP32: отправка данных в Google Sheets'
date: 2019-07-06 11:28:41.000000000 +01:00
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
- Google
- MicroPython
- OAuth2
- Интернет вещей
- Электроника
meta:
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '153'
  _edit_last: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_focuskw: ESP32
  _yoast_wpseo_metadesc: 'Отправка данных в Google Sheets c ESP32: MicroPython, сложности
    с ESP8266, процесс OAuth2 аутентификации с Google, JWT, NTP и подпись RSA.'
  _yoast_wpseo_linkdex: '66'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618223800'
  _yoast_wpseo_estimated-reading-time-minutes: '4'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/ru/diy-electronics-ru/micropython-on-esp32-sending-data-to-google-sheets-ru.html"
---
<!-- wp:paragraph -->

Однажды в один прекрасный летний выходной вместо того, чтобы пойти на на пляж или куда-нибудь еще, я находился дома и размышлял, как бы отправить что-нибудь с ESP32 в документ Google Sheets, используя MicroPython. Пусть даже ситуация будет совсем классической, и роль данных будут играть температура и влажность, героически измеренные всенародно любимым датчиком DHT22. Так всё и началось.

<!-- /wp:paragraph -->

<!-- wp:image {"id":3302} -->

![MicroPython на ESP32: отправка данных в Google Sheets]({{ site.baseurl }}/assets/images/2019/07/esp32_dev_board_and_dht22.jpg)

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:paragraph -->

Быстрый поиск в Google показал, что идея не новая, и существуют люди, которые уже пытались отправлять что-то в Google таблицы, и их попытки даже увенчались успехом. Прочитав несколько статей об этом, были замечены две вещи: либо использовалась таблица, доступ к которой не ограничен ни для кого, либо проект использует посредника, такого как PushingBox или IFTTT.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Но ведь Google Sheets предоставляют прекрасный HTTP API, почему же просто не послать POST-запрос прямо с ESP, чтобы записать что-то в таблицу? Однако, на самом деле это не так просто, ведь Google Sheets заботиться о безопасности и требует аутентификации. Google Sheets API предлагает нам ровно два способа для подтверждения нашей с вами подлинности: API keys или OAuth2. Может показаться, что всё хорошо, и мы можем просто создать API key в Google IAM, забыть о безопасности и ротации этого API key и использовать его вечно для записи данных в нашу таблицу. Однако, к сожалению (или к счастью), этот сценарий не сработает, ведь Google Sheets API принимает API key, если мы хотим только почитать данные из таблицы, но если мы хотим туда что-то записать, то будьте любезны, используйте OAuth2.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

И тут начинаются сложности. Сначала нам нужно создать service account в Google IAM, а потом там же сгенерировать ключик RSA для этого service account. Хорошо, пока вроде бы несложно. Дальше наш ESP микроконтроллер должен соорудить JWT запрос, включить в него текущее время, а потом еще и подписать этот запрос с помощью алгоритма RSA-SHA256. Потом наш ESP должен отправить этот запрос в Google OAuth2 service, который должен нам вернуть OAuth2 token, который наконец-то может быть использован при обращении к Google Sheets API.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Итак, перечислим то, что обязательно должно быть на ESP, чтобы записать строчку в Google Sheets: HTTP клиент, JSON parser, часы с правильным временем или же NTP client, реализация подписи с RSA-SHA256. К счастью, MicroPython нам любезно предоставляет почти всё из этого списка за исключением RSA-SHA256. Если быть точным, то MicroPython умеет считать SHA256, но вот создавать RSA подписи он еще не научился. Омрачает жизнь еще тот факт, что RSA подпись требует довольно много вычислений и памяти, но ESP микроконтроллеры не такие быстрые и ёмкие, как процессоры на обыкновенных PC.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Предположу, что сложность процесса получения OAuth2 token и отсутствие реализации RSA алгоритма для MicroPython и побудили использовать либо таблицы с открытым доступом, либо посредников, которые могут взять на себя процесс аутентификации. Конечно, как security engineer по роду деятельности, я не мог смириться с таблицей в открытом доступе, которая тем более будет содержать такую страшно конфиденциальную информацию, как температура и влажность воздуха в моем жилище. И опять же, как security engineer по роду занятий, я не люблю всяких middleman'ов. В результате не осталось другого выбора кроме воплощения в жизнь процесса аутентификации на ESP.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Как уже было сказано выше, MicroPython любезно предоставляет следующее:

<!-- /wp:paragraph -->

<!-- wp:list -->

- [ujson](https://docs.micropython.org/en/latest/library/ujson.html) для работы с JSON
- [ntptime.py](https://github.com/micropython/micropython/blob/master/ports/esp8266/modules/ntptime.py) для получения времени по протоколу NTP
- [uhashlib](https://docs.micropython.org/en/latest/library/uhashlib.html) для хэшей SHA256
- [http.client](https://github.com/micropython/micropython-lib/tree/master/http.client) для работы с HTTP

<!-- /wp:list -->

<!-- wp:paragraph -->

Остается только раздобыть RSA подпись. Одно из главных правил в криптографии гласит: никогда не пишите сами никакие криптографические алгоритмы, а используйте уже существующие библиотеки. Это правило мы знаем, и быстро нашли довольно популярную библиотеку [python-rsa](https://github.com/sybrenstuvel/python-rsa/), которая предоставляет алгоритм RSA, написанный на чистом Python. Помимо создания подписей, библиотека позволяет их проверять, шифровать и дешифровать тексты, загружать ключики из файлов и так далее. Все эти операции делают библиотеку немного тяжеловатой для худенького ESP. Поэтому было решено библиотеку укоротить и оставить лишь создание подписи, которое нужно для формирования JWT запросов. Пришлось даже удалить загрузку ключей из PKCS1, потому как это требовало портирования [pyasn1](https://github.com/etingof/pyasn1). В процессе обрезания библиотеку внезапно выяснилось, что в MicroPython отсутствуют некоторые стандартный для CPython функции, например, возведение в степень по модулю. [Попутно пришлось это упущение исправить](https://en.wikipedia.org/wiki/Modular_exponentiation#Right-to-left_binary_method). В результате получилась новая маленькая библиотечка [micropython-rsa-signing](https://github.com/artem-smotrakov/micropython-rsa-signing), все желающие могут ей насладиться.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

За основу был взят [один из прошлых проектов](https://blog.gypsyengineer.com/ru/diy-electronics-ru/how-to-run-micropython-on-esp8266.html). В целях удобства и экономии времени, сначала все было написано и протестировано на ноутбуке. И вот настал момент залить и запустить всё это дело на ESP. Первая попытка была с ESP8266, но она провалилась. RSA подпись работала очень медленно, но основная проблема заключалась в том, что процесс не мог завершиться, потому что попросту не хватало памяти во время возведения в степень по модулю. Были испробованы некоторые оптимизации, была даже попытка скомпилировать код вместе с МicroPython, но всё это не увенчалось успехом.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Последняя надежда была на ESP32, которая, как известно, чуть более быстрая и вместительная, чем ESP8266. До этого я не экспериментировал с ESP32, и c eBay была выписана новая платка с ESP32. И внезапно всё заработало. Код больше не ругался на недостаток памяти и вообще работал куда быстрее.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

[Код лежит на GitHub](https://github.com/artem-smotrakov/esp32-weather-google-sheets). Там же есть инструкции, правда, не сильно подробные и пока только на английском. Еще был создан проект на [Hackaday.io](https://hackaday.io/project/166197-esp32-weather-station-and-google-sheets). Надеюсь, удастся найти время и описать всё подробнее и на русском языке.

<!-- /wp:paragraph -->
