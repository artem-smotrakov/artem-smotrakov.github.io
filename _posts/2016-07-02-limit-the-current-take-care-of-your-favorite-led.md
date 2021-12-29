---
layout: post
title: Current limiting resistor for an LED
date: 2016-07-02 18:54:00.000000000 +01:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- DIY and electronics
tags:
- Electronics
meta:
  blogger_blog: blog.gypsyengineer.com
  blogger_author: Artem Smotrakov
  blogger_permalink: "/2016/07/limit-current-take-care-of-your.html"
  blogger_internal: "/feeds/5492409931645402059/posts/default/8514169295678329950"
  _edit_last: '1'
  _wp_rp_image: empty
  _wpgo_column_layout_save: ''
  _wpgo_column_layout: default
  mask_links: default
  _aioseop_description: How to calculate the value of a current limiting resistor
    if you want to drive an LED from microcontroller's I/O pin.
  _aioseop_title: A current limiting resistor for an LED
  rp4wp_auto_linked: '1'
  _yoast_wpseo_primary_category: '155'
  _yoast_wpseo_content_score: '90'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:0;s:7:"replies";i:0;s:7:"authors";i:0;s:14:"recent_authors";a:0:{}}
  _yoast_wpseo_focuskw: Current limiting resistor
  _yoast_wpseo_linkdex: '53'
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617997168'
permalink: "/en/diy-electronics/limit-the-current-take-care-of-your-favorite-led.html"
---
I am a beginner in electronics, and just learned that LEDs and microcontroller's I/O ports are so sensitive ... Thanks Mr. Ohm whose law saves us!

&nbsp;

[peg-image src="https://lh3.googleusercontent.com/-GzIpnAzXJn8/V3hxKVXjzCI/AAAAAAAAEo4/GgrEO8C0K0sNmIU1zMxJLahjXMzAeKx\_wCHM/s144-o/20160702\_185719.jpg" href="https://picasaweb.google.com/103813056835863838724/6302552616686914609?locked=true#6302912100853599266" caption="STM32, a resistor, an LED" type="image" alt="STM32, a resistor, an LED" image\_size="3264x2448"]

<!--more-->

A current limiting resistor should be used for driving an LED from a microcontroller's I/O pin to limit the current through the LED. The max current value for an LED can be found in its datasheet. It is usually around 20mA. Another reason for using a current limiting resistor is that microcontroller's I/O ports have a maximum I/O current rating. It can be found in a datasheet for microcontroller you use. For example, it is 25mA for STM32F103&nbsp;(see "Table 7. Current characteristics" in&nbsp;[datasheet](http://www.st.com/content/ccc/resource/technical/document/datasheet/33/d4/6f/1d/df/0b/4c/6d/CD00161566.pdf/files/CD00161566.pdf/jcr:content/translations/en.CD00161566.pdf)).&nbsp;

To calculate the value for a current limiting resistor, you just need to use your favorite&nbsp;[Ohm's law](https://en.wikipedia.org/wiki/Ohm%27s_law). Here is a circuit and calculations.

&nbsp;

[peg-image src="https://lh3.googleusercontent.com/-ktWt2rb5xp4/V3cqNo\_iNFI/AAAAAAAAEo4/eWp0A7M7zH4OuX5Kq35t9gwLXoIWZQmSQCHM/s144-o/20160701\_194142.jpg" href="https://picasaweb.google.com/103813056835863838724/6302552616686914609?locked=true#6302552617358865490" caption="How to calculate the value of a current limiting resistor" type="image" alt="How to calculate the value of a current limiting resistor" image\_size="3264x2448"]

"Vcc" is a power supply voltage (it is 3.3V in case of STM32F103). We assume that high level input voltage of I/O pin is equal to Vcc (which is not actually true, see section "5.3.13 I/O port characteristics" in [datasheet](http://www.st.com/content/ccc/resource/technical/document/datasheet/33/d4/6f/1d/df/0b/4c/6d/CD00161566.pdf/files/CD00161566.pdf/jcr:content/translations/en.CD00161566.pdf)).

The desired current through the LED should be less than its max current value (usually around 20mA). I used 5mA because I wanted just an indication, not lighting.

The formula told me that I should be happy with 260 Ohms. The closest standard resistor is 270 Ohms.

&nbsp;

[peg-image src="https://lh3.googleusercontent.com/-l7oAOn7X04o/V3c9HX\_Aa2I/AAAAAAAAEo4/I\_mVv2cDUMA8FiihLvv1vVM8\_XaKLZwXACHM/s144-o/20160701\_205658.jpg" href="https://picasaweb.google.com/103813056835863838724/6302552616686914609?locked=true#6302573400434961250" caption="STM32, a resistor, an LED" type="image" alt="STM32, a resistor, an LED" image\_size="3264x2448"]

By the way, there are lots of online calculators which you can use if you don't want to waste paper. For example, see&nbsp;[http://www.ohmslawcalculator.com/led-resistor-calculator](http://www.ohmslawcalculator.com/led-resistor-calculator)

Have a next day!
