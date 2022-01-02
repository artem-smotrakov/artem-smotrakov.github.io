---
layout: post
title: Problems with running MicroPython on ESP8266 with 512K
date: 2017-02-20 22:46:49.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY and electronics
tags:
- DIY
- Electronics
- ESP8266
- Internet of Shit
- IoT
- MicroPython
- Python
permalink: "/en/diy-electronics/problems-with-running-micropython-on-esp8266-with-512k.html"
---
In my previous post about [running MicroPython on ESP8266](http://blog.gypsyengineer.com/fun/diy-electronics/getting-started-with-esp8266-and-micropython.html), I mentioned that ESP8266 boards may have different amount of flash. Similarly there are two versions of MicroPython: limited version for 512K, and full version for boards which have more than 512K of flash. In that post, I played with ESP-07 which had only 512K, so I had to use a limited version of MicroPython. This limited MicroPython version was enough just to turn on/off an LED, but it turned out that it actually doesn't work well.

[peg-image src="https://lh3.googleusercontent.com/-R3VwoQTgucI/WKi6OUtAf7I/AAAAAAAAIOM/7lQm-sbab2MZdftJBdJUnDAFI6Ha3i6MgCCo/s144-o/IMG\_20170218\_131903.jpg" href="https://picasaweb.google.com/103813056835863838724/6383844083469641761#6388560826663468978" caption="" type="image" alt="ESP8266" image\_size="4096x2304"]



After successful flashing ESP8266 with limited MicroPython, I tried to upload a simple script which just turns on/off&nbsp;an LED in a loop:

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

I tried a couple of tools, but nothing worked.

[rshell](https://github.com/dhylands/rshell) reported the following error:

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

And&nbsp;[mpfshell](https://github.com/wendlers/mpfshell) reported this:

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

[WebREPL](https://github.com/micropython/webrepl) didn't work either. After connecting to the board via USB-Serial adapter, and trying to start WebREPL, it reported the following error:

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

I have not looked into the problems above, but looks like that&nbsp;limited MicroPython doesn't contain something which is required for those tools. Or, this may be just a bug.

Finally, I found an ESP8266 board with 1M of flash, so that I could use full MicroPython (in my case, it was v1.8.7).&nbsp;And it worked!&nbsp;I was able to upload a Python script to the board. Here is how it can be done with mpfshell:

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

After resetting the board, I was happy to see that the LED was blinking.

The conclusion is obvious:

- Buy ESP8266 with more than 512K of flash
- Use full version of MicroPython
