---
layout: post
title: Transistor delay circuit
date: 2019-06-16 13:02:02.000000000 +01:00
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
meta:
  _edit_last: '1'
  _yoast_wpseo_primary_category: '155'
  _yoast_wpseo_content_score: '60'
  _yoast_wpseo_focuskw: transistor delay circuit
  _yoast_wpseo_metadesc: The transistor delay circuit may be helpful to learn some
    electronics basics. Let’s see how we can choose elements for the circuit, and
    how the delay depends on parameters of the elements.
  _yoast_wpseo_linkdex: '78'
  rp4wp_auto_linked: '1'
  _wpdiscuz_statistics: a:4:{s:7:"threads";i:0;s:7:"replies";i:0;s:7:"authors";i:0;s:14:"recent_authors";a:0:{}}
  _oembed_57d72e6511eab5903ae5be60bb95cd38: <a class="twitter-timeline" data-width="625"
    data-height="938" data-dnt="true" href="https://twitter.com/artem_smotrakov?ref_src=twsrc%5Etfw">Tweets
    by artem_smotrakov</a><script async src="https://platform.twitter.com/widgets.js"
    charset="utf-8"></script>
  _oembed_time_57d72e6511eab5903ae5be60bb95cd38: '1617982480'
  _yoast_wpseo_estimated-reading-time-minutes: '4'

permalink: "/en/diy-electronics/transistor-delay-circuit.html"
---


The transistor delay circuit may be helpful to learn some electronics basics. The circuit is pretty simple. It only contains a transistor, a capacitor, several resistors, a switch and an LED. The circuit uses an RC filter to turn an LED on with a little delay. Let's see how we can choose elements for the circuit, and how the delay depends on parameters of the elements.





![Transistor delay circuit on a breadboard]({{ site.baseurl }}/assets/images/2019/06/transistor_delay_circuit_on_breadboard.jpg)



  
  




Below you can see the schema. The LED is connected to the collector of the transistor. The resistor R1 limits the current to prevent damaging the LED. The transistor Q1 and the resistor R2 make a switch. The transistor is controlled by the RC filter which is built by the variable resistor R3 and capacitor C1. The RC filter defines the delay.





![Transistor delay circuit]({{ site.baseurl }}/assets/images/2019/06/Transistor_delay_schem-1.png)





## How the transistor delay circuit works





At first, the capacitor is not charged, and the switch S1 is off. It means that no current comes to the base of the transistor, and the LED is off. If we hold the button S1, the capacitor begins charging. The resistor R3 defines how fast the capacitor charges. The more the resistance of R3 is, the slower the capacitor charges. The RC filter implements a voltage divider. While the capacitor is charging, the voltage across the capacitor grows. It means that the more the capacitor is charged, the more voltage applies to the base of the transistor. After some time, the voltage applied to the base of transistor becomes high enough to open the transistor. The current starts flowing through the transistor, and the LED turns on.





The delay can be adjusted by changing the resistance of the variable resistor R3.





![Transistor delay circuit on a breadboard]({{ site.baseurl }}/assets/images/2019/06/Transistor_delay_bb.png)





## Choosing components for the transistor delay circuit





Let's see how we can choose elements for the transistor delay circuit.





We use a standard LED which has the following parameters





- [latex]V\_{led} = 2V[/latex] - voltage drop 
- [latex]I\_{led(max)} = 20mA[/latex] - maximum forward current





Next, we use 2N3904 transistor. This is an NPN transistor which has the following parameters according to its datasheet





- [latex]V\_{CA(sat)} = 0.2V[/latex] - voltage drop between collector and emitter
- [latex]V\_{BE(sat)} = \frac{V\_{BE(sat)max} + V\_{BE(sat)min}}{2} = \frac{0.85V + 0.65V}{2} = 0.75V[/latex] - average voltage drop between base and emitter
- [latex]H\_{fe} = \frac{I\_{c}}{I\_{b}} = 30[/latex] - the lowest current gain of the transistor





### Choosing a current limiting resistor for an LED 





Let's calculate the value of the resistor R1 which limits the current for the LED.





The voltage supply is [latex]V\_{s} = 3V[/latex]. Let’s set the LED current to be [latex]I\_{led} = 10mA = 0.01A[/latex] which is less than the maximum allowed current for the LED. This current should be enough to turn the LED on. [latex]I\_{led}[/latex] is also the collector current [latex]I\_{c}[/latex]:





[latex]I\_{c} = I\_{led}[/latex]





Now we can apply Ohm's law to calculate R1:





[latex]R1 = \frac{V\_{s} - V\_{led} - V\_{CA(sat)}}{Ic}=\frac{3V - 2V - 0.2V}{0.01A} = 80 Ohm[/latex]





We can pick up a standard resistor of 100 Ohm.





### Choosing a base resistor for a transistor





First, let’s calculate the base current which keeps the transistor open





[latex]Ib = \frac{I\_{c}}{H\_{fe}} = \frac{10mA}{30} = 0.33mA[/latex]





To make sure the transistor turns on fully, let’s add a factor of two for safety and use a base current of [latex]I\_{b} = 0.7mA = 0.0007A[/latex]





Now we can apply Ohm’s law and calculate R2





[latex]R2 = \frac{V\_{s} - V\_{BE(sat)}}{I\_{b}} = \frac{3V - 0.75V}{0.0007A} = 3214Ohm = 3.2KOhm[/latex]





We can pick up a standard resistor of 3.3 KOhm.





### How an RC filter defines a delay





The delay is related to the charging time of the capacitor C1. The charging time of the capacitor is related to the product [latex]R3 \* C1[/latex]





[latex]\tau = R3 \* C1[/latex]





It is the time required to charge the capacitor, through the resistor, from an initial charge voltage of zero to approximately 63.2% of the value of an applied voltage.





The rise time from 20% to 80% can be calculated as the following





[latex]t\_{r} = 1.4\tau[/latex]





If we the capacitor C1 is 470 mkF, and the resistor R3 is 200 KOhm, then the approximate maximum delay is going to be





[latex]t\_{r} = 1.4 \* 200KOhm \* 470mkF = 1.4 \* 200000Ohm \* 0.00047F = 131s[/latex]





Note that the LED actually turns on much faster. It happens because the transistor starts opening even before the capacitor is charged to 80%. After some time, once the voltage on the base is high enough, the collector current starts slightly growing. As a result, the LED starts turning on.





## References





1. [Ohm's\_law](https://en.wikipedia.org/wiki/Ohm%27s_law)
2. [Resistor](https://en.wikipedia.org/wiki/Resistor)
3. [Capacitor](https://en.wikipedia.org/wiki/Capacitor)
4. [Light-emitting\_diode](https://en.wikipedia.org/wiki/Light-emitting_diode)
5. [How transistors work](https://www.build-electronic-circuits.com/how-transistors-work/)
6. [RC circuit](https://en.wikipedia.org/wiki/RC_circuit)
7. [RC time constant](https://en.wikipedia.org/wiki/RC_time_constant)
8. [LED datasheet](https://www.sparkfun.com/datasheets/Components/YSL-R596CR3G4B5C-C10.pdf)
9. [2N3904 datasheet](https://www.onsemi.com/pub/Collateral/2N3903-D.PDF)



