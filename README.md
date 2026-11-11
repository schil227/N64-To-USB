# N64-To-USB (2026 Version)

This code was forked from the [N64-To-USB Repo](https://github.com/MickMad/N64-To-USB) owned by one Michele Perla ("MickMad"). 
They combined the N64 controller protocol code from [Andrew Brown](http://www.instructables.com/id/Use-an-Arduino-with-an-N64-controller/) and 
the ArduinoJoystick HID/USB code from someone named Darren (dead link), dropped it in their github repo 14 years ago and never touched it again. This repo corrects some of the issues I observed.

## Info
### Disclaimer
Full disclosure, I'm writing this like a month after I did all the work, so I'm kinda hazy on the details.

I'm also quite tired, so this guide will assume a higher level of competency on the reader. If that pisses you off, imagine how I'll feel after I try to set this shit up after 3 years.

### What it does
- Supports 1 N64 controller as a Game Controller USB HID (Human Interface Device)
- Rumble/Save Cartridge unsupported

### Required
- N64 controller
- Arduino Uno (others will probably work, use at your own knowledge/risk)
- USB type B connector
- 3 wires to connect the controller to the Arduino board

If you wanna get fancy, you can invest in a female controller port (i.e. the port where you'd plug it in on the console), otherwise you may have to get creative with jamming some wires into the male controller plug - your milage may vary.

In addition, you'll need some software:
- [dfu-programmer](https://github.com/dfu-programmer/dfu-programmer/releases)
- (and maybe) [zadig](https://zadig.akeo.ie/)

### How to set it up (short)
- Download repo
- Connect the controller to 3.3v, ground, and Digial pin 2
	- If you're lookin' at the controller plug, like you would stick it into your eye, it's (from left to right) Ground, Data, Power. But don't take my word for it, google it ya dummy.
- Upload Arduino sketch to Arduino Uno
- Boot Arduino into Device Mode (google it)
- Download DFU Programmer, run the following (in this case, I'm in the same directory as the dfu executable)
	- `.\dfu-programmer.exe atmega16u2 erase`
	- `.\dfu-programmer.exe atmega16u2 flash path_to_repo_code\N64-To-USB\N64_To_USB_Atmega16u2\Arduino-joystick.hex`
	- `.\dfu-programmer.exe atmega16u2 reset`
		- To turn your Arduino back into an Arduino, go and find `Arduino-usbserial-atmega16u2-Uno-Rev3.hex` wherever your Arduino stuff is installed, and repeat the above (google it)
- Observe your N64 controller is now a USB game controller\* (e.g. in Windows, check out "setup game controller", find one called "Arduino Joystick")

\*You _may_ have to change the device into a WinUSB device, which would require something like zadig

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
