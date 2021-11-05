---
title: "Raspberry Pi GPIO in Go and C - RGB LED"
description: "Use Pulse Width Modulation (PWM) on a Raspberry Pi to drive an RGB LED in Go and C"
date: 2021-11-04T13:13:42-06:00
draft: false
image: "images/pwmfordummies/rgbledlandscape1a.png"
tags: ["raspberry-pi", "Go", "C", "GPIO"]
categories: ["raspberry-pi", "Go", "GPIO"]
GHissueID: 1
toc: true
---

<!--more-->

## Overview

This is the second article in a series that explores GPIO programming on a Raspberry Pi 3B+. The first is [Raspberry Pi GPIO in Go and C - Blinking LED](https://youngkin.github.io/post/sunfoundergpionotesled/). It is a supplement to the [Sunfounder RGB LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) project. You can find the full series [here](https://youngkin.github.io/categories/gpio/). 

This article explores the use of Pulse Width Modulation (PWM) to drive an RGB LED, as well as how to control an individual LED pin's brightness. The code samples will be in Go and C.

## Prerequisites

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'stretch' version of the Raspbian OS. Given that the Sunfounder Ultimate Starter Kit is advertised to work with a Raspberry Pi 4, I would expect the 4 series to work as well. I'm less sure about other Raspberry Pi versions, especially versions with 26 vs. 40 GPIO pins.

Next you'll need

1. A [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==)
2. Some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==)
3. A [220 Ohm resistor, and a RGB LED](https://www.amazon.com/EDGELEC-Tri-Color-Multicolor-Diffused-Resistors/dp/B077XGF3YR/ref=asc_df_B077XGF3YR/?tag=hyprod-20&linkCode=df0&hvadid=242051162351&hvpos=&hvnetw=g&hvrand=11064062033670066895&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=9028749&hvtargid=pla-430228081645&psc=1).

You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the cable will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however. [Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). If you expect to follow this series I recommend buying the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).

<img style="border:1px solid black" src="/images/pwmfordummies/RaphaelKit.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder Ultimate Starter/Raphael kit</i></center></figcaption>

You will also need some basic C  and Go programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like Vi or nano.

To compile and run the C program you'll need the [WiringPi](https://github.com/WiringPi/WiringPi)[^4] libary. It's easy to get:

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

