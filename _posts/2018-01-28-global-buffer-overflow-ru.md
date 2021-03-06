---
layout: post
title: Переполнение буфера в глобальной области памяти
date: 2018-01-28 22:38:34.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Security
permalink: "/ru/security-ru/global-buffer-overflow-ru.html"
---
<p style="text-align: justify;">Написано много статей, постов и даже книг о переполнении буферов в стеке. Чуть меньше про переполнение буфера в куче. Но есть еще одна вещь, которую можно переполнить, и о которой пишут мало. Это буфер в глобальной области памяти (global memory). Хотя все эти проблемы сильно похожи друг на друга, тем не менее попробуем заполнить этот небольшой пробел с переполнениями буфера в глобальной памяти.</p>
<p><a href="/en/security/global-buffer-overflows.html">English version</a></p>

<h2>О глобальной области памяти (global memory)</h2>
<p style="text-align: justify;">Есть два места, где могут располагаться глобальные и статические переменные:</p>
<ul style="list-style-type: square;">
<li>сегмент для инициализированных переменных и буферов</li>
<li>сегмент для неинициализированных переменных и буферов (BSS сегмент)</li>
</ul>
<p style="text-align: justify;">Те переменные, которые не инициализированы явно (то есть им не присваивается никакое значение в момент объявления), располагаются в BSS сегменте, после чего они автоматически заполняются нулями.</p>
<p>Вот так примерно выглядит память:</p>
<pre class="console">         старшие адреса
+-----------------------------+
| параметры командной строки  |
| и переменные окружения      |
+-----------------------------+
|           стек              |
+-------------+---------------+
|             |               |
|             V               |
|                             |
|                             |
|                             |
|                             |
|             ^               |
|             |               |
+-------------+---------------|
|            куча             |
+-----------------------------+ 
| неинициализированные данные |
|       (BSS сегмент)         |
|     (заполнено нулями)      |
+-----------------------------+
|  инициализированные данные  |
+-----------------------------+
|            код              |
+-----------------------------
+ младшие адреса

## Пример глобального переполнения буфера

Вот и простой пример, который демонстрирует переполнение буфера в глобальной области памяти:

<script src="https://gist.github.com/artem-smotrakov/6d0c774621afcfc3b2b1c72bf7cca549.js"></script>

## Как использовать переполнение буфера в глобальной памяти?

Как обычно, все зависит, от того, что именно и как переполняется. Вот несколько типичных факторов:

- какие данные хранятся в памяти рядом с переполняемым буфером
- можем ли мы писать в память рядом с переполняемым буфером
- можем ли мы читать из памяти рядом с переполняемым буфером

Посмотрим на несколько примеров.

### Перезапись важных данных

Ниже приведен пример приложения, которое спрашивает у пользователя пароль. Если пароль правильный, то приложение печатает секретную фразу.

<script src="https://gist.github.com/artem-smotrakov/218e8b185c5e2d2a1d73610bc657e317.js"></script>

Вызов `strcpy` может переполнить&nbsp;`buffer`, если пароль содержит больше, чем 15 символов (`strcpy` добавляет \0 в конец строки). В результате мы можем переписать переменную&nbsp;`access`:

(мы будем использовать Python для генерирования длинный строк)

```
$ gcc -g gbo.c -o gbo
$ ./gbo wrong
access denied
$ ./gbo `python3 -c "print('x' * 16 + 'y')"`
this is a secret
```

Это происходит, потому что&nbsp;`buffer`&nbsp;и&nbsp;`access` находятся в сегменте для неинициализированных данных. Более того, в памяти&nbsp;`access` располагается сразу после&nbsp;`buffer`. На самом деле, расположение эти переменных в памяти может быть и другим, потому что порядок следования глобальных переменных не определен. Поэтому код выше может быть и неуязвим в некоторых случаях (это может зависеть, например, от компилятора).

Здесь есть еще одна интересная деталь. Проблема уйдет, если мы явно инициализируем переменную&nbsp;`access`&nbsp;при ее объявлении:

```
char access = 'n';
```

В этом случае компилятор разместит переменную&nbsp;`access` в сегмент для инициализированных переменных, который обычно идет до сегмента для неинициализированных данных. В результате переполнение&nbsp;`buffer`&nbsp;не будет приводить к перезаписи переменной&nbsp;`access`.

### Перезапись объектов в куче или простой segfault

Обычно куча начинается где-то после BSS сегмента. Но фактический адрес начала кучи может быть разным. Посмотрим на вот такой код:

<script src="https://gist.github.com/artem-smotrakov/9638c9a39e6968f19481d8ccf67aef5a.js"></script>

Сначала определяем&nbsp;`buffer`&nbsp;в глобальной области памяти и&nbsp;`allocated` в куче. Потом копируем строку "test" в буфер&nbsp;`allocated`. Дальше печатаем адреса и содержимое буферов. И в самом конце копируем первый параметр командной строки в глобальный буфер и опять печатаем&nbsp;`allocated`.  
На моем Линуксе следующая команда вызывает segfault:

```
$ ./gbo `python3 -c "print('x' * 2**12)"`
buffer address = 0x601070
allocated address = 0x773010
allocated address - buffer address = 1515424
allocated (before) = test
Segmentation fault (core dumped)
```

