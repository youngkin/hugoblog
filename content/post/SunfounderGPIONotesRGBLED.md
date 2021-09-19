---
title: "Raspberry Pi GPIO in Go and C - RGB LED"
description: "Use Pulse Width Modulation (PWM) on a Raspberry Pi to drive an RGB LED in Go and C"
date: 2021-09-14T13:13:42-06:00
draft: false
image: "images/sunfounderLED.jpeg"
tags: ["raspberry-pi", "Go", "C", "GPIO"]
categories: ["raspberry-pi", "Go", "GPIO"]
GHissueID: 1
toc: true
---

This is the second article in a series that explores GPIO programming on a Raspberry Pi 3B+. It is a supplement to the [Sunfounder RGB LED project](../sunfoundergpionotesled/). You can find the full series [here](../../categories/gpio). It explores the use of Pulse Width Modulation (PWM) to drive an RGB LED as well as how to control an individual's LED brightness. The code samples will be in Go and C.

<!--more-->

## Overview

This is the second article in the series of Raspberry Pi GPIO programming. The first is [Sunfounder Raspberry Pi Kit Notes for Go and C - Blinking LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html).

## Prerequisites

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'stretch' version of the Raspbian OS. Given that the Sunfounder Ultimate Starter Kit is advertised to work with a Raspberry Pi 4, I would expect the 4 series to work as well. I'm less sure about other Raspberry Pi versions, especially versions with 26 vs. 40 GPIO pins.

Next you'll need is a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), [a 220 Ohm resistor, and a RGB LED](https://www.amazon.com/EDGELEC-Tri-Color-Multicolor-Diffused-Resistors/dp/B077XGF3YR/ref=asc_df_B077XGF3YR/?tag=hyprod-20&linkCode=df0&hvadid=242051162351&hvpos=&hvnetw=g&hvrand=11064062033670066895&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9028749&hvtargid=pla-430228081645&psc=1). You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the cable will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however. [Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). If you expect to follow this series I recommend buying the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).

<img style="border:1px solid black" src="/images/RaphaelKit.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Ultimate Starter/Raphael kit</i></center></figcaption>

You will also need some basic C  and Go programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like Vi or nano.

To compile and run the C program you'll need the WiringPi libary. It's easy to get:

```
sudo apt-get install wiringpi
```

Then test the installation using:

