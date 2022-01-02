---
layout: post
title: Что нового в Java 12
date: 2019-03-03 16:03:26.000000000 +00:00
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
permalink: "/ru/tech-ru/what-is-new-in-java-12-ru.html"
---


В Java 12 содержится в половину меньше больших нововведений в сравнении с предыдущей версией: в Java 12 засунули [восемь](https://openjdk.java.net/projects/jdk/12/) JEPов, а в Java 11 из было аж целых [семнадцать](https://openjdk.java.net/projects/jdk/11/). Как вы конечно же помните, JEP означает JDK Enhancement Proposal. Иными словами, это единица измерения больших изменений в JDK. Что касается малых изменений в Java, то их количество в Java 11 и 12 не слишком сильно отличается: Jira говорит нам, что в Java 11 из было порядка [2700](https://bugs.openjdk.java.net/browse/JDK-8218034?jql=project%20%3D%20JDK%20AND%20resolution%20in%20(Fixed%2C%20Delivered)%20AND%20fixVersion%20%3D%20%2211%22), а в Java 12 их около [2400](https://bugs.openjdk.java.net/browse/JDK-8219012?jql=project%20%3D%20JDK%20AND%20resolution%20in%20(Fixed%2C%20Delivered)%20AND%20fixVersion%20%3D%20%2212%22). Мы же сосредоточимся только на больших новинках.





(English version is [here](/tech/what-is-new-in-java-12.html))



  
  




## JEP 189: Shenandoah: A Low-Pause-Time Garbage Collector (Experimental)





Если присмотреться, то можно заметить, что последние версии Java содержат в себе довольно много измерений в JVM и особенно в том, что касается сборки мусора (garbage collection ну или просто GC). JEP 189 это как раз из этой оперы. Этот JEP добавляет новый алгоритм сборки мусора, имя которому Shenandoah. Как можно догадаться из названия JEPа, основная особенность этого нового алгоритма это уменьшение пауз при сборке мусора. Посмотрим, что нам об этом говорят авторы этого JEPа:





> ... reduces GC pause times by doing evacuation work concurrently with the running Java threads. Pause times with Shenandoah are independent of heap size, meaning you will have the same consistent pause times whether your heap is 200 MB or 200 GB.





А вот еще одна полезная цитата для лучшего понимания:





> Shenandoah is an appropriate algorithm for applications which value responsiveness and predictable short pauses. The goal is not to fix all JVM pause issues. Pause times due to reasons other than GC like Time To Safe Point (TTSP) issues or monitor inflation are outside the scope of this JEP.





Если коротко, то новый алгоритм пытается сократить паузы, однако не все и не всегда. Еще заявляется, что работа алгоритма не зависит от размера кучи. Подробности и ссылку на полное описание алгоритма можно найти в самом [JEP 189](https://openjdk.java.net/jeps/189). Новый алгоритм сборки мусора не включается по умолчанию. Тем, кому хочется поиграться с новым сборщиком мусора, необходимо скормить JVM пару дополнительных параметров, а именно:





`-XX:+UnlockExperimentalVMOptions -XX:+UseShenandoahGC`





## JEP 230: Microbenchmark Suite





JEP 230 добавляет набор microbenchmark в исходники OpenJDK для того, чтобы их было удобно запускать и измерять производительность. Этот JEP скорее будет интересен тем, кто принимает участие в разработке самой Java, нежели тем, кто ее просто использует. Но если вдруг вам интересно почитать о performance testing в Java и о Java Microbenchmark Harness (JMH) в частности, то добро пожаловать в [JEP 230](https://openjdk.java.net/jeps/230).





## JEP 325: Switch Expressions (Preview)





Это уже должно быть поинтереснее для рядовых пользователей Java, потому как здесь дело пойдет об изменения в самом языке Java. JEP 325 добавляет слегка упрощенные `switch`. Давайте сразу перейдем к примерам:





```
static void howMany(int k) {
    switch (k) {
        case 1 -> System.out.println("one");
        case 2 -> System.out.println("two");
        case 3 -> System.out.println("many");
    }
}
```





Иными словами, если мы попали в один case, то будет выполнена лишь одна инструкция справа от стрелки, а следующий `case` выполнен не будет несмотря на отсутствие `break`. Вот еще один пример:





```
static boolean isWeekend(String day) {
    return switch (day) {
        case "mon", "tue", "wed", "thu", "fri" -> false;
        case "sat", "sun" -> true;
        default -> throw new IllegalArgumentException("oops!");
    };
}
```





Здесь мы видим, что `case` может содержать несколько меток, а также выражение справа от стрелки может означать возвращаемое значение. Довольно удобно, не так ли? Ну и последний пример:





```
static boolean isWeekend(String day) {
    return switch (day) {
        case "mon", "tue", "wed", "thu", "fri" -> false;
        case "sat", "sun" -> true;
        default -> {
            if (day.startsWith("s")) {
                System.out.println("looks like weekend");
                break true;
            }

            System.out.printf("unknown day: %s%n", day);
            break false;
        }
    };
}
```





Иными словами, если нужно больше кода для `case` или `default`, то его всегда можно запихать в фигурные скобки. А еще `break` может вести себя как `return` и возвращать значение. Нужно отметить, что все `case` и `default` в упрощенном `switch` всегда должны возвращать значение или бросать exception, иначе компилятор расстроится.





Внимательный читатель заметит слово "preview" в названии JEPа, что означает это это лишь "preview language feature", которая не включена по умолчанию. Если вам хочется поиграться с новыми switch, то необходимо передать компилятору параметры `--enable-preview --release 12` и потом запустить Java c параметром `--enable-preview`. Выглядит это примерно так:





```
$ javac -d classes --enable-preview --release 12 Test.java
$ java -classpath classes --enable-preview Test
```





Авторы хорошо описали новые switch, так что кому интересно, то все подробности в [JEP 325](https://openjdk.java.net/jeps/325). Вот небольшая цитата:





> … These changes will simplify everyday coding …





Авторы JEP 325 хотят облегчить наше ежедневное программирование. Это очень трогательно.





## JEP 334: JVM Constants API





В скомпилированных классах имеется constant pool, где операнды для bytecode инструкций. Содержимое constant pool описывает как классы и методы, так и более простые сущности как строки и числа, которые используются в классе. Все это может быть использовано в качестве операндов для инструкции `ldc`, которая загружает элементы из constant pool и делает их аргументами для последующих инструкций. Инструкция `ldc` побуждает JVM к созданию объекта, такого как String, int или даже Class, в соответствии с его статическим описанием в constant pool.





JEP 334 добавляет новый API для моделирования содержимого constant pool. Этот новый API может быть полезным для приложений и библиотек, которые позволяют манипулировать bytecode инструкциями. Таким образом, скорее всего подобное нововведение будет не так интересно большинству пользователей Java, и наверное JEP 334 не сделает наше ежедневное программирование легче.





Новый API для работы с constant pool вы всегда найдете в пакете [java.lang.constant](http://cr.openjdk.java.net/~iris/se/12/build/latest/api/java.base/java/lang/constant/package-summary.html) , а если вам интересны подробности, то ищите их в [JEP 334](https://openjdk.java.net/jeps/334).





## JEP 340: One AArch64 Port, Not Two





Возможно вы знаете, что Java может работать на ARM. Более того, в настоящее время в JDK есть аж целых две реализации поддержки ARM. Если вдруг вы знакомы со структурой исходников JDK, то вы можете посмотреть на них в директориях src/hotspot/cpu/arm и open/src/hotspot/cpu/aarch64. Такое положение вещей может показаться немного избыточным, поэтому JEP 340 удаляет все, что связано с arm64, оставляет только aarch64 поддержку 32-bit ARM. Вот что нам говорят авторы:





> Removing this port will allow all contributors to focus their efforts on a single 64-bit ARM implementation, and eliminate the duplicate work required to maintain two ports





В общем, если вы не работаете над самой Java на ARM, то маловероятно, что это JEP вам будет чем-то полезен. Авторы наверное об этом догадывались, поэтому решили сильно не трудиться над описанием [JEP 340](https://openjdk.java.net/jeps/340).





## JEP 341: Default CDS Archives





CDS означает Class-Data Sharing. CDC позволяет упаковать некоторое множество классов в некий архив, предварительно подвергнув их некоторой обработке. Зачем это нужно? JVM делает множество магических манипуляций на классами во время их загрузки. JVM разбирает класс по косточкам, убеждается, что он хороший и правильные, сохраняет его в специальном внутреннем формате, ищет всех его друзей и подруг, и часто проделывает с ними тоже самое. Только после всего этого класс считается готовым к работе. Как вы могли уже догадаться, все эти приготовления занимают некоторое ненулевое время. Более того, каждый экземпляр JVM загружает обычно обязательный набор классов, таких как String, Integer или URLConnection, и проделывает с ними одинаковые манипуляции во время загрузки. Все эти классы требуют памяти для их хранения. Тут-то и пригодится CDS: если мы можем проделать большинство рутинных операций над классами один раз и упаковать результаты в специальный архив, то это архив может быть потом загружен несколькими JVM, которым не нужно будет проделывать эти рутинные операции. В теории, да и на практике тоже, подобный подход позволяет уменьшить время запуска JVM и объем занимаемой памяти.





В настоящее время JDK содержит такой набор классов в директории `lib`. Этот набор подготавливается во время сборки JDK и содержит стандартные классы из JDK. Чтобы использовать преимущества, которые дает нам CDS с этим стандартным набором классов, нам нужно запустить &nbsp;`java -Xshare:dump`





Итак, до этого момента было введение. Теперь посмотрим, что же делает JEP 341. К счастью, тут все намного проще. Этот JEP просто добавляет вызов `java -Xshare:dump` сразу после сборки JDK, в результате чего, архив CDS создается в директории `lib/server`. Таким образом этот архив будет использоваться всегда при старте JVM. Но отключить это можно с помощью параметра &nbsp;`-Xshare:off`&nbsp;





Все подробности в [JEP 341](https://openjdk.java.net/jeps/341).





## JEP 344: Abortable Mixed Collections for G1





Следующее нововведение в Java 12 опять касается сборки мусора. Возможно вы слышали о сборщике мусора под кодовым названием G1. Как известно, этот самый G1 приостанавливает все потоки Java-приложения (что называется stop-the-world), когда ему надо собрать мусор. G1 можно также попросить не останавливать весь мир надолго, например, это можно сделать параметром &nbsp;`-XX:MaxGCPauseTimeMillis`. В этом случае G1 будет честно пытаться уложиться в заданное время.





К сожалению, у G1 это не всегда получается. Это может часто происходить, например, когда G1 пытается собрать мусор в новых и старых областях памяти.





JEP 334 призван сделать G1 еще более умным и снабжает его дополнительным механизмом по остановке сборки мусора, если вдруг она занимает слишком долгое время. Вот что нам говорят авторы на этот счет:





> If G1 discovers that the collection set selection heuristics repeatedly select the wrong number of regions, switch to a more incremental way of doing mixed collections: split the collection set into two parts, a mandatory and an optional part. The mandatory part comprises parts of the collection set that G1 cannot process incrementally (e.g., the young regions) but can also contain old regions for improved efficiency. […]  
> After G1 finishes collecting the mandatory part, G1 starts collecting the optional part at a much more granular level, if there is time left. The granularity of collection of this optional part depends on the amount of time left, at most down to one region at a time. After completing collection of any part of the optional collection set, G1 can decide to stop the collection depending on the remaining time.  
> As the predictions get more accurate again, the optional part of a collection are made smaller and smaller, until the mandatory part once again comprises all of the collection set. […] If the predictions becomes inaccurate again, then the next collections will consist of both a mandatory and optional part again.





Если коротко, то G1 теперь будет пытаться разбивать процесс сборки мусора на несколько частей, причем одна часть будет обязательной, а остальные нет. После того, как основная часть работы сделана, сборщик мусора будет проверять не истекло ли время, и если нет, то приступать к одной из необязательных частей. Идея заключается в том, что если разбить всю работу на небольшие части, то будет легче отследить момент, когда время истекло и пора останавливаться.





Все подробности в [JEP 344](https://openjdk.java.net/jeps/344) и в документации для [G1](https://docs.oracle.com/en/java/javase/11/gctuning/garbage-first-garbage-collector.html#GUID-ED3AB6D3-FD9B-4447-9EDF-983ED2F7A573).





## JEP 346: Promptly Return Unused Committed Memory from G1





G1 точно не был обделен вниманием в Java 12. В настоящее время этот сборщик мусора возвращает память обратно операционной системе в случае full garbage collection или же во время concurrent cycle (подробности об этих важных событиях в жизни сборщика мусора можно найти в документации к G1). Оба эти события могут быть не слишком уж частыми. В результате память может не освобождаться довольно долго.





JEP 346 обучает G1 отслеживать моменты, когда приложение не активно и использовать это время для того, чтобы вернуть память операционной системе. Авторы JEPа нашли очень хорошее обоснование для этого нововведения:





> This behavior is particularly disadvantageous in container environments where resources are paid by use. Even during phases where the VM only uses a fraction of its assigned memory resources due to inactivity, G1 will retain all of the Java heap. This results in customers paying for all resources all the time, and cloud providers not being able to fully utilize their hardware.





Иными слвоами, JEP 346 попросту призван сэконовиться ваши денежки. Это тоже очень мило. Можно было бы еще добавить параметр `-XX:PriceOfMegabyte`, который задавал цену за мегабайт, и потом G1 мог бы посчитать сколько он сберег благодаря [JEP 346](https://openjdk.java.net/jeps/346).





## Заключение





Большинство больших нововведений в Java 12 связаны более или менее с JVM. Ну и еще есть новый синтаксис для `switch`, однако он еще находится в экспериментальном режиме. Надеемся, что новый `switch` появится уже скоро в Java 13.





![Java]({{ site.baseurl }}/assets/images/2019/03/duke-small-1-861x1024.png)



