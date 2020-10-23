---
layout: post
title: 'Tutorial: Timers and PWM (and a cheeky AM radio transmission) using STM32CubeIDE'
subtitle: 
gh-repo: kiwih/cubeide-timers-demo
gh-badge: [star, fork, follow]
share-img: 
tags: [tutorial, stm32, STM32CubeIDE, embedded, c]
---

Measuring, monitoring, and reacting to the passage of time in embedded systems is an ubiquitous requirement.
For instance, you might decide that you want to toggle an output every millisecond.
You might be tasked to implement a system that samples an ADC once a second.
You might want to change your `main()`'s `while(1)` to feature a delay such that it becomes a loop with a fixed period.

In order to achieve this, you must become proficient with one of the most useful peripherals you'll ever encounter within your microcontrollers: Timers.
These are embedded counters which measure the passage of time as a function of counting microcontroller clock ticks (you can actually get them to count other things as well, for instance pulses on an external pin).

Setting up timers, however, can be a little tricky - especially in microcontrollers as capable as the STM32F series. 
So, in this, my third tutorial using STM32CubeIDE, I'm going to walk through a quick explanation of how we can get started with some of the most common use cases, and I'll finish with a (very silly) demo application where we (ab)use a timer to produce an AM radio signal.

I'll be assuming that you already know the basics of creating a project and setting up debugging and so on.
If not, then please [check out this earlier tutorial, in which I walk through getting started with STM32CubeIDE](https://01001000.xyz/2020-05-11-Tutorial-STM32CubeIDE-Getting-started/).

So what are we waiting for? Let's get started!

TL;DR: Timers. The complete code project is available [here](https://github.com/kiwih/cubeide-timers-demo).

# Equipment for this tutorial

Today I will be using the following:

* (Free) Ubuntu Linux 20.04 because developing on Linux makes me happy and Ubuntu makes it easy. Don't worry if you're on Windows, you should be able to follow along with roughly the same steps. 
* (Free) STM32CubeIDE
* ($27.26 from [Amazon Prime](https://amzn.to/2BDK6ID), $15.05 from [Amazon](https://amzn.to/3gEp6AA)) The `Nucleo-F303RE` development board.
* ($6 from [Amazon Prime](https://amzn.to/33GRwXe)) Easy-to-use ribbon cables (there's more than you need here - to be honest, you'll need only one, but they're handy to have around).

_Note: The above Amazon links are affiliate links. As always I encourage you to shop around, but Amazon usually has pretty good pricing._

# Setting up the Project

1. Open STM32CubeIDE.
2. Start a new project for the `Nucleo-F303RE` dev board (or w/e you're using) called something sensible e.g. _cubeide-timers-demo_.
3. Answer 'Yes' to _Initialize all peripherals in their default configuration?_.
4. Answer 'Yes' to _Open device configuration view?_. 

The Device Configuration View is where you configure exactly which pins/peripherals are enabled and what their settings are.
For now we'll leave it at its defaults, but let's quickly note the hardware available to us:

/picture of LED pin/

As you can see there is a push button (called ...) at pin ... and an LED called LD2 at pin ... .
We also have a UART at ..., which connects to the programmer's virtual COM port (if you're unsure about this, do [check out my earlier tutorial](https://01001000.xyz/2020-05-11-Tutorial-STM32CubeIDE-Getting-started/) where I discuss it in some detail).

For now, let us head over to the code window. We're going to make that LED blink to get an idea of how the timer can be made to be useful.
Navigate using the project explorer on the left to `main.c`.

/ picture of navigator /  

# The Basics: Blink an LED using delay() 

* Show using delay() to begin with to blink LED, then make custom delay function

# The Basics: Measure time using a custom delay function

* Configure Timer1 using the setup window.

# A timer interrupt

* Replace delay() with interrupts to blink LED using Timer2
* Add more interrupts to show we're doing it in parallel (using breadboard?)

| SD Adapter side | `Nucleo-F303RE` side |
| :-------------- | :------------------- |
| CS              | PB1 (GPIO SD_CS)     |
| SCK             | PB13 (SPI2 SCLK)     |
| MOSI            | PB15 (SPI2 MOSI)     |
| MISO            | PB14 (SPI2 MISO)     |
| VCC             | 5V                   |
| GND             | GND                  |

* Configure Timer3 to have a UART output every 1s. 

# Segue: A watchdog interrupt

* Reboot

# Generating PWM:

* Delete Timer2/3 interrupt/code.

* The basics, talk about theory of PWM

* Blink LED using Timer2 PWM.

# Putting it together: Let's make an AM radio transmission

* AM radio format + carrier waves etc

Adding a pin:

Since we want to be emitting a radio signal, we're going to need some kind of antenna.
We'll make this very easy: We'll use a single wire connected to one of the microcontroller's pins that has PWM capability. 

So, let's quickly see what our pin options are. 

On the `Nucleo-F303-RE` we have both Arduino style headers as well as ST's branded _morpho_ headers, which are the double rows of pins down each side. 
For now, I'm going to use the pin that is labelled PWM/D9 on the silk screen.

[This document](https://www.st.com/resource/en/user_manual/dm00105823-stm32-nucleo-64-boards-mb1136-stmicroelectronics.pdf) from ST provides us with the correct pinout for the F303-RE, or alternatively (and in a more attractive and detailed way) the same info is presented on ST's mbed OS website [here](https://os.mbed.com/platforms/ST-Nucleo-F303RE/).

(Update after here) Straight away I can see that SPI2 is the winner - it is broken out onto the pins in the bottom right, along with PB1 which we will use for the chip select line.

![SPI2 Pins]({{ 'assets/img/cubeide-sd-card/nucleo_f303re_morpho_spi2.png' | relative_url }}){: .mx-auto.d-block :}

So, let's set up our SPI2 and GPIO in the Device Configuration View. Click SPI2 on the left, and then set it to Full Duplex Master with no Hardware NSS.
Then set the Data Size to 8 bits, and the clock prescaler to 128 (SD cards start up with low speeds and switch to higher speeds later. We'll look at how to do this soon).

![SPI2 Config]({{ 'assets/img/cubeide-sd-card/cubeide-spi2-setup.png' | relative_url }}){: .mx-auto.d-block :}

Then, create your Chip Select line on PB1 - I find setting a sensible name is also good:

![SD_CS Config]({{ 'assets/img/cubeide-sd-card/cubeide-sd-cs-setup.png' | relative_url }}){: .mx-auto.d-block :}

Finally, as we're going to be using the SD card with the FAT file system, scroll down in the device categories to `Middleware`, and expand this, then enable `FATFS` as `User-defined`. You may leave all other parameters as their defaults.

![FATFS Config]({{ 'assets/img/cubeide-sd-card/cubeide-fatfs-setup.png' | relative_url }}){: .mx-auto.d-block :}

Now save your Device Configuration, and when it asks, 'Yes' to _Do you want to generate Code?_ and 'Yes' to _Do you want to open [the C/C++] perspective now?_.

* Musical notes

* PWM to make note (square wave) over carrier wave 

* Listening using radio or SDR



*In `user_diskio.c` Decl:*
```c
/* USER CODE BEGIN DECL */

/* Includes ------------------------------------------------------------------*/
#include <string.h>
#include "ff_gen_drv.h"
#include "user_diskio_spi.h"
```

# Conclusions

If you would like the complete code that accompanies this blog post, it is made available in the associated Github repository [here](https://github.com/kiwih/cubeide-twinkle).