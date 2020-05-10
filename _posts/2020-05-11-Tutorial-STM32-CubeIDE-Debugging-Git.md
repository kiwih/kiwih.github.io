---
layout: post
title: 'Tutorial: Getting started with an ST development board using STM32CubeIDE'
subtitle: Project initialisation, intro to debugging, and getting going with Git
share-img: assets/img/cubeide-intro/boards.jpg
gh-repo: kiwih/stm32cubeide-nucleo-f303re-hello
gh-badge: [star, fork, follow]
tags: [tutorial, stm32, STM32CubeIDE, tooling, c, git]
---

Getting started with an 32-bit ARM-based microcontroller is always a little daunting. 
There's a plethora of available microcontrollers, platforms, development boards, tools, and software.
Everyone seems to have their own opinion on what is best to use, and debates between the available options range in tone from pleasant and insightful to divisive and vitriolic.
It can be quite confusing to work out exactly how one should get going - I certainly remember it being so, at any rate!

Well, in this tutorial I'm aiming to de-mystify things a little, targeting the very challenges that I myself faced when I first got started.
I'll walk through project initialisation, introducing the peripheral and clock configuration options, and demonstrate the ST CubeMX code generator traps.
I'll then show off debugging, something that everybody should know a little about (and no, I'm not talking about liberal use of `printf`).
Finally, I'll introduce how you can integrate Git into your workflow, for better project management and version control of your software.

While my normal ARM programming environment is currently based on Visual Studio Code, for this tutorial blogpost I'm actually going to use the STM32CubeIDE, as it's free, it only semi-recently came out, and I'm interested in its capabilities --- especially with the debugger.
Debugging for me in the past has been through the nightmare-inducing (but indisputably powerful) command line interface for OpenOCD.

## Hey, is this blog post an advertisment?

No, but I wish it was! How does one get into that gig? Seriously, let me know. :-)

Sadly, I'm just writing about ST offerings since I currently only own three ARM-based development boards.
Two of them are made by ST, are quite cheap, and are reasonably friendly to hobbyists and beginners. 
My other board is the Terasic DE10-Standard, which is none of those things - however, as the DE10-Standard is _freaking awesome_, I will no doubt blog about it in future.

I've also worked extensively with ST products throughout my academic and industrial career, so I have a bit of institutional knowledge to go along with these dev boards.

# Getting started: Choose a development board / platform

First, before you download and instally anything, make sure you actually have something to program! 
As mentioned, I have two ST dev boards; the stm32f4-discovery series `STM32F429I-DISC1` and the stm32nucleo series `NUCLEO-F303RE`, as seen here:

![ST dev boards]({{ 'assets/img/cubeide-intro/boards.jpg' | relative_url }}){: .mx-auto.d-block :}

If you're looking for a review of these, I like both of them quite a lot. The discovery board is great since it has an integrated LCD screen and a big on-device SDRAM chip, and the nucleo board is neat since it is quite cost effective and has both Arduino-compatible headers as well as its own larger array of I/O to work with.

As it's the cheaper board, this tutorial will focus on the `NUCLEO-F303RE` development board. 
However, as the steps can easily be generalised, you should be able to follow along with almost any other STM32-based development board.

I'm not going to provide a purchase link for any dev boards since I'm not affiliated with anyone, but shop around for a deal that looks good to you. 
The official ones are nice, but you might be able to find third party ones that are cheaper (although they often come with less documentation / worked examples).

# Getting started: Install STM32CubeIDE

There are a lot of different software development environments that will work with the STM32 line of microcontrollers. 
Today I'm going to focus on ST's own IDE: STM32CubeIDE. 
It includes the necessary drivers, compilers, and the debugger all in a nice eclipse package.

So, let's get started. You can download and install CubeIDE [from their website](https://www.st.com/en/development-tools/stm32cubeide.html).
It's free, although you do have to sign up an account / give them your email address.
My first impressions are very positive - ST have provided binaries for every platform (yay Debian/Linux).

