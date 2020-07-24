---
layout: post
title: 'Updating the Firmware on an Anet A8 (Mainboard Version 1.7)'
subtitle: Steps for 2020
#share-img: /assets/img/taniwha-and-ladders/base-game.jpg
tags: [walkthrough]
---

For the first time ever, I have a 3d printer! Well, it's not actually *mine*, but it's on my desk - it's mine to use (and abuse).

![The printer]({{ 'assets/img/anet-a8/the-printer.jpg' | relative_url }}){: .mx-auto.d-block :}

It came as a kitset from Amazon, and I had to assemble it myself. This wasn't too tricky, but ... (short story)

![Assembling the printer 1]({{ 'assets/img/anet-a8/assembly-1.jpg' | relative_url }}){: .mx-auto.d-block :}

![Assembling the printer 2]({{ 'assets/img/anet-a8/assembly-2.jpg' | relative_url }}){: .mx-auto.d-block :}

![Assembling the printer 3]({{ 'assets/img/anet-a8/assembly-3.jpg' | relative_url }}){: .mx-auto.d-block :}

Once finished, I needed to calibrate it, which was its own journey. (Couple of points, couple of pix of boats).

I had problems with stringing, but I got it to the point where I was happy enough.

For a task at work I now had to set to one of the goals of this 3d printer - being able to update the firmware, and that takes us to the meat of this walkthrough.

# Updating the Firmware

There are (link)plenty (link)of (link)tutorials (link)on (link)updating the (link)firmware for me to find, searching the internet in 2020. 
https://blog.thomasmarcussen.com/anet-a8-plus-upgrade-to-marlin-2-0-x/
https://all3dp.com/2/anet-a8-firmware-which-to-choose-and-how-to-change-it/
https://github.com/SkyNet3D/anet-board
https://www.youtube.com/watch?v=ePgpzkjriso
https://ktln2.org/2018/08/13/update-marlin-fw-for-anet/

However, plenty of them leave open-ended questions around _'does this work with the 1.7 revision of the board?'_, or '_is this Github still maintained?'_

Other issues too:
https://reprap.org/forum/read.php?415,861799
https://www.thingiverse.com/groups/anet-a8-prusa-i3/forums/general/topic:35035
https://reprap.org/forum/read.php?1,856698
https://github.community/t/flashing-marlin-2-0-bugfix-on-an-anet-a8-3d-printer/12682
https://www.reddit.com/r/AnetA8/comments/egwenl/flashing_marlin_issues_with_x_min_pin_is_not/


In the end, these were the steps:

1. (Optional) Using Usbasp, back up the existing firmware
2. (Optional) Using Usbasp, burn a new bootloader (I didn't need to do this)
3. Get Anet board definition from https://github.com/benlye/anet-board/raw/master/package_anet_board_index.json
4. Download https://github.com/MarlinFirmware/Marlin/tree/bugfix-1.1.x and apply Anet-A8 configuration
5. Change the configuration to disable interrupts by commenting `//#define ENDSTOP_INTERRUPTS_FEATURE` and disable EEPROM (so it doesn't overwrite the original, which I expect uses different format) (note that some people say PID needs further tuning but for me the values were very close to EEPROM so I left them and they were perfect!)
6. Compile + Download firmware
7. Recalibrate the z-heights (physical) on the board
8. Print!

# Improvements in quality

Compare the two boaties

# Conclusions

Not too difficult and some quick wins in terms of print quality!