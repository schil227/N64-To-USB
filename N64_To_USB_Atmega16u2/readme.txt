Note: This code compiles for LUFA version 111009 (https://github.com/abcminiuser/lufa/tree/LUFA-111009). Getting it to work with 
other versions will likely be a big pain in the ass.

To setup the project and upload the Arduino usbserial application firmware to an ATMEGA8U2 using the Arduino USB DFU bootloader:
1. unpack the source into LUFA's Projects directory
2. set ARDUINO_MODEL_PID in the makefile as appropriate
3. do "make clean; make"
4. put the 8U2 into USB DFU mode:
4.a. assert and hold the 8U2's RESET line
4.b. assert and hold the 8U2's HWB line
4.c. release the 8U2's RESET line
4.d. release the 8U2's HWB line
5. confirm that the board enumerates as either "Arduino Uno DFU" or "Arduino Mega 2560 DFU"
6. do "make dfu" (OS X or Linux - dfu-programmer must be installed first) or "make flip" (Windows - Flip must be installed first)

