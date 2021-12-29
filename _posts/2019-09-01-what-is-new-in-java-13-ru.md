---
layout: post
title: Что нового в Java 13
date: 2019-09-01 13:01:19.000000000 +01:00
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
- Open Source
- OpenJDK
meta:
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '261'
  _edit_last: '1'
  _yoast_wpseo_content_score: '30'
  _yoast_wpseo_focuskw: Java 13
  _yoast_wpseo_linkdex: '71'
  _yoast_wpseo_metadesc: 'Java 13 выйдет в свет 17го сентября 2019 года. Посмотрим,
    что есть внутри: text blocks, switch expressions, новая начинка для старых Socket
    API, изменения в сборщике мусора ZGC и динамические CDS архивы.'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1618052934'
author:
  login: artem
  email: artem.smotrakov@gmail.com
  display_name: Artem
  first_name: Artem
  last_name: Smotrakov
permalink: "/ru/tech-ru/what-is-new-in-java-13-ru.html"
---
<!-- wp:paragraph -->

Java 13 выйдет в свет 17го сентября 2019 года. Помимо ~2300 мелких изменений новая версия Java содержит пять существенных нововведений, которые принято именовать JEP, что означает Java Enhancement Proposal. Посмотрим, что же это за новинки: text blocks, switch expressions, новая начинка для старых Socket API, изменения в сборщике мусора ZGC и динамические CDS архивы.

<!-- /wp:paragraph -->

<!-- wp:image {"id":2646,"align":"center","width":419,"height":498,"className":"noborder"} -->

![Что нового в Java 13]({{ site.baseurl }}/assets/images/2019/09/duke-small-861x1024.png)

<!-- /wp:image -->

<!-- wp:more -->  
<!--more-->  
<!-- /wp:more -->

<!-- wp:heading -->

## JEP 355: Text Blocks (Preview)

<!-- /wp:heading -->

<!-- wp:paragraph -->

Иногда нам бывает нужно объявить длинную строку, которая еще и может состоять из нескольких строчек. Например, это может быть кусочек HTML странички или же SQL запрос. Вот как это обычно выглядит:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
String html = "<html>\n" +
              " <body>\n" +
              " <p>Hello, world</p>\n" +
              " </body>\n" +
              "</html>\n";
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Не очень лаконично. При детальном рассмотрении можно заметить обилие символов перевода строки, плюсика, кавычек, которые делают код чуть более длинным и чуть более трудночитаемым, чем хотелось бы. В некоторых языках программирования, например, в Python, можно объявлять длинные строки с помощью тройных кавычек `"""`. Это сильно помогает делать код лучше. JEP 355 наконец-то привнесет text blocks в Java, что позволит объявлять длинные строки легко и непринужденно. Посмотрим, что сказано в самом JEPе:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> Add text blocks to the Java language. A text block is a multi-line string literal that avoids the need for most escape sequences, automatically formats the string in a predictable way, and gives the developer control over format when desired.

<!-- /wp:quote -->

<!-- wp:paragraph -->

А вот как будет выглядеть вышеприведенный пример, если переписать его с использованием text blocks:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
String html = """
              <html>
                  <body>
                      <p>Hello, world</p>
                  </body>
              </html>
              """;
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Во время выполнения эта строка будет представлена как обычный объект `String`, только выравнивающие пробелы будут удалены:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
<html>
    <body>
        <p>Hello, world</p>
    </body>
