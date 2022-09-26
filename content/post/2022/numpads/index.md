---
title: "Numpads"
date: 2022-09-25T21:15:15+02:00
draft: false
author: Dennis Kruyt
categories:
- keyboards
cover:
  image: /images/2022/09/front-pad.jpg
description: ""
slug: numpads
summary: Building my own mechanical numpds
tags:
- keyboards
- keyboard
- qmk
- mechanical
title: Building mechanical numpads
---

I am always fascinated about mechanical keyboards. And a couple of weeks ago I had some spare time left. So I thought lets make my own keyboard from scratch from stuff that I had laying around en my 3d printer. Well maybe a full keyboard is to big, so let's start with something smaller. Why not a numpad, that goes along with my 65% keyboard?

The end result are 2 numpad, yhea could stop with one ;) One numpad with a function row and RGB underglow. And a bigger numpad wit 2 function rows and a OLED display.  

{{< figure src="/images/2022/09/rgb-numpad.jpg" caption="rgb numpad" >}}
{{< figure src="/images/2022/09/oled-mpad.jpg" caption="oled numpad" >}}

### 3D design

I started with looking around for existing 3d designs. I found one one tinkercad from FedorSosin. I remix the top plate to add function rows and a place for a oled display. This I did in tinkercad. The bottom/base part is a new design that also will fit a Raspberry Pico Pi. For the RGB one I sliced in a extra layer that I printed with transparent PLA and glued it to the top plate.

[Download STL files](/files/numpads.zip "download")


{{< figure src="/images/2022/09/3d-rgb.png" caption="3d design RGB" >}}
{{< figure src="/images/2022/09/3d-oled.png" caption="3d design OLED" >}}
{{< figure src="/images/2022/09/3dprinter.jpg" caption="3D printing in action" >}}

### Components
BOM of stuff that I used (had laying around) to create these numpads.
- Gateron brown switches
- RP2040, Pico Pi clone with USBC
- 0.96 inch OLED Display 128*x64
- WS2812 RGB LED strip
- PLA black and transparent (for RGB galore)
- Diodes 1n4148
- 5mm LED's
- 470 Ohm resistors for the LED's
- Shrinking tube
- Random wiring

### Handwire

I handwired the switches, not the best soldering job, but for a prototype it's working fine. handwiring can look daunting, but when you are into it, it's pretty easy. 

{{< figure src="/images/2022/09/wiring-oled.jpg" caption="Wiring OLED" >}}

{{< figure src="/images/2022/09/wiring.png#center" height="480" caption="Wiring Diagram" >}}

### QMK Firmware

{{< figure src="https://qmk.fm/assets/images/badge-dark.png#center" height="100" >}}

The RP2040 is now supported in QMK firmware. KMK should also work, but with KMK I could get the OLED display to work. So I used QMK with VIA support.

You can find the QMK files on my [Github page](https://github.com/dkruyt/qmk_firmware/tree/phantagom-pads/keyboards/phantagom). 

I am using also layer support, the keypad also doubles as a mouse, navigations and macro's. The oled gives info about the lock status and which layer I am on.

{{< figure src="/images/2022/09/oled-display.jpg#center" height="260" caption="OLED Display" >}}

### Desktop

Now I have a companion for my 65% keyboard.

{{< figure src="/images/2022/09/gammaykay-mpad.jpg" caption="65% keyboard and the macro numpad" >}}

