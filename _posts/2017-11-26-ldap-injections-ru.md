---
layout: post
title: LDAP-инъекции
date: 2017-11-26 22:11:40.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- LDAP
- Python
- Security
meta:
  mask_links: default
  _edit_last: '1'
  _aioseop_opengraph_settings: a:14:{s:32:"aioseop_opengraph_settings_title";s:0:"";s:31:"aioseop_opengraph_settings_desc";s:0:"";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}
  _wpgo_column_layout_save: ''
  _wpgo_column_layout: default
  _aioseop_keywords: LDAP, инъекция, injection, blind, java, python, пример, злоумышленник,
    перебор
  _aioseop_description: 'Зарисовка на тему LDAP-инъекций: что такое LDAP-инъекция,
    пример уязвимого приложения, пример blind LDAP-инъекции, как предотвратить LDAP-инъекции.'
  _aioseop_title: LDAP-инъекции
  rp4wp_auto_linked: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_primary_category: ''
  _yoast_wpseo_focuskw_text_input: LDAP-инъекции
  _yoast_wpseo_focuskw: LDAP-инъекции
  _yoast_wpseo_metadesc: Каждый знает про SQL-инъекции (SQL injection). Это как знаменитость
    в мире ИБ. Но кроме них еще существует множество других разновидностей инъекций,
    которые могут лишь позавидовать популярности SQL-инъекций. И это не совсем справедливо.
    Устраним же эту несправедливость и поговорим о LDAP-инъекциb (LDAP injection).
  _yoast_wpseo_linkdex: '79'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618014599'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/ru/security-ru/ldap-injections-ru.html"
---
Каждый знает про SQL-инъекции (SQL injection). Это как знаменитость в мире ИБ. Но кроме них еще существует множество других разновидностей инъекций, которые могут лишь позавидовать популярности SQL-инъекций. И это не совсем справедливо. Устраним же эту несправедливость и поговорим о LDAP-инъекции (LDAP injection).

