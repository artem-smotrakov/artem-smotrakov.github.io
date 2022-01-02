---
layout: post
title: 'Что нового в Java 10: часть первая'
date: 2018-02-14 18:49:06.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Tech
tags:
- GC
- Hotspot
- Java
- JEP
- JVM
permalink: "/ru/tech-ru/what-is-new-in-java-10-ru.html"
---
Java 10 должна увидеть свет в марте 2018. Это будет следующий большой short-term release после Java 9, которая была выпущена недавно в сентябре 2017.

После того как конце 2017-го года я покинул Java Team в Oracle и переехал на другую сторону Атлантического океана, я обещал себе под Новый Год, что буду стараться следить за тем, что происходит с Javа. Ну хотя бы в 2018-м году.

Так как Java 10 уже не за горами, то самое время взглянуть на JEP'ы (Java Enhancement Proposal), которые [запланированы в Java 10](http://openjdk.java.net/projects/jdk/10). Вот краткий обзор больших изменений в Java 10. Наслаждайтесь!

[English version is here.](/en/tech/what-is-new-in-java-10-episode-one.html)

&nbsp;



## Local-Variable Type Inference

Этого нововведение в языке Java. Оно подразумевает добавление возможности не указывать тип инициализированной переменной. Вот, что нам говорят авторы на прекрасном английском языке:

> We seek to improve the developer experience by reducing the ceremony associated with writing Java code, while maintaining Java's commitment to static type safety, by allowing developers to elide the often-unnecessary manifest declaration of local variable types. This feature would allow, for example, declarations such as:
> 
> ```
> var list = new ArrayList<String>(); // infers ArrayList<String>
> var stream = list.stream(); // infers Stream<String>
> ```
> 
> This treatment would be restricted to local variables with initializers, indexes in the enhanced&nbsp;for-loop, and locals declared in a traditional&nbsp;for-loop; it would not be available for method formals, constructor formals, method return types, fields, catch formals, or any other kind of variable declaration.

Если коротко, то предлагается ввести в язык Java новое ... ну не совсем ключевое слово ... и даже не зарезервированное слово ... а нечто, что будет называться "var". Предоставим слово авторам новинки:

> The identifier&nbsp;`var`&nbsp;is not a keyword; instead it is a&nbsp;_reserved type name_&nbsp;or a&nbsp;_context-sensitive keyword_. This means that code that uses&nbsp;`var`&nbsp;as a variable, method, or package name will not be affected; code that uses&nbsp;`var`&nbsp;as a class or interface name will be affected (but these names are rare in practice, since they violate usual naming conventions).

Основной вопрос в переносимости кода. Вы же не хотите, чтобы ваше приложение сломалось при переезде на Java 10 только потому, что вы где-то в коде используете слово "var"? И авторы любезно сделали это слово зависимым от контекста. Если ваш код уже использует слово "var" в качестве переменной, имени метода или пакета, то ничего плохого с вами не должно случиться. Ваш код должен компилироваться и работать как прежде. Но вот если ваш код вдруг использует слово "var" в качестве имени класса или интерфейса, то вам придется отказаться от такой роскоши. Авторы ссылаются на то, что подобное наименование классов попирает всяческие нормы написания кода, и вообще полное "кю".

Это нововведение, конечно, выглядит куда более скромно, чем, например, те же самые лямбды. Но все-таки кто-то мог ждать этого давно. А кто-то может быть все еще живет с мечтами о том, что нас когда-нибудь избавят от&nbsp;`public static void main(String[] args)` ...&nbsp; Может быть пора сделать для этого JEP?

Еще авторы напоминают, что следует использовать мозг во время написания кода:

> Like any other language feature, local variable type inference can be used to write both clear and unclear code; ultimately the responsibility for writing clear code lies with the user.

