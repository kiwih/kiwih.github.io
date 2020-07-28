---
layout: post
title: 'Updating the Firmware on an Anet A8 (Mainboard Version 1.7)'
subtitle: The steps, in 2020
share-img: /assets/img/anet-a8/the-printer.jpg
tags: [walkthrough, tooling, 3dprinting, embedded]
---

For the first time ever, I have a 3d printer! Well, it's not actually *mine*, but it's on my desk - it's mine to use (and abuse).
I wanted to update the firmware on it, but was worried about bricking it. 
This blogpost is about the procedure I went through to update it to Marlin 1.1.9.

TL;DR: If you're just interested in the steps, scroll down to 'Updating the Firmware'.

# Background on the Anet A8

The Anet A8 is a relatively inexpensive 3d printer you can buy from e.g. [Amazon](https://amzn.to/305Kk4r).  

![The printer]({{ 'assets/img/anet-a8/the-printer.jpg' | relative_url }}){: .mx-auto.d-block :}

It comes as a Kitset, which I had great fun unboxing...

![Assembling the printer unboxing]({{ 'assets/img/anet-a8/unboxing.jpg' | relative_url }}){: .mx-auto.d-block :}

...and then assembling.

![Assembling the printer 1]({{ 'assets/img/anet-a8/assembly-1.jpg' | relative_url }}){: .mx-auto.d-block :}

![Assembling the printer 2]({{ 'assets/img/anet-a8/assembly-2.jpg' | relative_url }}){: .mx-auto.d-block :}

Once finished, it was pretty easy getting started on the first print,

![The first print]({{ 'assets/img/anet-a8/first-print.jpg' | relative_url }}){: .mx-auto.d-block :}

but it was pretty clear that I needed to calibrate it - just look at that stringing!

I spent a lot of time with the calibration guides provided with the printer, changing extrusion speed/distance, printer temperature, movement acceleration, and so forth, but I only ever seemed to improve the print quality a little. Well, until I had a go at updating the firmware that is. Then, with absolutely no changes to the gcode between print 3 and print 4 (and a different dye in the filament) I managed to get a significant improvement!

![4 boats]({{ 'assets/img/anet-a8/4-boats.jpg' | relative_url }}){: .mx-auto.d-block :}

# So many online resources

There are [plenty](https://blog.thomasmarcussen.com/anet-a8-plus-upgrade-to-marlin-2-0-x/) [of](https://all3dp.com/2/anet-a8-firmware-which-to-choose-and-how-to-change-it/) [resources](https://github.com/SkyNet3D/anet-board) [online](https://www.youtube.com/watch?v=ePgpzkjriso) [regarding](https://ktln2.org/2018/08/13/update-marlin-fw-for-anet/) [updating](https://www.thingiverse.com/groups/anet-a8-prusa-i3/forums/general/topic:28969) the firmware on an Anet A8 in 2020. 

However, most of them leave open-ended questions around _'does this work with the 1.7 revision of the board?'_, or '_is this Github still maintained in 2020?'_

For instance, these are just a subset of the issues I found when checking the procedure would work:
* [Problem with compilation](https://reprap.org/forum/read.php?415,861799)
* [More problems with compilation](https://github.community/t/flashing-marlin-2-0-bugfix-on-an-anet-a8-3d-printer/12682)
* [Yet more problems with compilation](https://www.reddit.com/r/AnetA8/comments/egwenl/flashing_marlin_issues_with_x_min_pin_is_not/)
* [Issue: new firmware printing off center](https://www.thingiverse.com/groups/anet-a8-prusa-i3/forums/general/topic:35035)
* [Problems with homing after updating](https://reprap.org/forum/read.php?1,856698)

I started feeling the same way as [DarkTerritory, here](https://www.thingiverse.com/groups/anet-a8-prusa-i3/forums/general/topic:28969), who started a forum post in 2018 with _"K so I'm thinking abut trying to upgrade my old Anet A8 motherboard to Marlin. I have been looking around on the intertubes and now I'm more confused than ever. Everyone uses a different flavor of Marlin and no one does the update in the same way. Many also seem to leave out important steps."_

In the end they did resolve their issue, but again left the steps largely as an exercise for the reader.

# Updating the firmware

In the end, these were the steps that I performed.:

1. (Optional) Using Usbasp, which you can buy e.g. from [Amazon](https://amzn.to/2X4I3EU), back up the existing firmware with (On Ubuntu) `sudo apt install gcc-avr avr-libc binutils-avr avrdude`, then `avrdude -c usbasp -p m1284p -U flash:r:aneta8-flash.bin:r`
2. (Optional) Using Usbasp, burn a new bootloader (I didn't need to do this)
3. Get Anet board definition from https://github.com/benlye/anet-board/raw/master/package_anet_board_index.json
4. Download https://github.com/MarlinFirmware/Marlin/tree/bugfix-1.1.x and apply Anet-A8 configuration
5. Change the configuration to disable interrupts by commenting `//#define ENDSTOP_INTERRUPTS_FEATURE` and disable EEPROM (so it doesn't overwrite the original, which I expect uses different format) (note that some people say PID needs further tuning but for me the values were very close to EEPROM so I left them and they were perfect!)
6. Compile + Download firmware
7. Recalibrate the z-heights (physical) on the board
8. Print!

# Conclusions

Not too difficult and some quick wins in terms of print quality!