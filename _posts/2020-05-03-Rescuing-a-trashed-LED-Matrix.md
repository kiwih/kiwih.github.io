---
layout: post
title: Rescuing a trashed LED Matrix
subtitle: Reverse engineering a simple embedded system!
gh-repo: kiwih/LedMatrix
gh-badge: [star, fork, follow]
tags: [embedded]
---

[Github project link](https://github.com/kiwih/LedMatrix)

# Background

I was cycling through a light industrial area in Auckland City when I came across a rubbish bin awaiting pickup on the side of the road. 
It caught my eye as it was practically overflowing with discarded LED Matrix modules - big ones, about a foot in each dimension!
I couldn't believe it - firstly, discarding e-waste in a manner such as this is actually illegal in New Zealand, and secondly, the LED modules - for the most part - actually looked to be in pretty good condition!
I grabbed as many as I could carry on the bike and carried on home, keen to see if I could get them going!

# First impressions

Here are pictures of the front and back of one of the LED modules.
![Matrix Front]({{ 'assets/img/ledmatrix/first-front.jpg' | relative_url }}){: .mx-auto.d-block :}

![Matrix Back]({{ 'assets/img/ledmatrix/first-back.jpg' | relative_url }}){: .mx-auto.d-block :}

Straight away we can see several interesting pieces of information. 
* The modules have a well-labelled 12V power supply input
* The modules appear to have an INPUT port and an OUTPUT port, presumably for data
* There is a part number 'FX13-P013V2.0', or possibly 'BR43.13'
* There is what could be a date from 2013 on a sticker (although it has 2 digits too many?), as well as a QC passed label

The input port is interesting:
![Input Port]({{ 'assets/img/ledmatrix/input-port.jpg' | relative_url }}){: .mx-auto.d-block :}

Thanks to the labels (Whoever designed this PCB, seriously, thanks) this gives us some idea of how these modules might be controlled.
'GND' are grounds, 'OE' is almost certainly 'output enable', 'CLK' is a clock which implies a serial control scheme of some kind, 'LAT' is probably short for 'latch', and R1/R2/G1/G2/B1/B2 are likely the colour inputs. 

I paused at this point for dinner, and flicked a message to a former colleague that used to work at a tech company involved in digital road signs (among other things), asking him if there was a standard control scheme for LED modules such as these. He said that usually they were designed upon standard shift registers - direct quote "The modules are usually dumb, intelligence is in the controllers".
I took a closer look at the ICs that were stamped around the board - I found only two kinds.
1. Located near the INPUT and OUTPUT ports are 74HC245 chips. These are 'Octal Bus Trancievers' and appear to be being used as I/O protection.
2. Located between groups of LEDs are MBI5024 chips. Looking these up online, they are Shift Registers designed for use in applications such as LED boards - fancy that! Also, it looks like my former colleague was correct - seems like we just need to shift in data to display it.

I recreate the block diagram for the MBI2054 here:
![MBI2054 block diagram]({{ '/assets/img/mbi5024-block-diagram.png' | relative_url }}){: .mx-auto.d-block :}
(Source: )

We can also get the following helpful information out of the data sheet:

| Port  | Function |
| :---- | :------- |
| SDI   | Serial-data input to the internal shift register |
| CLK   | Clock input terminal. Data shifts in on a rising edge |
| LE    | Data strobe input terminal. Serial data is transferred to the output latch when LE is high, and the data is latched when LE is low. |
| nOE   | Output enable terminal (active low). When low, the output drivers are enabled, when high, output drivers are disabled. |

As a refresher or in case you're unfamiliar, shift registers operate according to the following generic internal block diagram:

![Matrix Back]({{ 'assets/img/ledmatrix/shift-register.png' | relative_url }}){: .mx-auto.d-block :}

Each D/Q there is a D type Flip Flop. On a rising clock edge, they capture the data at D, to be then emitted at Q. 
As such, after the first clock tick, the first flip flop will be emitting the first bit of data from the serial input.
You then change the serial input to the next bit of data, and pulse the clock again. Now, the first flip flop is outputting the second bit of data, and the second flip flop is outputting the first.
On and on this goes until you completely fill your shift register, and your input data stream is now present on the outputs of the shift register in parallel.

From the datasheet then, we can conclude that inputs LAT, CLK, and OE are going to all shift registers in parallel. Presumably then the R1/R2/G1/G2/B1/B2 are the data inputs to the shift registers. Why, though, are there two wires for each?

# Investigations with a microcontroller

It's worth pointing out here that due to the coronavirus outbreak, at the time of performing this work, I was stuck at home without my usual work-provided equipment and components. At any rate, I figured that I could probably get away with what I had on hand.
Firstly, I needed to make sure I could safely power up the LED module. I found a 12V 2A power supply, and applied it. Then, I pulled the OE pin low, and brushed noise against the LAT, CLK, and data pins. Every LED abruptly turned on - I had invited the sun into my office. These boards are hecking bright!
Fortunately, nothing exploded or melted. A good start! Using a multimeter, I could see that my 12V power supply was just coping, with a reading of 11.5V.

My first experiment was to find out (a) how the loaded data made its way to pixels on the screen, (b) the purpose of the two pins for each colour, and (c) how much current this board was actually trying to draw. Unfortunately for me, my multimeter ammeter had a blown fuse, and I couldn't go get another fuse from anywhere.

Fortunately, I had an Atmel Xplained Mini board (Atmega328PB based) at home to work with, so I used this as my controller. I quickly wrote a small program to bit bash out data to the shift register's G1 pin (chosen at random). I was interested to see what would happen if I pumped different data into the system.

```c
//Note: this will only control one bit of R1/R2/G1/G2/B1/B2, all others are forced to zero.
//(i.e. this is not suitable for actual use, just for experimentation)
void Disp_Send(uint8_t val, uint8_t color_pin) {
	//for each bit in val
	for(int i=0; i < 8; i++) {
		//put the i-th bit onto the color_pin pin
		PORT_COLORS = 0;
		if((0x80 >> i) & val) {
			PORT_COLORS |= (1 << color_pin);
		}
		
		//now toggle the CLK pin to store the bit
		_delay_us(1); //first delay is for settling time
		PORT_CTRL |= (1 << PIN_CLK);
		_delay_us(1);
		PORT_CTRL &= ~(1 << PIN_CLK);
	}

	//now latch the output by toggling the LAT pin
	_delay_us(1); //first delay is for settling time
	PORT_CTRL |= (1 << PIN_LAT);
	_delay_us(1);
	PORT_CTRL &= ~(1 << PIN_LAT);
}
```

I had it wired as follows:
```c
//CLK and LAT are on PORT C
#define DDR_MATRIX_CTRL DDRC
#define PORT_MATRIX_CTRL PORTC
#define PIN_CLK 0
#define PIN_LAT	1

//OE is associated with PORT B3 as this goes with PWM for TIMER2 (OC2A pin is same pin as PB3)
#define DDR_MATRIX_OE DDRB
#define PORT_MATRIX_OE PORTB
#define PIN_OE 3

//color pins on PORT D
#define DDR_MATRIX_COLORS DDRD
#define PORT_MATRIX_COLORS PORTD
#define PIN_R1 0
#define PIN_R2 1
#define PIN_B1 2
#define PIN_B2 3
#define PIN_G1 4
#define PIN_G2 5
```

In the `main()`, I simply called `Disp_Send_G1(0xFF)` with a short delay over and over again in a loop. 
To prevent cooking my power supply and further searing of my eyeballs, I also configured the OE pin as a PWM, keeping the display on only 6.25% of the time (16 cycles of every 256).

I grabbed one of the other LED modules now, one which was missing several LEDs on the PCB, so that if something went wrong I wouldn't break one of the modules that appeared to be in better condition. I was then able to produce the following output:
<video width="384" height="288" controls>
  <source src="{{ '/assets/vid/led-matrix-fill.mp4' | relative_url }}" type="video/mp4">
Your browser does not support the video tag.
</video>



[final]