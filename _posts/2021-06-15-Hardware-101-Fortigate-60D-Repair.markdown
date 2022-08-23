---
layout: post
title: "Hardware 101: Fortigate 60D Repair"
---

I guess the title of this is fairly self explanatory. Going to go through the recent repair I made on a 
fortiate 60D firewall.

First thing of note, I got this firewall rather cheaply from ebay and as such had no idea if it was in fully working
condition. It came without a power adapter (obviously this is normally a red flag) but as it was so cheap that 
didn't phase me too much. The cost of an official power adapter for these devices is scary, but no matter what
I'd be able to fabracobble up something to make it work.

Fairly quickly found posts on the internet detailing which connector it uses for power as this is of the newer 
generation of fortigate gear that has moved on from barrel jacks and uses a molex style power connector. Also luckily
there was mention of pre-terminated cables being available. With that I quickly found it on mouser as
[part 538-245135-0210](https://au.mouser.com/ProductDetail/538-245135-0210). So next mouser order I put that item in.

Expected it to just be one connector with bare wires at the end, instead it was terminated with the connector on both 
ends. Call that a win, next time I need one of these connectors (which arrived sooner than later as I picked up
more newer gen fortigate gear without power supplies quite soon after) then it was already here. Cut the cable in two
and verified that the red/black wires matched up with what the firewall used for its pinout (easy with a multimeter 
in continuity mode). Added a screw on barrel jack for now. Plugged in a 12V power supply with a higher output current
capacity than required, plugged in a usb cable for the usb console and...

Well it enumerated, connected in and got the bootloader. Let it try to boot and well, it crashed. Oh well, luckily I
acquired the firmware through means I won't go into. So a tftp load of that and see if it boots. Negative, it ain't
happy. Tried formatting the onboard flash from the bootloader, looks like the flash is just all kinds of corrupt.
Probably an earlier firmware version was logging to the onboard flash, and that's pushed it over its write limits.

So some more googling later, found out that if you're really lucky you can plug in an external flash drive and it might
enumerate earlier than the onboard. That's definitely the case for the bootloader. Can format the flash, tftp a firmware
onto it. But go to boot, and the onboard ends up as the primary again and that just crashes everything. So a bit more
searching showed jumpers on the board that could be used to disable the onboard flash. They should've been labelled as
"JUSBn" (with n for whichever jumper it was). Mine didn't have any labelled as that, but did have a group of jumpers
set up similar. So what harm could it do?

(insert image of header here)

At this point I was getting sick of how slow and unreliable the USB console was. Annoyingly this was the version
of the FG60D that doesn't have the raw serial console port. But as I knew where the console port was supposed to be
and there were some headers in the same area, grabbed a multimeter and probed around. Found where ground was 
(multimeter, continuity), added ground between a ttl serial to usb adapter and added power. No smoke, so we're going 
well. Probed around for voltages, everything else on the headers was 3.3V. Time to grab the RX on the UART and just 
start plugging it in. Can't really hurt it much with those voltages. If it's a power line we get nothing, if it's 
anything other than a UART TX, we get nothing. Well after a few attempts of moving the signal around and resetting
the firewall, I got a console log! So now I need to find the RX line of the header. This one was just sheer guessing 
based on how the wires were already hooked up. Plug TX of the USB-UART in, wait for the bootloader to get to a point
of accepting input, and...FIRST GO!

For anyone that has this issue in the future and comes across this post. Here's a hopefully useful image to help you
out.
![Pinout of console port](/assets/2021-06-15/fg60d-serial.jpg)

So now I had much easier access to the console. Removed the jumpers from the suspected flash jumpers. Power on, tftp,
load as default firmware. We're booooooooting! Got to formatting the shared storage partition. This took forever
and I was worried it'd crashed. Went off for a while, came back to a factory reset fresh install!

![Flash part 1](/assets/2021-06-15/fg60d-flash1.png)
![Flash part 2](/assets/2021-06-15/fg60d-flash2.png)
With that done, I now have a very cheap but working fortios v6 supported firewall. I'm sure this was as boring to read
as it was to write. But maybe it'll be useful to have preserved.
