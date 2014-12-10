---
layout: post
title: 	"RGB LED Controller"
date: 	2014-12-07 13:38:00
author: David Richmond
categories:	electronics led
---
There are many types of LED strips available these days.  One of the most common however is essentially 3 strings (ie. an R, G & B string separate) which is driven from 12V and has a common cathode topology.  

Normally these also ship with a driver which provides several effect functions and several pre-set fixed colours, and controlled via an IR remote control.  While the driver itself works, it tends to have a very low pwm rate, and if you crack one open you will notice it has been built to a price.  I suspect that on long strings (ok, longer than they normally ship with), the current draw might actually cause the driver transistors - TO-92 package - will burn.  Also, the power supply for the digital section on the drivers is typically a zener "regulator" which although works just screams "we're too cheap to even get a 5 cent 78L05, this 3.5 cent zener and 1 cent resistor will save us many dollars over the production run".

My aim is to build a drop-in replacement driver.  It should work with the same remote, not suffer from the same design issues, and show what's possible if you don't use price as your primary design drive.

The only restrictions I've applied are based on which parts I have on hand and don't require ordering anything else (ie. left-overs from other projects). As such, the following parts will be selected based on those criteria:
1. Microcontroller: NXP LPC810M021fn8 - Cortex M0+, 4KB Flash, 1KB RAM, DIP-8 Package. Selected because I ordered them from E14 a while back, recieved twice as many as I ordered, and haven't had a chance to do any further projects with them.
2. Regulator: LM317 - Jellybean adjustable regulator.  Who doesn't have these spare? Based on digital current requirements I could've used a TO-92 78L33, but I didn't have those on hand. Absolute overkill in a TO220, but that's what I had on hand.
3. String driver transistors: IRF820N MOSFET.  These are 10A power n-mosfets.  Slightly overspecced, but compared to the underspec job on the original drivers, who cares?
4. MOSFET Drivers: TC4427A. Maybe with a 5V micro and a logic-level mosfet I could've got away without a mosfet driver, but even in that case it could've been debatable as to whether the mosfet would've been driven into saturation and driven quickly.  These are dual mosfet drivers, since I only have 3 mosfets one will have a non-used driver - in a future version maybe the spare could be used to provide a power-off function?

Apart from those parts, there will only be a few other jellybeans in use (resistors to set the regulator, decoupling caps, etc).

Although not implemented at the current step, reverse protection would be a good thing to add in.

First prototype
---------------

Before even attempting to drive a real string, what was needed was a very basic prototype with a single RGB LED, basic power supply and the microcontroller with associated debug adapters.  This was to allow nice and easy code development without too many additional requirements beyond a usb connection to power the whole setup.

First difference from the final product is that all I could find was a common-anode RGB LED (with pnp bjt transistors as drivers because the LPC810 is very limited in it's output current drive).  I forgot about this later on, until I needed to un-invert the output of the PWM.

TODO: Schematic
TODO: Picture of breadboard.

TODO: Finish article

1. overview
2. initial design
  * design decisions and implementation
  * breadboard prototype
  * source code
3. second breadboard prototype
  * assembly mistake (trying to power digital section off regulator adjust pin)
  * pwm black box (issue: inverted output from initial prototype)
4. schematic
5. pcb
6. conclusion