Здесь мы пытаемся запихать в&nbsp;`buffer`&nbsp;строку, которая состоит из 4096 символов 'x', и которая его успешно переполняет. Буфер&nbsp;`allocated`&nbsp;располагается по адресу 0x773010 (вообще этот адрес меняется от запуска к запуску из-за динамического выделения памяти). Заметим, что разница между адресами намного больше (1515424), чем 4096. В результате мы получаем segfault, потому что мы пытаемся записать что-то по некорректному адресу. Не очень похоже, что есть возможность перезаписывать объекты в куче, если мы можем переполнить буфер в глобальной области памяти. Но всегда можно уронить приложение.

### Перезапись указателя на функцию в глобальной области памяти

Указатель на функцию просто содержит адрес этой функции в памяти. Указатель на функцию может быть использовать для вызова этой функции. Все довольно просто. Вот пример перезаписи указателя на функцию:

<script src="https://gist.github.com/artem-smotrakov/b5ec30cd841cac298d60fbf39e43a5b7.js"></script>

Этот код похож на тот, что мы рассмотрели ранее, только здесь мы балуемся с указателем на функцию. Сначала указатель на функцию `func` не инициализирован. Затем мы помещаем в него адрес функции&nbsp;`do_something`. Если пароль правильный, то помещаем в указатель адрес функции&nbsp;`print_secret`. И наконец с помощью указателя&nbsp;`func`&nbsp;мы вызываем функцию, на которую он указывает.

Вызов `strcpy` может переполнить&nbsp;`buffer` , если параметр командной строки больше 15ти&nbsp;символов (не забываем, что `strcpy` добавляет \0 в конец строки). Так как и&nbsp;`func`, и `buffer` не инициализированы сразу, то они оба живут в сегменте для неинициализированных данных. В результате чего мы и может перезаписать указатель&nbsp;`func`:

```
$ gcc -g gbo.c -o gbo
$ ./gbo `python3 -c "print('w' * 256)"`
Segmentation fault (core dumped)
```

Мы только что записали в&nbsp;`func` адрес 0x77777777 (0x77 это ASCII-код символа 'w'). Далее наше наивное приложение попыталось вызвать функцию, которая располагается по этому адресу. Так как этот адрес некорректный, мы получили segfault. Но простое падение приложение это не интересно. Интереснее заставить приложение выполнить то, что мы хотим. Предположим, что мы хотим вызвать функцию&nbsp;`print_secret`&nbsp;, но мы не знаем пароля. Сначала выясним адрес функции&nbsp;`print_secret`. GDB поможет нам с этим:

```
$ gdb --args ./gbo test
Reading symbols from ./gbo...done.
(gdb) break gbo.c:36
Breakpoint 1 at 0x40068c: file gbo.c, line 36.
(gdb) run
Starting program: /home/artem/tmp/gbo test

Breakpoint 1, main (argc=2, argv=0x7fffffffdcf8) at gbo.c:36
36 func();
(gdb) p func
$1 = (void (*)(void)) 0x7777777777777777
(gdb) p print_secret 
$2 = {void (void)} 0x400607 
(gdb) quit
```

Теперь мы знаем, что адрес функции&nbsp;`print_secret`&nbsp;это 0x400607. Дальше нам нужно передать приложению такую строку, чтобы оно записало адрес 0x400607 в указатель&nbsp;`func`. Для этого надо учесть следующее:

- надо записать 16 байтов, чтобы заполнить&nbsp;`buffer`
- помним, что мы в 64-битной системе, поэтому нам надо 8 байтов для перезаписи указателя&nbsp;`func`
- помним, что мы в little-endian системе, так что вместо 0x400607 нужно писать 0x070640

Следующая команда заставляет приложение вызвать функцию&nbsp;`print_secret`, даже если предоставлен неправильный пароль:

```
$ ./gbo `python3 -c "print('w' * 16 + '\x00\x00\x00\x00\x00\x07\x06\x40')"`
this is a secret
```

### Чтение конфиденциальных данных из глобальной памяти

Наверное каждый слышал про Heartbleed и OpenSSL. Это прекрасный пример так называемой "buffer overread" уязвимости, которая означает, что хитрый злоумышленник может читать память за пределами буфера. В случае с Heartbleed, коварный злоумышленник мог читать конфиденциальные данные из кучи. Подобные уязвимости возможны и с глобальной памятью, где тоже могут храниться всякие конфиденциальные данные. Вот простой пример уязвимого приложения:

<script src="https://gist.github.com/artem-smotrakov/7fc2d7390f99974b6a663d77d198a739.js"></script>

Приложение получает количество символов, которое надо напечатать. Оно копирует обозначенное количество байтов из глобального буфера&nbsp;`public` в локальный&nbsp;`buffer`. Дальше оно печатает все строки в&nbsp;`buffer`. Если количество запрашиваемых символов больше, чем размер&nbsp;`public`, то приложение будет послушно читать память за пределами&nbsp;`public`. Это приводит к чтению буфера&nbsp;`secret`, который следует сразу же за&nbsp;`public` в сегменте для неинициализированных данных. В результате содержимое&nbsp;`secret` печатается на экран.

## Как предотвратить переполнение буфера

Все тоже самое, что и в случае с переполнениями в стеке и куче. Разница между ними не большая. Разработчикам следует мыть руки с мылом перед работой с памятью и быть с ней очень внимательными. Использование мозга и трепетный подход к программированию может помочь избежать подобных проблем. Разумные сроки и отсутствие постоянного аврала и штурмовщины создают положительные условия для предотвращения переполнений буферов (дорогие менеджеры, вы можете вашим программистам). Code review, статические и динамические анализаторы также помогают вовремя отловить возникшие проблемы.

