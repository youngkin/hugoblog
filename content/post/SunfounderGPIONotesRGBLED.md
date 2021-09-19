---
title: "Raspberry Pi GPIO in Go an C - RGB LED"
description: "Use Pulse Width Modulation (PWM) to drive an RGB LED in Go and C"
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

```go
  1 //
  2 // Build - gcc -o rgbled rgbled.c  -lwiringPi -lpthread
  3 //
  4 #include <wiringPi.h>
  5 #include <softPwm.h>
  6 #include <stdio.h>
  7 #include <signal.h>
  8 #include <stdlib.h>
  9
 10 #define uchar unsigned char
 11 #define LedPinRed    24
 12 #define LedPinGreen  1
 13 #define LedPinBlue   23
 14
 15 static volatile int keepRunning = 1;
 16
 17 void interruptHandler(int);
 18
 19 void ledInit(void) {
 20     softPwmCreate(LedPinRed,  0, 100);
 21     softPwmCreate(LedPinGreen,0, 100);
 22     softPwmCreate(LedPinBlue, 0, 100);
 23 }
 24
 25 void ledColorSet(uchar r_val, uchar g_val, uchar b_val) {
 26     softPwmWrite(LedPinRed,   r_val);
 27     softPwmWrite(LedPinGreen, g_val);
 28     softPwmWrite(LedPinBlue,  b_val);
 29 }
 30
 31 int main(void) {
 32     if(wiringPiSetup() == -1){ //when initialize wiring failed, printf messageto screen
 33         printf("setup wiringPi failed !");
 34         return 1;
 35     }
 36
 37     signal(SIGINT, interruptHandler);
 38     printf("Hit ^-c to exit\n");
 39
 40     ledInit();
 41     while(keepRunning) {
 42             printf("Red\n");
 43             ledColorSet(0xff,0,0);
 44             delay(1000);
 45
 46             printf("Green\n");
 47             ledColorSet(0,0xff,0);
 48             delay(1000);
 49
 50             printf("Blue\n");
 51             ledColorSet(0x00,0x00,0xff);
 52             delay(1000);
 53
 54             printf("Yellow\n");
 55             ledColorSet(0xff,0xff,0x00);
 56             delay(1000);
 57
 58             printf("Purple\n");
 59             ledColorSet(0xff,0x00,0xff);
 60             delay(1000);
 61
 62             printf("Cyan\n");
 63             ledColorSet(0xc0,0xff,0x3e);
 64             delay(1000);
 65
 66             printf("Off\n");
 67             ledColorSet(0,0,0);
 68             delay(1000);
 69     }
 70
 71     return 0;
 72 }
 73
 74 void interruptHandler(int sig) {
 75     // Turn off LED
 76     // Don't understand why, but ledColorSet(0,0,0); doesn't work
 77     pinMode(LedPinRed, OUTPUT);
 78     digitalWrite(LedPinRed, LOW);
 79     pinMode(LedPinGreen, OUTPUT);
 80     digitalWrite(LedPinGreen, LOW);
 81     pinMode(LedPinBlue, OUTPUT);
 82     digitalWrite(LedPinBlue, LOW);
 83
 84     printf("\nExiting...\n");
 85
 86     exit(0);
 87 }
 ```

 Line 2 provides the command needed to build the program - `gcc -o rgbled rgbled.c  -lwiringPi -lpthread`. The `-l` flags reference the libraries needed to build the program. `-lwiringPi` refers to the WiringPi library. It should be installed in the correct place along with WiringPi and the build command should just work. Libraries are usually located in `/usr/lib`.

 Note the pin numbering used in lines 11 - 13. I'm using WiringPi pins 24, 1, and 23 for red, green, and blue, which correspond to BCM pins 19, 18, and 13 respectively. The Sunfounder wiring diagram above using BCM pins 17, 18, and 27 for red, green, and blue, which correspond to WiringPi pins 0, 1, and 2 respectively. The Sunfounder C code uses the pins as shown in the above diagram. Make sure your breadboard is wired to BCM pins 19, 18, and 13 for red, green, and blue; wiringPi pins 24, 1, and 23, or change the above code to to use 0, 1, and 2 for `LedPinRed`, `LedPinGreen`, and `LedPinBlue` respectively. This will be good practice for navigating your way around a breadboard. See the Sunfounder page [GPIO Extension Board](https://docs.sunfounder.com/projects/raphael-kit/en/latest/gpio_extension_board.html) for a quick cross reference between BCM, board, and WiringPi pin numbers.

 The `softPwm*` functions called on lines 20-22 and 26-28 indicate that software PWM is being used.

 The calls to `ledColorSet(...)` within the `while(keepRunning)` loop use hex numbers to set the colors. These are used to generate the full range of available colors. I did change the values somewhat as the green LED used apparently has less resistance as it's quite a bit brighter than the red and blue LEDs and therefore throws off the generated colors. Similarly, the blue LED seems to have more resistance as it is quite a bit dimmer than the other 2 LEDs. Changing the values, at least for my specific RGB LED, generated truer colors.

 Line 37 registers an interrupt handler to catch the SIGINT interrupt generated when ^-C is entered at the keyboard. Line 17 declares the interrupt handler function. Lines 74 thru 87 implement the interrupt handler function. Within that function the LED is turned off.

 The program can be run, after compiling, using `./rgbled`.

 ### Hardware PWM in C

 Since the Sunfounder documentation doesn't include a hardware PWM solution in C I decided to create one for myself. Partly to assure myself that the hardware PWM behavior I saw using the Go go-rpio library wasn't limited to that library. 

