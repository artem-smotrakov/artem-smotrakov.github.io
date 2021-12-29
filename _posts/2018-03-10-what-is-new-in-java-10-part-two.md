---
layout: post
title: 'Что нового в Java 10: часть вторая'
date: 2018-03-10 18:33:42.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tech
tags:
- Hotspot
- Java
- JEP
- JVM
meta:
  _edit_last: '1'
  _aioseop_opengraph_settings: 'a:15:{s:32:"aioseop_opengraph_settings_title";s:55:"Что
    нового в Java 10: часть вторая";s:31:"aioseop_opengraph_settings_desc";s:231:"Java
    10 уже не за годами, поэтому самое время посмотреть, что в ней будет. от краткий
    обзор больших изменений в Java 10. Наслаждайтесь!";s:32:"aioseop_opengraph_settings_image";s:71:"https://blog.gypsyengineer.com/wp-content/uploads/2016/08/java_logo.png";s:36:"aioseop_opengraph_settings_customimg";s:0:"";s:37:"aioseop_opengraph_settings_imagewidth";s:0:"";s:38:"aioseop_opengraph_settings_imageheight";s:0:"";s:32:"aioseop_opengraph_settings_video";s:0:"";s:37:"aioseop_opengraph_settings_videowidth";s:0:"";s:38:"aioseop_opengraph_settings_videoheight";s:0:"";s:35:"aioseop_opengraph_settings_category";s:7:"article";s:34:"aioseop_opengraph_settings_section";s:0:"";s:30:"aioseop_opengraph_settings_tag";s:0:"";s:34:"aioseop_opengraph_settings_setcard";s:7:"summary";s:44:"aioseop_opengraph_settings_customimg_twitter";s:0:"";s:44:"aioseop_opengraph_settings_customimg_checker";s:1:"0";}'
  _aioseop_description: Java 10 уже не за годами, поэтому самое время посмотреть,
    что в ней будет. от краткий обзор больших изменений в Java 10. Наслаждайтесь!
  _aioseop_title: 'Что нового в Java 10: часть вторая'
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '261'
  _yoast_wpseo_focuskw_text_input: Java 10
  _yoast_wpseo_focuskw: Java 10
  _yoast_wpseo_linkdex: '83'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_metadesc: В новой версии Java нас ждет довольно много улучшений в JVM.
    Помимо улучшений языка и виртуальной машины Java 10 содержит еще одно нововведение,
    которое совместно с новой моделью релизов Java наделало много шуму в сообществе
    любителей Java. Представляем вашему вниманию обзор больших изменений в Java 10.
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:0;s:7:"replies";i:0;s:7:"authors";i:0;s:14:"recent_authors";a:0:{}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618039045'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/ru/tech-ru/what-is-new-in-java-10-part-two.html"
---
Java 10 уже не за горами, а именно её обещают выпустить уже совсем скоро в марте. В новой версии Java нас ждет довольно много улучшений в JVM. Но кажется, что все в основном заинтересованы лишь в одном нововведении в самом языке, а именно [новом слове "var"](http://openjdk.java.net/jeps/286). Помимо улучшений языка и виртуальной машины Java 10 содержит еще одно нововведение, которое совместно с [новой моделью релизов Java](https://mreinhold.org/blog/forward-faster#Proposal) наделало много шуму в сообществе любителей Java.

Итак, представляем вашему вниманию краткий обзор остатков больших изменений в Java 10, которые не были рассмотрены в [предыдущей статье](https://blog.gypsyengineer.com/ru/tech-ru/what-is-new-in-java-10-ru.html). Наслаждайтесь!

&nbsp;

<!--more-->

## Time-Based Release Versioning

Чтобы стазу стало все понятно без лишних слов, процитируем авторов&nbsp;[JEP 322](http://openjdk.java.net/jeps/322), как они определяют новый формат версий Java:

[1-9][0-9]\*((\.0)\*\.[1-9][0-9]\*)\*

На этом мне следовало бы, пожалуй, остановиться, потому что к этому изящному определению довольно сложно добавить что-то еще. Но я попробую. Данное определение допускает, что версия Java может быть произвольной длины, но нескольким первым элементам придается особенное сакральное значение. Обратимся к первоисточнику:

> `$FEATURE.$INTERIM.$UPDATE.$PATCH`
> 
> $FEATURE — The feature-release counter, incremented for every feature release regardless of release content. Features may be added in a feature release; they may also be removed, if advance notice was given at least one feature release ahead of time. Incompatible changes may be made when justified. (Formerly $MAJOR.)
> 
> $INTERIM — The interim-release counter, incremented for non-feature releases that contain compatible bug fixes and enhancements but no incompatible changes, no feature removals, and no changes to standard APIs. (Formerly $MINOR.)
> 
> $UPDATE — The update-release counter, incremented for compatible update releases that fix security issues, regressions, and bugs in newer features. (Formerly $SECURITY, but with a non-trivial incrementation rule.)
> 
> $PATCH — The emergency patch-release counter, incremented only when it's necessary to produce an emergency release to fix a critical issue. (Using an additional element for this purpose minimizes disruption to both developers and users of in-flight update releases.)
> 
> The fifth and later elements of version numbers are reserved for use by downstream consumers of the JDK code base. The fifth element may be used to, e.g., identify implementor-specific patch releases.

Коротко о главном. `$FEATURE` увеличивается на единичку в двух случаях: либо когда выходит Java с каким-то нововведением, которое можно называть "фичей", либо когда какая-то "фича" удаляется из Java (о чем полагается сообщать заблаговременно).&nbsp; `$INTERIM` увеличивается для версий Java, которые содержат изменения меньшего калибра, которые не должны повлечь проблем с совместимостью. `$UPDATE` увеличивается на единичку всякий раз, когда выходит новая Java с починенными регрессиями, проблемами с безопасностью и дефектами в новых "фичах".

Такой форма версий удовлетворяет [новой модели релизов Java](https://mreinhold.org/blog/forward-faster#Proposal), которая вызвала много бурлений. Вот что авторы говорят по этому поводу:

> Under the six-month release model the elements of version numbers vary as follows:
> 
> $FEATURE is incremented every six months: The March 2018 release is JDK 10, the September 2018 release is JDK 11, and so forth.
> 
> $INTERIM is always zero, since the six-month model does not include interim releases. We reserve it here for flexibility, so that a future revision to the release model could include such releases and say that JDK $N.1 and JDK $N.2 are compatible upgrades of JDK $N. As examples, the JDK 1.4.1 and 1.4.2 releases were, in essence, interim releases, and would have been numbered 4.1 and 4.2 under this scheme.
> 
> $UPDATE is incremented one month after $FEATURE is incremented, and every three months thereafter: The April 2018 release is JDK 10.0.1, the July release is JDK 10.0.2, and so forth.

Новый форма версий также влечет за собой следующие изменения:

- новые значения, которые печатает `java -version`
- добавлены две новые system properties `java.version.date`&nbsp;и&nbsp;`java.vendor.version`
- обновлен Runtime.Version API

Авторы [JEP 322](http://openjdk.java.net/jeps/322) предоставляют нам хорошее описание нововведения, поэтому добро пожаловать всем желающим.

## Remove the Native-Header Generation Tool (javah)

Кто запускал команду&nbsp;`ls ${JAVA_HOME}/bin`? Результаты могут удивить и даже шокировать. В&nbsp;`bin`&nbsp;можно обнаружить солидный набор инструментов, которые входят в состав JDK. Один из них называется&nbsp;`javah`, который используется для генерации JNI header'ов, если у вас в коде есть native-методы in your code. Кто использует JNI?

Однако, генерирование JNI header'ов уже давно поддерживает компилятор&nbsp;`javac`, а именно начиная с Java 8. `javah`&nbsp;выглядит не совсем нужным в хозяйстве, поэтому его удаляют. Кстати, существует&nbsp;[Project Panama](http://openjdk.java.net/projects/panama/),&nbsp;который теоретически сможет не только заменить JNI в будущем, но и добавить еще много чего интересного. Ознакомьтесь на досуге.

[Авторы JEP-а 313 немногословны.](http://openjdk.java.net/jeps/313)

## Additional Unicode Language-Tag Extensions

Знаете ли вы, что такое языковой тег (language tag)? Уверен, что многие из нас уже когда-то видели их. Например,&nbsp;`en-GB`&nbsp;и&nbsp;`en-US` это языковые теги. Как вы уже успели догадаться, тег может состоять из нескольких элементов. Например,&nbsp;`en` обозначает английский язык, а `GB` и `US` обозначают регион. Таким образом, эти теги обозначают британский и американский языки соответственно. Языковые теги определены в документах под кодовым названием BCP 47 и в нескольких RFC.

Довольно просто, правда? Но как всегда надо было все усложнить. Языковой тег может содержать более двух элементов. Один тип элементов называется "extension subtag", который призван нести дополнительную информацию о теге, которая может даже не идентифицировать язык. Например, extension subtag может содержать информацию о календаре или часовом поясе.&nbsp;Common Locale Data Repository (CLDR) приводит длинный список таких атрибутов. Для элементов с информацией о locale даже выделили специальное название "Extension U". Вот пара примеров:

- `th-TH-u-ca-buddhist` обозначает тайский язык (th) в Таиланде (TH) с буддистским календарем (buddhist)
- `th-TH-u-ca-buddhist-nu-thai` обозначает тайский язык (th) в Таиланде (TH) с буддистским календарем (buddhist) и тайскими числами (nu-thai)

А как обстоят дела с языковыми тегами в Java? Предоставим слово авторам JEP-а:

> Support for BCP 47 language tags was was initially added in Java SE 7, with support for the Unicode locale extension limited to calendars and numbers. This JEP will implement more of the extensions specified in the latest LDML specification, in the relevant JDK classes.

Другими словами, следующие extensions будут добавлены в Java 10:

- cu (денежная еденица)
- fw (первый день недели)
- rg (регион)
- tz (часовое пояс)

JEP подразумевает обновление некоторых классов в пакетах&nbsp;`java.text`, `java.time` и `java.util`. Подробности в&nbsp;[JEP 314](http://openjdk.java.net/jeps/314), хотя их там не очень много.

## Heap Allocation on Alternative Memory Devices

Оказывается, существуют разные другие виды памяти помимо DRAM. Например,&nbsp;[NV-DIMM](https://en.wikipedia.org/wiki/NVDIMM), который может даже пережить отключение электричества и сохранить свое содержимое. И если такой тип памяти существует, должен же кто-то разместить на нем heap, правда? Начиная с Java 10, Hotspot JVM будут уметь это делать. Вот что нам говорят авторы нововведения:

> With the availability of cheap NV-DIMM memory, future systems may be equipped with heterogeneous memory architectures. One example of such technology is Intel's 3D XPoint. Such an architecture, in addition to DRAM, will have one or more types of non-DRAM memory with different characteristics.
> 
> This JEP targets alternative memory devices that have the same semantics as DRAM, including the semantics of atomic operations, and can therefore be used instead of DRAM for the object heap without any change to existing application code. All other memory structures such as the code heap, metaspace, thread stacks, etc., will continue to reside in DRAM.

Авторы сообщают нам, что некоторые операционные системы уже предоставляют возможность работы с отличными от DRAM видами памяти. Например,&nbsp;[NTFS DAX mode](https://channel9.msdn.com/events/build/2016/p470)&nbsp;и&nbsp;[ext4 DAX](https://lwn.net/Articles/618064). Доступ к памяти организован через файл. JEP обещает предоставить нам возможность размещать heap в такой памяти. Другими словами, совсем скоро станет возможным разместить heap на жестком диске. Скорее всего это будет медленнее, но будут и плюшки ведь содержимое жесткого диска (обычно) сохраняется после выключения питания.&nbsp; Авторы собираются добавить новый параметр для JVM&nbsp;`-XX:AllocateHeapAt=<path>`, с помощью которого можно будет указать путь к файлу, где надо разместить heap.

Авторы JEP-а говорят, что остальные опции для JVM, такие как&nbsp;`-Xmx`&nbsp;и`-Xms`, будут работать без изменений.

За деталями обращайтесь к&nbsp;[JEP 316](http://openjdk.java.net/jeps/316), хотя опять JEP не балует нас обилием подробностей.

## Experimental Java-Based JIT Compiler

Знакомы ли вы с таким явлением, как runtime compilation? Если да, то вероятно вы можете пропустить этот параграф. Дело вот в чем. После того, как вы написали свой изумительный код на Java, вам необходимо его немедленно скомпилировать, чтобы потом сразу же и запустить. Обычно вы используете для компиляции `javac`, который преобразует вашу программу на Java в набок класс-файлов, в которых будет содержаться Java байт-код. Далее вы запускаете вашу ненаглядную программу командой `java`&nbsp;, которая стартует JVM и запускает там ваш байт-код. А что происходит дальше? У JVM есть интерпретатор, который понимает инструкции байт-кода и преобразует из в инструкции процессора. Но помимо этого JVM также содержит свой собственный компилятор, который может разом преобразовать байт-код в инструкции процессора. Это явление и называется "runtime compilation". Скомпилированный таким образом код может потом запускаться быстрее. JVM может использовать интерпретатор байт-кода для выполнения Java программ, но некоторые участки байт-кода могут быть заранее скомпилированы. Для лучшего понимания какой байт-код следует скомпилировать, компилятор собирает статистику, какие кусочки байт-кода чаще используются. Самый популярный байт-код обычно подлежит компиляции. Компилятор в процессе компиляции может пытаться оптимизировать код, что обычно позволяет чуточку ускорить выполнение Java-программы. Однако, иногда компилятор может специально и деоптимизировать код. У компилятора обычно есть несколько стратегий. Одна из них называется JIT-компиляция, где "JIT" означает "just-in-time". Это означает, что компиляция выполняется прямо во время выполнения программы. Подводя итоги, мы можем догадаться, что внутренний компилятор виртуальной машины это довольно важный компонент, и обычно он довольно сложно устроен.

В настоящее время у Hotspot JVM есть два компилятора, которые имеют кодовые имена C1 и C2. Если кратко, то C1 это относительно быстрый и простой компилятор, который не делает много оптимизаций кода. Он хорош для приложений, которые недолго работают, и для которых важно быстро стартовать. C2 наоборот больше пытается оптимизировать код. Он больше подходит для долгоиграющих серверных приложений. Оба являются JIT компиляторами, и оба реализованы на C/C++/Asm. Кстати, C2 даже содержит свой собственный внутренний язык для описания оптимизаций кода. C2 вообще довольно сложный и тяжеловесный компонент Hotspot JVM.

Но в мире существуют и другие компиляторы Java байт-кода. Один из них называется&nbsp;[Graal](https://github.com/oracle/graal). Интересно, что Грааль написан на Java. JEP, о котором мы говорим в этом разделе, включает Грааль в состав Hotspot JVM в качестве экспериментального JIT-компилятора на платформе Linux-x64. Предоставим слово авторам JEP-а:

> Enable Graal to be used as an experimental JIT compiler, starting with the Linux/x64 platform. Graal will use the JVM compiler interface (JVMCI) introduced in JDK 9. Graal is already in the JDK, so enabling it as an experimental JIT will primarily be a testing and debugging effort.

Если вы хотите принять участие в эксперименте, то можете начать запускать ваши приложения с опциями&nbsp;&nbsp;`-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler`. Я думаю, что разработчики Hotspot/Graal будут рады вашему участию, если вы поделитесь с ними своим опытом.

За деталями обращайтесь к&nbsp;[JEP 317](http://openjdk.java.net/jeps/317) , который уже традиционно не радует нас их обилием.

## Root Certificates

Знакомы ли вы с цифровыми сертификатами и удостоверяющими центрами? Если да, то следующие пара параграфов могут быть не слишком интересными для вас. В асимметричных шифрах у нас есть пара ключей: открытый (public) и закрытый (private). Открытый ключ известен всем, но закрытый ключ держится в секрете. Ключи могут быть использованы, например, для шифрования или цифровой подписи. Цифровой сертификат содержит следующее:

- открытый ключ
- техническую информацию о ключе (например, его длину)
- информацию о владельце ключа
- цифровую подпись для информации выше

А кто ставит эту самую подпись? Предположим, что у нас есть открытый ключ&nbsp;`PublicKey1` и закрытый ключ&nbsp;`PrivateKey1`. Чтобы подписать наш сертификат для ключа&nbsp;`PublicKey1`, нам еще одна пара ключей. Назовем их&nbsp;`PublicKey2` и `PrivateKey2`&nbsp;соответственно. Закрытый ключ&nbsp;`PrivateKey2`&nbsp; используется для создания подписи, а открытый ключ&nbsp;`PublicKey2` для ее проверки. Ключ&nbsp;`PublicKey1` вместе с информацией выше подписывается ключом&nbsp;`PrivateKey2`&nbsp;, и подпись включается в сертификат. Теперь у нас есть подписанный сертификат&nbsp;`C1` для открытого ключа&nbsp;`PublicKey1`. Подпись может быть проверена ключом&nbsp;`PublicKey2`. Как вы уже могли догадаться, в природе должен существовать сертификат также и для&nbsp;`PublicKey2`. Но кто подписал этот сертификат для ключа&nbsp;`PublicKey2`? Для этого нужна еще одна пара ключей ... Так мы приходим к цепочке сертификатов.

К конечном счете, одной из главных целей создания цифровых сертификатов является возможность проверить, что открытый ключ из сертификата `C1` действительно принадлежит владельцу, который указан в сертификате. Проверка эта основана на цифровой подписи: пользователь может взять открытый ключ из следующего в цепочке сертификата `C2`&nbsp;и использовать его для проверки подписи в сертификате `C1`. Но потом нам необходимо также проверить подпись в сертификате `C2`. И мы берем следующий сертификат `C3`... Но ведь это же не может продолжаться бесконечно? Когда мы останавливаемся? И вот мы приходим к доверенным сертификатам. У нас может быть список сертификатов, которым мы доверяем. Такие сертификаты в быту еще называют корневыми или "root certificates". Мы останавливаем проверку цепочки сертификатов, когда наткнемся на сертификат из нашего списка доверенных сертификатов.

Обычно цифровой сертификат выпускается удостоверяющим центром (Certificate Authority или просто CA). Digicert, Comodo, VeriSign это примеры известных удостоверяющих центров. У удостоверяющих центров есть свои сертификаты, которые они используют для подписи выпускаемых ими сертификатов.

А зачем нам вообще нужны эти цепочки сертификатов? Например, сертификаты широко используются при установки защищенных TLS соединений. Да-да, когда вы видите маленький зеленый замочек в адресной строке вашего браузера напротив адреса сайта, то это означает, что ваш браузер успешно проверил цепочку сертификатов для этого сайта, и таким образом сумел установить защищенное соединение с сервером. У браузеров обычно есть свое хранилище доверенных сертификатов.

Хорошая новость: у Java тоже есть свое хранилище корневых сертификатов, которое называется&nbsp;`cacerts`. Это просто файл, который лежит в директории `${JAVA_HOME}/lib/security`.&nbsp; Но есть и плохая новость:&nbsp;`cacerts` в OpenJDK пуст. Но это досадное недоразумение наконец-то будет исправлено в Java 10.&nbsp;&nbsp;`cacerts` хранилище в OpenJDK будет содержать доверенные сертификаты из Oracle JDK. Предоставим слово авторам JEP-а:

> The cacerts keystore will be populated with a set of root certificates issued by the CAs of Oracle's Java SE Root CA Program. As a prerequisite, each CA must sign the Oracle Contributor Agreement (OCA), or an equivalent agreement, to grant Oracle the right to open-source their certificates. Below are the CAs that have signed the required agreement and, for each, a list of the root certificates (identified by the Distinguished Name) that will be included. This list includes a majority of the CAs that are currently members of Oracle's Java SE Root CA Program. Those that do not sign an agreement will not be included at this time. Those that take longer to process will be included in the next release.

И далее авторы приводят длинный список сертификатов, которые будут включены в OpenJDK. Вообще это кажется довольно простой затеей: надо было просто скопировать `cacerts`&nbsp;файл из Oracle JDK в OpenJDK.&nbsp; Но на самом деле на реализацию этого JEP-а ушло много трудов, ведь чтобы включить сертификат в OpenJDK, нужно получить подтверждение у соответствующего удостоверяющего центра, что он не против этого. Даже если этот сертификат и так находится в публичном доступе.

За деталями обращайтесь к&nbsp;[JEP 319](http://openjdk.java.net/jeps/319).

## Заключение

Наконец-то мы обсудили все основные нововведения в Java 10. Если кому-то интересно, то первая половина описана в [предыдущей статье](https://blog.gypsyengineer.com/ru/tech-ru/what-is-new-in-java-10-ru.html). Всего в&nbsp;[Java 10 будет 12 довольно больших нововведений](http://openjdk.java.net/projects/jdk/10/). Выпуск Java 10 намечен на март 2018 года. Будем надеяться, что Java 10 не будет содержать много регрессий.

P.S. Если я написал какую-нибудь глупость, то буду признателен, если вы об этом сообщите :)

