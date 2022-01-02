---
layout: post
title: Трудности с запуском MicroPython на ESP8266 с 512K памяти
date: 2017-04-26 11:00:33.000000000 +01:00
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
permalink: "/ru/diy-electronics-ru/problems-with-running-micropython-on-esp8266-with-512k-2.html"
---
[ESP8266 могут иметь разное количество памяти на борту](/fun-ru/diy-electronics-ru/how-to-run-micropython-on-esp8266.html). У MicroPython на этот случай есть две версии: ограниченная для бедных устройств с объемом памяти меньше 512K и полная версия для счастливых устройств с объемом памяти более 512K. Прошлый раз мы баловались с&nbsp;ESP-07, которая как раз имела лишь 512K памяти, поэтому использовалась урезанная версия MicroPython. Этого было достаточно, чтобы помигать светодиодом подключившись к ESP8266 через USB-Serial адаптер, но дальше начались трудности.

English version -&nbsp;[Problems with running MicroPython on ESP8266 with 512K](/fun/diy-electronics/problems-with-running-micropython-on-esp8266-with-512k.html)

[peg-image src="https://lh3.googleusercontent.com/-R3VwoQTgucI/WKi6OUtAf7I/AAAAAAAAIOM/7lQm-sbab2MZdftJBdJUnDAFI6Ha3i6MgCCo/s144-o/IMG\_20170218\_131903.jpg" href="https://picasaweb.google.com/103813056835863838724/6383844083469641761#6388560826663468978" caption="" type="image" alt="ESP8266" image\_size="4096x2304"]

После успешной заливки урезанной версии MicroPython, было решено загрузить простенький Python-скрипт, который включает и выключает светодиод в цикле:

```
import time
from machine import Pin

pin = Pin(13, Pin.OUT)
while True:
    pin.high()
    time.sleep(1.0)
    pin.low()
    time.sleep(1.0)
```

Однако скрипт отказывался загружаться. Были даже испробованы разные инструменты по заливке файлов на ESP8266.

[rshell](https://github.com/dhylands/rshell)&nbsp;выругался следующим образом:

```
$ sudo rshell --port /dev/ttyUSB0 
Connecting to /dev/ttyUSB0 ...
Traceback (most recent call last):
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 1184, in connect
    ip_address = socket.gethostbyname(port)
socket.gaierror: [Errno -2] Name or service not known

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/usr/local/bin/rshell", line 9, in 
    load_entry_point('rshell==0.0.9', 'console_scripts', 'rshell')()
  File "/usr/local/lib/python3.5/dist-packages/rshell/command_line.py", line 4, in main
    rshell.main.main()
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 2644, in main
    real_main()
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 2606, in real_main
    connect(args.port, baud=args.baud, wait=args.wait, user=args.user, password=args.password)
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 1190, in connect
    connect_serial(port, baud=baud, wait=wait)
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 1214, in connect_serial
    dev = DeviceSerial(port, baud, wait)
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 1434, in __init__
    Device. __init__ (self, pyb)
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 1269, in __init__
    self.root_dirs = ['/{}/'.format(dir) for dir in self.remote_eval(listdir, '/')]
  File "/usr/local/lib/python3.5/dist-packages/rshell/main.py", line 1351, in remote_eval
    return eval(self.remote(func, *args, **kwargs))
  File "", line 0
    
    ^
SyntaxError: unexpected EOF while parsing
```

[mpfshell](https://github.com/wendlers/mpfshell)&nbsp;сказал вот что:

```
$ sudo mpfshell

**Micropython File Shell v0.7.6, sw@kaltpost.de** 
-- Running on Python 3.5 using PySerial 3.2.1 --

mpfs [/]> open ttyUSB0
Connected to esp8266
mpfs [/]> ls
Traceback (most recent call last):
  File "/usr/local/bin/mpfshell", line 8, in 
    main()
  File "/usr/local/lib/python3.5/dist-packages/mp/mpfshell.py", line 687, in main
    mpfs.cmdloop()
  File "/usr/lib/python3.5/cmd.py", line 138, in cmdloop
    stop = self.onecmd(line)
  File "/usr/lib/python3.5/cmd.py", line 217, in onecmd
    return func(arg)
  File "/usr/local/lib/python3.5/dist-packages/mp/mpfshell.py", line 205, in do_ls
    files = self.fe.ls(add_details=True)
  File "/usr/local/lib/python3.5/dist-packages/mp/mpfexp.py", line 503, in ls
    files = MpFileExplorer.ls(self, add_files, add_dirs, add_details)
  File "/usr/local/lib/python3.5/dist-packages/mp/retry.py", line 43, in f_retry
    return f(*args, **kwargs)
  File "/usr/local/lib/python3.5/dist-packages/mp/mpfexp.py", line 169, in ls
    res = self.eval("os.listdir('%s')" % self.dir)
  File "/usr/local/lib/python3.5/dist-packages/mp/pyboard.py", line 150, in eval
    ret = self.exec_('print({})'.format(expression))
  File "/usr/local/lib/python3.5/dist-packages/mp/pyboard.py", line 157, in exec_
    raise PyboardError('exception', ret, ret_err)
mp.pyboard.PyboardError: ('exception', b'', b'Traceback (most recent call last):\r\n File "", line 1, in \r\nAttributeError: \'module\' object has no attribute \'listdir\'\r\n')
```

[WebREPL](https://github.com/micropython/webrepl)&nbsp;тоже не работал. После подключения к устройству через USB-Serial адаптер попытки запуска WebREPL приводили к следующим ошибкам:

```
could not open file 'boot.py' for reading
could not open file 'main.py' for reading

MicroPython v1.8.7-292-g3d739eb on 2017-02-18; ESP module with ESP8266
Type "help()" for more information.
>>> import webrepl_setup
Traceback (most recent call last):
  File "", line 1, in 
  File "webrepl_setup.py", line 111, in 
  File "webrepl_setup.py", line 80, in main
  File "webrepl_setup.py", line 49, in get_daemon_status
AttributeError: 'NoneType' object has no attribute ' __exit__'
>>>
```

Признаться, я не разбирался, в чем же там беда, но похоже, что урезанные версии MicroPython не содержат в себе чего-то, что так необходимо для заливки файлов вышеупомянутыми инструментами. Или это просто баг в MicroPython.

В конце концов, я нашел ESP8266 с объемом памяти в 1 мегабайт, на которую можно залить полную версию MicroPython (в моем случае это была версия 1.8.7). И все заработало. Python-скрипты&nbsp;стали не только загружаться, но и даже исполняться. Вот как это было сделано с помощью&nbsp;mpfshell:

```
$ sudo mpfshell

**Micropython File Shell v0.7.6, sw@kaltpost.de** 
-- Running on Python 3.5 using PySerial 3.2.1 --

mpfs [/]>; open ttyUSB0
Connected to esp8266
mpfs [/]>; ls

Remote files in '/':

mpfs [/]> put blink_led.py main.py
mpfs [/]> ls

Remote files in '/':

       main.py
mpfs [/]>
```

После перезагрузки устройства я был так счастлив увидеть мигающий светодиод.

Выводы:

- Надо было покупать ESP8266&nbsp;с объемом памяти больше 512K
- Надо использовать полные версии MicroPython
