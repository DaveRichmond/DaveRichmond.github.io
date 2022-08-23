---
layout: post
title: "Gotcha #3 - Systemd Networkd + VLANs"
---

## Gotcha's
This is a series where I make notes of various things that I encounter weird issues, that maybe should be documented
somewhere. Or at least have a source somewhere that you can find what you need when you search

# Systemd Networkd + VLANs
I know a bunch of people hate systemd, and this is not the place to go on about that. Playing around with arch, and that
pretty much requires dealing with systemd. Well today I went to try and configure a vlan on there. Making sure it's 
a persistent config is important, never know when a reboot will happen and gotta be sure things come back up correctly.

Well with that out of the way, my issue was due to me not liking the interface name this got usb to ethernet adapter
got. I forget off the top of my head, but it was awkwardly long. So, according to the docs all I needed to do to rename 
it was 
```
[Match]
MACAddress=nn:nn:nn:ab:cd:ef

[Link]
Name=ethusb0
```

Well that worked. However I also needed a vlan on top of that interface. No matter what I'd do the interface would stick
in various down states (should've checked the exact states before writing this, but memory bad).

Bunch of googling later and end up in the systemd bug tracker. Finally at the very end of a thread with a similar 
problem, but not my problem someone links to another bug. in the "[Match]" section, we need "Kind=!vlan".

What is happening is because the root interface and every vlan interface share the same MAC address, systemd in it's
smortness tries to rename every vlan (because hey, you've got the mac address I need to rename) but the name has already
been taken.

Ugh.
