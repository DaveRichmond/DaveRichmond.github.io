---
layout: post
title: 	"RGB LED Controller"
date: 	2015-08-11 16:36:00
author: David Richmond
categories:	electronics led
---
There are many types of LED strips available these days.  One of the most common however is essentially 3 strings (ie. an R, G & B string separate) which is driven from 12V and has a common cathode topology.  

Normally these also ship with a driver which provides several effect functions and several pre-set fixed colours, and controlled via an IR remote control.  While the driver itself works, it tends to have a very low pwm rate, and if you crack one open you will notice it has been built to a price.  I suspect that on long strings (ok, longer than they normally ship with), the current draw might actually cause the driver transistors will burn.  Also, the power supply for the digital section on the drivers is typically a zener "regulator" which although works just screams "we're too cheap to even get a 5 cent 78L05, this 3.5 cent zener and 1 cent resistor will save us many dollars over the production run".

![Cheap Driver](/images/led_controller/chinese_led_controller.jpg)

As we can see from the (abeit potato quality) photo of one of these drivers, they have seriously cut the BOM back.  There are obviously spaces for various components that have been skipped.  There is not a single bypass capacitor in the design.  The brains itself has the part number removed, but I'm going to assume it's a mask rom job due to the 24C02 eeprom external which must store the various colour settings. Here I also noticed that they completely skipped the I2C pullups for this eeprom - I guess they just got lucky that it "works".  The driver transistors are "A2SHB" which after a very short google session turns out to be the Vishay/Siliconix Si2302DS (http://www.vishay.com/docs/70628/70628.pdf) N-Channel MOSFET. However, this being such a cheap item, I assume they're counterfeit. Yet another cutback is visible here, there's nothing to stop the gates of the MOSFETs from floating, but the original design must've intended to have them due to the "1K" on the silkscreen near each of traces.

But enough pointing out issues with their design.

My aim is to build a drop-in replacement driver.  It should work with the same remote, not suffer from the same design issues, and show what's possible if you don't use price as your primary design drive.

The only restrictions I've applied are based on which parts I have on hand and don't require ordering anything else (ie. left-overs from other projects). As such, the following parts will be selected based on those criteria:

1. Microcontroller: NXP LPC810M021fn8 - Cortex M0+, 4KB Flash, 1KB RAM, DIP-8 Package. Selected because I ordered them from E14 a while back, recieved twice as many as I ordered, and haven't had a chance to do any further projects with them. Also, stupidly small but powerful, gotta have a "fun" part to each project.
2. Regulator: LM317 - Jellybean adjustable regulator.  Who doesn't have these spare? Based on digital current requirements I could've used a TO-92 78L33, but I didn't have those on hand. Absolute overkill in a TO220, but that's what I had on hand. Because of vague power supply requirements, we need the regulator to be able to handle a fairly high input range (meaning we can't just use a lil' LDO) - the LM317 and LM78xx series both handle ~30V in without complaining and the very low current requirements of the digital section (probably about the same as the quiescent for the regulator) mean that using a switcher would be more trouble than it's worth.
3. String driver transistors: IRF820N MOSFET.  These are 10A power n-mosfets.  Slightly overspecced, but compared to the underspec job on the original drivers, who cares?
4. MOSFET Drivers: TC4427A. Maybe with a 5V micro and a logic-level mosfet I could've got away without a mosfet driver, but even in that case it could've been debatable as to whether the mosfet would've been driven into saturation and driven quickly.  These are dual mosfet drivers, since I only have 3 mosfets one will have a non-used driver - in a future version maybe the spare could be used to provide a power-off function?

Apart from those parts, there will only be a few other jellybeans in use (resistors to set the regulator, decoupling caps, etc).

Although not implemented at the current step, reverse protection would be a good thing to add in.

First prototype
---------------

Before even attempting to drive a real string, what was needed was a very basic prototype with a single RGB LED, basic power supply and the microcontroller with associated debug adapters.  This was to allow nice and easy code development without too many additional requirements beyond a usb connection to power the whole setup.

First difference from the final product is that all I could find was a common-anode RGB LED (with pnp bjt transistors as drivers because the LPC810 is very limited in it's output current drive).  I forgot about this later on, until I needed to un-invert the output of the PWM.

![Prototype Schematic](/images/led_controller/proto1_schematic.png)

FIXME: Schematic missing base resistors.

At this point, we can see that several design decisions have been made.

1. All pins are allocated.
2. We had a choice between having a reset + bootloader programming setup, or full debug capability but no reset (apart from holding in reset on power-up, the IR receriver should idle high, so unless it's constantly receiving a carrier then we should come out of reset fairly quickly).  I've gone the debug route - there's no way I'll live without debugging capabilities these days (I anticipate many flame wars from "professionals" who claim that all they need for debugging is a single LED).
3. Due to debugging, we have two pins constantly allocated to said debugger.  This overcomes most of the reset issue as a debugger can reset the microcontroller over the SWD connection - any resetting in the field will just require pulling the power.

Final-ish Design
----------------

The prototype design was used for all the coding, but at the same time it was completely inappropriate to use for anything other than a single RGB LED.  As such, parallel to the coding, a closer to final design was implemented.

![Schematic](/images/led_controller/schematic_20150811.png)

The major changes between the prototype and this is:

1. Power MOSFETs used in a low-side drive configuration (required due to the design of the strings).
2. LM317-based power supply.  It's cheap, it does the job, we don't care about efficiency as the quiescent is going to be noise compared to how much power goes through the LEDs, we're running off mains so we don't need to account for every milliamp.
3. MOSFET drivers. As mentioned earlier, they make the driving of power MOSFETs simple.  They'll drive rather large gates very fast.

This design was built up on a breadboard.  On first power-on there the whole thing was just crazy.  After scratching my head for a few minutes and double checking things, my hand went straight to my forehead.  I'd connected the power for the microcontroller up to the adjustment pin on the regulator and not the output.  Fixing that, and all was good...until I realised that the colours were completely wrong.  The original design used the PWM inverted relative to this design.  A quick fix to make the output polarity configurable and everything was working as well as can be expected.



Notes section (to be removed when done)
---------------------------------------

This is a WIP (Work in Progress).

TODO: Picture of breadboard.

TODO: Finish article

TODO: Write in a more natural style

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

Edits
-----

2015-08-11 - Added in photos of the original driver. Finished code.  Started proper schematic of new driver. I'm not dead, just really slow at working on projects.