---
title: "Raspberry Pi GPIO in Go and C - LED Bar Graph"
description: "How to control an LED bar graph in Go and C"
date: 2021-11-04T13:13:42-06:00
draft: false
image: "images/ledbargraph/ledbargraphheader.png"
tags: ["raspberry-pi", "Go", "C", "GPIO"]
categories: ["raspberry-pi", "Go", "GPIO"]
GHissueID: 1
toc: true
---

<!--more-->

## Overview

This is the third article in a series that explores GPIO programming on a Raspberry Pi 3B+. It is a supplement to the [Sunfounder LED Bar Graph](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) project. You can find the full series [here](https://youngkin.github.io/categories/gpio/). The code for the series can be found in my [gpio](https://github.com/youngkin/gpio) repository.

The [Sunfounder LED Bar Graph](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) project provides very good documentation regarding how to set up the project as well as describing the C code. This article is more geared towards describing what an LED bar graph is, what it's good for, and how to control an LED bar graph using Go as well as C.

LED bar graphs consist of several LEDs embedded into a single component. In the picture to the right the bar graph contains 10 LEDs arranged side-by-side.

<img style="border:1px solid white; padding: 15px;" src="/images/ledbargraph/bargraphimage.png" align="right" width="300"/>

LED bar graphs have several uses including:

* Progress indicators
* Battery charge
* Voltmeter
* Sound meter
* Speed

In short, just about anything that requires an indication of a value at a relative position on a scale can be represented by an LED bar graph.

## Prerequisites

This section is repeated in all articles in my [Raspberry Pi GPIO series](https://youngkin.github.io/categories/gpio/). If you've already completed a project from one of these articles you can skip this section.

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

## LED Bar Graph in C

The code for this program can be found in the [ledbargraph.c](https://github.com/youngkin/gpio/blob/main/ledbargraph/ledbargraph.c) file in the [github respository](https://github.com/youngkin/gpio) accompanying this series. It substantially similar to the [Sunfounder program](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html), but there are some important differences:

* This program includes an initialization function that turns all the LEDs on then off prior to continuing with the remainder of the program. This can be useful to verify that the LED bar graph is wired up correctly.
* This program includes an interrupt handler that cleans up when the program is interrupted via entering `ctl-C` at the terminal. The Sunfounder code does not. This cleanup includes resetting the bar graph to its state prior to the program starting, e.g., turning all the LEDs off.
* Finally, this program also includes a function that randomly lights individual LEDs.

The major parts of the program are described in more detail below. The code is pretty well described in the [Sunfounder article](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) so I won't repeat what's already covered there.

{{< gist youngkin d3870141ea2a8de68cad62135fa488ed >}}

Lines 7 & 8 show how to build and run the program. Line 19 declares the interrupt handler that's used to intercept `ctl-C` input from the terminal. The rest of this section shows the `init()` function I added. It iterates through the `pins` array and first sets all the pins to `LOW` which has the effect of lighting them up. They're set to `LOW` because the positive terminals of the LED bar graph are connected to the positive power bus. The pins are connected to the negative terminals of the bar graph. To get current to flow the pin states must be set to 0 volts, or `LOW`. The bar graph stays lit for 500 microseconds, then the LEDs are turned off, then there's another delay of 500 microseconds before the program continues. The delays provide enough time to see the entire bar graph light up then turn off before the program continues. This provides a visual indication that the bar graph is connected properly.

{{< gist youngkin 7c1b89402b9e73cbf4d1e86995472a7b >}}

This next section contains the functions that control the bar graph LEDs. The `oddLedBarGraph()`, `evenLedBarGraph()`, and `allLedBarGraph()` functions are taken directly from the Sunfounder code. I added the `randomBarGraph()` function on lines 1 - 12 just to make things more interesting. It uses the C standard library `rand()` function, line 5, to generate a random number that is adjusted on line 6 to be within the range of the `pins` array, offsets 0 through 9. That resulting number is used as the offset in the array to find the pin to be toggled (lines 7 and 9). There is a very brief delay between lighting the individual LEDs.


{{< gist youngkin 95819fcaa718f6fcc5f55821e63431b2 >}}

This last section of code shows the `main()` function and the interrupt handler, `interruptHandler()`. With the exception of the addition of lines 18 and 19, `main()` is identical to the Sunfounder code.

Lines 26 through 34 define the `interruptHandler()` function. It's parameter, sig, contains the integer value of the signal being passed to the function. In our case it doesn't matter what signal is passed, it'll exit the program in any case. 

Lines 27 thorugh 30 iterate through the `pins` array making sure each pin is set to `OUTPUT` mode, i.e., it can be written to, and setting it's value to `HIGH`. Recall from above that setting the pin to `LOW` will light the pin, setting it to `HIGH` will turn it off.

Finally, the program exits on line 33 with a "success", 0, exit code.

## LED Bar Graph in Go

## References
