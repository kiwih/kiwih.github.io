---
layout: post
title: 'Tutorial: Getting started with an ST development board using STM32CubeIDE'
subtitle: Project initialisation, debugging, and getting going with Git
#share-img: assets/img/agilent-rs232/share.png
gh-repo: kiwih/stm32cubeide-nucleo-f303re-blink
gh-badge: [star, fork, follow]
tags: [tutorial, stm32, STM32CubeIDE, tooling, c, git]
---

Getting started with an 32-bit ARM-based microcontroller is always a little daunting. 
There's a plethora of available microcontrollers, platforms, development boards, tools, and software.
Everyone seems to have their own opinion on what is best to use, and debates between the available options range in tone from pleasant and insightful to divisive and vitriolic.
It can be quite confusing to work out exactly how one should get going - I certainly remember it being so, at any rate!

Well, in this tutorial I'm aiming to simplify things a little, targeting the very questions that I myself had when I got started.
I'll walk through project initialisation, introducing the peripheral and clock configuration options, and demonstrate the ST CubeMX code generator traps.
I'll then show off debugging, something that everybody should know a little about (and no, I'm not talking about liberal use of `printf`).
Finally, I'll introduce how you can integrate Git into your workflow, for better project management and version control of your software.

While my normal ARM programming environment is currently based on Visual Studio Code, for this tutorial blogpost I'm actually going to use the STM32CubeIDE, as it's free, it only semi-recently came out, and I'm interested in its capabilities --- especially with the debugger.
Debugging for me in the past has been through the nightmare-inducing (but indisputably powerful) command line interface for OpenOCD.

## Is this product placement / is this an advertisment?

No, but I wish it was! How does one get into that gig?
Anyway, I'm writing about ST offerings since I currently only own three ARM-based development boards.
Two of them are made by ST, are quite cheap, and are reasonably friendly to hobbyists and beginners. 
My other board is the Terasic DE10-Standard, which is none of those things - however, as the DE10-Standard is _freaking awesome_, I will no doubt blog about it in future.

I've also worked extensively with ST products throughout my academic and industrial career, so I have a bit of institutional knowledge to go along with these dev boards.

# Getting started: Choose a development board / platform

First, before you download and instally anything, make sure you actually have something to program! 
As mentioned, I have two ST dev boards; the stm32f4-discovery series `STM32F429I-DISC1` and the stm32nucleo series `NUCLEO-F303RE`.
If you're looking for a review of these, I like both of them quite a lot. The discovery board is great since it has an integrated LCD screen and a big on-device SDRAM chip, and the nucleo board is neat since it is quite cost effective and has both Arduino-compatible headers as well as its own larger array of I/O to work with.

I'm not going to provide a purchase link for any dev boards since I'm not affiliated with anyone, but shop around for a deal that looks good to you. 
The official ones are nice, but you might be able to find third party ones that are cheaper (although they often come with less documentation / worked examples).

As it's the cheaper board, this tutorial will focus on the `NUCLEO-F303RE` development board. 
However, as the steps can easily be generalised, you should be able to follow along with almost any other STM32-based development board.

# Getting started: Install STM32CubeIDE

There are a lot of different software development environments that will work with the STM32 line of microcontrollers. 
Today I'm going to focus on ST's own IDE: STM32CubeIDE. 
It includes the necessary drivers, compilers, and the debugger all in a nice eclipse package.

So, let's get started. You can download and install CubeIDE [from their website](https://www.st.com/en/development-tools/stm32cubeide.html).
It's free, although you do have to sign up an account / give them your email address.
My first impressions are very positive - ST have provided binaries for every platform (yay Debian/Linux).

Anyway, here's the steps, I'm sure I don't need to go into detail here:
1. Download the correct version for your operating system (I grabbed the Debian bundle)
2. Extract the installer
3. Install with adminstrator priveliges 
4. Run

# Your first project

* Create a project
* Board selector
* Looking around inside CubeMX
* Setting up clocks/GPIO

# Let's C what we can do here

* Where to put your code
* Calling HAL functions
* Blink an LED in a loop

# Compiling the project

# Debugging execution

# The virtual COM port

# 