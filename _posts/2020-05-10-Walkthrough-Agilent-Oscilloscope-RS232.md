---
layout: post
title: 'Walkthrough: Reading an Agilent 5000 series oscilloscope via RS232'
subtitle: Enabling data transfer from scope to PC
#gh-repo: kiwih/LedMatrix
#gh-badge: [star, fork, follow]
tags: [tooling, python]
---

Recently I acquired an older Agilent 54621A oscilloscope for my at-home workstation, and yes, that is a floppy drive on the front.

![Scope Front]({{ 'assets/img/agilent-rs232/front.jpg' | relative_url }}){: .mx-auto.d-block :}

According to the person I got it from, the floppy drive still works, and they successfully used it to load data captures to their PC as recently as last year. Bit old fashioned, but I suppose it's servicable, given that I do own a USB floppy drive... 

However, while I was looking it over, I noted the presence of an RS232 port:

![Scope RS232 Port]({{ 'assets/img/agilent-rs232/port.jpg' | relative_url }}){: .mx-auto.d-block :}

(I also noted this ominous warning message)

![Scope Warning]({{ 'assets/img/agilent-rs232/warning.jpg' | relative_url }}){: .mx-auto.d-block :}

The presence of the RS232 port got me thinking - Having never owned an Agilent oscilloscope before, I wonder what sort of data can be sent and recieved via this port?

Armed with google, I soon found [this](https://web.sonoma.edu/esee/manuals/5462xUG.pdf), the User's guide for my oscilloscope, which mentions that software called BenchLinkXL 54600 software can be used to communicate with the oscilloscope! Success!
I downloaded the software, but --- turns out it's not going to be quite that easy. It won't install nor work on Windows, nor on Wine under Ubuntu.

Still, data must be able to go in and out of this port. I did more googling, and found [this](https://www.keysight.com/upload/cmc_upload/All/5000_series_prog_guide.pdf), the "Agilent InfiniiVision 5000 Series Oscilloscopes Programmer's Guide", an 884 page behemoth that, among other things, provides intricate details of an Agilent programming language to be used via a different set of custom software, the "Agilent IO Libraries Suite". 

But, my oscilloscope isn't a 5000 series InfiniiVision, and the manual describes connecting the software via either LAN, USB, or GPIB, none of which I have present. I have only the RS-232 port, and the manual does not describe this at all as a connection option.
Yet, one paragraph stood out to me, on page 36, which said _The command set [in this manual] is similar to the 6000 Series oscilloscopes (and the *54620/54640 Series oscilloscopes* before them)_.

How interesting! That implies there is a programmer's guide for my oscilloscope as well... So with yet more googling, I found [this](http://web.mit.edu/8.13/8.13d/manuals/agilent-54621a-programmers-guide.pdf) backed up on an mit.edu domain (thanks MIT), which is indeed the older programmer's guide for my specific oscilloscope!

Now we're in business. This manual specifically discusses talking to the RS-232 port, in fact, has an entire chapter devoted to it. 

# Linking the machines

The manual's first step in getting an oscilloscope talking to a computer via RS232, is, as expected, configuring both machines settings so that they can understand one another. 
If you aren't already familiar with how RS232 works, this is an important step. 
Unlike with newer protocols such as USB, RS232 devices need to be configured manually to speak with the same settings, i.e. speed, handshaking, parity checking, and so forth. If you don't get these settings equivalent on both devices, then despite the fact that both devices speak the same language, they won't be able to understand one another. 
From a metaphorical point of view, you might consider it as getting two language speakers to use the same dialect (e.g. american english vs british english) except more severe.

In order to configure the RS232 port on the oscilloscope, we just need to jump through a few on-screen menus, via `Utility (Physical button) > I/O (on-screen button)`
You can see we have only a couple of options, namely the baud rate and the handshaking method.

![Scope Warning]({{ 'assets/img/agilent-rs232/rs232-onscreen-config.jpg' | relative_url }}){: .mx-auto.d-block :}

In the original user manual (the first document I linked), on page 6-8, these two options are explained thus:

![Manual explanation]({{ 'assets/img/agilent-rs232/serial-setup.png' | relative_url }}){: .mx-auto.d-block :}

Given this information, we can now use any suitable tool to match these settings and try talk to the oscilloscope!

For Windows, a popular tool is PuTTY. On Linux/Ubuntu, I would use minicom. On a mac, I guess minicom is also available but I'm honestly not sure.

In this tutorial, I'll use Windows/PuTTY. 

First things first - I own a USB to Serial cable, and it outputs the male side of the RS232. Annoyingly, the oscilloscope also has a male RS232 connector, so I need a female-to-female cable such as this one:

![Female female RS232 cable]({{ 'assets/img/agilent-rs232/female-female-rs232.jpg' | relative_url }}){: .mx-auto.d-block :}

Now, after connecting all these together, we can find our COM port number using device manager:

![Device manager]({{ 'assets/img/agilent-rs232/device-manager.png' | relative_url }}){: .mx-auto.d-block :}

I now want to open PuTTY and configure it, setting it to use XON/XOFF flow control (I also set the oscilloscope to use this), and baud rate 9600 (this is slow, but fast enough for experimentation purposes), as well as match the other settings in the user manual (8 data bits, 1 stop bit, no parity).

![Device manager]({{ 'assets/img/agilent-rs232/putty-config.jpg' | relative_url }}){: .mx-auto.d-block :}

Let's now `[open]` the port, and try a command!

The programmer's guide, from page 8-5 onwards, lists all available commands and queries. This one, `*IDN?`, seems to be a nice place to start:

![IDN definition]({{ 'assets/img/agilent-rs232/idn-def.jpg' | relative_url }}){: .mx-auto.d-block :}

So what happens if we type this in and press [Enter]?

reading the oscilloscope user manual  to set up the RS-232 port, which is reasonable, but I was getting tired of reading manuals at this point so I just poked at buttons till I found it:



The 


