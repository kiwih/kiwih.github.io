---
layout: post
title: 'Tutorial: An SD card with an STM32 using STM32CubeIDE'
subtitle: Let's have some fun with files
gh-repo: kiwih/cubeide-sd-card
gh-badge: [star, fork, follow]
#cover-img: /assets/img/path.jpg
share-img: /assets/img/anet-a8/the-printer.jpg
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
Well, there are a few good tutorials and resources floating around online (including even my own from 2017) but some of them are a bit dated, relying on older tools and libraries.

In this tutorial I'm going to walk through the steps that one would use to get an SD card working over the SPI interface on a STM32 dev board. 
I'll be using the STM32CubeIDE development environment, and I'll assume you already know the basics of creating a project and setting up debugging and so on.
If not, then please [check out my earlier blog post in which I introduce STM32CubeIDE](https://01001000.xyz/2020-05-11-Tutorial-STM32CubeIDE-Getting-started/).

# Equipment for this tutorial

Today I will be using the following:

* (Free) Ubuntu Linux 20.04 because developing on Linux makes me happy and Ubuntu makes it easy. Don't worry if you're on Windows, you should be able to follow along with roughly the same steps. 
* (Free) STM32CubeIDE
* ($27.26 from [Amazon Prime](https://amzn.to/2BDK6ID), $15.05 from [Amazon](https://amzn.to/3gEp6AA)) The `Nucleo-F303RE` development board.
* ($5 from [Amazon Prime](https://amzn.to/3ik1wJV)) An SD card breakout board (comes in a pack of two).
* ($6 from [Amazon Prime](https://amzn.to/33GRwXe)) Easy-to-use ribbon cables (there's more than you need here but they're handy to have around).

_Note: The above Amazon links are affiliate links. As always I encourage you to shop around, but Amazon usually has pretty good pricing._

# Preliminaries