</html>
```

<!-- /wp:code -->

<!-- wp:paragraph -->

К сожалению, новый синтаксис это лишь preview language feature, который выключен по умолчанию. Чтобы его включить, нужно запустить компилятор с параметрами `--enable-preview --release 13`, а потом передать `java` параметр `--enable-preview`. Например:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
$ javac -d classes --enable-preview --release 13 Test.java
$ java -classpath classes --enable-preview Test
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Хотя текстовые блоки могут показаться довольно простым нововведением, JEP 355 содержит довольно много букв и рассуждает об отступах, игнорируемых пробелах, символах ознаменующих конец строки, некоторых особенных символах, соединение строк. Более того, там приводится описание новых методов, которые добавляет JEP 355: `String::stripIndent()`, `String::translateEscapes()` и `String::formatted(Object… args)`. Все подробности [тут](https://openjdk.java.net/jeps/355).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 354: Switch Expressions (Preview)

<!-- /wp:heading -->

<!-- wp:paragraph -->

А это небольшая добавочка к [JEP 325](https://openjdk.java.net/jeps/325), который добавил новый синтаксис для `switch` блоков в JDK 12 в качестве preview language feature. Если коротко, то JEP 325 добавил упрощенную форму для `switch` блоком, которая содержит выражения вида `case L -> ...`. Чуть побольше подробностей можно увидеть [здесь](https://blog.gypsyengineer.com/ru/tech-ru/what-is-new-in-java-12-ru.html).

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

После добавления нового синтаксиса в JDK 12, [разработчики OpenJDK решили узнать мнение общественности на этот счет](https://mail.openjdk.java.net/pipermail/jdk-dev/2019-April/002770.html). Собранное общественное мнение и вылилось в новый JEP 354, который делает лишь одно маленькое изменение в новом синтаксисе для `switch` блоков. Изначально новые `switch` блоки разрешали использовать `break` со значением, которое используется в качестве результирующего значения всего `switch` блока. Теперь же вместо `break` нужно использовать новое слово `yield`. Посмотрим, что нам говорят сами авторы этого нововведения:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> To yield a value from a switch expression, the `break` with value statement is dropped in favor of a `yield` statement.

<!-- /wp:quote -->

<!-- wp:paragraph -->

Лучше один раз увидеть, чем сто раз услышать, поэтому немедленно проиллюстрируем это примером:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
int result = switch (s) {
    case "Foo" -> 1;
    case "Bar" -> 2;
    default -> {
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
    }
};
```

<!-- /wp:code -->

<!-- wp:paragraph -->

И еще один пример:

<!-- /wp:paragraph -->

<!-- wp:code {"className":"console"} -->

```
int result = switch (s) {
    case "Foo": 
        yield 1;
    case "Bar":
        yield 2;
    default:
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
};
```

<!-- /wp:code -->

<!-- wp:paragraph -->

