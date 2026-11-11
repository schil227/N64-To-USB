# N64-To-USB (2026 Version)

This code was forked from the [N64-To-USB Repo](https://github.com/MickMad/N64-To-USB) owned by one Michele Perla ("MickMad"). 
They combined the N64 controller protocol code from [Andrew Brown](http://www.instructables.com/id/Use-an-Arduino-with-an-N64-controller/) and 
the ArduinoJoystick HID/USB code from someone named Darren (dead link), dropped it in their github repo 14 years ago and never touched it again.

## Background
I had a hankerin' to play a Majora's Mask randomizer, so I broke out my 15 year old N64-to-usb converter, and found it wasn't working. After attempting to fix it, I was reminded
(again) that I cannot solder, and it was completely fucked. I looked online for a replacement, balked at the reasonable prices, and began to give up. Then stared at the dusty 
Arduino Uno on my desk - and I resolved to DIY. "Surely, someone's already done this" I thought, "surely, it won't take more than an hour or two to get it all setup." 

Just kidding, I'm not that niave.

I took the N64 connect from the aforementioned busted converter, strategically googled a few things, and found MickMad's code. "Super" I thought. I assembled a few wires, 
and then without much ceremony, set everything up, and it kinda/sorta/not-really worked. 

### State of the Code
MickMad really appeared to cobble things together; iirc the HID stuff was built for a Gamecube controller (16 buttons, 2 joysicks), and a N64 controller 
has roughly half of that, so in his mind you could have two N64 controllers and apply them to the same input device (I guess?). The reality was, 
that the ArduinoJoystick (HID USB device) was set up wrong; there were 32 buttons as inputs, the first 16 were based on the X/Y orientation of the joystick (so mostly chaos).
The N64 controller interpreter was also not quite working, as it registered the A button as always being held down. 

## The Fixes
To make this the 2026 edition, I had to fixup the original repo: this required a bunch quite a bit of debugging, making some simple assembly changes, 
wrestling with Windows USB drivers, messing around with LUNA, emailing .hex files from my linux laptop back to my Windows PC... ya know, "hour or two" stuff.
