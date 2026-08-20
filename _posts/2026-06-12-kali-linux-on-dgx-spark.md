---
layout: post
title: "Kali linux on dgx spark with virt manager"
date: 2026-06-12
---
Kali Linux running in Virt-Manager on DGX Spark
Took me a while to get Kali Linux running
Turns out there's a small trick required to the default settings to get it going
You have to select configure machine before boot checkbox

![kali1.png](/assets/images/kalilinux/kali1.png)

Then in the configuration under overview

![kali2.png](/assets/images/kalilinux/kali2.png)

I tried AAVMF both secboot and no sec boot and both worked fine
however without this, the machine didn't boot
I also chose Debian 13


Some guides recommend Debian 11, that's out of date.
I found that when I booted up by default virt-manager had not supplied a graphical interface for the gui or display manager to run on.
I also found that enabling spice before install made it impossible to send keystrokes to grub to select the graphical install.


TLDR - - - - - - - ADD SPICE(graphics display) AFTER INSTALL - - - - - - - - - - -

So I added the hardware for spice and virtio AFTER installing Kali worked fine to display the UI despite indications QXL was required.
I still wasn't able to send keystrokes or mouse movements so I looked into connecting to the display directly

![kali3.png](/assets/images/kalilinux/kali3.png)

It turns out you have to add inputs which are the basic usb inputs as hardware to get the spice server working properly so there's a bunch of hardware to add, 1 input USB keyboard 1 input USB mouse 1 graphics, display spice 1 video virtio .
I had no issue with qxl or whatnot
Your list of hardware should look something like this

![kali4.png](/assets/images/kalilinux/kali4.png)

When installing kali packages, I added gnome for simplicity of getting it working without using lightdm which is also fine

I found that I wasn't able to get the install working with the spice graphics server, so I recommend adding that hardware after install through the serial console.
Incidentally I found that enabling opengl support/3d acceleration isn't fully reversible through the UI, once enabled, I couldn't properly fix the vm to boot again without it.

Lastly I found I had to add a usb tablet input device to make the mouse cursor function not just the USB mouse, working kali linux

![kali5.png](/assets/images/kalilinux/kali5.png)
