---
title: "Raspberry Pi GPIO in Go and C - Using a Shift Register & 7 Segment Display"
description: "Why use a shift register and how to use it with a 7 segment display"
date: 2021-11-14T13:13:42-06:00
draft: false
image: "images/sevensegdisplay/header2.png"
tags: ["raspberry-pi", "Go", "C", "GPIO"]
categories: ["raspberry-pi", "Go", "GPIO"]
GHissueID: 1
toc: true
---

<!--more-->

## Overview

This is the fourth article in a series that explores GPIO programming on a Raspberry Pi 3B+. It is a supplement to the [Sunfounder 7-Segment Display](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.4_7-segment_display_c.html) project. You can find the full series [here](https://youngkin.github.io/categories/gpio/). The code for the series can be found in my [gpio repository](https://github.com/youngkin/gpio).

Like the [Sunfounder RGB LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) project, there are some complicated aspects to this project that aren't well covered in the Sundfounder project documentation. The purpose of this article is to fill those gaps, specifically:

1. What is the goal of the project?
2. What is a "shift register" (the 74HC595 chip)?
3. What are the uses of a shift register?
4. What is a 7-segment display?
5. How is a shift register used on conjuction with a 7-segment display?

___WHAT IS THE GOAL OF THIS PROJECT?!??!?!___

<img style="border:1px solid white; padding: 15px;" src="/images/sevensegdisplay/flatbreadboard2.png" align="center" width="1000"/>
<figcaption align="left"><center><i style="color:black;">LED Bar Graph</i></center></figcaption>

___AND HERE TOO !!!___

## Prerequisites

This section is repeated in all articles in my [Raspberry Pi GPIO series](https://youngkin.github.io/categories/gpio/). If you've already completed a project from one of these articles you can skim this section for required items not included in other projects (e.g., an LED Bar Graph).

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'Stretch' version of the Raspbian OS. The Raspberry Pi website has instructions on how to [setup a new Raspberry Pi from scratch](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up) if you decide to go that way vs. buying a complete kit.

Other items you'll need include:

* a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) _(You may find [this tutorial on breadboards](http://wiki.sunfounder.cc/index.php?title=Breadboard_Basics_%E2%80%93_Types) helpful)_,
* some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==),
* a [220 Ohm resistor](https://www.amazon.com/Resistor-Tolerance-Resistors-Assortment-Certificated/dp/B08QR72BFW/ref=sr_1_8?crid=1E3LWKK431TDV&keywords=resistor+assortment&qid=1636936543&qsid=143-6049131-7886617&s=industrial&sprefix=resistor%2Cindustrial%2C203&sr=1-8&sres=B085NQZNTY%2CB072BL2VX1%2CB07N1ZK8CC%2CB098BKR447%2CB08QR72BFW%2CB07L851T3V%2CB09CZ18Z31%2CB08PF3HNMD%2CB07D54XMFK%2CB088WQMNYK%2CB08NY3XR96%2CB06WRQS97C%2CB07PXYVP3J%2CB07D2Z45CG%2CB089Q88QPN%2CB08ZRYH9VC%2CB08FD1XVL6%2CB08QRH6HFT%2CB07PTNN78Z%2CB07P3MFG5D) - this set actually has an assortment of resistors including 220 Ohm resistors.
* a [74HC595 shift register](https://www.amazon.com/Texas-Instruments-SN74HC595N-Registers-3-State/dp/B01J6WI7RA/ref=sr_1_5?keywords=shift+register+74hc595&qid=1636936939&qsid=143-6049131-7886617&s=industrial&sr=1-5&sres=B07HFWB9L9%2CB01J6WI7RA%2CB0993RQGQY%2CB09CTHBH9P%2CB07B9DCR17%2CB08Z8B9QXY%2CB06WD3W8Q3%2CB07ZHGL8LN%2CB0842PRWJG%2CB07DR7PYYT%2CB01HEPJOV4%2CB07WNHBP86%2CB07RL1398S%2CB07B9D7SPC%2CB08Z3NH5BK%2CB07MQ5X9Q3%2CB07DL13RZH%2CB08JTXNP9Q%2CB08BR1PPQM%2CB01D8KOZF4)
* a [common cathode 7-segment display](https://www.amazon.com/microtivity-7-segment-Display-Common-Cathode/dp/B004S95VJE/ref=sr_1_11?crid=AOG8YK9L0NA1&keywords=7+segment+display&qid=1636937044&qsid=143-6049131-7886617&s=industrial&sprefix=7+segment%2Cindustrial%2C211&sr=1-11&sres=B07GTQZ286%2CB00XW2L6SS%2CB07MCGDST2%2CB00EZBGUMC%2CB08THC6NGS%2CB07GTQ8NDC%2CB07GTPYXNF%2CB004S95VJE%2CB00XW2NSU2%2CB0060FGD3M%2CB087B8WTRZ%2CB01D0WSCJA%2CB0060FGCW4%2CB08N15PLSQ%2CB081VDVVSS%2CB07CLCC82N%2CB07GTQFL6J%2CB07GTQ42VR%2CB085WMBYH4%2CB08XPSCG4K)
* You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the T-Type adapter will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however.

[Here's a simple kit](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) that has all (most?) of the above]. The photo of the kit appears to show a shift register, but I can't be sure. I'm finding the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1) especially useful. NOTE: The Ultimate Starter Kit and the Raphael Kit are the same product.

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

## Information that would have been helpful

### What is a shift register and what is it good for?

[Provide a reference for this](https://www.arduino.cc/en/Tutorial/Foundations/ShiftOut) - _Not covered, daisy chaining shift registers_

### What is a 7-segment display?

### How will this project use the shift register and 7-segment display?

## Setup and Code

<img style="border:1px solid black" src="/images/ledbargraph/breadboard.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">FIX THIS!!!</i></center></figcaption>

<img style="border:1px solid black" src="/images/ledbargraph/ledbargraph.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">FIX THIS!!!</i></center></figcaption>

The breadboard should be wired as illustrated in the above diagram. If you're unfamiliar  with breadboards and breadboard diagrams this [breadboard tutorial ](https://www.sciencebuddies.org/science-fair-projects/references/how-to-use-a-breadboard) should be helpful.

___MORE DESCRIPTION HERE!!!___

### Shift Register/7-Segment Display in C

TODO

### Shift Register/7-Segment Display in Go

TODO

## Summary

TODO

## References

* The [Sunfounder 7-Segment Python](https://docs.sunfounder.com/projects/electronic-kit/en/latest/lesson_19_7-segment.html) project contains some additional information not available in the C version of the project.
* [Raspberry Pi GPIO Pinout diagram](https://pinout.xyz/)
* [Shift Registers: Serial-in, Parallel-out (SIPO) Conversion](https://www.allaboutcircuits.com/textbook/digital/chpt-12/serial-in-parallel-out-shift-register/) is a good resource for getting a detailed introduction to shift registers in general, and the 74HC595 shift register in particular.
* [Texas Instruments CD74HC595 Datasheet](https://www.ti.com/lit/ds/symlink/cd74hc595.pdf?ts=1636840974607&ref_url=https%253A%252F%252Fwww.google.com%252F) contains some interesting information including a timing diagram.