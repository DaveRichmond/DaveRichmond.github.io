---
layout: post
title: U-boot flash dumping
---

# leadin bit (forgot words)

Right now I'm porting openwrt to a few cheap access points as a bit of fun. 
The latest one that came across my desk is an extremenetworks aerohive ap305c. 
From the internet you won't find that many details on it. So once I got a 
console into it (I wasn't paying for one of the official cables, a usb cable, 
usb to screw terminal breakout, and a usb-uart adapter do the job and I don't 
have to worry about ordering one with the wrong pinout). 

First thing I got hit by is you need a password to get into u-boot on this 
device. Thankfully they've kept the same password for a long time, and the 
openwrt wiki/table-of-hardware has it [documented](https://openwrt.org/toh/aerohive/hiveap-330).

## Actually dumping the firmware

Not wanting to reinvent the wheel, the most mentioned piece of software for this 
is [depthcharge](https://github.com/nccgroup/depthcharge). Unfortunately the 
actual documentation for using it is pretty sparse. This may not be the actual 
way to do things, but it's what I found seemed to work.

First of all, follow the docs in /python on getting it installed (create python virtualenv, pip install).

### Create config

    $ depthcharge-inspect -c aerohive.cfg -i /dev/ttyACM0:9600

Replacing your serial port as required, baud rate as required (this is a weird 
device with an old-school 9600 baud console, and I was too lazy to change it).

### read flash to memory on device

Before we can transfer the flash, u-boot really only supports transferring data 
from ram to other places, so we have to load the flash into ram.

Maybe there is some extra handy features in u-boot I missed, but we'll have 
to do some maths to determine the size of the flash. I did notice somewhere 
in the boot logs that it's a 4MB flash chip, so that's a size of 0x400000 
bytes. U-boot also usually has the variable loadaddr set to somewhere in 
ram you can safely stick data

    uboot> nand read ${loadaddr} 0 0x400000

### finally transfer it across

We'll have to make a note of loadaddr as we can't just refer to the variable on other devices

    $ depthcharge-read-mem -c aerohive.cfg -a 0x10000000 -l 0x400000 -f aerohive-nand.bin -i /dev/ttyACM0:9600

After a really long time, we'll have our flash dumped to a file and we can now 
analyse it for any useful tidbits like the device-tree.
