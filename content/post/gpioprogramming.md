---
title: "Raspberry Pi GPIO - Introduction To Programming Broadcom BCM2835 ARM Peripherals"
description: "Learn how to directly program the BCM2835 using registers to perform tasks such as writing to a GPIO pin or using advanced capabilites such as SPI."
date: 2022-03-14T13:13:42-06:00
draft: false
image: "/images/bcm2835programming/Raspberry-Pi-3B-top_1500_dark.jpg"
tags: ["raspberry-pi", "C", "GPIO"]
categories: ["raspberry-pi", "C", "GPIO"]
GHissueID: 1
toc: true
---

<!--more-->

## Overview

This is the fifth article in a series that explores [GPIO programming on a Raspberry Pi 3B+](https://youngkin.github.io/categories/gpio/). It is an introduction to controlling GPIO peripheral devices by directly interacting with the appropriate registers on the BCM2835 board. It is an introduction in that it only covers one of the capabilities, using the basic GPIO capability as described in [Section 6 of the BCM2835 datasheet](). This excludes BCM2835 support for protocols such as SPI, Pulse Width Modulation, and IC2 which overlay specific protocol capability by utilizing the underlying pins mapped to a given protocol (e.g., GPIO pins 8-11 are associated with the SPI protocol and have behaviors that can be enabled that are specific to that protocol). That said, using the basic pin level functionality covers the same concepts needed to utilize advanced protocol capabilites such as SPI. This article, coupled with the [BCM2835 datasheet](), should provide enough background to drive devices that are controlled via those capabilities. Of course there are multiple libraries such as the C [BCM2835]() and [WiringPi] libraries, the Python [pgpio]() and [RPi.GPIO] libraries, and the Go [rpio]() library that make this much easier. And while these libraries are usually more appropriate for most projects it can be helpful to understand how the BCM2835 board works, and how those libraries interact with the board. This background is also helpful if you find yourself reading library code and/or would like to contribute to these projects, or even write your own library.

This article assumes you already have some familiarity with the BCM2835 and GPIO programming on the Raspberry Pi. If you don't you may want to consider trying the projects covered in some of [my other GPIO articles](https://youngkin.github.io/categories/gpio/). The simplist article in the series is [Raspberry Pi GPIO in Go and C - Blinking LED](https://youngkin.github.io/post/sunfoundergpionotesled/). It demonstrates the same capability, blinking an LED, but it demonstrates the usage of the [WiringPi]() and the Go [rpio]() libraries. It is a good introduction to using GPIO on the Raspberry Pi, but it only covers basic GPIO concepts and how to wire a breadboard to a Raspberry Pi's GPIO outputs.

The following topics will be covered:

1. **Prerequisites** - describes the hardware and libraries you'll need for this article.
2. **An introduction to the BCM2835 GPIO Peripherals board** - provides an overview the board's architecture and capabililites.
3. **An introduction to the [BCM2835 C library]()** - provides a brief overview of the BCM2835 C library. This library is the source of much is what is covered in this article.
4. **Using the BCM2835 board to control an LED via the GPIO pins** - provides the details, including code, for controlling an LED.
5. **Summary** - summarizes the important concepts covered in this article.
6. **References** - provides a list of references I found helpful and some that were used in the creation of this article.

## Prerequisites

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'stretch' version of the Raspbian OS. Given that the Sunfounder Ultimate Starter Kit is advertised to work with a Raspberry Pi 4, I would expect the 4 series to work as well. I'm less sure about other Raspberry Pi versions, especially versions with 26 vs. 40 GPIO pins.

Next you'll need is a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), [a 220 Ohm resistor, and an LED](https://www.amazon.com/ELEGOO-Electronics-Component-resistors-Potentiometer/dp/B01ERPXFZK/ref=sr_1_7_sspa?crid=3EJQNCOWP00IF&dchild=1&keywords=resistors&qid=1631478270&s=industrial&sprefix=resis%2Cindustrial%2C219&sr=1-7-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEzTktVSzNYMkxMSDlKJmVuY3J5cHRlZElkPUEwMjAzMDY0NVRERkFLVjVRTUFWJmVuY3J5cHRlZEFkSWQ9QTA5MjM2NjUxUFZYQUlETVAzRDA3JndpZGdldE5hbWU9c3BfbXRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the adapter will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however. [Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). If you expect to follow this series I recommend buying the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).

<img style="border:1px solid black" src="/images/pwmfordummies/RaphaelKit.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Ultimate Starter/Raphael kit</i></center></figcaption>

You will need some basic C programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like Vi or nano. Finally, you'll need basic familiarity with the Linux command line.

## An introduction to the BCM2835 ARM Peripherals

This section provides an overview of the BCM 2835 peripherals. It starts with an overview overview of the capabilites of the various peripherals. It then moves to an overview of BCM2835 addressing. Understanding addressing is fundamental to understanding the rest of the article. It concludes with a more detailed discussion about how registers are used to access the peripherals and their associated I/O functions.

This is not an exhaustive description of the BCM2835 board. The only capability that will be covered in any detail, and is the focus of this article, is interacting with the GPIO I/O functions, namely defining the function of a pin and setting and clearing the value of the pin (e.g., setting it to HIGH and LOW), specifically programming the the BCM2835 GPIO to blink an LED.

There are multiple pin numbering schemes. The Raspberry Pi has physical pin numbers, i.e., the numbers on the Pi board. BCM2835 has a numbering scheme called GPIO numbering. It is different than, and incompatible with, the Pi's physical numbering scheme. Finally some libraries such as the C WiringPi library have their own numbering schemes which are incompatible with the Pi and BCM2835 schemes. This article uses the BCM2835 numbering scheme exclusively. There is a good [pin out diagram](https://pinout.xyz) available that maps between these 3 numbering schemes. Where a pin supports a particular I/O function, as described below, that is also called out on this pin out diagram.

A quick note on terminology... The term "peripherals" is used in the title and in some parts of this article. To me the term peripherals is something of a misnomer. At its most basic the GPIO capability of the BCM2835 is made available through a set of physical pins on the device. Some of these pins provide power and ground needed to drive external devices like LEDs, sensors, and motors. Other pins can be controlled programatically to take input from, or output to, these external devices. At a higher level, subsets of the GPIO pins provide support for complete protocols that implement more sophisticated functions like controlling the speed and direction of a motor and controlling LED displays that display text like those in highway signs. These basic and more sophisticated capabilities are controlled by setting the function of a pin. Input and output are 2 functions that can be set. There are a variety of others that will be covered in the __I/O Functions__ section below. I titled that section __I/O Functions__ because I think that more accurately describes the concept than the term "peripherals" does. To me "peripherals" are the external devices. Setting the function of a pin describes how the BCM2835 interacts with a device or peripheral.

### I/O Functions

The BCM2835 supports a variety of I/O functions. These are the subject of the BCM2835 datasheet. This section will only describe a few of these functions in detail. There are other sources like Wikipedia that can provide information about the others.

The main I/O functions supported by the BCM2835 are:

  * GPIO
  * SPI
  * PWM
  * AUX
  * BSC
  * DMA
  * External Mass Media Controller
  * Interrupts
  * Audio (PCM/I2s Audio)
  * System Timer
  * UART

As I have direct experience with the first 3 I/O functions, GPIO, SPI, and PWM, this section will focus on these. This will provide enough context in order to get a basic understanding of the kinds of things the BCM2835 is capable of as well as leveraging the information in the Addressing and Registers sections below in order to learn about and use the remaining I/O functions.

#### GPIO

GPIO stands for General Purpose Input Output. In the BCM2835 datasheet the term GPIO refers to the most general, or basic, level of control of a physical pin. This is what's used to control the function of a pin, such as setting it as an input or output pin. It's also used to set or get the value of a pin. Values are conceptually referred to as HIGH and LOW, but they are represented by voltage, or lack thereof, passing across the pin. There are other types of settings for pins. One of these settings is used to control whether to detect a state change on the rising or falling edge of a pin's voltage change. Another setting controls what are called pull-up/pull-down resistors attached to each pin. These resistors are used to explicitly control the value of a pin, 1 or 0, when a pin's voltage is in an indeterminate state. [This tutorial](https://www.electronics-tutorials.ws/logic/pull-up-resistor.html) has a reasonably good explanation for why pull-up and pull-down resistors are needed.

The code associated with this article causes and LED to blink. Accomplishing this involves the following steps:

1. Set the function of the pin to output, i.e., it's going to be written to in order to send a signal to the LED.
2. Write LOW to the pin to cause the LED to turn on.
3. Pause
4. Write HIGH to the pin to cause the LED to turn off.
5. Repeat steps 2 through 4.

Accomplishing this uses the GPIO function. More accurately it uses the GPIO "function" to set the pin's function (e.g., output).

#### SPI (Serial Peripheral Interface)

SPI is used to send data serially to a peripheral that can accept or requires data in parallel. This is helpful because a relatively large set of parallel inputs can be written to using just 3 GPIO pins. If SPI wasn't used, one GPIO pin would be required for each parallel input. This could easily be prohibitive since pins are a limited resource.

<img style="border:1px solid black" src="/images/sevensegdisplay/7segmentdisplaypinout.png" align="center" width="200" height="100"/>
<figcaption align="left"><center><i style="color:black;">Image credit: Circuit Basics</i></center></figcaption>

The diagram above shows a 7 segment display that is commonly used to display display numeric characters in peripherals like calculators and clocks. It has 8 input lines and 2 ground lines. Controlling this directly using GPIO would use 8 of the 40 GPIO pins. Controlling 6 of these displays would take all 48 GPIO pins, but there are only 40. Clearly the approach to using GPIO to control this peripheral isn't practical. Enter a device called a shift register. Shift registers can control 8 input lines of a connected peripheral. It uses 5 GPIO pins, 3 for input and 2 for control. To control multiple peripherals it's possible to daisy-chain shift registers without requiring additional GPIO pins. Taking our previous example of 6 7 segment displays, using a shift register requires 5 GPIO pins vs. the 48 that would be required by directly using GPIO pins. If you're interested in learning more about shift registers see my article [Raspberry Pi GPIO in Go and C - Using a Shift Register & 7 Segment Display](http://youngkin.github.io/post/shiftregistersevensegdisplay/).

The SPI interface requires at least 4 GPIO pins. One of the pins, MOSI, sends data to a peripheral. MOSI stands for Master Out Slave In. The second pin, MISO (Master In Slave Out) accepts input. 

 > I reluctantly use the terms "master" and "slave". However these terms are used in all of the documents I've read on SPI. I'll continue to use them in order to avoid confusion.

 The 3rd pin is a clock which controls the data transmission between the master and slave. The 4th pin is called the chip select (CS) or chip enable (CE) pin. It is used to prepare the slave device to interface with the BCM2835. An additional slave device can be accessed by using a 5th pin, the second CE/CS pin. The 2 CE/CS pins are used to enable each of the slave devices, but only one device can be enabled at a time. 

The primary SPI interface, SPIO, is implemented on GPIO pins 7-11. Pins 7 & 8 are the 2 CE/CS pins available on the BCM2835. Pin 9 is MISO, 10 is MOSI, and 11 is the clock (SCLK). The BCM2835 has 2 auxilary SPI interfaces, SPI1 and SPI2. From the BCM2835 datasheet it looks like SPI1 is available on pins 16-21 and SPI0 on pins 35-39. These auxilary interfaces are available via the AUX I/O function.

SPI can be used to control a variety of peripherals including matrixed LED or LCD displays to display characters or images, take input from touchscreens, and interact with various sensors. [Wikipedia has a good article](https://en.wikipedia.org/wiki/Serial_Peripheral_Interface) describing SPI in more detail.

I have an upcoming article devoted to SPI. I'll update this document adding a reference when it becomes available.

#### PWM

PWM stands for Pulse Width Modulation. It is used to convert a digital signal, like that produced by the BCM2835, into a simulated analog signal. It is simulated in the sense that it's not a true analog signal. It's still a digital signal but it cycles so fast it appears to be an analog signal. Many peripherals like electric motors, dimmable LEDs, and color LEDs require an analog signal. As with SPI there are dedicated pins that can be enabled to utilize the PWM I/O function. These are GPIO pins 12, 13, 18, and 19. With these pins the BCM2835 supports something called hardware PWM. The PWM functionality is embedded into the BCM2835. It implements a hardware based clock loop that controls the frequency of the simulated analog signal. It is possible to use other GPIO pins for PWM, this is called software PWM. However, in software PWM the clock loop is written in the code that runs on the Raspberry Pi CPU. Since that CPU is doing many other things it isn't possible to create a reliable clock signal. This can result in some undesirable side effects such as a flickering LED in place of a dim LED. See my article [Raspberry Pi GPIO in Go and C - RGB LED](https://youngkin.github.io/post/sunfoundergpionotesrgbled/) for more information about PWM as well as examples of PWM in use.

### Addressing

All GPIO I/O functions are accessed via registers. These registers are located at various offsets in the on-board memory. Accessing the registers requires a knowledge of how addressing works on the BCM2835 board was well as on the Raspberry Pi. Here's a diagram from the [BCM2835 datasheet](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf), section 1.2.1, showing how memory is mapped from the Pi's physical address to the BCM2835 addresses.

<img style="border:1px solid black" src="/images/bcm2835programming/BCM2835Addressing.jpg" align="center" width="1000" height="500"/>
<figcaption align="left"><center><i style="color:black;">BCM2835 Addressing</i></center></figcaption>

The Raspberry Pi maps the BCM2835 memory to the Pi's physical address of `0x2000 0000`. This is shown in the middle column labeled "ARM Physical Addresses". The gray shaded area labeled I/0 peripherals is the Pi's physical memory location of the BCM2835's peripherals (e.g., GPIO pins). Moving towards the left you'll see a gold box labeled "VC/ARM MMU". This is the [Memory Management Unit](https://en.wikipedia.org/wiki/Memory_management_unit) that is responsible for mapping the Pi's physical memory to the BCM2835 memory. Following the line from the middle column's "I/O Peripherals" partition, through the VC/ARM MMU, to the far left column titled "VC CPU Bus Addresses" leads to the "I/O Peripherals partition on the BCM2835's CPU address bus. It's address is `0x7E00 0000`.

The I/O peripheral addresses can be made available to the application through one of 2 ways. The traditional way is via the `/dev/mem` device mounted on the file system. Accessing this device requires root privileges. A newer method became available starting with the Raspberry Pi 2. This method accesses the I/O peripheral addresses via `/dev/gpio`. An advantage of using this method is that it doesn't require root access. A disadvantage to using this method is that only the GPIO functionality of the BCM2835 can be accessed. This means advanced capabilities like PWM and SPI are not available. Accessing the I/O peripherals using `/dev/mem` also requires a little more work, namely mapping the physical memory from `/dev/mem` to the application's virtual memory. This will be covered in more detail later.

### Registers
  
As mentioned above, specifying the I/O function of a pin or set of pins is accomplished through registers on the BCM2835. Each I/O function has an associated register set located at an offset within the BCM I/O peripheral address space. For example, the GPIO register set starts at the CPU bus address `0x7E20 0000`. The SPI register set starts at CPU bus address offset `0x7E20 4000`. The PWM register set starts at offset `0x7E00 C000`. And so on, each I/O function has it's own set of registers to manage its behavior. That said, they all work on the same principles. Namely, find the offset of the beginning of each I/O function's register set and then set (or get) the appropriate bits at the appropriate offsets to control the behavior. To make this example more explicit, and to highlight additional details, let's do a high level walkthrough of how to control a GPIO pin to blink an LED.

As mentioned previously the GPIO register starts at bus address `0x7e2000 0000`. Section 6.1 of the [BCM2835 datasheet](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf) starting on page 90 provides the details about the GPIO registers. To blink an LED we'll first need to specify the pin's function, output in this case, and then toggle the value of the LED between HIGH and LOW.

#### Specify a pin's function

First the I/O function of the pin needs to be set. A pin's functionality is defined via one of 8 alternative functions. Encoding each function requires 3 bits, 000 - 111. These bits are contained within what is called a GPIO Function Select Register. Each register is 32 bits long. Therefore each 32 bit register can control 10 pins with 2 bits left over. In order to control all of the GPIO pins there are 6 function select registers (the BCM2835 has the capability to control 53 pins via function select registers). The first function select register is located at `0x7E20 0000`, zero-offset from the start of the GPIO register set. The 6th function select register starts at offset `0x7E20 0014`. The 8 alternative functions are:

* Input - represented by bit pattern `000` 
* Output  - represented by bit pattern `001`
* and Alternate functions 0 through 5 represented by bit patterns `010` through `111` respectively.
  
In this example let's use pin 17 as an output pin. Inspecting the BCM2835 datasheet we see that pin 17 is in function select register 1, AKA GPFSEL1. GPFSEL1 is at offset `0x7E20 0004`. Pin 17's function select offset within the register is at bits 21-23. Recall that the output bit pattern is `001` so we'll need to put that bit pattern in bits 21-23. In the section on coding below we'll see how this is actually accomplished.

#### Set the pin value

This article assumes that pin 17 is attached to the ground terminal of the LED (the cathode). This means that we'll have to set the pin to LOW in order for current to flow from the power input to the anode through to the LED ground on pin 17 and light the LED. Setting the pin to HIGH will cause the LED to turn off. Switching between LOW and HIGH will cause the LED to blink, which is what we're looking for.

GPIO pin values are controlled via the GPIO Pin Output Set register. There are 2 GPIO Pin Output Set registers. Pin 17 is in the first register, GPIO Pin Output Set 0, AKA GPSET0,, at GPIO register offset `0x7E20 002C`. Pin 17 is at bit offset 17 within GPSET0. Setting bit 17 to 1 sets the register to HIGH. GPIO Pin Output Set registers can only set a pin to HIGH, setting a pin to 0 has no effect. But recall we need to set the pin to LOW. Since setting a pin in GPSET0 won't result in the pin being set to LOW how can we accomplish this? It turns out there is another set of GPIO registers called GPIO Pin Output Clear registers. These are used to set a pin's value to LOW. Pin 17 is in GPIO Pin Output Clear register 0, AKA GPCLR0, starting at offset `0x7E20 0028`. It's position within the register is also at bit 17. Setting bit 17 to 1 will set the pin to LOW. So we'll need to left shift a `1` into bit 17 in order to turn on the LED. We'll use GPSET0 to turn the LED off. As with specifying a pin's function, how this is accomplished will be covered later in this article. By using the GPCLR0 and GPSET0 registers in turn we can cause the LED to blink.

#### Other GPIO settings

The GPIO register set can be used to control many other functions of a GPIO pin in addition to selecting the function and setting/clearing a pin's value. It can also be used to read the value of a pin. As discussed in the I/O functions section above other things that can be managed include specifying how state changes are to be detected (rising/falling edge detection) and how to set the voltage of a pin when it's in an indeterminate state (pull-up/down resistors). Discussing these other registers is an advanced topic and is beyond the scope of this article.

## Setup and Code

<img style="border:1px solid black" src="/images/pwmfordummies/blinkingLED.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Image credit: Sunfounder</i></center></figcaption>

The breadboard should be wired as illustrated in the above diagram. One very important thing to note, something that I spent way too much time debugging, is that the resistor connecting the ground pin on the 7-segment display must be connected to the ground or negative bus on the breadboard. In all my prior projects I've connected the resistor to the positive breadboard bus. I initially missed this detail and the 7-segment display didn't display anything. One other thing I got wrong on the initial wiring is that I had the 74HC595 output register pins connected incorrectly to the 7-segment display. This is easy to do. I debugged this by noting which LED segments lit up for which expected number. After going through several numbers it became apparent that I had the 'G' and 'E' pins on the 7-segment display reversed.

If you're unfamiliar  with breadboards and breadboard diagrams this [breadboard tutorial ](https://www.sciencebuddies.org/science-fair-projects/references/how-to-use-a-breadboard) should be helpful.

### Controlling an LED

{{< gist youngkin 84d8e9f350d19d184255adb3fb7ad93c >}}

## Summary

___Refer to SPI article for more details about SPI? Also, provide an advanced section in the SPI article covering working directly with the board vs. via a library?___

Comments and questions about this article are welcome.

## References

* [The Sunfounder Blinking LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html) is the source of the breadboard diagram. It was also the inspiration for this series on Raspberry Pi GPIO programming. See my [Blinking LED project](http://10.0.0.223:1313/post/sunfoundergpionotesled/) for more details.
* [Raspberry Pi GPIO Pinout diagram](https://pinout.xyz/) including the physical board pin numbers, the BCM/GPIO pin numbers, and the WiringPi pin numbers.
* Other articles in my [Raspberry Pi GPIO series](https://youngkin.github.io/categories/gpio/)
* [The gpio repository](https://github.com/youngkin/gpio) containing the code for this article
* [How to setup a new Raspberry Pi from scratch](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up)
* [How to use a breadboard](https://www.sciencebuddies.org/science-fair-projects/references/how-to-use-a-breadboard)