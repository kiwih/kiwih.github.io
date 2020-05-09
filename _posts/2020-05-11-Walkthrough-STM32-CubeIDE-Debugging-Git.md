---
layout: post
title: 'Walkthrough: Getting started with an ST development board using STM32CubeIDE'
subtitle: Project initialisation, debugging, and getting going with Git
#share-img: assets/img/agilent-rs232/share.png
gh-repo: kiwih/stm32cubeide-nucleo-f303re-blink
gh-badge: [star, fork, follow]
tags: [stm32, tooling, c, git]
---

Getting started with an 32-bit ARM-based microcontroller is always a little daunting. 
There's a plethora of available microcontrollers, platforms, development boards, tools, and software.
Everyone seems to have their own opinion on what is best to use, and while debates between the available options range in tone from pleasant and insightful to divisive and vitriolic, it can be quite confusing to work out exactly how one should get going.
I certainly remember it being so, at any rate!

Well, in this walkthrough I'm aiming to simplify things a little, targeting the very questions that I myself had when I got started.
I'll walk through project initialisation, introducing the peripheral and clock configuration options, and demonstrate the ST CubeMX code generator traps.
I'll then show off debugging, something that everybody should know a little about (and no, I'm not talking about liberal use of printf).
Finally, I'll introduce how you can integrate Git into your workflow, for better project management and version control of your software.

While my normal workflow is currently based on Visual Studio Code, for this blogpost I'm actually going to focus on the STM32CubeIDE, as it only recently came out and I'm interested in its capabilities. 
Debugging for me in the past has been through the nightmare-inducing (but indisputably powerful) command line interface for OpenOCD.

## Is this product placement / is this an advertisment?

No, but I wish it was! How does one get into that gig?
Anyway, I'm writing about ST offerings since I currently only own three ARM-based development boards.
Two of them are made by ST, are quite cheap, and are reasonably friendly to hobbyists and beginners. 
My other development board is the Terasic DE10-Standard, which is none of those things, but which I absolutely will do some blogging on in future because it's _freaking awesome_.

# Getting started