Подробности в&nbsp;[JEP 286](http://openjdk.java.net/jeps/286).

## Consolidate the JDK Forest into a Single Repository

Честно говоря, сложно назвать это чем-то полезным для пользователей Java, однако этот проект реализован в рамках JEP процесса, так что&nbsp; технически это "feature". Речь здесь идет о перелопачивании исходников JDK, что скорее всего будет интересно лишь тем, кто принимает участие в разработке JDK. Авторы JEP'а совместно с многими другими перемещают туда-сюда куски JDK, что-то переименовывают, объединяют Mercurial репозитории и так далее. Вряд ли это как-то затронет пользователей JDK. Надеемся лишь, что всякие переименования и перемещения не внесут дополнительных регрессий.

Предоставим слово самим зачинщикам:

> Combine the numerous repositories of the JDK forest into a single repository in order to simplify and streamline development.

Подробности в&nbsp;[JEP 296](http://openjdk.java.net/jeps/296), если вы вдруг обнаружили себя в этом [списке](http://openjdk.java.net/census).

## Garbage-Collector Interface

Это опять же не сильно походит на нечто полезное для пользователей Java. Основная цель этого JEP'а это упростить жизнь тем, кто хочет добавить в JVM свой собственный сборщик мусора с Го и гейшами. Кто интересуется реализацией сборки мусора? Этот долгожданный JEP для вас.

Что говорят эксперты:

> Improve the source code isolation of different garbage collectors by introducing a clean garbage collector (GC) interface.

А еще эксперты преследуют следующие цели:

> - Better modularity for HotSpot internal GC code
> - Make it simpler to add a new GC to HotSpot without perturbing the current code base
> - Make it easier to exclude a GC from a JDK build

В рамках этого JEP'а авторы хотят получше изолировать код, который относится к сборке мусора, и сделать более понятным интерфейс самих сборщиков мусора. Скорее всего почитать [подробности](http://openjdk.java.net/jeps/304) будет интересно тем, кто принимает участие в разработке JVM.

## Parallel Full GC for G1

Кто любит убираться дома? Не уверен, что таких найдется много. Представим, что мы должны делать генеральную уборку каждые выходные. И ведь это запросто может занять несколько часов, а то и половину нашего заслуженного выходного. И это ужасно. Я стараюсь избегать таких генеральных уборок и мне кажется, что большинство на моей стороне. Но к сожалению наши жилища все равно покрываются пылью, а то и грязью, они захламляются, и в результате генеральная уборка становится неизбежна. Но нам скучно делать это грязное дело в одиночку, и мы слезно просим окружающих помочь нам в этом нелегком деле.

Hotspot JVM ведет себя примерно так же. У JVM есть сборщик мусора под псевдонимом G1. Товарищ G1 изо всех сил пытается уклониться от масштабных сборок мусора, но иногда они все-таки неизбежны. когда память уже на исходе, и добыть ее другими более щадящими способами не представляется возможным. Тогда товарищ G1 пускается во все тяжкие и начинает генеральную уборку. Проблема заключается в том, что делает он это всегда в гордом одиночестве, а именно в одном потоке. И вот теперь в Java 10 товарищ G1 будет обучен собирать мусор в нескольких потоках, а еще авторы этого нововведения любезно разрешат нам поиграться с количеством потоков с помощью параметра&nbsp;`-XX:ParallelGCThreads`.

Кстати, сборщик мусора под кодовым названием G1 начиная с Java 9 является почетным сборщиком мусора по умолчанию.

К нашему величайшему сожалению,&nbsp;[JEP 307](http://openjdk.java.net/jeps/307) не балует нас обилием подробностей.

Будем надеяться, что мусор будут выносить вовремя.

## Application Class-Data Sharing

Знаете ли вы, что такое CDS? Если да, то можете смело пропускать этот абзац. CDS означает Class-Data Sharing. Да, речь пойдет о class-файлах. CDS позволяет нам выбрать подмножество классов, подвергнуть их специальной обработке и упаковать в специальный архив. Но что это за специальная обработка? - спросит внимательный и дотошный читатель. Дело в том, что JVM производит множество магических операций во время загрузки класса. Например, JVM разбирает класс по косточкам, сохраняет его содержимое у себя в закромах, придирчиво выполняет над классом разные проверки, ищет зависимости и так далее. Только после всего этого долгожданный класс готов к использованию. Как вы уже могли догадаться, все эти манипуляции занимают время. Более того, каждый экземпляр JVM, как правило, загружает один и тот же набор классов, в который обычно входят такие классы как String, Integer, URLConnection и многие другие, которые входят в стандартную библиотеку Java. И как вы можете догадаться, все эти классы требуют памяти. И вот когда у нас есть тот самый специальный волшебный архив классов, то мы можем его поместить в память и попросить разные экземпляры JVM его совместно использовать. Такой социализм и коллективизация на уровне JVM может уменьшить время загрузки приложений и сэкономить память.

Хорошо, это все очень даже замечательно, только вот CDS у нас есть уже давным-давно, а именно с момента выпуска Java 5. Но давайте повнимательней посмотрим на название JEP'а -&nbsp;"Application Class-Data Sharing". При ближайшем рассмотрении названия мы&nbsp; видим, что помимо&nbsp;"Class-Data Sharing" там еще затесалось словечко&nbsp;"Application". Дело в том, что в настоящее время CDS работает лишь для bootstrap class loader. "Application CDS" или в простонародье "AppCDS" призван&nbsp;расширить возможности Class-Data Sharing и устранить дискриминацию между class loader'ами путем дозволения другим class loader'ам загружать эти самые специальные архивы с классами. Вот какие class loader'ы у нас есть:

- built-in system class loader
- built-in platform class loader
- custom class loaders

Авторы упоминают некоторый анализ, который был проведен над large-scale enterprise applications и даже над serverless cloud services. По словам авторов, этот анализ показывает, что AppCDS должен всячески им помочь в нелегком деле экономии памяти и сокращении времени старта:

> Analysis of the memory usage of large-scale enterprise applications shows that such applications often load tens of thousands of classes into the application class loader. Applying AppCDS to these applications will result in memory savings of tens to hundreds of megabytes per JVM process.
> 
> Analysis of serverless cloud services shows that many of them load several thousand application classes at start-up. AppCDS can allow these services to start up quickly and improve the overall system response time.

Звучит многообещающе. Посмотрим, как оно будет работать.

Авторы хорошо поработали и предоставили нам не только прекрасное описание новинки, но и примеры использования. Подробности в [JEP 310](http://openjdk.java.net/jeps/310).

## Thread-Local Handshakes

Знаетели вы, что такое JVM safepoint? Хотя возможно, что здоровее и полезней этого не знать ... Если кратко, то JVM safepoint означает остановку маленького мира JVM, когда останавливаются все потоки, которые выполняют Java-код. Это необходимо для того, чтобы мы имели эксклюзивный доступ к закромам JVM. Safepoint'ы используются для разных операций, например, сборки мусора, деоптимизации кода (и такое случается ...), biased lock revocation и так далее.

Если кратко, то целью этого нововведения под названием "Thread-Local Handshakes" является сокращение количества этих самых JVM&nbsp; safepoint'ов. Вот, что нам пишут авторы:

> Introduce a way to execute a callback on threads without performing a global VM safepoint. Make it both possible and cheap to stop individual threads and not just all threads or none.

Довольно просто, не так ли? Но вот следующая цитата из JEP'а может уже взорвать мозг (пожалуйста, уберите от экранов детей и впечатлительных животных):

> A handshake operation is a callback that is executed for each JavaThread while that thread is in a safepoint safe state. The callback is executed either by the thread itself or by the VM thread while keeping the thread in a blocked state. The big difference between safepointing and handshaking is that the per thread operation will be performed on all threads as soon as possible and they will continue to execute as soon as it’s own operation is completed. If a JavaThread is known to be running, then a handshake can be performed with that single JavaThread as well.
> 
> In the initial implementation there will be a limitation of at most one handshake operation in flight at a given time. The operation can, however, involve any subset of all JavaThreads. The VM thread will coordinate the handshake operation through a VM operation which will in effect prevent global safepoints from occurring during the handshake operation.
> 
> The current safepointing scheme is modified to perform an indirection through a per-thread pointer which will allow a single thread's execution to be forced to trap on the guard page. Essentially, at all times there will be two polling pages: One which is always guarded, and one which is always unguarded. In order to force a thread to yield, the VM updates the per-thread pointer for the corresponding thread to point to the guarded page.

С ума сойти ...&nbsp; но если этого вам мало, то добро пожаловать в&nbsp;&nbsp;[JEP 312](http://openjdk.java.net/jeps/312)

[Продолжение следует ...](/ru/tech-ru/what-is-new-in-java-10-part-two.html)

P.S. Если я написал какую-нибудь глупость, то буду признателен, если вы об этом сообщите :)

