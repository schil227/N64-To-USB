# N64-To-USB (2026 Version)

This code was forked from the [N64-To-USB Repo](https://github.com/MickMad/N64-To-USB) owned by one Michele Perla ("MickMad"). 
They combined the N64 controller protocol code from [Andrew Brown](http://www.instructables.com/id/Use-an-Arduino-with-an-N64-controller/) and 
the ArduinoJoystick HID/USB code from someone named Darren (dead link), dropped it in their github repo 14 years ago and never touched it again. This repo corrects some of the issues I observed.

# Info
## Disclaimer
Full disclosure, I'm writing this like a month after I did all the work, so I'm kinda hazy on the details.

I'm also quite tired, so this guide will assume a higher level of competency on the reader. If that pisses you off, imagine how I'll feel after I try to set this shit up after 3 years.

## What it does
- Supports 1 N64 controller as a Game Controller USB HID (Human Interface Device)
- Rumble/Save Cartridge unsupported

## Required
- N64 controller
- Arduino Uno (others will probably work, use at your own knowledge/risk)
- USB type B connector
- 3 wires to connect the controller to the Arduino board

If you wanna get fancy, you can invest in a female controller port (i.e. the port where you'd plug it in on the console), otherwise you may have to get creative with jamming some wires into the male controller plug - your milage may vary.

In addition, you'll need some software:
- [dfu-programmer](https://github.com/dfu-programmer/dfu-programmer/releases)
- (and maybe) [zadig](https://zadig.akeo.ie/)

## How to set it up (short)
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

Just kidding, I'm not that naive.

I took the N64 connect from the aforementioned busted converter, strategically googled a few things, and found MickMad's code. "Super" I thought. I assembled a few wires, 
and then without much ceremony, set everything up, and it kinda/sorta/not-really worked. 

## The Original Code
MickMad really appeared to cobble things together; iirc the HID stuff was built for a Gamecube controller (16 buttons, 2 joysticks), and a N64 controller 
has roughly half of that, so in his mind you could have two N64 controllers and apply them to the same input device (I guess?). The reality was, 
the ArduinoJoystick (HID USB device) was set up wrong; there were 32 buttons as inputs, the first 16 were based on the X/Y orientation of the joystick (so mostly chaos). The N64 controller interpreter was also not quite working, as it registered the A button as always being held down, and the Y axis on the joystick was inverted. 

# The Fixes
To make this the 2026 edition, I had to fixup the original repo: this required a bunch quite a bit of debugging, making some simple assembly changes, 
wrestling with Windows USB drivers, messing around with LUNA, emailing .hex files from my linux laptop back to my Windows PC... ya know, "hour or two" stuff.

## Controller Input
As mentioned, there were two problems: the A button was usually always registered as pressed, and the Y axis was inverted - but first, a bit on the protocol, and how controller data is obtained.

### The Code
Other places explain this much better than I'll attempt to ([1](https://www.scribd.com/document/675857081/N64-Controller-Protocol), [2](https://sites.google.com/site/consoleprotocols/home/nintendo-joy-bus-documentation/n64-specific/standardcontroller)), so I'll only shoot for a high-level explanation. As you can see, there's only 3 ports in the controller: power, ground, and data. But with 14 buttons and a control stick, you may be asking "how does all that data go through?" The answer is, simply, there's a protocol. 

First, the console sends a signal to the controller. It says "Hey, gimmie your state". It can say other things, but we're mainly interested in this. Then it waits for a response. The controller, in turn, responds with the state of all it's inputs, in a certain order (A, B, Z, Start, etc.). Thus communication is established. In order to actually sends the data across the line, the data is further "encoded". When the controller is giving the state, it's a series of 1s and 0s (32 of them, specifically). When it's a `1`, the line is high for 1us, low for 1us, then high again for 3us. When it's a `0`, the line is high for 1us, low for 3us, and then high for 1us. The takeaway from this is, there's a lot of stuff transmitted very fast.

That being the protocol, now we must interact with it. Andrew Brown, the original author of the arduino sketch (I suspect), created two functions: `N64_send(...)` and `N64_get(...)`. The sending function deals with making the request from the "console" (technically, the way it's implemented it can send _any_ of the permitted signals, however we're only using it for the one). The getting function must then rapidly "catch" all the output from the controller; it does this by writing the bits received into a byte array `N64_raw_dump` (effectively using 1/8th of the space available, but whatever). 

As mentioned, all this stuff has to happen very fast. Mr Brown made the (in my opinion, valid) choice of using assembly and gotos to make the code as _reliable_ as possible. Most of the code is spent "waiting" for 1us to pass; the microcontroller runs at 16MHz, meaning it performs 16,000,000 operations per second, or 16 cycles per microsecond (1us). Thus, he was able to read the assembly, figure out how many cycles the `N64_QUERY` opt cycle takes (2 cycles), then pad the remaining 14 cycles with `NOP` (No Operation) opt-codes (1 cycle). Thus, timing of everything is good. 

Anyways; the code calls `N64_send(...)` and `N64_get(...)`, and then we have the state of the controller stored in `N64_raw_dump`. This section of code is tight, and must be strictly performant, but after the data is in `N64_raw_dump` we can go back to our slow-ass c/c++ code and re-interpret the data into the `N64_status` `struct`. This `struct` contains the N64 controller state, and writes it out to the hardware serial (where, it's interpreted by the ArduinoJoystick HID USB). This runs on loop about 100 times per second

### Correcting the inverted Y axis
I'm actually not sure if the Y axis being inverted was intentional, or something that may have been incorrect downstream (i.e. in my Windows usb game controller app), but I figure if it's happening for me now, it'll probably happen in the future. Fixing it wasn't so bad, after I figured out that negative values are stored as 2's compliment. Basically it boiled down to this:

```c++
N64_status.stick_y |= N64_raw_dump[24+i] ? 0 : (0x80 >> i); 
```

This checks the last 8 bytes of data, and if the value from `N64_raw_dump` returned a 1, then the value of the byte at that position would be inverted. For example, `0001011` becomes `1110100`. This byte of data is then passed on to the HID, and it interprets the stick as the right-way-around.

### Fixing the A button
The tricker task was figuring out why the A button was nearly always interpreted as being "pressed". It's no co-incidence that it's the first button being sent, because the issue ultimately lived on the seam between `N64_send(...)` and `N64_get(...)`. At the end of `N64_send(...)`, the data line is brought _high_ by calling `N64_HIGH` (signaling the console is done with the data line, allowing the controller to respond). However there appears to be a bit of a race condition: `N64_get(...)` checks if the line is _low_ before reading the input; my theory is that the `N64_HIGH` command is not instantaneous; and there is a slight delay between going from _low_ to _high_. In that time, `N64_get(...)` already checks the state, reads it as low, and thinks that it's time to read the input - which shortly thereafter becomes _high_. 

And so, the fix is a fun one: tell the CPU not to do anything for 16 cycles. At the end of `N64_send(...)`:

```c++
    asm volatile ("nop\nnop\nnop\nnop\nnop\n"
                "nop\nnop\nnop\nnop\nnop\n"  
                "nop\nnop\nnop\nnop\nnop\nnop");
```

This extra bit of times makes it so `N64_get(...)` starts reading at the right time, and everything else works.

## Controller Output
As for the `_to_USB` half of this project, things were a little off. The provided ArduinoJoystick .hex file flashed without issue, however it was way off from a typical N64 Output. The HID (Human Interface Device) descriptor (the thing that explains what this device can do; e.g. it has a joystick, it has buttons, etc.) displayed the connected controller as having 32 buttons, the first 16 of which were driven by the X/Y joystick values. There was also some "rotation" output or something, not really applicable either. The goal with this was simply to make it correct.

### Setting up LUNA
What turned out to be the biggest pain in the ass with this project was, in fact, not messing around with ASM but just trying to build the ArduinoJoystick code (see N64_To_USB_Atmega16u2/). I don't know what version of LUNA the code had originally been built with (and, bare in mind, the link to the original code is dead) - so I had to take a best guess, and make it work. Going off of when MickMad had published this repo (~14 years ago at time of writing) I went with LUFA version 111009 (https://github.com/abcminiuser/lufa/tree/LUFA-111009). Even with that, I had to make numerous changes to the code to get it to compile; so I put on my big programmer pants and figured it out. If you have any aspirations to edit that stuff, then I suggest you do the same.

Finally after successfully running make, I found two things that needed to be fixed.

### Fixing the Descriptor object

Full disclosure, I know about this much 🤏 about how HID devices work, and the protocol and what-not; and to be honest, I caved and asked the copilot AI to tell me what was wrong. Afterwords I felt cheap, and foolish. Basically, there's a `ReportSize` and a `ReportCount`; the size is the size of the report data (8 bits to carry the joystick X values and 8 bits to carry the joystick Y values), while the count is the number of reports, which is 2 (one for X, one for Y). However the size was originally 4 - which makes sense if you're working with a fancy two-stick Gamecube controller, but not on the humble N64. This same change was needed for the button count; not 32 buttons as originally reported, but 16. 

Updating the descriptor `ReportSize` and `ReportCount` values was correct - but it didn't fix it. In fact, after that change nothing was working. This is because I was missing the other piece of this - updating the Report data source.

### Fixing the Descriptor object source
Remember way back at the beginning of this README when I mentioned that MickMad had "implemented" support for two N64 controllers? Well I took all that out; one because I was skeptical if it would work as intended, and two because I only had one controller port to plug into. That is, I almost took it all out, but I missed something:

```c++
typedef struct {
    int8_t  x1; /**< Current absolute joystick X position, as a signed 8-bit integer */
    int8_t  y1; /**< Current absolute joystick Y position, as a signed 8-bit integer */
    uint8_t button1a; /**< Bit mask of the currently pressed joystick buttons */
	uint8_t button1b; /**< Bit mask of the currently pressed joystick buttons */
	
	int8_t  x2; /**< Current absolute joystick X position, as a signed 8-bit integer */
    int8_t  y2; /**< Current absolute joystick Y position, as a signed 8-bit integer */
    uint8_t button2a; /**< Bit mask of the currently pressed joystick buttons */
	uint8_t button2b; /**< Bit mask of the currently pressed joystick buttons */
} USB_JoystickReport_Data_t;
```

This guy 64 bits - twice as big as it should be. See, this is the recipient of the stream written data from the sketch I mentioned above, What happens when you only write half the values? Well honestly this was over a month ago so I no longer remember, but the answer boils down to, shit doesn't work right. Updating that `struct` to remove the `2` properties fixed it - and finally, my two week 1 to 2 hour task was complete.

# Conclusion 
I almost didn't do any of this (making the fork and uploading my changes and this README), partially because 95% of it isn't "my code", and I barely even understand the majority of it - but ultimately, this is useful. Not just useful, it's _good_, it's the "right thing to do". So much of the world goes to rot - projects that could be saved if only _somebody_ would care to. Maybe I read The Lorax to my kids too much, or maybe I'm giving myself too much credit, idk. But in closing, I'm tired, and yeah.

