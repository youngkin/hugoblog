---
title: "Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi"
description: "An overview of PWM, with code examples, on the Raspberry Pi."
date: 2021-09-19T13:13:42-06:00
draft: false
image: "images/pwmfordummies/PWMPulsePerioddim.png"
tags: ["raspberry-pi", "Golang", "C", "GPIO"]
categories: ["raspberry-pi", "Golang", "GPIO"]
GHissueID: 1
toc: true
---

This article provides some details about hardware and software based PWM on the Raspberry Pi, specifically the 3B+ with the [Broadcomm BCM2835 board](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf). This article has an accompanying application, [PWM Explorer](https://github.com/youngkin/gpio)[^20], that can be used to experiment with PWM settings and capabilities.

<!--more-->

## Overview

I was writing an [article](https://youngkin.github.io/post/sunfoundergpionotesrgbled) as a companion to the [SunFounder RGB LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html) to cover some areas that were missing in that project such as what is PWM, how does it work, and why the code was written as it was. I had no idea what PWM was or what I was doing or why. To fill the gap I started researching PWM and found numerous articles about PWM[^1] [^2] [^3], and also software libraries like [WiringPi](https://github.com/WiringPi/WiringPi) (C)[^4], [go-rpio](https://github.com/stianeikeland/go-rpio) (Go)[^5], [BCM2835 by Mike McCalley](https://www.airspayce.com/mikem/bcm2835) (C)[^6] and [pigpio](https://abyz.me.uk/rpi/pigpio/pdif2.html) (Python & C)[^13]. So with my focus still on controlling an RGB LED via GPIO and PWM I started writing programs in C and Go. I quickly had difficulty in several areas:

1. Terminology is used inconsistently across articles and the libraries. For example, some used the terms _Cycle_ and _Range_ for the same thing.
2. The libraries were documented to varying degrees. Coupled with a lack of common terminology it was difficult to figure out how to use them.
3. Going hand in hand with figuring out how to use the libraries, it was difficult to understand how different parameters affected the behavior of the hardware, in this case an RGB LED.

My intent in writing this is to:

1. Provide an overview of PWM and its main concepts.
2. Provide a glossary that links the various PWM terms to a consistent definition.
3. Explain how the various PWM settings interact with each other (not just _duty cycle_).
4. Provide the information needed (sometimes via external references) to install a GPIO test bed on a Raspberry Pi 3B+.
5. Provide programs, written in Go and C, that demonstrate how to effectively use WiringPi[^4] and go-rpio[^5].
6. Provide an application that can be used to experiment with the various PWM parameters in real time.
7. Provide hard to find information on some of the concepts and PWM settings (e.g., minimum/maximum PWM frequencies).

Lest anyone think I'm presenting myself as an expert in GPIO and PWM, I'm not. Despite my best attempts to summarize what I've learned, I'm sure I've made mistakes or made things more confusing. Any mistakes I've made are my own. If anyone comes across any mistakes topics that could be clarified, I'd appreciate comments so I can modify this article.

I'm not writing this to be a definitive source about all things PWM. There are better resources for that. This article lists those [resources](#references) that I found particularly helpful.

Now to the outline of this article. The sections are as follows:

1. [Prerequisites](#prerequisites) covers areas like how to get the software libraries used in this article as well as things like the hardware needed by the _PWM Explorer_ application to illustrate behavior.
2. [What is PWM?](#what-is-pwm) provides a very basic introduction to PWM. It provides definitions of common terms and the aliases used in the articles and software libraries.
3. [Overview of PWM on a Raspberry Pi 3B+](#overview-of-pwm-on-a-raspberry-pi-3b) provides an overview of how GPIO and PWM are implemented on a Raspberry Pi.
4. [Exploring PWM on a Raspberry Pi](#exploring-pwm-on-a-raspberry-pi) provides a description of the various PWM settings, how they interact with each other, some hard to find tidbits about the PWM settings, and an overview of the application, _PWM Explorer_, that can be used to drive PWM on a Raspberry Pi to see how the settings interact with each other in real time.
5. A [Summary](#summary) to wrap things up.
6. And a set of useful [References](#references).

## Prerequisites

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'Stretch' version of the Raspbian OS. The Raspberry Pi website has instructions on how to [setup a new Raspberry Pi from scratch](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up)[^16] if you decide to go that way vs. buying a complete kit.

Other items you'll need include:

* a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) _(You may find [this tutorial on breadboards](http://wiki.sunfounder.cc/index.php?title=Breadboard_Basics_%E2%80%93_Types) helpful[^18])_, 
* some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), 
* [a 220 Ohm resistor, and a RGB LED](https://www.amazon.com/EDGELEC-Tri-Color-Multicolor-Diffused-Resistors/dp/B077XGF3YR/ref=asc_df_B077XGF3YR/?tag=hyprod-20&linkCode=df0&hvadid=242051162351&hvpos=&hvnetw=g&hvrand=11064062033670066895&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9028749&hvtargid=pla-430228081645&psc=1). 
* You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the T-Type adapter will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however. 
 
[Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). I'm finding the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1) especially useful. NOTE: The Ultimate Starter Kit and the Raphael Kit are the same product.

<img style="border:1px solid black" src="/images/pwmfordummies/RaphaelKit.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Ultimate Starter/Raphael kit</i></center></figcaption>

You will also need some basic C  and Go programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like vim or nano.

To compile and run the C program you'll need the WiringPi[^4] [^19] library. It's easy to get:

```bash
sudo apt-get install wiringpi
```

Then test the installation using:

```bash
pi@pi-node1:~/go/src/github.com/youngkin/gpio/rgbled $ gpio -v
gpio version: 2.50
Copyright (c) 2012-2018 Gordon Henderson
This is free software with ABSOLUTELY NO WARRANTY.
For details type: gpio -warranty

Raspberry Pi Details:
  Type: Pi 3B+, Revision: 03, Memory: 1024MB, Maker: Sony
  * Device tree is enabled.
  *--> Raspberry Pi 3 Model B Plus Rev 1.3
  * This Raspberry Pi supports user-level GPIO access.
```

In the above you'll notice `gpio version: 2.50`. If you're using a Rasberry Pi 4, use the instructions given in the Sunfounder [Checking the WiringPi](https://docs.sunfounder.com/projects/raphael-kit/en/latest/check_the_wiringpi_c.html).

WiringPi is unique in that it includes a command line tool, `gpio`, as shown above, that can be used to manage, control, and query the GPIO board. This can be very handy. See the [gpio reference](http://wiringpi.com/the-gpio-utility/) for more information on what it can do and how to use it.

If you're interested in Go development on a Raspberry Pi you'll need to install the development environment onto the Raspberry Pi. [Here's a simple source](https://www.jeremymorgan.com/tutorials/raspberry-pi/install-go-raspberry-pi/) that explains how to accomplish this. This source is a little dated, but the only significant issue is with the version of Go to install. The source shows installing Go __1.14.4.linux-arm64.tar.gz__ and __1.14.4.linuxarmv6l.tar.gz__. The current versions are __1.17.1.linux-arm64.tar.gz__ and __1.17.1.linuxarmv6l.tar.gz__. For the Raspberry Pi 3B+ the correct choice will be __1.17.1.linuxarmv6l.tar.gz__. The other is intended for 64 bit systems like the Raspberry Pi 4 series. You can find current ARM versions of Go at the [Golang download site](https://golang.org/dl/).

For Go development you'll also need the [go-rpio](https://github.com/stianeikeland/go-rpio)[^5] library.

If you want to veer away from the cookbook style of the Sunfounder docs you'll need some basic knowledge of Linux . For example, I won't be explaining what __root privileges__ are.

Finally, I wrote an application that supports experimenting with the various PWM settings on the Raspberry Pi 3B+. There are 2 options for getting the code. First, you can clone or fork the [project](https://github.com/youngkin/gpio.git) from GitHub. If you do that you'll need to have experience with git and have it installed on the Raspberry Pi. See this [article on installing git](https://linuxize.com/post/how-to-install-git-on-raspberry-pi/) for more details. After installing `git` you can download the project by running `git clone https://github.com/youngkin/gpio.git`. If you would like to contribute the project please fork the respository instead. As an alternative to using git you can also download a _zip_ file of the project by navigating to the [project's URL](https://github.com/youngkin/gpio.git), clicking on the _Code_ button above and to the right of the file listing, and selecting _Download ZIP_.

## What is PWM?

At it's most basic, PWM is used to simulate an analog signal using a digital source (like a GPIO pin). As an example, an LED's brightness and/or color can be modified by varying the voltage supplied to the LED. A variety of analog devices, such as a motor's speed, can be controlled in the same way. PWM simulates varying voltages by varying the length of the digital power pulse within a given duration.
  
In addition to what's presented here, [Introduction to PWM: How Pulse Width Modulation works](https://www.kompulsa.com/introduction-pwm-pulse-width-modulation-works/) describes some of the same concepts described here as well as a few more examples of how PWM can be used. The [References](#references) section has links to more sources of PWM information.

The follow sections cover the main points:

* [Terminology](#Terminology) defines terms that will be used throughout the document
* [Primary concepts](#Primary-concepts) describes the major concepts

### Terminology

My research into PWM involved reading several articles[^1] [^2] [^3] as well as examining the code of several PWM software libraries in various languages [^4] [^5] [^6]. These various sources aren't completely consistent in the terminology they use. Here are some of the common terms, their aliases, and definitions. See the diagram below for a visual representation of the terms.

<img style="border:1px solid black" src="/images/pwmfordummies/PWMPulsePeriod.png" align="center" width="800" height="400"/>
<figcaption align="left"><center><i style="color:black;">PWM timing diagram</i></center></figcaption>

* __Frequency__ - Per Wikipedia [^11], _frequency is the number of occurrences of a repeating event per unit of time_. In electronics an event is the peak of a wave to the next peak of the wave (analog). In digital terms an event is from the leading edge of one pulse to the leading edge of the next pulse. Frequency is measured in Hertz, which is the number of repeating events per second. 
* __Period__ - Also from Wikipedia [^11], period is the duration of time of one cycle in a repeating event. Period is measured in seconds. So period is how long something takes vs. frequency which is how many times an event occurs in a given duration of time. This makes period the reciprocal of the frequency[^10] and vice versa, ie., _unit of time per event_ vs. _events per unit of time_. In the above diagram the period is 10 milliseconds (ms). So the frequency in the above diagram is `1/.01` or 100. Since frequency is measured in Hertz, or events/second, the frequency in the above diagram is 100Hz.
* __Clock source__[^8] - Also called a clock, it sets the rate at which the clock advances. 
* __PWM clock__ - Sometimes also called a clock so this gets a bit confusing. The PWM clock's underlying input is the clock source. Different hardware devices, such as motors and servos, only work within specific period ranges. Often the source clock is too fast for these devices. The PWM clock is created by dividing the clock source's frequency with a number that will result in the PWM clock operating at a frequency that's appropriate for a given device. Different devices will need different PWM clock speeds. The number used as the denominator in this calculation is frequently called the __divisor__ in software libraries and the BCM2835 Data Sheet[^9].
* __Pulse__ - From the "P" in PWM. This is the minimum length of time a PWM pin's output is set to high or low. Its minimum length is governed by the speed of the PWM clock. I'll use pulse throughout this document, mostly because it's in the name. 
* __Range__ - Range can be thought of as a counter that counts PWM clock pulses. The ratio of range to PWM clock frequency can be thought of as the frequency of the signal sent to a PWM pin. I've also seen the term cycle length used as as an alias for range[^5].
*  __Pulse width__ - Pulse width is the duration of a pulse. In the various software libraries I've seen it called width[^2], value[^4], data[^6] [^8], and duty length[^5].

### Primary concepts

* __Duty cycle__ is the ratio of Pulse Width to Range, i.e., `Pulse-Width/Range`. For a range of 10 and a pulse of 5, the duty cycle is 5/10 or 50%. Duty cycle regulates the output voltage of a PWM device. For a 50% duty cycle and an input voltage of 5 volts, the output voltage will be 2.5 volts.
* __Software vs. Hardware PWM__ - For the purposes of this article there are 2 ways to generate a PWM signal, software-based and hardware-based[^17]. Hardware-based PWM is generated by a dedicated hardware PWM device that can be configured to generate a PWM signal as described above[^17]. Hardware-based PWM produces a very uniform signal with regard to timing. A uniform signal is required, for example, to produce a flicker-free light source such as an LED. Software-based PWM is directly implemented in the executing program using a `while(true)` for loop that never ends which controls the amount of time a pin is allowing current to flow (pulse) vs. the amount of time the pin isn't allowing current to flow. In this case the uniformity of the signal is determined by the accuracy of a language's `sleep()` function and the OS (Linux) scheduler. A less uniform signal, for example, may result in a flickering light source. There is a more complete description of [the difference between soft PWM and PWM](https://raspberrypi.stackexchange.com/questions/100641/whats-the-difference-between-soft-pwm-and-pwm) and associated pros and cons on the Raspberry Pi Stack Exchange site.
* __Balanced vs. Mark/Space__ refers to the method used to determine how the PWM output signals are to be generated. There are 2 algorithms, balanced and mark/space. Balanced indicates that the duty cycle will be evenly spread across the range. That is to say, the pulse width will be split into a set of shorter pulses that are distributed across the range. In contrast, in mark/space, the pulse is generated as a single signal called a "mark". The time remaining in the range, `range-pulseWidth` , is called the "space". No signal is present in the space duration. Mark/Space is often good enough, but as periods get longer so does the absolute time difference between the mark and space durations. For large ratios of `range/PWMClockFrequency`, e.g., 1 (_which equates to 1Hz since the denominator unit is frequency_) and a duty-cycle of 50%, the space will be 500 milliseconds and the mark will be 500 milliseconds. This difference is large enough to be discernable in the behavior of the device. For example a motor might surge or a light flicker. In contrast, balanced mode will smooth out these differences. For the same 1Hz range and 50% duty-cycle, balanced mode might produce 500 1 millisecond signals every 2 milliseconds (`1/(500 * .002) = 1 = 1Hz`). The same 50% duty-cycle is produced, but the output signal is much smoother. Hardware implementations like the BCM2835 support both algorithms. It is possible to support both algorithms in software, but depending on the algorithms are implemented they may consume a significant amout of CPU.
 
## Overview of PWM on a Raspberry Pi 3B+

Physically PWM is implemented via the BCM2835's GPIO pins. The BCM2835 board has 40 pins, a subset of which are GPIO pins. Of the GPIO pins[^12] there are 4 hardware PWM pins, 13, 19, 12, and 18. The remaining GPIO pins, as well as the hardware GPIO pins, can be used for software PWM.

There are several clock sources available on the BCM2835 board.  The clock source used by the GPIO libraries in this article is called the oscillator. It's frequency as documented in several references[^4] [^5] [^6] is 19.2MHz with a period of about 52 nanoseconds (1 / 19,200,000 = ~0.000000052 seconds).

The BCM2835 board also implements something called a channel[^8]. A hardware PWM pin is controlled by a channel. The PWM Clock, range, and pulse width are specified for a channel. All hardware pins connected to that channel will share the same range, pulse-width, and duty-cycle. The BCM2835 board has 2 channels. GPIO pins[^12] 18 and 12 on one channel, 13 and 19 on the other. This means that a signal that's sent to either pin that share a channel will go to both pins. For example, sending a signal on GPIO12 will also be shared with GPIO18 and vice versa[^8] [^9] [^10].

## Exploring PWM on a Raspberry Pi

The setup for this exercise is identical to a combination of the [SunFounder Blinking LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html) and [Sunfounder RGB LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) projects. If you're familiar with wiring a breadboard the diagrams below may be all you need to get started. Otherwise it may be worth while looking at the Sunfounder LED projects and an [introduction to breadboards](http://wiki.sunfounder.cc/index.php?title=Breadboard_Basics_%E2%80%93_Types)[^18]. The resistor used in both diagrams is 220 Ohms.

<img style="border:1px solid black" src="/images/pwmfordummies/blinkingLED.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Blinking LED breadboard setup</i></center></figcaption>

This setup will be used to demonstrate software PWM on a non-PWM pin.

<img style="border:1px solid black" src="/images/pwmfordummies/RgbLed.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder RGB LED breadboard setup</i></center></figcaption>

This setup will be used to demonstrate hardware and software PWM on a PWM pin. This setup shows an RGB LED pin (green) connected to the GPIO hardware pin 18. A different hardware PWM pin may be used such as 12, 13, and 19. Also, the use of an RGB LED pin isn't required. Any type of LED will work. It will have to be wired up in a similar manner however, i.e., the LED positive lead should be connected to GPIO 18 (green wire) and the LED's ground lead (the longest lead) to the board's ground. If you do choose to use an RGB LED pin it's important to note that only 2 of the colors can be controlled by hardware PWM. The pins need to be on different PWM channels. In the diagram above the LED colors attached to GPIO pins 17 and 27 (blue and red) can only be used for software PWM.

It'll be easier to play with the different combinations of PWM settings when both of the above setups are wired up at the same time. It allows for any of the GPIO pins to be used by _PWM Explorer_, with the exception that for hardware PWM the LED(s) must be installed on hardware PWM pins (GPIO pins 13, 19, 18, and 12). If more than one hardware pin is used they must be on different channels.

### Driving PWM using PWM Explorer

PWM Explorer supports C and Go. Choosing C you can experiment with both Mark/Space and Balanced PWM Modes. The Go library I used, go-rpio[^5] only supports Mark/Space.

As described in the [Prerequisites](#prerequisites) section above, I wrote an application, called _PWM Explorer_, to experiment with PWM on a Raspberry Pi. This software can be used to drive PWM on both PWM and non-PWM pins. It also supports changing the various PWM parameters like divisor, range, and pulse width to provide visual feedback on the effect of these parameters on the behavior of the LEDs connected to the pins. This software is available on my [gpio GitHub repository](https://github.com/youngkin/gpio)[^20].

<img style="border:1px solid black" src="/images/pwmfordummies/pwmexplorer.jpg" align="center" width="1000" height="1000"/>
<figcaption align="left"><center><i style="color:black;">PWM Explorer</i></center></figcaption>

The diagram above is a screenshot of the main program and has labels with brief descriptions of the various sections of the user interface. It's a text based UI so that it can be run on a Raspberry Pi that doesn't have the desktop capability installed. It can be run at the command line by navigating to the installation directory under the `gpio/pwmdemo/pwmexplorer` directory and running `sudo /usr/local/go/bin/go run main.go`. `sudo` is needed because some of the GPIO access requires `root` permissions. In addition to the main UI application there are 2 supporting programs in the `gpio/pwmdemo/pwmexplorer/apps` directory, `freqtest.go` and `freqtest.c`. These are the actual implementations of the code required to interact with the PWM capabilities on the Raspberry Pi 3B+ using the go-rpio and WiringPi libraries. They are written in Go and C respectively.

There are a variety of PWM parameters supported. These are:

1. __PWM Pin__ - this item allows you to choose a PWM hardware pin to use. The pins available in the dropdown are specific to the language chosen. C uses the WiringPi[^6] library which uses its own pin numbering scheme. Go uses the standard GPIO pin numbers
2. __Non-PWM Pin__ - this item allows you to specify a PWM pin to use, even a hardware pin. PWM Pin and Non-PWM Pin are mutually exclusive and the program will prevent you from specifying both. As with PWM Pin above, the numbering scheme is specific to the language, C or Go, chosen. The program offers no protection against using the wrong pin numbering scheme so be careful what you specify. If the pin chosen doesn't behave as expected it may be that you used the wrong pin numbering scheme.
3. __Clock Frequency/Clock Divisor__ - this item is used to set the PWM clock frequency. The go-rpio library supports specifying the PWM clock frequency directly. The C WiringPi library uses the concept of divisor defined above to set the PWM clock frequency. You can calculate the divisor to use by dividing the Raspberry Pi 3B's oscillator clock's frequency of 19,200,000 Hertz by the desired PWM clock frequency. For example. to get a 100kHz PWM clock frequency divide 19,200,000 by 100,000. This calculation gives the Clock Divisor to use, 192 in this case. To avoid confusion, when C is the chosen language the label will be _Clock Divisor_. When Go is the chosen language this items label will be _Clock Frequency_.
4. __PWM Mode__ - this item is used to specify whether Mark/Space or Balanced modes will be used. Note: some combinations of language, pin type (PWM vs. non-PWM), and PWM Type (hardware/software) don't support Balanced mode. When this is the case a message will be displayed in the _Messages_ area. The Go go-rpio library doesn't support balanced mode.
5. __Range__ - the desired range as defined in the [Terminology](#Terminology) section above.
6. __Pulse Width__ - the desired pulse width as defined in the [Terminology](#Terminology) section above.
7. __PWM Type__ - this item is used to specify whether hardware or software PWM is to be used.

#### Experimenting with PWM Parameters

As was stated earlier, the WiringPi and go-rpio libraries both use the Raspberry Pi 3B's Oscillator clock which has a frequency is 19.2MHz. This is fixed and cannot be changed. But besides the pins chosen, the Clock Frequency/Divisor, PWM Mode, Range, Pulse Width, and PWM Type can all be modified. All of these interact with one another either directly or indirectly. In this section I will explain these relationships and how they interact. You can use the PWM Explorer to follow along and directly see the effects that I'll explain.

This section uses an LED to demonstrate the effect of the various parameters on a device. There is a property of the human eye that needs to be understood. The human eye perceives linear changes in in brightness in a logarithmic fashion. Specifically, at the lower end of a pulse width setting (lower voltage) changes in a given setting will produce what looks like a more significant result than the same change at the higher end (higher voltage)[^14]. The PWM Explorer doesn't compensate for this.

##### PWM Clock Frequency / Clock Divisor

The first thing to decide is what frequency you want the PWM clock to run at. It's frequency is specified directly when using Go and via the divisor when using C. Choosing this frequency is impacted by the type of PWM device being used, e.g., an LED or a motor. This article doesn't cover how to calculate this frequency, but there are sources that do[^2] [^3]. Instead I'll focus on the general impact of clock frequency on LED devices.

You may or may not be aware that the human eye can detect flickering starting at about 60Hz and below. Flickering is more apparent using peripheral vision. Given this, a PWM clock frequency to Range, `PWMClockFrequency / Range`, below 60Hz isn't ideal unless you're trying to create a blinking LED. Assuming the range can't be modified, choosing the right Clock Frequency/Divisor will directly impact whether an LED appears to be a steady light source, flickering, or blinking.

Here are some settings to try this out _(these assume the LED is wired up to GPIO pin 18)_:

| Setting | Go | C |
| ------- | -- | - |
| PWM Pin | 18 | 1 |
| Non-PWM Pin | N/A | N/A |
| Clock Frequency/Clock Divisor | 5000 (10000) | 3840 (1920) |
| PWM Mode | markspace | markspace |
| Range | 167 | 167 |
| Pulse Width | 2 | 2 |
| PWM Type | hardware | hardware |

Using the above settings will result in a 30Hz frequency at the pin. As this is well below 60Hz flickering will be apparent. Changing the PWM Clock Frequency/Clock Divisor will result in a pin frequency of 62.5Hz. The result will be that there is no apparent flickering.

__NOTE:__ _The "General Purpose Clock Divisors" on the BCM2835 have a register width of 12 bits (see[^9], page 108, `DIVI` field bits 23 thru 12). This means the maximum value of the Clock Divisor is 4095 (0 to 2^12-1). The go-rpio[^5] further states that PWM Clock frequencies below 4688Hz will result in "unexpected behavior" (rpio.go, see comments for `SetFreq()` function). Other sources[^15] state that 9.6Mhz is the highest available PWM Clock frequency (19.2Mhz/2). I've seen unexpected results with both frequencies below 4688Hz and above 9.6Mhz._

##### Range

Range effectively determines the frequency of the signal at the GPIO pin. This means that the frequency at the pin is defined by the ratio of `PWM Clock Frequency/Range`. Another thing range determines is the resolution of the signal going to the device. Recall that Duty Cycle is the ratio of Pulse Width to Range. Starting with a lower value of range, say 4, limits duty cycle to 0%, 25%, 50%, 75%, or 100%. To say it a different way, with a Range of 4 the only values for Pulse Width that make sense are 0, 1, 2, 3, 4. This in turn limits things like the range of LED brightness or blinking that's available.

Since range impacts frequency at the pin _and_ the resolution, it is important to choose the correct PWM clock frequency, Range, and Pulse Width in combination. Starting with a low PWM Clock frequency limits the choice of range which in turn limits the available duty cycles.

As an example let's choose 2 extremes. For the first let's choose a PWM Clock frequency of 5kHz and a range of 5. The resolution is 5. This means are only 5 available duty cycles including full on and full off. As described above this limits the available brightness settings for a light like an LED.

Here are the settings to try this example _(the settings for the second example are in parentheses)_:

| Setting | Go | C |
| ------- | -- | - |
| PWM Pin | 18 | 1 |
| Non-PWM Pin | N/A | N/A |
| Clock Frequency/Clock Divisor | 5000 (600000) | 3840 (32) |
| PWM Mode | markspace | markspace |
| Range | 5 (10000) | 5 (10000) |
| Pulse Width | 1 (5, 2, 100, 1000, 10000) | 1 (5, 2, 100, 1000, 10000) |
| PWM Type | hardware | hardware |

Then change to Pulse Width to 5. This will be the brightest setting of the LED. While it is noticably brighter than a Pulse Width of 1, due to the narrow range it's not that much brighter.

At the other extreme let's configure an LED for an extremely smooth transition from off to full brightness. This will require a higher resolution. Let's go crazy and decide that we'd like to have 10,000 steps of brightness available. This translates to a range of 10,000. To avoid visible flickering we need to have at least a 60Hz signal at the GPIO pin. Since the ratio of `PWM Clock Frequency/Range` determines the signal frequency at the pin, we will need a PWM Clock frequency of at least 600kHz. A range of 10000 times a minimum pin signal frequency of 60Hz is 600kHz (10,000 * 60 = 600,000). To check our work, `PWM Clock Frequency / Range`, 600,000 / 10,000 = 60 (Hz).

##### Pulse width

At higher GPIO pin frequencies pulse width effects the brightness of an LED. At lower GPIO pin frequencies pulse width is visible as the length of time the LED is on vs. the blink rate. Let's again use extreme examples to illustrate this.

For the first example let's use a PWM clock frequency of 1MHz and a range of 10,000. The frequency at the GPIO pin will be 100Hz 1,000,000/10,000), fast enough that no blinking or flickering will be visible. The range of 10,000 is likewise high enough that we won't be able to discern discrete steps in the brightness of the LED.

Here are the settings for the first example:

| Setting | Go | C |
| ------- | -- | - |
| PWM Pin | 18 | 1 |
| Non-PWM Pin | N/A | N/A |
| Clock Frequency/Clock Divisor | 1000000 (5000) | 19 (3840) |
| PWM Mode | markspace | markspace |
| Range | 10000 | 10000 |
| Pulse Width | 100 (5000) | 100 (5000) |
| PWM Type | hardware | hardware |

For the second example change the Clock Frequency (Go) to 5000, Clock Divisor (C) to 3840, the Range to 10000 and the Pulse Width to 5000. The LED will flash for a very short period of time and will repeat flashing about once every 2 seconds (Clock Frequency/Range, 5000/10000 = 0.5Hz or once every 2 seconds). Now change the Pulse Width to 5000. Now the LED will be on for 1 second and off for 1 second. This changed the duty cycle from 2% to 50% with the corresponding change in the duration of the LED flash. This example shows how to create a blinking light using PWM, just choose a PWM Clock Frequency and range that result in a very low GPIO pin frequency, well below 60Hz.

##### PWM Mode

The available PWM modes are balanced and mark/space. Recall that with mark/space the signal is either on or off (high or low) for a fixed duration within the range. For example, with a duty cycle of 50% and a range of 10, the signal will be on for 5 consecutive seconds and off for 5 consecutive seconds. As explained above, balanced mode will spread this 50% duty cycle evenly across the range which will make the effects of the duty cycle less apparent, except for a dimming of the LED (e.g., a 10ms pulse, every 10ms, 500 times over 10 seconds).

To illustrate this behavior set the Clock Frequency to 4688 (Go), Clock Divisor to 4095 (C), the PWM Mode to markspace, the Range to 1000, the Pulse Width to 10, and the PWM Type to hardware. This results in a blinking LED at a pin frequency of about 4.69Hz, or almost 5/second. Now change the PWM Mode to balanced. The language must be C to use balanced mode as go-rpio only supports mark/space. Now the blinking is no longer apparent.

| Setting | Go | C |
| ------- | -- | - |
| PWM Pin | 18 | 1 |
| Non-PWM Pin | N/A | N/A |
| Clock Frequency/Clock Divisor | 4688 | 4095 |
| PWM Mode | markspace (~~balanced~~) | markspace (balanced) |
| Range | 1000 | 1000 |
| Pulse Width | 10 | 10 |
| PWM Type | hardware | hardware |

##### PWM Type

The available PWM Types are _hardware_ and _software_. The examples above used hardware PWM only. This is because true balanced mode is only available on hardware. In addition, the software PWM as implemented in Go and C is more akin to balanced mode and definitely not mark/space. Both the Go and C versions have a hardcoded 100MHz clock. Some of the examples in previous sections rely on mark/space behavior, especially the PWM Mode section.

##### Other things to try

To see how the above works with non-PWM pins, try some or all of the above on non-hardware PWM pins using a PWM Type of software. Monitoring CPU usage during these tests might show some differences between CPU usage for hardware vs. software PWM.

##### PWM Explorer code

The code in the [PWM Explorer](https://github.com/youngkin/gpio) GitHub respository is extensively commented. To avoid repeating what is best illustrated directly in the code there's not much to say here. The structure of both programs is substantially similar although there are differences is how the Go and C libraries are used to accomplish the same task.

##### Key takeaways

1. On the Raspberry Pi there are only 4 hardware PWM pins and 2 channels. There are 2 pins per channel. This reduces the number of effective hardware PWM pins to 2.
2. Care must be taken when specifying the Clock Frequency or Clock Divisor. The acceptable range for PWM Clock frequency is 4688Hz to 9.6Mhz.
3. Range determines both the frequency at the GPIO pin and the resolution of the signal to a device like an LED or motor. The first, frequency, is significant because at lower pin frequencies the on/off states of the pin become very apparent, as in a blinking LED light. The second, resolution, is important because it defines how fine grained the control of the device will be. For example, at low resolutions there are only a limited set of LED brightnesses that can be chosen and the differences are very apparent. At higher resolutions small changes in pulse width will result in a very small, hardly discernable, change in brightness. This is important when implementing something like a dimmer where the change in brightness from off to fully on should not change in large, discernable steps of brightness. I.e., a smoother transition through the range is desired.
4. For higher resolutions (Range) a higher PWM clock frequency should be chosen. This ensures that the GPIO pin frequency won't drop too low, e.g., below the 60Hz frequency needed to avoid a flickering LED light.
5. At higher GPIO pin frequencies (e.g., above 60Hz) changing the pulse width will cause the LED to appear dimmer or brighter.
6. At lower GPIO pin frequencies (e.g., below 60Hz, and especially below 10Hz) pulse width can be used to cause obvious pulsing, as in a blinking LED light. The length of time the light is on is directly proportional to the pulse width.
7. Choice of PWM mode also impacts the "smoothness" of the signal. Even at low GPIO frequencies, which would normally cause an LED to blink, balanced mode will reduce that effect, often to the point of rendering it unnoticable.
8. PWM type (software vs. hardware) can affect the consistency of the period of a PWM signal to the point that even above 60Hz an LED might noticably flicker. It can also result in an inordinate amount of CPU load needed to support the desired clock rate. 
9.  Software PWM can be used, but the resulting signal can be too ragged to be used in some devices.
10. Setting a range that is greater than the PWM Clock frequency may seem odd, but it can be used effectively to create a low frequency, pulsing signal that can be used for, among other things, causing an LED to blink.

## Summary

Hopefully by now you have enough knowledge to start effectively using PWM on a Raspberry Pi 3B+, or perhaps on other Raspberry Pi models or even on other platforms entirely. Specifically:

* We've gone through the various terms so you should now be able to read most of the literature on PWM and understand the concepts and terms used in those sources.
* If you followed along setting up and experimenting with PWM on your Raspberry Pi you now have a full, working, GPIO test bed installed on your Raspberry Pi.
* You should have an understanding of the various PWM settings and how they interact with each other.
* If you used the _PWM Explorer_ application you have valuable hands-on experience with using various combinations of PWM settings. You can continue to use the _PWM Explorer_ to experiment with various settings when you have questions or to test assumptions.
* By reading the Go and/or C code you now know how to effectively use the go-rpio and/or WiringPi libraries. Even if you decide to use other libraries, like pigpio[^13], you should understand the terminology well enough to use them effectively, or at least have a good start on learning how to use them.

## References

[^1]: [Introduction to Microcontroller Timers: Periodic Timers](https://www.allaboutcircuits.com/technical-articles/introduction-to-microcontroller-timers-pwm-timers/) is a good general introduction to timers. PWM hardware is one type of periodic timer. It is useful to read this introduction.
[^2]: [Pulse-width Modulation (PWM) Timers in Microcontrollers](https://www.allaboutcircuits.com/technical-articles/introduction-to-microcontroller-timers-pwm-timers/) is a good detailed discussion about PWM timers. It's an excellent read if you want more detail than presented here.
[^3]: [Raspberry Pi And The IoT In C - - Pulse Width Modulation, Servos And More](https://www.iot-programmer.com/index.php/books/22-raspberry-pi-and-the-iot-in-c/chapters-raspberry-pi-and-the-iot-in-c/60-raspberry-pi-and-the-iot-in-c-pulse-width-modulation-servos-and-more) is a detailed book about programming GPIO on the Raspberry Pi. There is a chapter devoted to PWM.
[^4]: [WiringPi](https://github.com/WiringPi/WiringPi) is a C library for GPIO programming
[^5]: [go-rpio](https://github.com/stianeikeland/go-rpio) is a Go library for GPIO programming
[^6]: [bcm2835 by Mike McCalley](https://www.airspayce.com/mikem/bcm2835/) is another C library for GPIO programming.
[^8]:The [full Broadcom spec for the BCM2835](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf), starting at page 138, for more details about how PWM is implemented on the BCM2835.
[^9]: There's a detailed discussion about [Which pin(s) on RPi 3B is PWM capable](https://www.raspberrypi.org/forums/viewtopic.php?t=150254). Specifically regarding the effect of sharing 2 PWM channels for 4 PWM pins. The Broadcom spec[^8] also discusses this in section 9.4 on page 139, but in a less obvious way.
[^10]: A [Frequency to Period Calculator](https://www.sensorsone.com/frequency-to-period-calculator/) is handy for quick calculations, or validation, of frequency to period conversion (period = 1/frequency).
[^11]: [Frequency page on Wikipedia](https://en.wikipedia.org/wiki/Frequency#CITEREFSerwayFaughn1989) discusses frequency and period
[^12]: [RaspberryPPi Pinout](https://pinout.xyz/) is a good source that describes each pins' role, physical pin number, and for GPIO pins the GPIO pin number for the BCM2835 board. It has tabs that can be used to highlight which pins serve which purpose, e.g., PWM pins.
[^13]: [pigpio](https://abyz.me.uk/rpi/pigpio/pdif2.html) is primarily a Python library. It also has a C library. It has the distinction of performing hardware PWM on any GPIO pin. I'm not sure how it does this.
[^14]: [Linear LED PWM](https://jared.geek.nz/2013/feb/linear-led-pwm) provides guidelines/formulas for getting linear scaling when changing the brightness of an LED via PWM. Googling "LED PWM linear brightness" brings up several other articles as well.
[^15]: [Driving PWM output frequency](https://raspberrypi.stackexchange.com/questions/53854/driving-pwm-output-frequency) provides some interesting information and discussion on Raspberry Pis with 26 and 40 GPIO pins.
[^16]: [Setting up your Raspberry Pi](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up) is the official documentation for getting started with a Raspberry Pi including things like required hardware as well as how to install the OS and other important details.
[^17]: There is a 3rd way to generate PWM signals, [DMA or Direct Memory Access](https://stackoverflow.com/questions/50427275/raspberry-how-does-the-pwm-via-dma-work). It won't be discussed here in this article.
[^18]: [Breadboard Basics - Types](http://wiki.sunfounder.cc/index.php?title=Breadboard_Basics_%E2%80%93_Types) is a useful introduction to breadboards and how to use them.
[^19]: [WiringPi library reference guide](http://wiringpi.com/reference/) documents the main WiringPi functions. The rest of the website also has some useful information.
[^20]: [PWM Explorer](https://github.com/youngkin/gpio) is an application that accompanies this article that can be used to experiment with various PWM settings. It demonstrates PWM using both Go and C libraries.