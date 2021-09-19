---
title: "Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi"
description: "An overview of PWM, with code examples, on the Raspberry Pi."
date: 2021-09-19T13:13:42-06:00
draft: false
image: "images/PWMPulsePeriod.png"
tags: ["raspberry-pi", "Golang", "C", "GPIO"]
categories: ["raspberry-pi", "Golang", "GPIO"]
GHissueID: 1
toc: true
---

This article provides some details about hardware and software based PWM on the Raspberry Pi, specifically the 3B+ with the [Broadcomm BCM2835 ARM Peripherals](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf).

<!--more-->

## Overview

I was writing an article titled [Raspberry Pi GPIO in Go and C - RGB LED](sunfounderergpionoesrgbled) as a companion to the [SunFounder RGB LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html) to cover some areas that were missing in that project like what is PWM, how does it work, and why the code was written as it was. I had no idea what PWM was or what I was doing or why. To fill the gap I started researching PWM and found numerous articles about PWM[^1] [^2] [^3], and also software libraries like [WiringPi](https://github.com/WiringPi/WiringPi) (C), [go-rpio](https://github.com/stianeikeland/go-rpio) (Go), [BCM2835 by Mike McCalley](https://www.airspayce.com/mikem/bcm2835) (C) and [pigpio](https://abyz.me.uk/rpi/pigpio/pdif2.html) (Python & C). So with my focus still on controlling an RGB LED via GPIO and PWM I started writing programs in C and Go. I quickly had difficulty in several areas:

1. Terminology was used inconsistently across articles and the libraries. For example, some used the term _Period_ while others used _Cycle_ or _Range_ for the same thing.
2. The libraries were documented to varying degrees. Coupled with a lack of common terminology it was difficult to figure out how to use them.
3. Going hand in hand with figuring out how to use the libraries, it was difficult to understand how different parameters affected the behavior of the hardware, in this case an RGB LED.

I wrote this article for a variety of reasons:

1. A reference for myself when I do more PWM development in the future.
2. As an index to articles I found helpful in understanding various concepts like _Duty Cycle_ and the relationships between _frequency_, _Period_, and _Duty Cycle_.
3. To serve as a set of sample programs that illustrate the various ways of writing an application to control an RGB LED, in C and Go, using the various libraries. With regard to libraries, I've chosen WiringPi (C) and go-rpio (Go) as the libraries as they seemed best suited for what I'm doing. Other libraries may be useful as well. For those that want to develop using Python, the terminology used in WiringPi and go-rpio should map well enough over to python libraries like pigpio and [RPi.GPIO](https://pypi.org/project/RPi.GPIO/).
4. To provide an application that makes it easy to explore the how the various PWM parameters affect hardware behavior.
5. To hopefully help others who find themselves in a similar situation when trying to develop PWM applications.

Lest anyone think I'm presenting myself as an expert in GPIO and PWM, I'm not. Despite my best attempts to summarize what I've learned I'm sure I've made mistakes or made things more confusing. Any mistakes I've made are my own. If anyone comes across any mistakes are things certain areas should be clarified I'd appreciate comments so I can modify this article.

I'm not writing this to be a definitive source about all things PWM. There are better resources for that. This article lists those resources that I found particularly helpful.

Now to the outline of this article. The sections are as follows:

1. [Prerequisites](#prequisites) covers areas like how to get the software libraries used in this article as well as things like the hardware needed to illustrate behavior.
2. [What is PWM?](#what-is-pwm) provides a very basic intro to PWM, tries to provide definitions of common terms and the aliases used in the articles and software libraries.
3. [Overview of PWM on a Raspberry Pi 3B+](#overview-of-pwm-on-a-raspberry-pi-3b) provides an overview of how GPIO and PWM are implemented on a Raspberry Pi.
4. [Exploring PWM on a Raspberry Pi](#exploring-pwm-on-a-raspberry-pi) has several subsections about controlling an LED, and an RGB LED, with software on a Raspberry Pi. It will have examples in Go and C, using different libraries, as well as showing the implementations of hardware and software PWM.
5. And a [Summary](#summary) to wrap things up.

## Prerequisites

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'stretch' version of the Raspbian OS. Given that the Sunfounder Ultimate Starter Kit is advertised to work with a Raspberry Pi 4, I would expect the 4 series to work as well. I'm less sure about other Raspberry Pi versions, especially versions with 26 vs. 40 GPIO pins.

Next you'll need is a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), [a 220 Ohm resistor, and a RGB LED](https://www.amazon.com/EDGELEC-Tri-Color-Multicolor-Diffused-Resistors/dp/B077XGF3YR/ref=asc_df_B077XGF3YR/?tag=hyprod-20&linkCode=df0&hvadid=242051162351&hvpos=&hvnetw=g&hvrand=11064062033670066895&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9028749&hvtargid=pla-430228081645&psc=1). You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the cable will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however. [Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). If you expect to follow this series I recommend buying the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).

<img style="border:1px solid black" src="/images/RaphaelKit.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Ultimate Starter/Raphael kit</i></center></figcaption>

You will also need some basic C  and Go programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like Vi or nano.

To compile and run the C program you'll need the WiringPi libary. It's easy to get:

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

I chose not to download the code from the Sunfounder site, preferring to write my own instead, even if all I did was copy directly from the project documentation. Due to this I created my own location to create the code. In fact, [my code is in Github](https://github.com/youngkin/gpio). If you do like downloading code you have the option of downloading, cloning, or forking it from my Github repository. As an added bonus, the project code written in Go is also located there. The code for this project is located at [gpio/ledblink](https://github.com/youngkin/gpio/tree/main/ledblink).

If you're interested in Go development on a Raspberry Pi you'll need to install the development environment onto the Raspberry Pi. [Here's a simple source](https://www.jeremymorgan.com/tutorials/raspberry-pi/install-go-raspberry-pi/) that explains how to accomplish this. This source is a little dated, but the only significant issue is with the version of Go to install. The source shows installing Go __1.14.4.linux-arm64.tar.gz__ and __1.14.4.linuxarmv6l.tar.gz__. The current versions are __1.17.1.linux-arm64.tar.gz__ and __1.17.1.linuxarmv6l.tar.gz__. For the Raspberry Pi 3B+ the correct choice will be __1.17.1.linuxarmv6l.tar.gz__. The other is intended for 64 bit systems like the Raspberry Pi 4 series.

Finally, I'm assuming a basic knowledge of Linux if you want to veer away from the cookbook style of the Sunfounder docs. For example, I won't be explaining what __root privileges__ are.

## What is PWM?

At it's most basic, PWM is used to simulate an analog signal using a digital source (like a GPIO pin). As an example, an LED's brightness and/or color can be modified by varying the voltage supplied to the LED. A variety of analog devices, such as a motor's speed, can be controlled in the same way. PWM simulates varying voltages by varying the length of the digital power pulse within a given duration (period). The ratio of the power pulse vs the period is called a duty cycle. For example, a time duration of 10 seconds and a pulse length of 1 second has a duty cycle of 1/10, or 0.1, or 10%. This example duty cycle means power is flowing for 10% of the time. A duty cycle of 10% with a 10 volt input would  result in a 1 volt output. [Introduction to PWM: How Pulse Width Modulation works](https://www.kompulsa.com/introduction-pwm-pulse-width-modulation-works/) provides a quick overview of PWM.

### Terminology

My research into PWM involved reading several articles[^1] [^2] [^3] as well as examining the code of several PWM software libraries in various languages [^4] [^5] [^6]. These various sources aren't completely consistent in the terminolgy they use. Here are some of the common terms, their aliases, and definitions. See the diagram below for a visual representation of the terms.

<img style="border:1px solid black" src="/images/PWMPulsePeriod.png" align="center" width="800" height="400"/>
<figcaption align="left"><center><i style="color:black;">PWM timing diagram</i></center></figcaption>

1. __Clock source__[^8] - Also called a clock, it sets the rate at which the clock advances. Several references[^4] [^5] [^6] indicate that BCM2835 the clock frequency is 19.2MHz or about every 50 nanoseconds (1 / 19,200,000).
2. __PWM clock__ - Sometimes also called a clock so this gets a bit confusing. The PWM clock's underlying input is the clock source. Different hardware devices, such as motors and servos, only work within specific periond ranges. Often the source clock is too fast for these devices. The PWM clock is created by dividing the clock source's frequency with a number that will result in the PWM clock operating at a frequency that's appropriate for a given device. Different devices will need different PWM clock speeds. The number used as the denominator in this calculation is frequently called the __divisor__ in software libraries.
3. __Frequency__ - Per Wikipedia [^11], "frequency is the number of occurrences of a repeating event per unit of time". 
4. __Period__ - Also from Wikipedia [^11], period is the duration of time of one cycle in a repeating event, so the period is the reciprocal of the frequency[^10]. So period is how long something takes vs. frequency which is how many times a thing occurs in a given duration of time. In the context of PWM, period is also called range [^5] [^6] or cycle[^8] [^4] .
5. __Pulse__ - From the "P" in PWM. This is the length of time a PWM pin's output is set to high. Its minimum length is governed by the speed of the PWM clock. I'll use pulse throughout this document, mostly because it's in the name.
6. __Range__ - Range can be thought of as a counter that counts PWM clock pulses. The ratio of range to PWM clock frequency can be thought of as the frequency of the signal sent to a PWM pin. I've also seen the term cycle length used as as an alias for range[^5].
7. __Pulse width__ - is the duration of a pulse. In the various software libraries I've seen it called width[^2], value[^4], data[^6] [^8], and duty length[^5].
8. __Duty cycle__ - is the ratio of Pulse Width to Range, i.e., Pulse-Width/Range. For a range of 10ms and a pulse of 5ms, the duty cycle is 5/10 or 50%. Duty cycle regulates the output voltage of a PWM device. For a 50% duty cycle and an input voltage of 5 volts, the output voltage will be 2.5 volts.
9. __Algorithm__ - Algorithm refers to the method used to determine how the PWM hardware will output signals. There are 2 algorithms, balanced and mark/space. Balanced indicates that the duty cycle will be evenly spread across the period. That is to say, the pulse will be split into a set of shorter pulses that are distributed across the period. In contrast, in mark/space, the pulse is generated as a single signal called a "mark". The time remaining in the period, period-length - pulse-length, is called the "space". No signal is present in the space duration. Mark/Space is often good enough, but as periods get longer so does the absolute time difference between the mark and space durations. For long periods, e.g., 1 second and a duty-cycle of 50%, the space will be 500 milliseconds and the mark will be 500 milliseconds. This difference is large enough to be discernable in the behavior of the device. For example a motor might surge or a light flicker. In contrast, balanced mode will smooth out these differences. For the same 1 second period and 50% duty-cycle, balanced mode might produce 500 1 millisecond signals every 2 milliseconds. The same 50% duty-cycle is produced, but the output signal is much smoother.
10. __Channel__ [^8] - a PWM pin is controlled by a channel. For example, the period and pulse are specified for a channel. Any pin(s) connected to that channel will share the same period/pulse/duty-cycle.

For the purposes of this article there are 2 ways to generate a PWM signal, software-based and hardware-based. Hardware-based PWM is generated by a dedicated hardware PWM device that can be configured to generate a PWM signal as described above. It produces a very uniform signal with regard to timing. A uniform signal is required, for example, to produce a flicker-free light source such as an LED. Software-based PWM is directly implemented in the executing program using a `while(true)` for loop that never ends which controls the amount of time a pin is allowing current to flow (pulse) vs. the amount of time the pin isn't allowing current to flow. In this case the uniformity of the signal is determined by the accuracy of a language's `sleep()` function and the OS (Linux) scheduler. A less uniform signal, for example, may result in a flickering light source. There is a more complete description of [the difference between soft PWM and PWM](https://raspberrypi.stackexchange.com/questions/100641/whats-the-difference-between-soft-pwm-and-pwm) and associated pros and cons on the Raspberry Pi Stack Exchange site.

## Overview of PWM on a Raspberry Pi 3B+

Software PWM can be created on any GPIO pin. For hardware PWM the Raspberry Pi 3B+ has 4 PWM pins, BCM GPIO pins 12,13, 18, and 19 [^8]. The Broadcom BC2835 GPIO board that's on the Raspberry Pi 3B+ has 2 PWM channels[^8]. GPIO12 and GPIO18 share one channel and GPIO13 and GPIO19 share the other channel. This means that a signal that's sent to either pin that share a channel will go to both pins. For example, sending a signal on GPIO12 will also be shared with GPIO18 and vice versa[^8] [^9] [^10].

So why is it important to know that there are only 2 PWM channels for 4 PWM pins? This project uses an RGB LED that has 4 pins. One pin is for the ground, the other 3 are used to control the red, green, and blue LEDs. So we need 3 PWM pins, one for each RGB color. See the problem? There are only 2 independent pins available for hardware PWM but we need 3 to control the RGB LED. Let's say that GPIO12 is configured as the color red and GPIO18 is configured as blue. If a signal is sent to GPIO12 which represents the color red, the same signal will be sent to GPIO18, blue. So the resulting color will be purple, not red as expected.

## Exploring PWM on a Raspberry Pi

The setup for this exercise is identical to the [SunFounder Blinking LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html). If you're familiar with wiring a breadboard the diagram below may be all you need to get started. The resistor is 220 Ohms.

<img style="border:1px solid black" src="/images/blinkingLED.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Blinking LED breadboard setup</i></center></figcaption>

### Dim LED

Where's the code?!?

#### Software

#### Hardware

##### Mark/Space

##### Balanced

Only in C/WiringPi, go-rpio doesn't support it

### Playing with PWM clock frequency, range, and duty cycle

## Summary

[^1]: [Introduction to Microcontroller Timers: Periodic Timers](https://www.allaboutcircuits.com/technical-articles/introduction-to-microcontroller-timers-pwm-timers/) is a good general introduction to timers. PWM hardware is one type of periodic timer. It is useful to read this introduction.
[^2]: [Pulse-width Modulation (PWM) Timers in Microcontrollers](https://www.allaboutcircuits.com/technical-articles/introduction-to-microcontroller-timers-pwm-timers/) is a good detailed discussion about PWM timers. It's an excellent read if you want more detail than presented here.
[^3]: [Raspberry Pi And The IoT In C - - Pulse Width Modulation, Servos And More](https://www.iot-programmer.com/index.php/books/22-raspberry-pi-and-the-iot-in-c/chapters-raspberry-pi-and-the-iot-in-c/60-raspberry-pi-and-the-iot-in-c-pulse-width-modulation-servos-and-more) is a detailed book about programming GPIO on the Raspberry Pi. There is a chapter devoted to PWM.
[^4]: [WiringPi](https://github.com/WiringPi/WiringPi) is a C library for GPIO programming
[^5]: [go-rpio](https://github.com/stianeikeland/go-rpio) is a Go library for GPIO programming
[^6]: [bcm2835 by Mike McCalley](https://www.airspayce.com/mikem/bcm2835/) is another C library for GPIO programming.
[^7]: The Raspberry Pi's GPIO PWM pins can also be configured as non-PWM pins
[^8]:The [full Broadcom spec for the BCM2835](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf), starting at page 138, for more details about how PWM is implemented on the BCM2835.
[^9]: There's a detailed discussion about [Which pin(s) on RPi 3B is PWM capable](https://www.raspberrypi.org/forums/viewtopic.php?t=150254). Specifically regarding the effect of sharing 2 PWM channels for 4 PWM pins. The Broadcom spec[^8] also discusses this in section 9.4 on page 139, but in a less obvious way.
[^10]: A [Frequency to Period Calculator](https://www.sensorsone.com/frequency-to-period-calculator/) is handy for quick calculations, or validation, of frequency to period conversion (period = 1/frequency).
[^11]: [Frequency page on Wikipedia](https://en.wikipedia.org/wiki/Frequency#CITEREFSerwayFaughn1989) discusses frequency and period
[^12]: [RaspberryPPi Pinout](https://pinout.xyz/) is a good source for for the BCM2835 chip. It has tabs that can be used to highlight which pins serve which purpose, e.g., PWM pins.
[^13]: [pigpio](https://abyz.me.uk/rpi/pigpio/pdif2.html) has a C library. It has the distinction of performing hardware PWM on any GPIO pin.
