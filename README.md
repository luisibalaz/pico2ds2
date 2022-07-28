# pico-ds
An xinput to DualShock(2) "translator" written for the Raspberry Pi Pico. Source code release pending.

It currently only emulates a Digital Pad. A test build is provided under the `build` directory.

This Pi Pico firmware uses the TinyUSB modified host stack to read inputs from Xbox controllers, using tusb_xinput, an implementation of xinput for the Pi Pico. However, this is not perfect. These issues are described below.

It also utilizes the powerful PIO feature of this MCU, to emulate the communication between the PS1/PS2 and the controllers.

This software is delivered as-is, and comes with no warranty. Do not make this mod if you don't have enough experience. I take no resposability for the things you mess up.

---

## Connections Schematic and parts required

The currently required parts are listed here:
* A voltage regulator to get 5V from the 7.5V (or 9V?) motor power line from the PS1/PS2. USB peripherals **need** 5V.  
The cheapest option is a LM7805 linear regulator. This is however inefficient and will produce a lot of heat if using wired controllers, but it will fit nicely in a small custom PCB and case.  
A more expensive option is a switched buck converter. Much more efficiente but big and bulky.  
But be very careful. There is a fuse on this line, that prevents more than 700mA (if I remember correctly) to be drawn. I take no responsability if something goes wrong.
* Jumper wires to connect everything.
* A micro USB OTG cable (or USB C OTG, depending on your Pico USB plug).
* (Optional) Resistors and LEDs for some visual cues.
* (Optional) USB to serial converter to see Debug information.
* (Optional but very recommended) A pushbutton to reboot the Pico every time it freezes, connected between a GND pin and the Run pin.

Maybe I (or someone) will design a custom PCB to accommodate everything in a small and nice case.

### Schematic

This is a quickly drawn schematic, but I hope it is informative. Double and triple check your very own LM7805 pinout.

![schematic](img/pico-ds.png "Schematic")

More information on the connection of things will be added later.

---

## Controllers notes

### PS1 Digital Pad

This controller, also known as DualDigital, only responds to command `0x01 0x42 ...` (check [references](#references) for more information). That means, that when a PS2 tries to enter config mode `0x01 0x043`, the controller does not acknowledge this, and then the PS2 recognizes it as a Digital Pad.

Here we can see a `0x42` command communication, and the ACK line go low after every (except the last) byte received by the controller.

![0x42 command response](img/dualdigital0x42.png "0x42 command response")

Here we can see a `0x43` command communication, and the ACK does NOT go low after the `0x43` byte is received by the controller, and communication ends.

![0x43 command response](img/dualdigital0x43.png "0x43 command response")

### Og Xbox Controller

This controller does not use a standard USB port, but the communication protocol is indeed a standard USB, and we only need an adapter (recommended, can always come back to use it with the Xbox), soldering a USB cable to the controller motherboard (not recommended) or cut the cable and solder a USB male plug to that (even less recommended), to get it ready to use.

My favorite method is to use an Xbox 360 USB Breakaway cable, and sand the plastic inside the "circular part" of this cable, enough to fit the Og Xbox part. If this is not clear enough, I'll be happy to make a small tutorial in this repo.

### Xbox One Pad initialization

I'm having trouble initializing Xbox One (S, specifically) every time. After plugging it in, nothing happens, USB can't even be used to give it power. You will need a power supply capable of more (milli) amps. The LM7805 should be enough.

---

## Some USB issues

There are a couple of problems that come with the use of TinyUSB and tusb_xinput.

### USB Hubs

They currently can't be used. This a big problem because OG Xbox pads present themselves as USB Hubs, and the controller part itself is a device connected to said Hub. This also limits this program to be used ONLY as a single USB device adapter, and my goal is to make it be able to use a Hub, so that you need a single Pico for both PS1/PS2 controller ports.

**Possible causes and solutions**

* If this is a TinyUSB problem, maybe a future update will get things fully working.
* If this is a tusb_xinput issue, maybe taking a deeper look at the Xinput implementation will get things properly working.

### Time interval between reports
This could an tusb_xinput issue, and could be a big one, because this time interval is around 8-10ms, which could slow things down for up to a frame, depending on when the button is pressed and when the press is reported.

**Possible causes and solutions**

* If this is a TinyUSB problem, maybe a future update will get things fully working.
* If this is a tusb_xinput issue, maybe taking a deeper look at the Xinput implementation will get things properly working.
* If this is a hardware limitation, nothing could be done.

For both problems, maybe writing a lower lever xinput driver will make things work much better but will be way harder and take much longer to have it finished.

---

## Source code notes:

Source code will be released once I reorganize it and get Left Joystick to DPad mapping working, and get some reconnection bugs fixed.

### File `psxSPI.pio`

This file has been taken from [PicoMemcard's pmc+ development branch](https://github.com/dangiu/PicoMemcard/blob/pmc%2B/development/psxSPI.pio) and not the main branch, because it seems that it is a bit more stable.

### TinyUSB modified stack

Follow instructions from https://github.com/Ryzee119/tusb_xinput.

Specifically, this commit: https://github.com/Ryzee119/tinyusb/commit/4a61b7ac61c7cfefb80a3abfd0891adf105c545c, tells you to modify files `src/host/usbh.c` and `src/tusb.h` to get things working.

### The tusb_xinput driver

A custom xinput driver written by Ryzee119 (https://github.com/Ryzee119/tusb_xinput) is used as a base for the currently working version. Further modifications will be requiered to adapt it to this specific case.

---

## References

Most documentation used is already available online:

* For information about PS1/PS2 "SPI" communication protocol:
  * https://psx-spx.consoledev.net/controllersandmemorycards
  * http://www.emudocs.org/PlayStation/psxcont/
  * https://store.curiousinventor.com/guides/PS2
  * https://gist.github.com/scanlime/5042071
  * https://github.com/Lameguy64/PSn00bSDK

* For information about Xbox/Xbox360/XboxOne USB descriptors and data format:
  * https://github.com/paroj/xpad
  * https://www.partsnotincluded.com/understanding-the-xbox-360-wired-controllers-usb-data/
  * https://github.com/medusalix/xone
  * https://github.com/quantus/xbox-one-controller-protocol

* `psxSPI.pio` code taken from PicoMemcard (pmc+/development branch): https://github.com/dangiu/PicoMemcard/tree/pmc+/development

* A modified version of TinyUSB (https://github.com/hathach/tinyusb) is used as the host stack for the Pi Pico, [see above](#tinyusb-modified-stack).

* An xinput implementation for the Pi Pico: https://github.com/Ryzee119/tusb_xinput