Anyway, here's the steps, I'm sure I don't need to go into detail here:
1. Download the correct version for your operating system (I grabbed the Debian bundle since I'm running an Ubuntu system)
2. Extract the installer
3. Install with adminstrator priveliges 
4. Run

(I trust it's as easy as this to install on Windows, but who knows?)

Alright, you're good to go - preliminaries finished! 

# Your first project

Before we can start writing code we need to create a project. 
This is similar to most other IDEs - projects are used to bundle together all of your settings, code, and definitions into a single collection all managed from the same application (hence the name IDE, which stands for Integrated Development Environment). 
The alternative would be to have multiple programs to handle your development, for instance using CubeMX for your chip configuration, `vi` or `emacs` to code, standalone ST-Link drivers for programming, OpenOCD for debugging, and so on. 
This is a more fiddly approach, but some developers prefer it. 

At any rate, here's what CubeIDE looks like - yep, that sure looks like an eclipse-based environment.

![first look]({{ 'assets/img/cubeide-intro/start.png' | relative_url }}){: .mx-auto.d-block :}

As CubeIDE is eclipse-based, I know to look under the top left icon (Or under the menu `File > New > STM32 Project`) to get started:

![new project]({{ 'assets/img/cubeide-intro/new-project.png' | relative_url }}){: .mx-auto.d-block :}

This will now think for a moment (a popup may appear briefly) and then show you the chip/board selector. 
This menu appears to be copied from their standalone STM32CubeMX program.

Since I have the Nucleo-F303RE, that's what I'm going to search for, after selecting `Board Selector` at the top:

![find a board]({{ 'assets/img/cubeide-intro/find-board.png' | relative_url }}){: .mx-auto.d-block :}

I like that it shows me the picture of the kit, that's kind of neat. Now, pick a sensible project name:

![name options]({{ 'assets/img/cubeide-intro/project-name.png' | relative_url }}){: .mx-auto.d-block :}

I didn't change anything in the final window. Hit finish and let's move on!

![firmware options]({{ 'assets/img/cubeide-intro/firmware.png' | relative_url }}){: .mx-auto.d-block :}

I then got a popup asking me if I wanted to initialise peripherals to their default configuration. 

![default peripheral options]({{ 'assets/img/cubeide-intro/default-peripherals.png' | relative_url }}){: .mx-auto.d-block :}

I assume this means the peripherals should be set up for the hardware on my development board. I'll hit yes.

I now get the CubeMX configuration menu view. This looks like this:

![cubemx view]({{ 'assets/img/cubeide-intro/cubemx-view.png' | relative_url }}){: .mx-auto.d-block :}

In here, we can set up our peripherals. By choosing `yes` a moment ago it appears to have pre-populated some settings for me:
* _PA5_ is _LD2_, the board's Green LED
* _PC13_ is _B1_, the board's Blue Push Button
* _PA2 and PA3_ are a USART
* _PB3, PA13/14_ are debugging wires
* and _PC14/15, PF1/0_ are the high and low speed oscillators (I'm not entirely sure why these are enabled by default since my dev kit does not have one of the crystals populated).

Quick pause for a sanity check, let's open up [the schematic](https://www.arrow.com/en/reference-designs/nucleo-f303re-stm32-nucleo-development-board-with-stm32f303ret6-mcu-supports-arduino-and-st-morpho-connectivity/982d4d8e7083037810fa1ca7cad4fcf6) for this development board (schematic available in menu to the right of that page). Has it put the LED in the right place?

![cubemx view]({{ 'assets/img/cubeide-intro/led-pin.png' | relative_url }}){: .mx-auto.d-block :}

Cool cool. We can check the switch in the same way, and that looks correct as well.

If you click on `Clock Configuration` in the top menu bar, you'll now see an intricate clock system diagram:

![clocks view]({{ 'assets/img/cubeide-intro/config-clocks.png' | relative_url }}){: .mx-auto.d-block :}

Using the radio buttons embedded into the multiplexers throughout this diagram you can change how different available clock sources propagate through the various PLLs and clock dividers in the device in order to generate all the different frequencies you need.

The default suggestion seems pretty good to me - It's using the on-chip high and low speed RC clocks, which are perfectly acceptable for little demo applications such as this.
Hopefully you can see from this diagram that it's then using a PLL to multiply the internal 8MHz up to 72MHz.
The 72MHz is then passed as-is to everything except the APB1 peripheral bus.

In theory, the clock system does its best to stop you from breaking anything. 
Observe, if I change the APB1 divider to `/1` instead of `/2`, the system detects that this will create a faulty system (it exceeds the 36MHz maximum for this bus), and thus flags the inappropriate setting in red.

![bad clock setting]({{ 'assets/img/cubeide-intro/bad-clock.png' | relative_url }}){: .mx-auto.d-block :}

I returned the setting to the original, and saved. It may ask you if you want to generate code - if you haven't already done so, hit yes. 
Now, onto the programming!

# Let's C what we can do here

CubeMX, which this functionality is developed upon, generates C files to work with under a Src directory, and puts a HAL (Hardware Abstraction Layer) into an Includes directory. 
It appears CubeIDE works the exact same way. 
Expand the folders on the right under the project view and see what it has generated to work for you:

![files and folders]({{ 'assets/img/cubeide-intro/files-folders-explanation.png' | relative_url }}){: .mx-auto.d-block :}

For the purposes of this tutorial we'll just focus on the main `Project and autogenerated code` section, starting with main.c.

You'll see that main.c is already quite large, containing a fair amount of autogenerated code. 
A key piece of information to remember here is that *main.c can be edited by the code generator*, so it's important to *only write code in the USER sections*.
What does this mean? Let's take a look under `int main()`:

```c
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_USART2_UART_Init();
  /* USER CODE BEGIN 2 */

  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
```

Observe in this block of code that there are a number of `USER CODE BEGIN` and `USER CODE END` sections marked out by comments.
Any code written inside these blocks is _safe_ and will not be deleted by the code autogenerator.
Any code written outside those blocks is _unsafe_ and *will be deleted* by the code autogenerator any time you edit the CubeMX settings that we looked at earlier.

Of course, files that you yourself add to the project (e.g. `MySuperCFile.c`) are also totally safe. It's just this set of autogenerated `.c` (and `.h`) files that you must be careful with when adding code.

Take a brief moment now to look at the function names used in the autogenerated code. 
* Any function call beginning with `HAL_` is from the STM32 HAL, and is provided in the library files. There's HAL functions to do all sorts of things, including using the UART, writing a Pin, etc.
* Any function call beginning with `MX_` is autogenerated by CubeMX/CubeIDE. These functions tend to be used to initialise functions.
* There are exceptions to these rules, including, annoyingly, `SystemClock_Config()`, which is also an autogenerated function.

Let's add a smidge of C code of our own now! After the `Infinite Loop` area, we're going to add code to toggle the LED under section 3.
I initially couldn't get the autosuggest to show up on its own, but then worked out it appears after you press `Ctrl+Space`:

![auto suggest]({{ 'assets/img/cubeide-intro/eclipse-autosuggest.png' | relative_url }}){: .mx-auto.d-block :}

We're going to change the overall loop to:

```c

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
    HAL_Sleep(1000);
  }
  /* USER CODE END 3 */
}
```
(Yes, indentation by the autogenerator is two spaces. You get used to it.)

# Compiling the project

# Debugging execution

# The virtual COM port

# 