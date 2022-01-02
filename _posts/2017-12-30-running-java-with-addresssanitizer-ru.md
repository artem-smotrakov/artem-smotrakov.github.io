---
layout: post
title: Запуск Java вместе с AddressSanitizer
date: 2017-12-30 16:42:07.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Security
tags:
- Java
- Security
permalink: "/ru/security-ru/running-java-with-addresssanitizer-ru.html"
---
OpenJDK и AddressSanitizer довольно известные open-source проекты. OpenJDK содержит код на C/C++, который в отличие от кода на Java, может напрямую работать с памятью, что в свою очередь может привести к известным проблемам связанными с доступом за пределы выделенной памяти. Это всем известные и всевозможные переполнения буферов, use-after-ftee и прочие проблемы. Подобные проблемы могут быть обнаружены с помощью инструментов, которые называются runtime memory checkers. Одним из таких инструментов является AddressSanitizer. И теперь стало гораздо легче использовать AddressSanitizer вместе с OpenJDK.

[English version is also available.](/fun/running-java-with-addresssanitizer.html)

## Что такое AddressSanitizer?

Как уже было отмечено, AddressSanitizer (или сокращенно ASan) инструмент, которые позволяет выявлять ошибки связанные с работой с памятью в C/C++ коде. Примерами таких ошибок могут быть переполнения буферов в разных областях памяти, чтение данных за пределами выделенной памяти, use-after-free, а также утечки памяти. Если кратко, то AddressSanitizer инструментирует код на C/C++ в процессе компиляции и вставляет дополнительные проверки, которые позволят обнаружить ошибки доступа к памяти во время выполнения инструментированного кода. Этот инструмент доступен в GCC 4.8+ и Clang 3.1+ работает с несколькими платформами включая&nbsp; Linux x64 и MacOS x64. Пользователям Windows повезло меньше - в настоящее время официальная версия AddressSanitizer официально не работает на многими любимой уютненькой операционной системе от Microsoft.&nbsp; AddressSanitizer может быть задействован, если передать параметр&nbsp;`-fsanitize=address`&nbsp;компилятору и linker'у. Подробности на официальной страничке AddressSanitizer:

