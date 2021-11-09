---
title: "Raspberry Pi GPIO in Go and C - LED Bar Graph"
description: "How to control an LED bar graph in Go and C"
date: 2021-11-08T13:13:42-06:00
draft: false
image: "images/ledbargraph/ledbargraphheader.png"
tags: ["raspberry-pi", "Go", "C", "GPIO"]
categories: ["raspberry-pi", "Go", "GPIO"]
GHissueID: 1
toc: true
---

<!--more-->

## Overview

This is the third article in a series that explores GPIO programming on a Raspberry Pi 3B+. It is a supplement to the [Sunfounder LED Bar Graph](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) project. You can find the full series [here](https://youngkin.github.io/categories/gpio/). The code for the series can be found in my [gpio repository](https://github.com/youngkin/gpio).

The [Sunfounder LED Bar Graph](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) project provides very good documentation regarding how to set up the project as well as describing the C code. This article is more geared towards describing what an LED bar graph is, what it's good for, and how to control an LED bar graph using Go as well as C.

LED bar graphs consist of several LEDs embedded into a single component. In the picture to the right the bar graph contains 10 LEDs arranged side-by-side.

<img style="border:1px solid white; padding: 15px;" src="/images/ledbargraph/bargraphimage.png" align="right" width="250"/>

LED bar graphs have several uses including:

* Progress indicators
* Battery charge
* Voltmeter
* Sound meter
* Speed

In short, just about anything that requires an indication of a value at a relative position on a scale can be represented by an LED bar graph.

## Prerequisites

This section is repeated in all articles in my [Raspberry Pi GPIO series](https://youngkin.github.io/categories/gpio/). If you've already completed a project from one of these articles you can skip this section.

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'Stretch' version of the Raspbian OS. The Raspberry Pi website has instructions on how to [setup a new Raspberry Pi from scratch](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up) if you decide to go that way vs. buying a complete kit.

Other items you'll need include:

* a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) _(You may find [this tutorial on breadboards](http://wiki.sunfounder.cc/index.php?title=Breadboard_Basics_%E2%80%93_Types) helpful)_,
* some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==),
* [a 220 Ohm resistor, and a RGB LED](https://www.amazon.com/EDGELEC-Tri-Color-Multicolor-Diffused-Resistors/dp/B077XGF3YR/ref=asc_df_B077XGF3YR/?tag=hyprod-20&linkCode=df0&hvadid=242051162351&hvpos=&hvnetw=g&hvrand=11064062033670066895&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9028749&hvtargid=pla-430228081645&psc=1).
* You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the T-Type adapter will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however.

[Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). I'm finding the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1) especially useful. NOTE: The Ultimate Starter Kit and the Raphael Kit are the same product.

<img style="border:1px solid black" src="/images/pwmfordummies/RaphaelKit.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Ultimate Starter/Raphael kit</i></center></figcaption>

You will also need some basic C  and Go programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like vim or nano.

To compile and run the C program you'll need the [WiringPi](https://github.com/WiringPi) library. It's easy to get:

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

For Go development you'll also need the [go-rpio](https://github.com/stianeikeland/go-rpio) library.

If you want to veer away from the cookbook style of the Sunfounder docs you'll need some basic knowledge of Linux . For example, I won't be explaining what __root privileges__ are.

## Setup and Code

<img style="border:1px solid black" src="/images/ledbargraph/breadboard.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder wiring diagram</i></center></figcaption>

<img style="border:1px solid black" src="/images/ledbargraph/ledbargraph.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Actual wiring</i></center></figcaption>

The breadboard should be wired as illustrated in the above diagram. If you're unfamiliar  with breadboards and breadboard diagrams this [breadboard tutorial ](https://www.sciencebuddies.org/science-fair-projects/references/how-to-use-a-breadboard) should be helpful.

One thing to note about the wiring diagram is that the resistors are all attached to the breadboard's positive power bus on one side and the positive terminals of the LED bar graph on the other (_Label Side_). This means the LED bar graph is always getting power. In order for power to flow through a component one of the component's terminals must be receiving power and the other must either not be receiving power or be connected to a negative or ground terminal. As noted, the resistors are directly attached to the power bus. In this diagram the GPIO pins are connected to the bar graph's negative terminals. GPIO pins are set to either __HIGH__ or __LOW__ when in output mode _(i.e., the pins are written to)_. __HIGH__ means voltage is flowing to the pin, __LOW__ means no voltage is flowing to the pin. Since completing a circuit requires one side of the circuit to have zero volts, and the bar graph is always receiving power via the power bus, the GPIO pins must be set to __LOW__ voltage for current to flow and the LEDs to illuminate. In the code sections below, the LEDs are set to __LOW__ when the LEDs should be illuminated.

### LED Bar Graph in C

The code for this program can be found in the [ledbargraph.c](https://github.com/youngkin/gpio/blob/main/ledbargraph/ledbargraph.c) file in the [github respository](https://github.com/youngkin/gpio) accompanying this series. It substantially similar to the [Sunfounder program](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html), but there are some important differences:

* This program includes an initialization function that turns all the LEDs on then off prior to continuing with the remainder of the program. This can be useful to verify that the LED bar graph is wired up correctly.
* This program includes an interrupt handler that cleans up when the program is interrupted via entering `ctl-C` at the terminal. The Sunfounder code does not. This cleanup includes resetting the bar graph to its state prior to the program starting, e.g., turning all the LEDs off.
* Finally, this program also includes a function that randomly lights individual LEDs.

The major parts of the program are described in more detail below. The code is pretty well described in the [Sunfounder article](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) so I won't repeat what's already covered there.

{{< gist youngkin d3870141ea2a8de68cad62135fa488ed >}}

Lines 9 & 10 show how to build and run the program. Line 19 declares the interrupt handler that's used to intercept `ctl-C` input from the terminal. The rest of this section shows the `init()` function I added. It iterates through the `pins` array and first sets all the pins to `LOW` which has the effect of lighting them up. They're set to `LOW` because the positive terminals of the LED bar graph are connected to the positive power bus. The pins are connected to the negative terminals of the bar graph. To get current to flow the pin states must be set to 0 volts, or `LOW`. The bar graph stays lit for 500 microseconds, then the LEDs are turned off, then there's another delay of 500 microseconds before the program continues. The delays provide enough time to see the entire bar graph light up then turn off before the program continues. This provides a visual indication that the bar graph is connected properly.

{{< gist youngkin 7c1b89402b9e73cbf4d1e86995472a7b >}}

This next section contains the functions that control the bar graph LEDs. The `oddLedBarGraph()`, `evenLedBarGraph()`, and `allLedBarGraph()` functions are taken directly from the Sunfounder code. I added the `randomBarGraph()` function on lines 1 - 12 just to make things more interesting. It uses the C standard library `rand()` function, line 5, to generate a random number that is adjusted on line 6 to be within the range of the `pins` array, offsets 0 through 9. That resulting number is used as the offset in the array to find the pin to be toggled (lines 7 and 9). There is a very brief delay between lighting the individual LEDs.

{{< gist youngkin 95819fcaa718f6fcc5f55821e63431b2 >}}

This last section of code shows the `main()` function and the interrupt handler, `interruptHandler()`. With the exception of the addition of lines 3, 18, and 19, `main()` is identical to the Sunfounder code. Line 3 registers the `interruptHandler()` function to receive the `SIGINT` signal associated with entering `ctl-C` at the terminal.

Lines 26 through 34 define the `interruptHandler()` function. It's parameter, sig, contains the integer value of the signal being passed to the function. In our case it doesn't matter what signal is passed, it'll exit the program in any case. 

Lines 27 thorugh 30 iterate through the `pins` array making sure each pin is set to `OUTPUT` mode, i.e., it can be written to, and setting it's value to `HIGH`. Recall from above that setting the pin to `LOW` will light the pin, setting it to `HIGH` will turn it off.

Finally, the program exits on line 33 with a "success", 0, exit code.

### LED Bar Graph in Go

The code for this program can be found in the [ledbargraph.go](https://github.com/youngkin/gpio/blob/main/ledbargraph/ledbargraph.go) file in the [github respository](https://github.com/youngkin/gpio) accompanying this series. This program in this section is very similar to its C counterpart above. There are some minor differences however. It doesn't include functions to light the even and odd numbered LEDs. The major parts of the program are described in more detail below.

{{< gist youngkin 83522e2fcba8b95bb29635743635ddb1 >}}

Line 6 provides the command to run the program. Lines 34 and 35, set up the slices that will contain the pin numbers and the associated go-rpio `rpio.Pin` instances. Line 33 shows the comparable WiringPi pin numbers as a cross-reference. 

The [Sunfounder GPIO Extension Board diagram](https://docs.sunfounder.com/projects/raphael-kit/en/latest/gpio_extension_board.html) doesn't show the GPIO pins associated with SDA1, SCL1, and SPICE0 pins. The [Raspberry Pi Pinout diagram](https://pinout.xyz/) does. Using this reference we can see that SDA1 maps to GPIO pin 2, SCL1 maps to GPIO pin 3, and SPICE0 maps to GPIO pin 8.

{{< gist youngkin c3d7ff9785e4f201937f52e806c4e9ac >}}

This next part of the program shows the `init` function. Lines 4 - 9 create `rpio.Pin` instances, define the pins as `rpio.OUTPUT` pins, and append them to the `gpins` slice defined in the first part of the program. Lines 10 - 13 set the pins to `LOW` state, lighting them up. Recall the discussion about the C program above that describes why the pins are set to `LOW`. Line 14 briefly pauses the program so the effect of lighting them up can be seen. Finally lines 15 - 17 turn the pins off by setting them to `HIGH`.

{{< gist youngkin c56a1e306dc1cbf2eb1469d2f25715ed >}}

This part of the program contains the `randBarGraph()` and `ledAll()` functions. Line 3 seeds the Go random number generator which is used in line 9 to generate the pin number to be lit. Lines 5 - 15 and lines 22 - 30 contain Go `select` blocks which are used to make sure the 2 functions will exit if a `ctl-C` signal is received from the terminal. Specifically, line 6 and 23 receive the stop command via the `stop` channel and lines 7 & 24 exit the functions. The rest of the code is self-explanatory, the LEDs are turned on and off.

{{< gist youngkin 78167aa8d18574e24f2a36af749f1d2b >}}

This final part of the program contains the `main()` and the `signalHandler()` functions. Lines 11 & 15 create unbuffered Go channels. The `stop` channel is used to signal other parts of the program that the program is exiting. This is needed because the `signalHandler()` is started in a separate goroutine on line 21. It must be able to stop the main goroutine before exiting the program. The `stop` channel is defined as taking an `interface{}` type. This means that the channel can contain any type. For this channel the type of the data sent isn't important.

The `sigs` channel, defined on line 15, is used by the Go runtime to notify the `signalHandler()` that an interrupt signal has been received. `signalHandler()` is registered with the Go runtime on line 20. Line 21 starts the goroutine that will run `signalHandler()`.

The rest of `main()` simply initializes the pins and runs the controlling functions.

LInes 28 - 42 contain the definition of `signalHandler()`.

## Summary

This brief article provided a quick overview of the LED bar graph component and its application in the real world. It also included some C code that wasn't included in the original Sunfounder program. This additional code is used to show how to turn on the different LEDs in random fashion. Finally an implementation of this same C program was provided in Go. This was meant to demonstrate the usage of how use the [go-rpio](https://github.com/stianeikeland/go-rpio) library implements functionality similar to [WiringPi](https://github.com/WiringPi/WiringPi).

Hopefully you've learned enough from this article to use LED bar graphs in your own applications.

## References

* [Sunfounder LED Bar Graph](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.3_led_bar_graph_c.html) project
* [Sunfounder GPIO Extension Board diagram](https://docs.sunfounder.com/projects/raphael-kit/en/latest/gpio_extension_board.html)
* [Raspberry Pi Pinout diagram](https://pinout.xyz/)
* [WiringPi](https://github.com/WiringPi/WiringPi) library for C
* [WiringPi website](http://wiringpi.com/)
* [go-rpio](https://github.com/stianeikeland/go-rpio) library for Go
* Other articles in my [Raspberry Pi GPIO series](https://youngkin.github.io/categories/gpio/)
* [The gpio repository](https://github.com/youngkin/gpio) containing the code for this article
* [How to setup a new Raspberry Pi from scratch](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up)
* [How to use a breadboard](https://www.sciencebuddies.org/science-fair-projects/references/how-to-use-a-breadboard)