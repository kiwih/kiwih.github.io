---
layout: post
title: A CubeMX compatible SD card driver
subtitle: For use with FatFS!
gh-repo: kiwih/cubemx-mmc-sd-card
gh-badge: [star, fork, follow]
#cover-img: /assets/img/path.jpg
tags: [stm32, c]
---

The first time that I tried to get an SD card working with CubeMX I found a most interesting affair - not particularly complicated, but definitely a little fiddly. 

*2020 edit: [There is a more detailed and up to date version of this post](https://01001000.xyz/2020-08-09-Tutorial-STM32CubeIDE-SD-card/). Use it instead*.

At any rate, the goal was to get a STM32CubeMX-generated system integrated with FatFs middleware managing an SPI-connected MMC/SD memory card.

The project was initially created in CubeMX, and then code written by ChaN was ported to the CubeMX HAL.

If you're interested, [the project github](https://github.com/kiwih/cubemx-mmc-sd-card) has the relevant details (2020 edit: this may be out of date)  and interesting files, including a SPI driver at `Src/user_diskio_spi.c` and `Inc/user_diskio_spi.h` which should be usable in any CubeMX project (although the SPI handle `hspi1` may need to be changed depending on your requirements).

The CubeMX project file is `nucleo-sdcard.ioc` in the root directory. You can load this with CubeMX and change any of the settings and regenerate without affecting any of the code in this code base.