Новые `switch` блоки всё еще заявлены в Java 13 в качестве preview language feature, что означает, как уже говорилось ранее, что новый синтаксис не включен по умолчанию. Для его включения нужно запустить компилятор с параметрами `--enable-preview --release 13`, а потом скормить `java` параметр `--enable-preview`, как было показано в примере выше.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Вот и всё. В [JEP 354](https://openjdk.java.net/jeps/354) в основном обсуждаются те же самые темы, что и в предыдущем JEP 325.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 353: Reimplement the Legacy Socket API

<!-- /wp:heading -->

<!-- wp:paragraph -->

А этот JEP призван заместить внутренности `java.net.Socket` и `java.net.ServerSocket`. Хотя многие, может быть, и не используют эти классы явно, это нововведение вероятно коснется каждого, кто использует JDK, потому как Java sockets это основа для всех сетевых операций в Java. Поэтому JEP 353 звучит немного пугающе, если взглянуть на него повнимательнее. Вот как авторы этого JEP объясняют, почему сие необходимо:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> The java.net.Socket and java.net.ServerSocket APIs, and their underlying implementations, date back to JDK 1.0. The implementation is a mix of legacy Java and C code that is painful to maintain and debug. The implementation uses the thread stack as the I/O buffer, an approach that has required increasing the default thread stack size on several occasions. The implementation uses a native data structure to support asynchronous close, a source of subtle reliability and porting issues over the years. The implementation also has several concurrency issues that require an overhaul to address properly. In the context of a future world of fibers that park instead of blocking threads in native methods, the current implementation is not fit for purpose.

<!-- /wp:quote -->

<!-- wp:paragraph -->

Иными словами начинка `java.net.Socket` и `java.net.ServerSocket` давно уже сильно устарела, содержит разные проблемы, тяжела в поддержке и изменении, и поэтому сильно тормозит дальнейшее развитие JDK.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Стоит обратить внимание, что этот JEP не трогает `java.net.DatagramSocket`.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Если всё пойдет гладко, то большинство пользователей не должны заметить эту новинку. Если же что-то пойдет не так, то проблемы вероятно возникнут у многих. К счастью, авторы учли такой поворот и не удаляют с концами старую начинку для Java sockets. В случае необходимости можно будет переключиться обратно на старую начинку, запустив приложение с параметром `-Djdk.net.usePlainSocketImpl`. Однако, авторы обещают всё-таки удалить этот параметр вместе с устаревшей начинкой в одной из будущих версий Java. Все подробности можно найти [здесь](https://openjdk.java.net/jeps/353).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 350: Dynamic CDS Archives

<!-- /wp:heading -->

<!-- wp:paragraph -->

CDS означает Class-Data Sharing и позволяет упаковывать набор часто используемых классов в специальный архив, которые позже может быть загружен несколькими экземплярами JVM. Зачем это нужно? Дело в том, что в процессе загрузки классов JVM делает довольно много сложных вещей, таких как чтение классов, сохранение их во внутренних структурах, проверка корректности прочитанных классов, поиск и загрузка зависимых классов и так далее, и тому подобное. Только после всего этого, классы готовы к работе. Как можно догадаться, все эти процедуры требуют времени и сил. Более того, экземпляры JVM часто могут загружать одни и те же классы, например, `String`, `Integer`, `ArrayList` или же классы одного и того же приложения. Все эти классы требуют памяти. Если бы мы выполнили все необходимые процедуры один раз и поместили переработанные классы в специальный архив, которые может быть подгружен в память нескольких JVM, то это могло бы существенно сэкономить память и сократить время запуска приложения. CDS как раз и позволяет такой архив создать.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Java 9 позволяла добавлять в архив только системные классы. Java 10 уже позволяла включать в архив классы приложения. [Создание такого архива](https://openjdk.java.net/jeps/310) включает в себя два шага. Сначала необходимо создать список классов, загружаемых приложением:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=classes.lst \
     -classpath application.jar Application
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

И лишь затем создать сам архив с найденными классами:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=classes.lst \
     -XX:SharedArchiveFile=application.jsa -classpath application.jar Application
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

JEP 350 призван улучшить CDS, сделав его чуть более умным, чтобы он мог создавать архив по завершении приложения. Это означает, что два шага, приведенные выше, теперь могут быть объединены в один:

<!-- /wp:paragraph -->

<!-- wp:preformatted {"className":"console"} -->

```
java -XX:ArchiveClassesAtExit=application.jsa \
     -classpath application.jar Application
```

<!-- /wp:preformatted -->

<!-- wp:paragraph -->

Стоит заметить, что только классы, которые были загружены во время работы приложения, будут добавлены в архив. Иными словами, те не слишком везучие классы, которые все же содержаться в `application.jar`, но по каким-то причинам на были загружены, не будут добавлены в архив.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

К сожалению, архив всё еще не может быть создан автоматически после первого запуска приложения. Авторы явно об этом нам сообщают:

<!-- /wp:paragraph -->

<!-- wp:quote -->

> A follow-up enhancement to this JEP could perform automatic archive generation during the first run of an application. This would eliminate the explicit archive creation step (step 2 above). The usage of CDS/AppCDS could then be completely transparent and automatic.

<!-- /wp:quote -->

<!-- wp:paragraph -->

Также авторы предоставляют довольно много других подробностей, которые можно прочитать в [JEP 350](https://openjdk.java.net/jeps/350).

<!-- /wp:paragraph -->

<!-- wp:heading -->

## JEP 351: ZGC: Uncommit Unused Memory

<!-- /wp:heading -->

<!-- wp:paragraph -->

[The Z Garbage Collector](https://wiki.openjdk.java.net/display/zgc/Main) (ZGC) это сборщик мусора, которые всеми силами старается сократить паузы во время сборки мусора. Он героически пытается делать всю грязную работу, в то время, когда обычные потоки приложения продолжают выполнятся.

<!-- /wp:paragraph -->

<!-- wp:paragraph -->

Когда типичный сборщик мусора выполняет свои непосредственные обязанности, как правило, он должен возвратить освобожденную память обратно операционной системе. Некоторые сборщики мусора, такие как G1 и Shenandoah, которые включены в Hotspot JVM, успешно это делают, однако, ZGC всё еще не возвращает память, даже если эта память давно уже не используется. JEP 351 призван устранить это недоразумение путем введения нового параметра `-XX:ZUncommitDelay`, который объясняет ZGC, когда тому следует возвращать освобожденную память операционной системе. Может показаться, что подобное нововведение довольно простое, и ZGC должен был уже давно научиться освобождать память, но все чуть более сложнее. Авторы [JEP 351](https://openjdk.java.net/jeps/351) приводят некоторые соображения на этот счет, а так же объясняют чуть более подробно, как теперь работает механизм возврата памяти в ZGC.

<!-- /wp:paragraph -->

<!-- wp:heading -->

## Заключение

<!-- /wp:heading -->

<!-- wp:paragraph -->

Java 13 не содержит слишком много больших нововведение. Вероятно, что только новые текстовые и `switch` блоки заинтересуют большинство пользователей Java. Однако, эти новинки в языке всё еще являются preview langeage features, которые выключены по умолчанию. Остальные нововведения в JDK 13 скорее всего не будут замечены большинством, что означает, что рядовой пользователь вряд ли заметит что-либо при переходе на новый JDK. Тем не менее, каждый может включить новинки в синтаксисе, опробовать их, составить свое мнение и отправить отзыв, чтобы сделать JDK лучше.

<!-- /wp:paragraph -->

