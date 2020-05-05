---
layout: post
title: The PIC16F-antastic picmicro clone
subtitle: Can we emulate a PIC16F core using an FPGA?
gh-repo: kiwih/pic16f-antastic
gh-badge: [star, fork, follow]
#cover-img: /assets/img/path.jpg
tags: [fpga]
---

In this project I set out to create a  cycle-accurate FPGA-ready clone of the picmicro-midrange core, i.e. it is a PIC on an FPGA!
While there are other FPGA ready clones of PICs, I still wanted to make my own - mainly as a learning experience! 
As a point of difference, however, I set out to make it cycle accurate (it seems most/all the other designs are their own implementations of the ISA, with potentially different timing and clock cycle properties).

*Note: This project was created before this blog existed, and as such is awaiting a more detailed write up.*

The PIC16F-antastic was designed entirely from the information in the Microchip datasheets.
It's mostly written in Verilog, but some small parts (mainly the testbenches) are written in SystemVerilog.
It doesn't yet have very many peripherals, as the focus of this work was on the core. However, PORTA, PORTB, TIMER0, and the UART (in asynchronous mode) are good to go!
(If you like the idea of this work, maybe you could write your own and submit pull requests to me on Github?)

One of my major goals with this project was to be able to write programs in MPLAB X, as if you were programming a real PIC - I'm pleased to say this works perfectly! Simply target the 16f628a using MPASM as your compiler, and then using the provided tool `hex2v` you may convert the outputted Intel-format `.hex` file to a Verilog-ready Hex file for inclusion in the Verilog code.

Another major goal for me was to have total automated testing of all the ISA instructions and all peripheral features. Using Modelsim and some SystemVerilog testbenches, this goal was also accomplished!

Feel free to check out the project [on its github!](https://github.com/kiwih/pic16f-antastic)