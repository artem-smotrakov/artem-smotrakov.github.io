---
layout: post
title: Sending text commands to STM32 with USART
date: 2016-10-06 10:10:41.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY and electronics
tags:
- Electronics
- STM32
permalink: "/en/diy-electronics/stm32-usart-sending-text-commands.html"
---
If you think that your STM32 board feels lonely, you can connect it to your laptop. One of the ways how you can do that is USART. That's probably the easiest way. For example, let's say that we want to send text commands from a laptop to STM32 board. One command should turn on an LED, and another one should turn it off. And of course, STM32 should curse us in case of invalid command.

![STM32F103, ST-LINK/V2 programmer/debugger and USB-Serial adapter]({{ site.baseurl }}/assets/images/2016/10/IMG_1202.jpg)

I used a [template project for STM32](http://blog.gypsyengineer.com/fun/a-template-project-for-stm32f103-on-linux.html) which I have described earlier. Everything that I described in that post applies here (same components, same software, same commands). But source code became a little bit more complicated:

[https://github.com/artem-smotrakov/stm32f103-usb-commands](https://github.com/artem-smotrakov/stm32f103-usb-commands)

## Hardware: STM32F103, ST-LINK/V2 and USB-Serial adapter

Here is a list of components we need:

1. STM32F103 board (I used [this one](https://www.aliexpress.com/item/1pcs-STM32F103C8T6-ARM-STM32-Minimum-System-Development-Board-Module-For-arduino/32478120209.html?spm=2114.13010608.0.57.JGYUo9) which is based on STM32F103C8T6 microcontroller).
2. An LED of your favorite color, and a resistor (200 or 330 Ohms should be fine). Here is another post which describes [how to select a current limiting resistor](http://blog.gypsyengineer.com/fun/limit-the-current-take-care-of-your-favorite-led.html).
3. ST-LINK/V2 debugger/programmer.
4. USB-Serial adapter. I used [this one](https://www.aliexpress.com/item/Free-shipping-FT232RL-FT232-FTDI-USB-3-3V-5-5V-to-TTL-Serial-Adapter-Module-Mini/32648254875.html?spm=2114.13010608.0.91.zmRfmD) which is based on FT232RL (but you can use any other adapter).

As I described in my previous post, I connected an LED to PA1 pin. Then, we need to connect an USB-Serial adapter to STM32. According to [datasheet for STM32F103](http://www.st.com/content/ccc/resource/technical/document/datasheet/33/d4/6f/1d/df/0b/4c/6d/CD00161566.pdf/files/CD00161566.pdf/jcr:content/translations/en.CD00161566.pdf), PA9 and PA10 pins can be configured for USART (see page #31 of the datasheet):

1. Connect PA9 (TX) pin to RX pin of USB-Serial adapter.
2. Connect PA10 (RX) pin to TX pin of USB-Serial adapter.
3. Connect GND of STM32 board to GND of USB-Serial adapter.

I hope your laptop has two free USB ports because we need one for ST-LINK/V2, and another one for USB-Serial adapter.

That's it about hardware.

## Software: STM32 and USART

I put the sources on GitHub:

```bash
git clone https://github.com/artem-smotrakov/stm32f103-usb-commands
```

The structure is similar to what I described in my post about template project for SMT32:

1. `Makefile` is a Make file for compiling sources and uploading binaries to STM32 board.
2. `main.h` and `stm32f10x_conf.h` include header files that we need, and define a couple of constants.
3. `main.c` contains the most interesting part.

Here is what we have in `main.c`:

1. `init_output()` initializes PA1 as an output pin to drive an LED. Then, we can turn it on/off with `turn_on_pa1()` and `turn_off_pa1()` functions
2. `usart_init()` initializes USART on PA9 and PA10 pins. Then, we can send bytes and strings with `usart_send()`, `usart_send_string()`, `usart_send_newline()` and `usart_send_line()` functions.
3. `USART1_IRQHandler()` is a handler for USART data. It receives incoming text commands.
4. `handle_command()` processes commands which came from USART.
5. `get_string_length()` returns length of a string, and `is_equal()` checks if two strings are equal. They are similar to `strlen()` and `strcmp()` functions. It's also possible to use [newlib](https://sourceware.org/newlib/) which contains those functions, but it would make the project more complicated. So I just quickly added `get_string_length()` and `is_equal()` instead.
6. `main()` function puts everything together.

This simple project supports only two text commands:

1. `a1 on` sets PA1 pin to "1" (turns on LED).
2. `a1 off` sets PA1 pin to "0" (turns off LED).

The sources can be compiled and uploaded to STM32 board with commands like the following (you need to specify a path to STM32 standard peripheral library):

```bash
STD_PERIPH_LIBS=/home/user/libs/STM32F10x_StdPeriph_Lib_V3.5.0 \
    make all
STD_PERIPH_LIBS=/home/user/libs/STM32F10x_StdPeriph_Lib_V3.5.0 \
    make burn
```

## Sending commands to USB-Serial port

If you use Linux, there should be a file in `/dev` associated with a USB port which you use for USB-Serial adapter. You can use `dmesg` tool to find it:

```bash
dmesg | grep tty
```

It should print something like the following:

```
[0.000000] console [tty0] enabled
[5.228613] systemd[1]: Created slice system-getty.slice.
[29158.699946] usb 1-2: FTDI USB Serial Device converter now attached to ttyUSB0
[29869.017886] ftdi_sio ttyUSB0: FTDI USB Serial Device converter now disconnected from ttyUSB0
[29877.238602] usb 1-2: FTDI USB Serial Device converter now attached to ttyUSB0
```

In my case, I had `/dev/ttyUSB0` file. You may need to set 666 access rights to it:

```bash
sudo chmod 666 /dev/ttyUSB0
```

After that you can send data to STM32 by writing to this file, for example:

```bash
# turn LED on
echo -e "a1 on\r" > /dev/ttyUSB0
# turn LED off
echo -e "a1 off\r" > /dev/ttyUSB0
```

Or, you can use your favorite terminal emulation program, for example, Minicon:

```bash
sudo apt-get install minicom
sudo minicom -s
```

The second command above shows a menu where you can configure minicom with proper settings for USB-Serial connection (file name, baud rate, word length, stop bits, parity and flow control). You can find those settings in `usart_init()` function. Then, it will offer you to save settings, and connect to the device). Or, you can just use the following command:

```bash
minicom --device /dev/ttyUSB0
```

That's pretty much it. Good luck!

![STM32F103, ST-LINK/V2 programmer and debugger and USB-Serial adapter]({{ site.baseurl }}/assets/images/2016/10/IMG_1200.jpg)