WiringPi is unique in that it includes a command line tool, `gpio`, as shown above, that can be used to manage, control, and query the GPIO board. This can be very handy. See the [gpio reference](http://wiringpi.com/the-gpio-utility/) for more information on what it can do and how to use it.

I chose not to download the code from the Sunfounder site, preferring to write my own instead, even if all I did was copy directly from the project documentation. Due to this I created my own location to create the code. In fact, [my code is in Github](https://github.com/youngkin/gpio). If you do like downloading code you have the option of downloading, cloning, or forking it from my Github repository. As an added bonus, the project code written in Go is also located there. The code for this project is located at [gpio/rgbled](https://github.com/youngkin/gpio/tree/main/rgbled).

If you're interested in Go development on a Raspberry Pi you'll need to install the development environment onto the Raspberry Pi. [Here's a simple source](https://www.jeremymorgan.com/tutorials/raspberry-pi/install-go-raspberry-pi/) that explains how to accomplish this. This source is a little dated, but the only significant issue is with the version of Go to install. The source shows installing Go __1.14.4.linux-arm64.tar.gz__ and __1.14.4.linuxarmv6l.tar.gz__. The current versions are __1.17.1.linux-arm64.tar.gz__ and __1.17.1.linuxarmv6l.tar.gz__. For the Raspberry Pi 3B+ the correct choice will be __1.17.1.linuxarmv6l.tar.gz__. The other is intended for 64 bit systems like the Raspberry Pi 4 series.

For Go development you'll also need the [go-rpio](https://github.com/stianeikeland/go-rpio)[^5] library. I chose it for several reasons:

1. It seems to be in fairly wide use
2. It seems to be fairly complete
3. It's relatively active
4. It comes with example code and good documentation
5. Its API is similar to WiringPi's

Another Go option is [periph](https://github.com/periph/host) (code) with [documentation](https://periph.io/). It is more active and the documentation is very good, better than go-rpio. But for the LED examples I was able to find, go-rpio better matched what I was looking for, especially with regard to this project. But this is an excellent alternative to go-rpio and vice-versa.

Finally, I'm assuming a basic knowledge of Linux if you want to veer away from the cookbook style of the Sunfounder docs. For example, I won't be explaining what __root privileges__ are.

## Information that would have been helpful

This project uses PWM (Pulse Width Modulation) on GPIO pins to achieve the desired effect, namely demonstrating how to create different colors with a simple RGB LED. Unfortunately the Sunfounder documentation leaves out information about PWM, like what is PWM and how is it implemented on a Raspberry Pi? I started to include in this section all the information I found missing from the [Sunfounder project documentation](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html), but it quickly became apparent that would better be left to a separate article. So if you don't already have a good understanding of PWM I'd recommend reading my [Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi](https://youngkin.github.io/post/pulsewidthmodulationraspberrypi/)[^1] article before continuing with this article. I will be using terminology explained in that article throughout this one.

## RGB LED in C (Software PWM)

Depending on your experience, you should consider reviewing the [RGB LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) project starting with [Components List and Introduction](https://docs.sunfounder.com/projects/raphael-kit/en/latest/component_list.html). Peruse it up through the _Play with C_ section. If you're experienced with basic electronics and components like breadboards and resistors you can skip it.

You should set up the breadboard as described in the project documentation or in the diagram below:

<img style="border:1px solid black" src="/images/pwmfordummies/RgbLed.png" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder RGB LED breadboard setup</i></center></figcaption>

The WiringPi `gpio` utility can help with debugging if necessary. You already have this utility, you used it to verify the WiringPi installation in the [Prerequisites](#prerequisites) section above.

The C code described in the Sunfounder [RGB LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) uses software PWM. When run, the colors of the RGB LED change as expected. Varying voltages to the respective RGB pins using software PWM produces light of varying brightness (off to full bright) which will produce a wide range of colors.

Here is a slightly modified version of the Sunfounder C program that the controls an RGB LED:

{{< gist youngkin 58aaaab75b2f16be7bd4a3621d17348b >}}

 Line 5 provides the command needed to build the program - `gcc -o rgbled rgbled.c  -lwiringPi -lpthread`. The `-l` flags reference the libraries needed to build the program. `-lwiringPi` refers to the WiringPi library. It should be installed in the correct place along with WiringPi and the build command should just work. Libraries are usually located in `/usr/lib`.

Lines 14-16 specify, using the WiringPi pin numbering scheme, the GPIO pins 0, 1, and 2 for `LedPinRed`, `LedPinGreen`, and `LedPinBlue` respectively.

Line 20 contains the declaration of a signal handler for interrupt signals (SIGINT).

 Lines 23-25 use the `softPwmCreate()` function to initialize the pins. The initial characters in the function name, `softPwm`, indicate that the pins are being initialized for software PWM. The function's first parameter is the pin number. The second parameter is the starting pulse width. The final parameter is the range. See [Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi](https://youngkin.github.io/post/pulsewidthmodulationraspberrypi/) if you don't understand these terms.

 Lines 29-31, `softPwmWrite()` send the desired signal/voltage to the associated pin. The first parameter is the pin number, the same as in `softPwmCreate()` above. The second parameter is the pulse width. Notice that the pulse width was `0` in `softPwmCreate()`. This has the effect of setting the pin to zero volts. In `softPwmWrite()` the pulse width is set to the values specified by `r_val`, `g_val`, and `b_val`. These values represent the red, green, and blue values respectively. The maximum effective value for the pulse width is the range. This will result in full brightness/voltage. Any values greater than range will have no additional impact. It is important to note that the desired voltage will continue to flow to the pins until reset by another `softPwmWrite()`.

 {{< gist youngkin 55e6a197e7da1d727e9d8cd616571c73 >}}

The code snippet above is a continuation of the program. It shows the `main()` function. Lines 2–5 initialize the WiringPi library. The program exits if this initialization fails.

Line 7 specifies a signal handler to be called when SIGINT signal is received by the program. This is the signal set when the user enters ctl-C at the keyboard. This signal handler will terminate the program. This function was declared in the previous code snippet.

The calls to `ledColorSet(...)` within the `while(keepRunning)` loop at line 11 use hex numbers to set the colors. These are used to generate the full range of available colors. They must fall in the range of `0` to `0xff`, recall the range value was specified in `softPwmCreate()`. I did change the values from the original Sunfounder code as the green LED used apparently has less resistance as it's quite a bit brighter than the red and blue LEDs and therefore throws off the generated colors. Similarly, the blue LED seems to have more resistance as it is quite a bit dimmer than the other 2 LEDs. Changing the values, at least for my specific RGB LED, generated truer colors. The code loops, changing the LED color, until terminated via a ctl-C at the keyboard.


{{< gist youngkin 7668e669ebbc578e9e7cafcc506ef203 >}}

The code snippet above is a continuation of the previous code snippet. It contains the implementation of the interrupt handler described previously. It turns off all the RGB LED's pins, resulting in the LED being completely off. `pinMode()` sets the mode for the specified pin (first parameter) to `OUTPUT` (second parameter). This changes the pin from a PWM pin to a pin that can only be set to ON or OFF. The voltage of an output pin cannot be varied. `digitalWrite()` sets the voltage for the specified pin to zero (LOW).

 The program can be run, after compiling, using `./rgbled`.

 ## RGB LED in C (Hardware PWM)

 Since the Sunfounder documentation doesn't include a hardware PWM solution in C I decided to create one for myself. For this program modify the breadboard as shown below:

<img style="border:1px solid black" src="/images/pwmfordummies/RgbLedHardware.jpg" align="center" width="600" height="300"/>
<figcaption align="left"><center><i style="color:black;">Sunfounder RGB LED breadboard setup</i></center></figcaption>

This time I'm using WiringPi pins 24, 1, and 23 for red, green, and blue, which correspond to BCM pins 19, 18, and 13 respectively. This is because these are 3 of the 4 hardware PWM pins on the Broadcomm BCM2835 board. Be sure to rewire the board to match these new pin assignments . The WiringPi `gpio` utility can help with debugging if necessary. You already have this utility, you used it to verify the WiringPi installation in the [Prerequisites](#prerequisites) section above.

{{< gist youngkin 1a249b318748b2ce7ec59ca445677d19 >}}

This version of RGB LED is very similar to the software PWM version above. There are 2 significant differences.

1. Note the pin numbering used in lines 14-16. The WiringPi pin numbers used here correspond to the hardware PWM pins available on the GPIO board.
2. The `ledInit()` and `ledColorSet()` functions (lines 30-48) are quite different. In these functions pin mode is set to `PWM_OUTPUT` vs. the use of `softPwmCreate` and `pwmWrite` is used instead of `softPwmWrite`. Line 34 sets the range. Note that the pin number isn't specified for the `pwmSetRange()` call. There are 2 reasons for this. The first is that range is set at the channel level, not for individual pins. The second is that the WiringPi library doesn't allow for the 2 channels to be specified separately. It sets both channels to the same value. Line 35, `pwmSetClock()`, (indirectly) sets the frequency. It specifies a _divisor_, in this case 2, that is used to divide the board's oscillator's clock frequency into the frequency used to control the pins. The divisor must be a number between 2 and 4095. As with `pwmSetRange()`, `pwmSetClock()` is specified at the channel level, not for individual pins. See [Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi](https://youngkin.github.io/post/pulsewidthmodulationraspberrypi/) for more discussion about clocks, frequency, and divisor as they relate to PWM.

{{< gist youngkin b46a3eccdbbab8cd577cbbda87824425 >}}

The code snippet above contains the remainder of the program, `main()` and the interrupt handler. The interrupt handler is exactly the same as the interrupt handler in the software PWM version. Like the software PWM version of the program, the program loops until interrupted via a ctl-C.

The program is run using root privileges with `sudo ./rgbledHardware`. `sudo` is needed because direct hardware access is limited to users with root privileges. `sudo` provides root privileges to commands prefixed with `sudo`.

When running the program you'll notice that the RGB LED doesn't produce the expected colors. Sometimes it might show purple when it should be showing blue. Sometimes it might be turned off completely when it's supposed to be showing a color. This behavior occurs because BCM pin 13 (blue) and BCM pin 19 (red) are on the same hardware channel. As explained above, when a signal is sent to one channel it will propagate to both pins that share that channel. Green, on BCM pin 12, is unaffected. One behavior I don't understand is why sometimes the LED is off when it should be showing a color. This only happens for red (BCM19) and blue (BCM 13) pins. Perhaps the pins that share a channel don't produce a signal at exactly the same time. Imagine a case where the blue pin is set to 0xff and the red pin is set to 0x00. If the red pin's signal comes in slightly after the blue signal it would override the blue pin signal turning the LED off. The takeaway from all this is that for all intents and purposes, there are only 2 hardware PWM pins can be in use at the same time, and they can't be on the same channel.

## RGB LED in Go (Hardware PWM)

This version of RGB LED will work with the same breadboard setup as the C hardware PWM version. That is, it uses the hardware PWM pins on the GPIO board. Unlike the Sunfounder C version, and the first C version in this article, the Go library only supports hardware PWM. In my [Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi](https://youngkin.github.io/post/pulsewidthmodulationraspberrypi/) article I do include a software version of PWM in Go. It's implemented in a companion program, [freqtest.go](https://github.com/youngkin/gpio/blob/main/pwmdemo/pwmexplorer/apps/freqtest.go) in the `runSoftwarePWM()` function. Here is the code:

{{< gist youngkin 072801bdf96d13eae5cba2e2774307e4 >}}

Lines 27-29 define the pins to use for the red, green and blue elements of the LED.

Lines 30-31 define default values to use for settings.

Lines 35–75 set the Mode to PWM and theDutyCycle for the LED's pins. In `DutyCycle()`, the first parameter is the pulse width, called _duty_ by go-rpio. The second parameter is the range, called _cycle_ by go-rpio. The remainder of the lines contain comments and code that address the issues with trying to use 2 hardware PWM pins residing on the same channel as noted in the [Hardware PWM in C](#hardware-pwm-in-c) section above.

{{< gist youngkin 11c97e8d84b134c40e3d15d6f3c639cf >}}

This code snippet is a continuation of the program above. This gist shows `ledInit()`. It initializes the GPIO pins for use in the program. They set the mode (PWM), frequency, and duty cycle (providing parameters for pulse width and range (aka cycle)). As described in [Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi](https://youngkin.github.io/post/pulsewidthmodulationraspberrypi/), the specified frequency must be between 4688 and 9,600,000.

{{< gist youngkin 188d35218c26003cfc029b5ca7a3f838 >}}

This is part 3 of the program and shows the implementation of `main()`. Lines 2-4 initialize the go-rpio library. Line 5 makes sure the resources used are released when the program exits. 

The remainder of the program prompts the user for the red, green, and blue values to use and the sets the pins approporiately.

The full program, [rgbled.go](https://github.com/youngkin/gpio/blob/main/rgbled/rgbled.go), can be found in the [gpio github repo](https://github.com/youngkin/gpio).

## Summary

That's it, I hope you found this article interesting. To quickly review, this article covered the following:

1. Setting up the physical environment needed to experiment with an RGB LED on a Raspberry Pi 3B+.
2. Provided, via a link to another article[^1], the detailed knowledge of PWM that's missing from the [Sunfounder RGB LED Project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html).
3. Provided and explained the C and Go code needed to set colors in an RGB LED using both hardware and software PWM.

## References

[^1]: [Pulse Width Modulation for Dummies, with a Slice of Raspberry Pi](https://youngkin.github.io/post/pulsewidthmodulationraspberrypi/) contains all you need to know about PWM on a Raspberry Pi, at least until you want to become a PWM expert!
[^2]: [Sunfounder RGB LED Project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html)
[^5]: [go-rpio](https://github.com/stianeikeland/go-rpio)
[^4]: [WiringPi](https://github.com/WiringPi/WiringPi)