```c
  1 //
  2 // Build - gcc -o rgbledHardware rgbledHardware.c  -lwiringPi -lpthread
  3 //
  4 #include <wiringPi.h>
  5 #include <softPwm.h>
  6 #include <stdio.h>
  7 #include <signal.h>
  8 #include <stdlib.h>
  9
 10 #define uchar unsigned char
 11 #define LedPinRed    24
 12 #define LedPinGreen  1
 13 #define LedPinBlue   23
 14
 15 static volatile int keepRunning = 1;
 16
 17 void interruptHandler(int);
 18
 19 //
 20 // Hardware PWM pins include WiringPi pins 1, 26, 23, 24 (BCM 18, 12, 13, and 19 respectively).
 21 // For practical purposes only 2 are usable as the other 2 are linked, turn one on they both turn on.
 22 // Pins 24 & 26 (BCM 19 & 12) and Pins 1 & 23 (BCM 13 and 18) are independent. Pins 1 and 26 (BCM 18 & 12)
 23 // are linked. Set one and the other will also be set. Ditto for pins 23 & 24 (BCM 13 & 19). In addition,
 24 // Writing to linked pins and one other results in inconsistent results. The independent pin will always
 25 // be set, but the dependent pins won't always be set correctly.
 26 //
 27 void ledInit(void) {
 28     pinMode(LedPinRed, PWM_OUTPUT);
 29     pinMode(LedPinGreen, PWM_OUTPUT);
 30     pinMode(LedPinBlue, PWM_OUTPUT);
 31
 32     pwmWrite(LedPinRed, 0xff);
 33     pwmWrite(LedPinGreen, 0x32);
 34     pwmWrite(LedPinBlue, 0xff);
 35
 36     delay(1000);
 37     printf("Initialization complete\n");
 38 }
 39
 40 void ledColorSet(uchar r_val, uchar g_val, uchar b_val) {
 41     pwmWrite(LedPinRed, r_val);
 42     pwmWrite(LedPinGreen, g_val);
 43     pwmWrite(LedPinBlue, b_val);
 44 }
 45
 46 int main(void) {
 47     if(wiringPiSetup() == -1){ //when initialize wiring failed, printf messageto screen
 48         printf("setup wiringPi failed !");
 49         return 1;
 50     }
 51
 52     signal(SIGINT, interruptHandler);
 53     printf("Hit ^-c to exit\n");
 54
 55     ledInit();
 56     while(keepRunning) {
 57             printf("Red\n");
 58             ledColorSet(0xff,0x00,0x00);   //red
 59             delay(2000);
 60             printf("Green\n");
 61             ledColorSet(0x00,0x32,0x00);   //green
 62             delay(2000);
 63             printf("Blue\n");
 64             ledColorSet(0x00,0x00,0xff);   //blue
 65             delay(2000);
 66             printf("Yellow\n");
 67             ledColorSet(0xff,0x32,0x00);   //yellow
 68             delay(1000);
 69             printf("Purple\n");
 70             ledColorSet(0xff,0x00,0xff);   //purple
 71             delay(1000);
 72             printf("Cyan\n");
 73             ledColorSet(0xc0,0x32,0xff);   //cyan
 74             delay(1000);
 75     }
 76     return 0;
 77 }
 78
 79 void interruptHandler(int sig) {
 80         // Turn off LED
 81         // Don't understand why, but ledColorSet(0,0,0); doesn't work
 82         pinMode(LedPinRed, OUTPUT);
 83         digitalWrite(LedPinRed, LOW);
 84         pinMode(LedPinGreen, OUTPUT);
 85         digitalWrite(LedPinGreen, LOW);
 86         pinMode(LedPinBlue, OUTPUT);
 87         digitalWrite(LedPinBlue, LOW);
 88
 89         printf("\nExiting...\n");
 90
 91         exit(0);
 92 }
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
  1 //
  2 // Hardware PWM pins include WiringPi pins 1, 26, 23, 24 (BCM 18, 12, 13, and 19 respectively).
  3 // For practical purposes only 2 are usable as the other 2 are linked, turn one on they both turn on.
  4 // Pins 24 & 26 (BCM 19 & 12) and Pins 1 & 23 (BCM 13 and 18) are independent. Pins 1 and 26 (BCM 18 & 12)
  5 // are linked. Set one and the other will also be set. Ditto for pins 23 & 24 (BCM 13 & 19). In addition,
  6 // writing to linked pins and one other results in inconsistent results. The independent pin will always
  7 // be set, but the dependent pins won't always be set correctly. This could be a result of timing issues
  8 // that affect the ordering of when signals are sent to pins on the same channel.
  9 //
 10
 11 package main
 12
 13 import (
 14     "bufio"
 15     "fmt"
 16     "os"
 17     "strconv"
 18     "strings"
 19
 20     "github.com/stianeikeland/go-rpio/v4"
 21 )
 22
 23 const (
 24     ledPinRed   = rpio.Pin(19)
 25     ledPinGreen = rpio.Pin(18)
 26     ledPinBlue  = rpio.Pin(13)
 27     freq        = 100000
 28     cycle       = 1024
 29 )
 30
 31 func ledColorSet(redVal, greenVal, blueVal uint32) {
 32     // This doesn't work as expected because GPIO pins 19 & 13 are essentially linked. This
 33     // is also true for GPIO pins 18 & 12, but since pin 12 isn't used here pin 18, green,
 34     // works as expected. So, when a blueVal or redVal are provided, the same value
 35     // is propagated to the linked pin. So for example, if redVal is 0 and blueVal
 36     // is 255, both the red and blue leds will light up (purple).
 37     //
 38     // Uncomment these lines if you want to see this behavior.
 39     ledPinGreen.Mode(rpio.Pwm)
 40     ledPinBlue.Mode(rpio.Pwm)
 41     ledPinGreen.DutyCycle(greenVal, cycle)
 42     ledPinBlue.DutyCycle(blueVal, cycle)
 43
 44     // Given the explanation above, a workaround is to set only accept a redVal of
 45     // 255. When this is the case the blue and green pins are changed to output mode
 46     // and turned off, resulting in only a red LED. If redVal isn't 255, then it's
 47     // disabled (like blue and green above) and the blue/green pins are reenabled
 48     // and set to their respective values. This is obviously a hack since the red
 49     // pin can only be set to 255 and can't be used in conjuction with the blue and
 50     // green pins.
 51     //
 52     // Comment these lines if you don't want to see this behavior.
 53     //  if redVal == 255 {
 54     //      ledPinRed.Mode(rpio.Pwm)
 55     //      ledPinRed.DutyCycle(redVal, cycle)
 56     //
 57     //      //ledPinGreen.Mode(rpio.Output)
 58     //      ledPinGreen.DutyCycle(greenVal, cycl)
 59     //      ledPinBlue.Mode(rpio.Output)
 60     //      ledPinBlue.Low()
 61     //  } else {
 62     //      ledPinRed.Mode(rpio.Output)
 63     //      ledPinRed.Low()
 64     //
 65     //      ledPinGreen.Mode(rpio.Pwm)
 66     //      ledPinBlue.Mode(rpio.Pwm)
 67     //      ledPinGreen.DutyCycle(greenVal, cycle)
 68     //      ledPinBlue.DutyCycle(blueVal, cycle)
 69     //  }
 70 }
 71
 72 func ledInit() {
 73     ledPinRed.Mode(rpio.Pwm)
 74     ledPinRed.Freq(freq)
 75     ledPinRed.DutyCycle(0, cycle)
 76
 77     ledPinGreen.Mode(rpio.Pwm)
 78     ledPinGreen.Freq(freq)
 79     ledPinGreen.DutyCycle(1, cycle)
 80
 81     ledPinBlue.Mode(rpio.Pwm)
 82     ledPinBlue.Freq(freq)
 83     ledPinBlue.DutyCycle(1023, cycle)
 84 }
 85
 86 func main() {
 87     if err := rpio.Open(); err != nil {
 88         os.Exit(1)
 89     }
 90     defer rpio.Close()
 91
 92     ledInit()
 93
 94     reader := bufio.NewReader(os.Stdin)
 95
 96     for {
 97         //
 98         // Get RGB values
 99         //
100         fmt.Println("Enter red value (0 to 1023):")
101         red, err := reader.ReadString('\n')
102         if err != nil {
103             fmt.Printf("Error reading from StdIn, %s", err)
104             os.Exit(1)
105         }
106         red = strings.TrimSuffix(red, "\n")
107
108         fmt.Println("Enter green value (0 to 1023):")
109         green, err := reader.ReadString('\n')
110         if err != nil {
111             fmt.Printf("Error reading from StdIn, %s", err)
112             os.Exit(1)
113         }
114         green = strings.TrimSuffix(green, "\n")
115
116         fmt.Println("Enter blue value (0 to 1023):")
117         blue, err := reader.ReadString('\n')
118         if err != nil {
119             fmt.Printf("Error reading from StdIn, %s", err)
120             os.Exit(1)
121         }
122         blue = strings.TrimSuffix(blue, "\n")
123
124         fmt.Printf("You entered red: %s, green: %s, blue: %s\n", red, green, blue)
125
126         //
127         // Convert string RGB values to int
128         //
129         blueNum, _ := strconv.Atoi(blue)
130         redNum, _ := strconv.Atoi(red)
131         greenNum, _ := strconv.Atoi(green)
132         fmt.Printf("red: %x, green: %x, blue: %x\n", redNum, greenNum, blueNum)
133
134         //
135         // Set LED color
136         //
137         ledColorSet(uint32(redNum), uint32(greenNum), uint32(blueNum))
138
139         //
140         // Quit?
141         //
142         fmt.Printf("Enter 'q' to quit\n")
143         quit, err := reader.ReadString('\n')
144         if err != nil {
145             fmt.Printf("Error reading from StdIn, %s, err")
146             os.Exit(1)
147         }
148         if quit == "q\n" {
149             ledColorSet(0, 0, 0)
150             break
151         }
152     }
153 }
```

## Summary

[^1]: The Raspberry Pi's GPIO PWM pins can also be configured and non-PWM pins.
[^2]: The [full Broadcom spec for the BCM2835](https://www.raspberrypi.org/app/uploads/2012/02/BCM2835-ARM-Peripherals.pdf), starting at page 138, for more details about how PWM is implemented on the BCM2835.
[^3]: There's a detailed discussion about [Which pin(s) on RPi 3B is PWM capable](https://www.raspberrypi.org/forums/viewtopic.php?t=150254). Specifically regarding the effect of sharing 2 PWM channels for 4 PWM pins. The Broadcom spec[^2] also discusses this in section 9.4 on page 139, but in a less obvious way.