[https://github.com/google/sanitizers/wiki/AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer)

## Как же использовать AddressSanitizer для OpenJDK?

Java (или точнее OpenJDK) содержит много кода на C/C++ во многих своих компонентах, например:

- Большая часть виртуальной машины Hotspot написана на C/C++
- Java Core Libraries используют JNI для доступа к системным функциям. Для примеров можно посмотреть на код Java I/O или код для работы с сетью.
- Java Security Libraries используют JNI для более быстрого выполнения разных криптографических операций, а также для доступа к системным функциям по стандарту PKCS11

Разработчики ОpenJDK это люди, которые могут совершать ошибки, которые могут привести к печальным последствиям, таким как переполнения буферов и прочим напастям. Для своевременного обнаружения и устранения таких проблем мы и можем использовать AddressSanitizer.

## Как использовать AddressSanitizer вместе с Java

Теперь можно построить OpenJDK с включенным AddressSanitizer. Поддержка AddressSanitizer была недавно добавлена в confiture/make файлы для OpenJDK:

[http://hg.openjdk.java.net/jdk10/master/rev/77a5f2ef1807](http://hg.openjdk.java.net/jdk10/master/rev/77a5f2ef1807)

Чтобы включить ASan при построении OpenJDK, нужно передать&nbsp;`--enable-asan` параметр для&nbsp;`configure`&nbsp;скрипта. Следующие команды позволяют построить OpenJDK с включенным AddressSanitizer:

```
# clone OpenJDK workspace
hg clone http://hg.openjdk.java.net/jdk10/master jdk10
cd jdk10

# configure
# you can also use 'fastdebug' or just --enable-debug
bash configure --with-boot-jdk=/path/to/jdk9 \
    --with-debug-level=slowdebug \
    --enable-asan

# build
make images
```

Для подробностей можно запустить&nbsp;`bash configure --help`&nbsp;или посмотреть здесь:

&nbsp;[http://hg.openjdk.java.net/jdk10/master/file/5a1afee9b9e2/doc/building.md](http://hg.openjdk.java.net/jdk10/master/file/5a1afee9b9e2/doc/building.md)

После того, как&nbsp;`make` завершит свою работу, запустите&nbsp;`java -version`&nbsp;для проверки, что Java собралась успешно. Здесь нужно будет отключить обработчик SEGV. Для этого нужно&nbsp;`handle_segv=0`&nbsp;параметр для AddressSanitizer (runtime параметры AddressSanitizer ищет в переменной окружения ASAN\_OPTIONS):

```
bash-4.2$ ASAN_OPTIONS="handle_segv=0" ./build/linux-x64/images/jdk/bin/java -version
java version "10-internal"
Java(TM) SE Runtime Environment (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan)
Java HotSpot(TM) 64-Bit Server VM (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan, mixed mode)
```

Если не передать&nbsp;`handle_segv=0` , то можно увидеть такую ошибку:

```
ASAN:SIGSEGV
=================================================================
==115049==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7f2e1c5eb513 sp 0x7f2e381709d8 bp 0x7f2e3df5a5c0 T1)
#0 0x7f2e1c5eb512 (+0x512)
AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV ??:0 ??
Thread T1 created by T0 here:
#0 0x7f2e418f18aa in __interceptor_pthread_create /opt/jprt/jprtadm/erik/jdk9-dev/build/devkit/src/gcc-4.9.2/libsanitizer/asan/asan_interceptors.cc:183
#1 0x7f2e41284416 in ContinueInNewThread0 (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0x1c416)
#2 0x7f2e4128006d in ContinueInNewThread (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0x1806d)
#3 0x7f2e4128467a in JVMInit (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0x1c67a)
#4 0x7f2e41275161 in JLI_Launch (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/../lib/jli/libjli.so+0xd161)
#5 0x401272 in main (/scratch/artem/jdk10_asan/build/linux-x64/images/jdk/bin/java+0x401272)
#6 0x7f2e40cc4b34 in __libc_start_main (/lib64/libc.so.6+0x21b34)
==115049==ABORTING
```

По умолчанию AddressSanitizer еще включает&nbsp;[LeakSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizerLeakSanitizer). Это инструмент для обнаружения утечек памяти, который встроен в AddressSanitizer. В настоящее время мне не совсем понятно, работает ли LeakSanitizer корректно с виртуальной машиной Hotspot. Поэтому я обычно отключаю LeakSanitizer. Для этого нужно просто передать&nbsp;`detect_leaks=0`&nbsp;через переменную окружения ASAN\_OPTIONS:

```
bash-4.2$ ASAN_OPTIONS="handle_segv=0 detect_leaks=0" ./build/linux-x64/images/jdk/bin/java -version
java version "10-internal"
Java(TM) SE Runtime Environment (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan)
Java HotSpot(TM) 64-Bit Server VM (slowdebug build 10-internal+0-2017-10-30-1135083.asmotrak.jdk10asan, mixed mode)
```

Подробности о параметрах AddressSanitizer можно найти здесь:

[https://github.com/google/sanitizers/wiki/SanitizerCommonFlags](https://github.com/google/sanitizers/wiki/SanitizerCommonFlags)

## Запуск Java тестов вместе с AddressSanitizer

Если вы участвуете в разработке OpenJDK, и ваши изменения касаются кода на С/C++, то вам может быть полезно запускать существующие тесты для OpenJDK с включенным AddressSanitizer. Для этого нужно просто собрать OpenJDK с вашими изменениями и запустить `make test`. Эта команда запустит все существующие автоматические тесты. Если вам необходимо запустить только какое-то подмножество тестов, то можно использовать переменную TEST, например,&nbsp;`make run-test TEST=jdk_lang`. Еще можно просто запустить необходимые тесты с помощью Jtreg (это test harness, который используется для тестирования OpenJDK). Подробности можно найти на&nbsp;[Testing OpenJDK](http://hg.openjdk.java.net/jdk10/master/file/5a1afee9b9e2/doc/testing.md)

## Известные проблемы

Код на Java, который запускает новые процессы с помощью ProcessBuilder может вызвать следующую ошибку если включен AddressSanitizer:

```
Address 0x7fceacab7280 is located in stack of thread T24 (MainThread)==10175==AddressSanitizer CHECK failed: ../../../../src/libsanitizer/asan/asan_thread.cc:231 "((ptr[0] == kCurrentStackFrameMagic)) != (0)" (0x0, 0x0)
```

Происходит это. потому что Process API использует вызовы fork() and vfork(), в тоже время виртуальная машина Hotspot использует Threads API. Похоже, что AddressSanitizer сходит от этого с ума, подробности можно почитать здесь:

[https://github.com/google/sanitizers/issues/255](https://github.com/google/sanitizers/issues/255)

## Ограничения

Несмотря на то, что&nbsp;AddressSanitizer является хорошо себя зарекомендовавшим инструментом по отлову переполнения буферов и прочих безобразий, это вовсе не серебряная пуля. Все равно надо использовать мозг при написании C/C++ кода и стараться избегать переполнения буферов, утечек памяти и прочих ошибок, которые могут возникнуть с работой с памятью. Уже в дополнение к этому было бы неплохо вооружиться хорошими инструментами, такими как runtime memory checkers и всевозможными статическими анализаторами кода.

## Что дальше?

Гриша Штойк заметил, что было бы здорово, если бы проверки AddressSanitizer'а вставлялись и в код, который генерирует Java runtime компилятор. Я не эксперт в runtime компиляции, поэтому спросил об это Игоря Игнатьева из&nbsp;[Hotspot Group](http://openjdk.java.net/census#hotspot). Игорь сказал, что это будет сложно реализовать в C1/C2 компиляторах, но должно быть проще с компилятором Graal.