```
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

## Information that would have been helpful

This project uses the PWM (Pulse Width Modulation) pins on the GPIO to achieve the desired effect, namely demonstrating how to create different colors with a simple RGB LED. Unfortunately the Sunfounder documentation leaves out information about PWM, like what is PWM and how is it implemented on a Raspberry Pi?

### What is PWM?

At it's most basic, PWM is used to simulate an analog signal using a digital source (like a GPIO pin). An LED's brightness can be modified by varying the voltage supplied to the LED. A variety of analog devices, such as a motor's speed, can be controlled in the same way. PWM simulates varying voltages by varying the length of the digital power pulse within a given duration (range). The ratio of the power pulse vs the range is called a duty cycle. For example, a time duration of 10 seconds and a pulse length of 1 second has a duty cycle of 10. This duty cycle of 10 means power is flowing for 10% of the time. A duty cycle of 10 with a 10 volt input would  result in a 1 volt output. This article, [Introduction to PWM: How Pulse Width Modulation works](https://www.kompulsa.com/introduction-pwm-pulse-width-modulation-works/), provides a good overview of PWM.

For the purposes of this article there are 2 ways to generate a PWM signal, software-based and hardware-based. Hardware-based PWM is generated by a dedicated hardware PWM device that can be configured to generate a PWM signal as described above. It produces a very uniform signal with regard to timing. A uniform signal is required, for example, to produce a flicker-free light source such as an LED. Software-based PWM is directly implemented in the executing program using a `while(true)` for loop that never ends which controls the amount of time a pin is allowing current to flow (pulse) vs. the amount of time the pin isn't allowing current to flow. In this case the uniformity of the signal is determined by the accuracy of a language's `sleep()` function and the OS (Linux) scheduler. A less uniform signal, for example, may result in a flickering light source. There is a more complete description of [the difference between soft PWM and PWM](https://raspberrypi.stackexchange.com/questions/100641/whats-the-difference-between-soft-pwm-and-pwm) and associated pros and cons on the Raspberry Pi Stack Exchange site.

The Sunfounder RGB LED project documentation doesn't mention that there are both software and hardware PWM modes. This became apparent when when I wrote a program that used the Go GPIO library which only supports hardware PWM. The project's example C code uses the software PWM functions of the WiringPi library. As will be explained in more detail below, the Go version of RGB LED didn't work as expected while the C version did.

### PWM using Raspberry Pi GPIO

Software PWM can be created on any GPIO pin. For hardware PWM the Raspberry Pi 3B+ has 4 PWM pins, BCM GPIO pins 12,13, 18, and 19 [^1]. The Broadcom BC2835 GPIO board that's on the Raspberry Pi 3B+ has 2 PWM channels[^2]. GPIO12 and GPIO18 share one channel and GPIO13 and GPIO19 share the other channel. This means that a signal that's sent to either pin that share a channel will go to both pins. For example, sending a signal on GPIO12 will also be shared with GPIO18 and vice versa[^2] [^3].

So why is it important to know that there are only 2 PWM channels for 4 PWM pins? This project uses an RGB LED that has 4 pins. One pin is for the ground, the other 3 are used to control the red, green, and blue LEDs. So we need 3 PWM pins, one for each RGB color. See the problem? There are only 2 independent pins available for hardware PWM but we need 3 to control the RGB LED. Let's say that GPIO12 is configured as the color red and GPIO18 is configured as blue. If a signal is sent to GPIO12 which represents the color red, the same signal will be sent to GPIO18, blue. So the resulting color will be purple, not red as expected. 

## RGB LED in C

If you haven't done so already, you should start with the [Introduction](https://docs.sunfounder.com/projects/raphael-kit/en/latest/introduction.html) to the project's in the Ultimate Starter/Raphael kit's documentation and work your way through the following sections up to and including the [Play with C](https://docs.sunfounder.com/projects/raphael-kit/en/latest/play_with_c.html) section, and then to the [RGB LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) project. You should set up the breadboard as described in the project documentation or in the diagram below:

<img style="border:1px solid black" src="/images/RgbLed.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder RGB LED breadboard setup</i></center></figcaption>

The C code described in the Sunfounder [RGB LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) uses software PWM. When run, the colors of the RGB LED change as expected. Varying voltages to the respective RGB pins using software PWM produces light of varying brightness (off to full bright) which will produce a wide range of colors.

Here is a slightly modified version of the Sunfounder C program that the controls an RGB LED:

```c
  1 // Copyright (c) 2021 Richard Youngkin. All rights reserved.
  2 // Use of this source code is governed by a MIT-style
  3 // license that can be found in the LICENSE file.
  4 //
  5 // Build - gcc -o rgbled rgbled.c  -lwiringPi -lpthread
  6 //
  7 #include <wiringPi.h>
  8 #include <softPwm.h>
  9 #include <stdio.h>
 10 #include <signal.h>
 11 #include <stdlib.h>
 12
 13 #define uchar unsigned char
 14 #define LedPinRed    24
 15 #define LedPinGreen  1
 16 #define LedPinBlue   23
 17
 18 static volatile int keepRunning = 1;
 19
 20 void interruptHandler(int);
 21
 22 void ledInit(void){
 23     softPwmCreate(LedPinRed,  0, 0xff);
 24     softPwmCreate(LedPinGreen,0, 0xff);
 25     softPwmCreate(LedPinBlue, 0, 0xff);
 26 }
 27
 28 void ledColorSet(uchar r_val, uchar g_val, uchar b_val) {
 29     softPwmWrite(LedPinRed,   r_val);
 30     softPwmWrite(LedPinGreen, g_val);
 31     softPwmWrite(LedPinBlue,  b_val);
 32 }
 33
 34 int main(void) {
 35     if(wiringPiSetup() == -1){ //when initialize wiring failed, printf messageto screen
 36         printf("setup wiringPi failed !");
 37         return 1;
 38     }
 39
 40     signal(SIGINT, interruptHandler);
 41     printf("Hit ^-c to exit\n");
 42
 43     ledInit();
 44     while(keepRunning) {
 45             printf("Red\n");
 46             ledColorSet(0xff,0,0);
 47             delay(1000);
 48
 49             printf("Green\n");
 50             ledColorSet(0,0x32,0);
 51             delay(1000);
 52
 53             printf("Blue\n");
 54             ledColorSet(0x00,0x00,0xff);
 55             delay(1000);
 56
 57             printf("Yellow\n");
 58             ledColorSet(0xff,0x32,0x00);
 59             delay(1000);
 60
 61             printf("Purple\n");
 62             ledColorSet(0xff,0x00,0xff);
 63             delay(1000);
 64
 65             printf("Cyan\n");
 66             ledColorSet(0xc0,0x32,0xff);
 67             delay(1000);
 68
 69             printf("Off\n");
 70             ledColorSet(0,0,0);
 71             delay(1000);
 72     }
 73
 74     return 0;
 75 }
 76
 77 void interruptHandler(int sig) {
 78     // Turn off LED
 79     // Don't understand why, but ledColorSet(0,0,0); doesn't work
 80     pinMode(LedPinRed, OUTPUT);
 81     digitalWrite(LedPinRed, LOW);
 82     pinMode(LedPinGreen, OUTPUT);
 83     digitalWrite(LedPinGreen, LOW);
 84     pinMode(LedPinBlue, OUTPUT);
 85     digitalWrite(LedPinBlue, LOW);
 86
 87     printf("\nExiting...\n");
 88
 89     exit(0);
 90 }
 ```

 Line 5 provides the command needed to build the program - `gcc -o rgbled rgbled.c  -lwiringPi -lpthread`. The `-l` flags reference the libraries needed to build the program. `-lwiringPi` refers to the WiringPi library. It should be installed in the correct place along with WiringPi and the build command should just work. Libraries are usually located in `/usr/lib`.

 Note the pin numbering used in lines 14-16. I'm using WiringPi pins 24, 1, and 23 for red, green, and blue, which correspond to BCM pins 19, 18, and 13 respectively. The Sunfounder wiring diagram above using BCM pins 17, 18, and 27 for red, green, and blue, which correspond to WiringPi pins 0, 1, and 2 respectively. The Sunfounder C code uses the pins as shown in the above diagram. Make sure your breadboard is wired to BCM pins 19, 18, and 13 for red, green, and blue; wiringPi pins 24, 1, and 23, or change the above code to to use 0, 1, and 2 for `LedPinRed`, `LedPinGreen`, and `LedPinBlue` respectively. This will be good practice for navigating your way around a breadboard. See the Sunfounder page [GPIO Extension Board](https://docs.sunfounder.com/projects/raphael-kit/en/latest/gpio_extension_board.html) for a quick cross reference between BCM, board, and WiringPi pin numbers.

 The `softPwm*` functions called on lines 23-25 and 29-31 indicate that software PWM is being used.

 The calls to `ledColorSet(...)` within the `while(keepRunning)` loop use hex numbers to set the colors. These are used to generate the full range of available colors. I did change the values somewhat as the green LED used apparently has less resistance as it's quite a bit brighter than the red and blue LEDs and therefore throws off the generated colors. Similarly, the blue LED seems to have more resistance as it is quite a bit dimmer than the other 2 LEDs. Changing the values, at least for my specific RGB LED, generated truer colors.

 Line 40 registers an interrupt handler to catch the SIGINT interrupt generated when ^-C is entered at the keyboard. Line 20 declares the interrupt handler function. Lines 77 thru 90 implement the interrupt handler function. Within that function the LED is turned off.

 The program can be run, after compiling, using `./rgbled`.

 ### Hardware PWM in C

 Since the Sunfounder documentation doesn't include a hardware PWM solution in C I decided to create one for myself. Partly to assure myself that the hardware PWM behavior I saw using the Go go-rpio library wasn't limited to that library. 

```c
  1 // Copyright (c) 2021 Richard Youngkin. All rights reserved.
  2 // Use of this source code is governed by a MIT-style
  3 // license that can be found in the LICENSE file.
  4 //
  5 // Build - gcc -o rgbledHardware rgbledHardware.c  -lwiringPi -lpthread
  6 //
  7 #include <wiringPi.h>
  8 #include <softPwm.h>
  9 #include <stdio.h>
 10 #include <signal.h>
 11 #include <stdlib.h>
 12
 13 #define uchar unsigned char
 14 #define LedPinRed    24
 15 #define LedPinGreen  1
 16 #define LedPinBlue   23
 17
 18 static volatile int keepRunning = 1;
 19
 20 void interruptHandler(int);
 21
 22 //
 23 // Hardware PWM pins include WiringPi pins 1, 26, 23, 24 (BCM 18, 12, 13, and 19 respectively).
 24 // For practical purposes only 2 are usable as the other 2 are linked, turn one on they both turn on.
 25 // Pins 24 & 26 (BCM 19 & 12) and Pins 1 & 23 (BCM 13 and 18) are independent. Pins 1 and 26 (BCM 18 & 12)
 26 // are linked. Set one and the other will also be set. Ditto for pins 23 & 24 (BCM 13 & 19). In addition,
 27 // Writing to linked pins and one other results in inconsistent results. The independent pin will always
 28 // be set, but the dependent pins won't always be set correctly.
 29 //
 30 void ledInit(void) {
 31     pinMode(LedPinRed, PWM_OUTPUT);
 32     pinMode(LedPinGreen, PWM_OUTPUT);
 33     pinMode(LedPinBlue, PWM_OUTPUT);
 34
 35     pwmWrite(LedPinRed, 0xff);
 36     pwmWrite(LedPinGreen, 0x32);
 37     pwmWrite(LedPinBlue, 0xff);
 38
 39     delay(1000);
 40     printf("Initialization complete\n");
 41 }
 42
 43 void ledColorSet(uchar r_val, uchar g_val, uchar b_val) {
 44     pwmWrite(LedPinRed, r_val);
 45     pwmWrite(LedPinGreen, g_val);
 46     pwmWrite(LedPinBlue, b_val);
 47 }
 48
 49 int main(void) {
 50     if(wiringPiSetup() == -1){ //when initialize wiring failed, printf messageto screen
 51         printf("setup wiringPi failed !");
 52         return 1;
 53     }
 54
 55     signal(SIGINT, interruptHandler);
 56     printf("Hit ^-c to exit\n");
 57
 58     ledInit();
 59     while(keepRunning) {
 60             printf("Red\n");
 61             ledColorSet(0xff,0x00,0x00);   //red
 62             delay(2000);
 63             printf("Green\n");
 64             ledColorSet(0x00,0x32,0x00);   //green
 65             delay(2000);
 66             printf("Blue\n");
 67             ledColorSet(0x00,0x00,0xff);   //blue
 68             delay(2000);
 69             printf("Yellow\n");
 70             ledColorSet(0xff,0x32,0x00);   //yellow
 71             delay(1000);
 72             printf("Purple\n");
 73             ledColorSet(0xff,0x00,0xff);   //purple
 74             delay(1000);
 75             printf("Cyan\n");
 76             ledColorSet(0xc0,0x32,0xff);   //cyan
 77             delay(1000);
 78     }
 79     return 0;
 80 }
 81
 82 void interruptHandler(int sig) {
 83         // Turn off LED
 84         // Don't understand why, but ledColorSet(0,0,0); doesn't work
 85         pinMode(LedPinRed, OUTPUT);
 86         digitalWrite(LedPinRed, LOW);
 87         pinMode(LedPinGreen, OUTPUT);
 88         digitalWrite(LedPinGreen, LOW);
 89         pinMode(LedPinBlue, OUTPUT);
 90         digitalWrite(LedPinBlue, LOW);
 91
 92         printf("\nExiting...\n");
 93
 94         exit(0);
 95 }
 ```

This version of RGB LED is very similar to the software PWM version above. It also uses the same pins as the above example. The primary difference is that the pin mode is set to `PWM_OUTPUT` vs. the use of `softPwmCreate`. It also uses `pwmWrite` instead of `softPwmWrite`.

The program is run using root privileges with `sudo ./rgbledHardware`. `sudo` is needed because direct hardware access is limited to users with root privileges. `sudo` provides root privileges to commands prefixed with `sudo`.

When running the program you'll notice that the RGB LED doesn't produce the expected colors. Sometimes it might show purple when it should be showing blue. Sometimes it might be turned off completely when it's supposed to be showing a color. This behavior occurs because BCM pin 13 (blue) and BCM pin 19 (red) are on the same hardware channel. As explained above, when a signal is sent to one channel it will propagate to both pins that share that channel. Green, on BCM pin 12, is unaffected. One behavior I don't understand is why sometimes the LED is off when it should be showing a color. This only happens for red (BCM19) and blue (BCM 13) pins. Perhaps the pins that share a channel don't produce a signal at exactly the same time. Imagine a case where the blue pin is set to 0xff and the red pin is set to 0x00. If the red pin's signal comes in slightly after the blue signal it would override the blue pin signal turning the LED off. The takeaway from all this is that for all intents and purposes, there are only 2 hardware PWM pins can be in use at the same time, and they can't be on the same channel.

## RGB LED in Go

This version of RGB LED will work with the same breadboard setup as the C version. Unlike the Sunfounder C version, and the first C version in this article, the Go library only supports hardware PWM.

First things first, we need a Go library to drive the GPIO interface. I'm using [go-rpio](https://github.com/stianeikeland/go-rpio) for several reasons:

1. It seems to be in fairly wide use
2. It seems to be fairly complete
3. It's relatively active
4. It comes with example code and good documentation
5. Its API is similar to WiringPi's

Another option is [periph](https://github.com/periph/host) (code) with [documentation](https://periph.io/). It is more active and the documentation is very good, better than go-rpio. But for the LED examples I was able to find, go-rpio better matched what I was looking for, especially with regard to this project. But this is an excellent alternative to go-rpio and vice-versa.

### Example 1 - RGB LED with hardware PWM

This program can operate in one of 2 ways. First, it can operate as a fully hardware PWM implementation, similar to [Hardware PWM in C](#hardware-pwm-in-c). It can also operate in a mixed-mode, part hardware and part software PWM. See the comments on lines 32-38 and lines 44-52 for details. Switching modes requires commenting/uncommenting code blocks as described in those comments.

```go
  1 // Copyright (c) 2021 Richard Youngkin. All rights reserved.
  2 // Use of this source code is governed by a MIT-style
  3 // license that can be found in the LICENSE file.
  4 //
  5 // Hardware PWM pins include WiringPi pins 1, 26, 23, 24 (BCM 18, 12, 13, and 19 respectively).
  6 // For practical purposes only 2 are usable as the other 2 are linked, turn one on they both turn on.
  7 // Pins 24 & 26 (BCM 19 & 12) and Pins 1 & 23 (BCM 13 and 18) are independent. Pins 1 and 26 (BCM 18 & 12)
  8 // are linked. Set one and the other will also be set. Ditto for pins 23 & 24 (BCM 13 & 19). In addition,
  9 // writing to linked pins leads to inconsistent results. The independent pin will always
 10 // be set, but the dependent pins won't always be set correctly. This could be a result of timing issues
 11 // that affect the ordering of when signals are sent to pins on the same channel.
 12 //
 13
 14 package main
 15
 16 import (
 17     "bufio"
 18     "fmt"
 19     "os"
 20     "strconv"
 21     "strings"
 22
 23     "github.com/stianeikeland/go-rpio/v4"
 24 )
 25
 26 var (
 27     ledPinRed   = rpio.Pin(19)
 28     ledPinGreen = rpio.Pin(18)
 29     ledPinBlue  = rpio.Pin(13)
 30     freq        = 100000
 31     cycle       = 1024
 32 )
 33
 34 func ledColorSet(redVal, greenVal, blueVal uint32) {
 35     // This doesn't work as expected because GPIO pins 19 & 13 are essentially linked. This
 36     // is also true for GPIO pins 18 & 12, but since pin 12 isn't used here pin 18, green,
 37     // works as expected. So, when a blueVal or redVal are provided, the same value
 38     // is propagated to the linked pin. So for example, if redVal is 0 and blueVal
 39     // is 255, both the red and blue leds will light up (purple).
 40     //
 41     // Uncomment these lines if you want to see this behavior.
 42     ledPinRed.Mode(rpio.Pwm)
 43     ledPinGreen.Mode(rpio.Pwm)
 44     ledPinBlue.Mode(rpio.Pwm)
 45
 46     ledPinRed.DutyCycle(redVal, uint32(cycle))
 47     ledPinGreen.DutyCycle(greenVal, uint32(cycle))
 48     ledPinBlue.DutyCycle(blueVal, uint32(cycle))
 49
 50     // Given the explanation above, a workaround is to set only accept a redVal of
 51     // 255. When this is the case the blue and green pins are changed to output mode
 52     // and turned off, resulting in only a red LED. If redVal isn't 255, then it's
 53     // disabled (like blue and green above) and the blue/green pins are reenabled
 54     // and set to their respective values. This is obviously a hack since the red
 55     // pin can only be set to 255 and can't be used in conjuction with the blue and
 56     // green pins.
 57     //
 58     // Comment these lines if you don't want to see this behavior.
 59     //  if redVal == 255 {
 60     //      ledPinRed.Mode(rpio.Pwm)
 61     //      ledPinRed.DutyCycle(redVal, cycle)
 62     //
 63     //      //ledPinGreen.Mode(rpio.Output)
 64     //      ledPinGreen.DutyCycle(greenVal, cycl)
 65     //      ledPinBlue.Mode(rpio.Output)
 66     //      ledPinBlue.Low()
 67     //  } else {
 68     //      ledPinRed.Mode(rpio.Output)
 69     //      ledPinRed.Low()
 70     //
 71     //      ledPinGreen.Mode(rpio.Pwm)
 72     //      ledPinBlue.Mode(rpio.Pwm)
 73     //      ledPinGreen.DutyCycle(greenVal, cycle)
 74     //      ledPinBlue.DutyCycle(blueVal, cycle)
 75     //  }
 76 }
 77
 78 func ledInit() {
 79     ledPinRed.Mode(rpio.Pwm)
 80     ledPinRed.Freq(freq)
 81     ledPinRed.DutyCycle(0, uint32(cycle))
 82
 83     ledPinGreen.Mode(rpio.Pwm)
 84     ledPinGreen.Freq(freq)
 85     ledPinGreen.DutyCycle(1, uint32(cycle))
 86
 87     ledPinBlue.Mode(rpio.Pwm)
 88     ledPinBlue.Freq(freq)
 89     ledPinBlue.DutyCycle(1023, uint32(cycle))
 90 }
 91
 92 func main() {
 93     if err := rpio.Open(); err != nil {
 94         os.Exit(1)
 95     }
 96     defer rpio.Close()
 97
 98     ledInit()
 99
100     reader := bufio.NewReader(os.Stdin)
101
102     for {
103         //
104         // Get RGB values
105         //
106         fmt.Println("Enter red value (0 to 1023):")
107         red, err := reader.ReadString('\n')
108         if err != nil {
109             fmt.Printf("Error reading from StdIn, %s", err)
110             os.Exit(1)
111         }
112         red = strings.TrimSuffix(red, "\n")
113
114         fmt.Println("Enter green value (0 to 1023):")
115         green, err := reader.ReadString('\n')
116         if err != nil {
117             fmt.Printf("Error reading from StdIn, %s", err)
118             os.Exit(1)
119         }
120         green = strings.TrimSuffix(green, "\n")
121
122         fmt.Println("Enter blue value (0 to 1023):")
123         blue, err := reader.ReadString('\n')
124         if err != nil {
125             fmt.Printf("Error reading from StdIn, %s", err)
126             os.Exit(1)
127         }
128         blue = strings.TrimSuffix(blue, "\n")
129
130         fmt.Printf("You entered red: %s, green: %s, blue: %s\n", red, green, blue)
131
132         //
133         // Convert string RGB values to int
134         //
135         blueNum, _ := strconv.Atoi(blue)
136         redNum, _ := strconv.Atoi(red)
137         greenNum, _ := strconv.Atoi(green)
138         fmt.Printf("red: %x, green: %x, blue: %x\n", redNum, greenNum, blueNum)
139
140         //
141         // Set LED color
142         //
143         ledColorSet(uint32(redNum), uint32(greenNum), uint32(blueNum))
144
145         //
146         // Quit?
147         //
148         fmt.Printf("Enter 'q' to quit\n")
149         quit, err := reader.ReadString('\n')
150         if err != nil {
151             fmt.Printf("Error reading from StdIn, %s, err")
152             os.Exit(1)
153         }
154         if quit == "q\n" {
155             ledColorSet(0, 0, 0)
156             break
157         }
158     }
159 }
```

## Summary

[^1]: The Raspberry Pi's GPIO PWM pins can also be configured and non-PWM pins.
[^2]: The [full Broadcom spec for the BCM2835](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf), starting at page 138, for more details about how PWM is implemented on the BCM2835.
[^3]: There's a detailed discussion about [Which pin(s) on RPi 3B is PWM capable](https://www.raspberrypi.org/forums/viewtopic.php?t=150254). Specifically regarding the effect of sharing 2 PWM channels for 4 PWM pins. The Broadcom spec[^2] also discusses this in section 9.4 on page 139, but in a less obvious way.
