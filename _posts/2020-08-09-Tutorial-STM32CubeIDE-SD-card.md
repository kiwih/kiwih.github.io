---
layout: post
title: 'Tutorial: An SD card over SPI with an STM32 using STM32CubeIDE and FatFS'
subtitle: Let's have some fun with files
gh-repo: kiwih/cubeide-sd-card
gh-badge: [star, fork, follow]
share-img: /assets/img/cubeide-sd-card/module-wired.png
tags: [tutorial, stm32, STM32CubeIDE, embedded, c]
---

Embedded projects often call for some kind of data storage and retrieval. 
For instance, you might have need for a data logger to record data points over long periods of time.
You might have user settings that need to be preserved after the device is shut off and rebooted.
You might want to profile your application, or debug certain features.

There are a number of ways to do this, and one of my favourites is to add an SD (or other memory) card to a project. 
It's not too complicated, and it adds a lot of flexibility, as you can easily take the SD card out of the project and examine it with another machine.
My second favourite method is to slap on a 20 cent EEPROM, but while that's cheaper it's a lot less flexible and usually stores a lot less data!

I've been asked more than once if I have any good tutorial resources for adding SD cards to a project. 
Well, there are a few good tutorials and resources floating around online (including even [my own resource from 2017](https://01001000.xyz/2017-08-17-CubeMX-SD-card)) but some of them are a bit dated, relying on older tools and libraries.
Further, while there are some _amazing_ resources (e.g. [ChaN's](http://elm-chan.org/docs/mmc/mmc_e.html)) on talking to SD cards over SPI, there are less that describe how to interface this with file systems such as FAT, and less again that describe how to do it while also working with STM32's build environments.
Complicating matters is that officially you should use the STM32 SDIO peripheral to interface with an SD card - however, not all STM32s have the SDIO peripheral, leaving us to fall back on the SPI method (That said, it is worth noting that *not all SD cards support the SPI interface*).

Well, in this tutorial I'm going to walk through the steps that one would use to get an SD card working over the SPI interface on a STM32 dev board (re-)using my FatFS driver from 2017. 
FatFS is an amazing open source project [also provided by ChaN](http://elm-chan.org/fsw/ff/00index_e.html) which has since been integrated into the STM32Cube tools.
If you're interfacing with an SD card using the SDIO peripheral, it's pretty easy and the tooling does most of it for you.
If you're working with other kinds of configurations, e.g. SD card over SPI, it's actually still pretty easy - you just need the appropriate driver!
So, today I want to show how you can use the FatFS libraries within the STM32CubeIDE development environment, and show how you can simply drop in the appropriate SPI driver to make everything work.

I'll be assuming that you already know the basics of creating a project and setting up debugging and so on.
If not, then please [check out this earlier tutorial, in which I walk through getting started with STM32CubeIDE](https://01001000.xyz/2020-05-11-Tutorial-STM32CubeIDE-Getting-started/).

# Equipment for this tutorial

Today I will be using the following:

* (Free) Ubuntu Linux 20.04 because developing on Linux makes me happy and Ubuntu makes it easy. Don't worry if you're on Windows, you should be able to follow along with roughly the same steps. 
* (Free) STM32CubeIDE
* ($27.26 from [Amazon Prime](https://amzn.to/2BDK6ID), $15.05 from [Amazon](https://amzn.to/3gEp6AA)) The `Nucleo-F303RE` development board.
* ($5 from [Amazon Prime](https://amzn.to/3ik1wJV)) An SD card breakout board (comes in a pack of two).
* ($6 from [Amazon Prime](https://amzn.to/33GRwXe)) Easy-to-use ribbon cables (there's more than you need here but they're handy to have around).
* A micro SD card - note that *not all SD cards will work in SPI mode*. I have an Apacer one that works, and a Kingston one that does not. YMMV.

_Note: The above Amazon links are affiliate links. As always I encourage you to shop around, but Amazon usually has pretty good pricing._

# The software stack

In this blog post I'm not terribly interested in the low-level behaviour we're using to get an SD card working beyond "we talk to it over SPI". 
It's worth knowing though, so go [check out what ChaN wrote](http://elm-chan.org/docs/mmc/mmc_e.html), and then come back.
The key knowledge that I want to show in this tutorial is around the architecture of an embedded application that wants to use an SD card with a FAT file system (using the FatFS library).

In general it's always useful to visualise the architecture of what you are working with. In a FatFS system it looks like this:

![Architecture with FatFS]({{ 'assets/img/cubeide-sd-card/stack.png' | relative_url }}){: .mx-auto.d-block :}

FatFS is provided as a _Middleware_ which can translate FAT file structures in memory into their actual files. Handy!
For it to do its magic, it needs access to a storage medium. It relies on several functions as ChaN notes here:

![FatFS functions]({{ 'assets/img/cubeide-sd-card/fatfs-functions.png' | relative_url }}){: .mx-auto.d-block :}

# Setting up the Project and Pins

1. Open STM32CubeIDE.
2. Start a new project for the `Nucleo-F303RE` dev board (or w/e you're using) called something sensible e.g. _cubeide-sd-card_.
3. Answer 'Yes' to _Initialize all peripherals in their default configuration?_.
4. Answer 'Yes' to _Open device configuration view?_. 

The Device Configuration View is where you configure exactly which pins/peripherals are enabled and what their settings are.
Since we want to be connecting to an SD card, we need to enable an SPI port and then decide where to wire it.

Let's quickly work out where our pins are going. Our goal here is to identify an SPI peripheral with easy-to-access pins as well as a GPIO pin to use as a _chip select_.

On the `Nucleo-F303-RE` we have both Arduino style headers as well as ST's branded _morpho_ headers, which are the double rows of pins down each side. 
While it's tempting to use the Arduino header's SPI port (since it's labelled on the silk screen) I actually don't like to, as the on-board LED shares one of the pins (one of the worst features of this particular development kit).
So, instead I will take a look at the morpho header pinouts.

[This document](https://www.st.com/resource/en/user_manual/dm00105823-stm32-nucleo-64-boards-mb1136-stmicroelectronics.pdf) from ST provides us with the correct pinout for the F303-RE, or alternatively (and in a more attractive and detailed way) the same info is presented on ST's mbed OS website [here](https://os.mbed.com/platforms/ST-Nucleo-F303RE/).

Straight away I can see that SPI2 is the winner - it is broken out onto the pins in the bottom right, along with PB1 which we will use for the chip select line.

![SPI2 Pins]({{ 'assets/img/cubeide-sd-card/nucleo_f303re_morpho_spi2.png' | relative_url }}){: .mx-auto.d-block :}

So, let's set up our SPI2 and GPIO in the Device Configuration View. Click SPI2 on the left, and then set it to Full Duplex Master with no Hardware NSS.
Then set the Data Size to 8 bits, and the clock prescaler to 128 (SD cards start up with low speeds and switch to higher speeds later. We'll look at how to do this soon).

![SPI2 Config]({{ 'assets/img/cubeide-sd-card/cubeide-spi2-setup.png' | relative_url }}){: .mx-auto.d-block :}

Then, create your Chip Select line on PB1 - I find setting a sensible name is also good:

![SD_CS Config]({{ 'assets/img/cubeide-sd-card/cubeide-sd-cs-setup.png' | relative_url }}){: .mx-auto.d-block :}

Finally, as we're going to be using the SD card with the FAT file system, scroll down in the device categories to `Middleware`, and expand this, then enable `FATFS` as `User-defined`. You may leave all other parameters as their defaults.

![FATFS Config]({{ 'assets/img/cubeide-sd-card/cubeide-fatfs-setup.png' | relative_url }}){: .mx-auto.d-block :}

Now save your Device Configuration, and when it asks, 'Yes' to _Do you want to generate Code?_ and 'Yes' to _Do you want to open [the C/C++] perspective now?_.

# Wiring the SD card adapter

Now that you have configured the pins in CubeIDE, we need to physically wire them in real life!

Using the SPI pins from the earlier figure, and the power pins depicted here,

![Power Pins]({{ 'assets/img/cubeide-sd-card/nucleo_f303re_morpho_power.png' | relative_url }}){: .mx-auto.d-block :}

Use your ribbon cables and connect these to the appropriate pins on the SD card adapter module (the module's pins are labelled on the silk screen, so this isn't much of a chore).

![SD adapter silkscreen labels]({{ 'assets/img/cubeide-sd-card/module-labels.png' | relative_url }}){: .mx-auto.d-block :}

In table form, the connections are as follows:

| SD Adapter side | `Nucleo-F303RE` side |
| :-------------- | :------------------- |
| CS              | PB1 (GPIO SD_CS)     |
| SCK             | PB13 (SPI2 SCLK)     |
| MOSI            | PB15 (SPI2 MOSI)     |
| MISO            | PB14 (SPI2 MISO)     |
| VCC             | 5V                   |
| GND             | GND                  |

Once you're finished it should look something like this:

![Wired up]({{ 'assets/img/cubeide-sd-card/module-wired.png' | relative_url }}){: .mx-auto.d-block :}

# Key files to make this work



* FatFS needs a driver structure. CubeIDE makes us one to fill in. We're going to link this to our own file by providing some function names.
* Modifications in `FATFS/Target/user_diskio.h`
* Copy in `Core/Src/user_diskio_spi.c`
* Copy in `Core/Inc/user_diskio_spi.h`
* A couple of `#define` in `main.h`.

# Testing and correct output.