[English version is here.](https://blog.gypsyengineer.com/fun/security/ldap-injections.html)

<!--more-->

## Что такое LDAP?

LDAP означает Lightweight Directory Access Protocol. Это сетевой протокол прикладного уровня для доступа к так называемой службе каталогов. Протокол является не текстовым (как, например, HTTP), а бинарным. LDAP протокол обычно используется поверх TCP/IP, но лучше еще использовать в рамках TLS соединения. За подробностями о LDAP протоколе стоит обратиться к&nbsp;[RFC 2251](https://www.ietf.org/rfc/rfc2251.txt).

Если вкратце, то LDAP сервер содержит базу данных, где данные хранятся в виде "атрибут = значение".&nbsp; LDAP описывает как эти данные могут быть получены и изменены.

На текущей момент известно некоторое количество&nbsp;[ПО, которое поддерживает LDAP](https://en.wikipedia.org/wiki/List_of_LDAP_software), например:

- OpenLDAP
- Java содержит в себе LDAP-клиент, который доступен через JNDI.

## Что такое LDAP-инъекция?

LDAP-инъекция (LDAP injection) это способ нападения, который позволяет злобному злоумышленнику коварно модифицировать LDAP-запрос, который использует приложение для обращения к LDAP-хранилищу. Достигается это злодейство&nbsp;путем отправки приложению хитроумно подготовленных данных, которые содержат части LDAP-запроса. В результате беспринципный злоумышленник может отправлять LDAP-хранилищу всевозможные запросы, порой даже абсолютно любые, которые приложение само никогда бы не отправило. Это в свою очередь может привести к несанкционированному чтению или даже изменению всякой важной информации, которая может храниться в LDAP-хранилище. Обычно LDAP-инъекции возникают, потому что приложение не проверяет входные данные от пользователя, который может оказаться злобным злоумышленником, и наивно использует их для построения LDAP-запроса к базе данных.

## Пример LDAP-инъекции

Давайте на минуточку представим, что у нас есть LDAP-сервер, который надежно спрятан в нашей внутренней сети, которую героически защищает от враждебного внешнего мира наша огненная стена, которая больше известна, как firewall. Пусть этот LDAP-сервер хранит данные пользователей, такие как адреса, пароли, явки. Еще представим, что у нас есть некое приложение, которое использует данные из нашего LDAP-хранилища для аутентификации пользователей. Другими словами, для предоставления доступа наше приложение просит пользователя ввести имя и пароль, которые оно потом сверяет с данными, которые хранятся на нашем LDAP-сервере.

В целях демонстрации мы можем запустить локальный LDAP-сервер с помощью [Ldaptor](https://ldaptor.readthedocs.io/en/latest/)&nbsp;(он еще нуждается в&nbsp;[Twisted](https://github.com/twisted/twisted), который можно установить с помощью pip). Разработчики Ldaptor любезно предоставили [пример LDAP-сервера](https://ldaptor.readthedocs.io/en/latest/quickstart.html#ldap-server-quick-start), который я благодарно позаимствовал:

<script src="https://gist.github.com/artem-smotrakov/0316503009bd1d6a6ce39aa1be171235.js"></script>

Вышеприведенный сервер может быть запущен с помощью команды&nbsp;`python ldapserver.py`

А вот и пример уязвимого приложения на Java:

<script src="https://gist.github.com/artem-smotrakov/b81033aee088c09091399774dd951308.js"></script>

Наше бедное приложение спрашивает у пользователя его имя и пароль, которые пользователь предоставляет через параметры командной строки. Далее наше милое приложение отправляет LDAP-запрос к нашему LDAP-хранилищу, чтобы получить соответствующую запись о нашем пользователе. Если такая запись была найдена, то счастливый пользователь получает доступ, который ознаменовывается появлением сообщения "Access granted". Приведем пример того, как это приложение работает, когда пользователь вводит правильные и неправильные пароли:

```
$ javac -d classes LDAPLogin.java 
$ java -classpath classes LDAPLogin bob secret
LDAP query: (&(uid=bob)(userPassword=secret))
Access granted
$ java -classpath classes LDAPLogin bob wrong
LDAP query: (&(uid=bob)(userPassword=wrong))
Access denied
```

Проблема этого простого приложения заключается в том, что оно ветрено подставляет имя и пароль, которые предоставляет ему пользователь, прямо в LDAP-запрос. Такое халатное поведение приложения дает хитрому злоумышленнику возможность модифицировать запрос, которые отправляется к LDAP-хранилищу. В свою очередь это может быть использовано, например, для обхода аутентификации. Если коварный злоумышленник не знает пароля Боба, то он может попросту использовать строку&nbsp;`bob)(|(uid=bob`&nbsp;в качестве имени пользователя и строку `wrong)` в качестве пароля, что в результате позволит ему получить долгожданный доступ.

```
$ java -classpath classes LDAPLogin "bob)(|(uid=bob" "wrong)"
LDAP query: (&(uid=bob)(|(uid=bob)(userPassword=wrong)))
Access granted
```

Можно видеть, что приложение отправило запрос&nbsp;`(&(uid=bob)(|(uid=bob)(userPassword=wrong)))`&nbsp;к LDAP-хранилищу. LDAP использует в запросах к базе данных так называемую "польскую нотацию". Этот запрос означает&nbsp;"(uid == bob) and (uid=bob or userPassword=wrong)". Это логическое выражение всегда истинно для записи Боба, поэтому запрос всегда возвращает запись Боба, если даже был подсунут неправильный пароль. В результате злобный злоумышленник может обойти аутентификацию.

Настоящий exploit для LDAP-инъекции может зависеть от разных вещей, например:

- LDAP-запроса, который используется приложением
- Версии LDAP-сервера
- Логики работы приложения

В природе можно наблюдать три основных разновидностей LDAP-запросов:

- Запрос без логических операторов. Выглядят они как `(field=value)`. Если даже мы можем поместить что угодно вместо `value`, то мы все равно не сможем использовать никакие логические операторы, потому что записи в Польской нотации требуют, чтобы логический оператор был вначале. Все что можно сделать, это попробовать использовать какую-то уязвимость в самом LDAP-сервере, если она может быть вызвана каким-то значением `value`. Например, это может быть переполнение буфера, если конечно сервер ему подвержен.
- Оператор AND находится вначале поискового запроса. Выглядит это как&nbsp;`(&(field1=value1)(field2=value2))`. Пример подобной ситуации был рассмотрен выше.
- Оператор OR находится вначале поискового запроса. Выглядит это как
- `(|(field1=value1)(field2=value2)).`

Для успеха предприятия очень хорошо и полезно знать, какой тип запроса используется приложением.

Приложение также может использовать LDAP-запросы для обновления данных в LDAP-хранилище. LDAP использует LDAP Data Interchange Format (LDIF) для обновления данных в хранилище. LDIF представляет собой список команд для добавления, удаления или изменения записей в базе данных. Это попросту набор записей, одна запись - одна команда. `ldapserver.py`, который приведен выше, содержит пример данных в формате LDIF, который добавляет три записи в базу данных. Если хитрый злоумышленник может внедрить свои данные в LDIF запрос, он может использовать CRLF символы для разделения команд. В результате, злобный злоумышленник потенциально может, например, добавить любые записи в базу данных. Это чем-то похоже на HTTP request splitting.

Версия LDAP-сервера тоже может иметь значение, потому что разные версии LDAP-серверов могут иметь разные особенности в обработке запросов. Иногда коварный злоумышленник может использовать LDAP-инъекцию таким образом, что LDAP-запрос включает в себя несколько поисковых запросов. Если LDAP-сервер использует только первый запрос и игнорирует остальные, вместо возвращения ошибки, то это может помочь в осуществлении атаки. К сожалению, синтаксис LDAP-запросов не допускает комментирования окончания запроса, как это можно сделать в SQL.

## LDAP-инъекции вслепую (Blind LDAP injections)

Blind LDAP-инъекции чем-то похожи на blind SQL-инъекции. Приложение может быть уязвимо к LDAP-инъекции, но может так случиться, что вредное приложение не будет отображать все поля записей. Такое печальное обстоятельство не позволит злобному злоумышленнику просто прочитать содержимое LDAP-хранилища. Но если приложение каким-то образом дает знать, что отправленный запрос с данными от злоумышленника завершился успешно или нет, то такое поведение может позволить хитрому злоумышленнику задавать серверу так называемые да/нет вопросы. В результате беспринципный злоумышленник может осуществить эффективный перебор и извлечь ценные данные из базы данных.

Представим, что у нас есть такое приложение на Java. Оно спрашивает у пользователя его идентификатор для поиска записи пользователя в базе данных. Если запись была найдена, то приложение печатает номер телефона пользователя. Если не найдена, то приложение просто сообщает, что ничего не было найдено. Приложение может использовать LDAP-сервер выше. А вот и оно само.

<script src="https://gist.github.com/artem-smotrakov/e4d3bac16fa3404d89c9f09b830d8513.js"></script>

А вот и пример штатного использования приложения:

```
$ javac -d classes LDAPInfo.java 
$ java -cp classes LDAPInfo bob
LDAP query: (&(uid=bob)(objectClass=person))
Phone: telephoneNumber: 555-9999
$ java -cp classes LDAPInfo boba
LDAP query: (&(uid=boba)(objectClass=person))
Nobody found!
```

Кто-то может заметить, что LDAPInfo подвержен LDAP-инъекции. Хотя приложение лишь печатает номер телефона, коварный злоумышленник все равно может извлечь ценные данные из других полей. Например, злобный злоумышленник может спросить у сервера, начинается ли поле&nbsp;`userPassword`&nbsp;в записи Боба с буквы 'a':

```
$ java -cp classes LDAPInfo "bob)(userPassword=a*"
LDAP query: (&(uid=bob)(userPassword=a*)(objectClass=person))
Nobody found!
```

Если передать строку&nbsp;`bob)(userPassword=a*` в качестве имени пользователя, то это приведет к появлению сообщения 'Nobody found!'. Приложение построило запрос&nbsp;`(&(uid=bob)(userPassword=a*)(objectClass=person))`&nbsp;, который ничего не вернул, потому что поле&nbsp;`userPassword`&nbsp;из записи Боба не начинается с буквы 'a'. Теперь коварный злоумышленник может перебрать первую букву пароля Боба:

```
$ java -cp classes LDAPInfo "bob)(userPassword=a*"
LDAP query: (&(uid=bob)(userPassword=a*)(objectClass=person))
Nobody found!
$ java -cp classes LDAPInfo "bob)(userPassword=b*"
LDAP query: (&(uid=bob)(userPassword=b*)(objectClass=person))
Nobody found!
$ java -cp classes LDAPInfo "bob)(userPassword=c*"
LDAP query: (&(uid=bob)(userPassword=c*)(objectClass=person))
Nobody found!
[...]
$ java -cp classes LDAPInfo "bob)(userPassword=s*"
LDAP query: (&(uid=bob)(userPassword=s*)(objectClass=person))
Phone: telephoneNumber: 555-9999
```

Как только злобный злоумышленник попробует букву 's', то приложение вернет номер телефона Боба. Это означает, что пароль Боба начинается с буквы 's'. Дальше хитрый злоумышленник начинает перебирать вторую букву пароля Боба подставляя строки вида&nbsp;`bob)(userPassword=sa*`, `bob)(userPassword=sb*"`, `"bob)(userPassword=sc*`&nbsp; и так далее. Это позволяет реализовать эффективный перебор, чтобы получить пароль Боба.

Вот пример реализации такого перебора:

<script src="https://gist.github.com/artem-smotrakov/e624953f7843f8a67239e8db332c3333.js"></script>

## Как предотвратить LDAP-инъекции

Ничего революционного перечислено не будет. Проверка входных данных спасут гиганта мысли и отца русской демократии. Приложениям следует экранировать все специальные символы в данных, которые приходят от пользователей. OWASP любезно предоставляет статью об этом (смотри ниже), которая может быть применена не только к Web-приложениям.

Дополнительно мерой может быть отключение индексирования полей, которые могут содержать важную информацию такую как пароли. Например, если поле `userPassword`&nbsp;не индексировано, то запрос, который содержит `(userPassword=a*)`&nbsp;, приведет к ошибке:

```
$ ldapsearch -h ldap.server m -x -b "dc=test,dc=com" "(&(uid=test)(userPassword=a*))"
....
# search result
search: 2
result: 53 Server is unwilling to perform
text: Function Not Implemented, search filter attribute userpassword is not indexed/cataloged

# numResponses: 1
```

Но коварный злоумышленник все еще может попробовать извлечь данные из других индексированных полей.

Ссылки:

- [RFC 4511: Lightweight Directory Access Protocol (LDAP): The Protocol](https://tools.ietf.org/html/rfc4511)
- [LDAP injection (OWASP)](https://www.owasp.org/index.php/LDAP_injection)
- [LDAP Injection Prevention Cheat Sheet (OWASP)](https://www.owasp.org/index.php/LDAP_Injection_Prevention_Cheat_Sheet)
- [LDAP injection & Blind LDAP injection in Web applications](http://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)