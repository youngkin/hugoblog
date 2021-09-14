---
title: "Sunfounder Raspberry Pi Kit Notes for Go and C - Blinking LED"
description: "Questions about Sunfounder RPi kits? Look no further!"
date: 2021-09-11T13:13:42-06:00
draft: false
image: "images/sunfounderLED.jpeg"
tags: ["raspberry-pi", "Go", "C", "GPIO"]
categories: ["raspberry-pi", "Go", "GPIO"]
GHissueID: 1
toc: true
---

This is the first article in a series that explores GPIO programming on a Raspberry Pi 3B+. It is a supplement to the [Sunfounder Blinking LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/play_with_c.html). The code samples will be in Go and C.
<!--more-->

## Overview

I recently became interested in GPIO programming on a Raspberry Pi. It's a nice way to see something concrete controlled by software. It's also a nice way to learn a little bit about electronics. This is the first article in a series I intend to write to document my journey in GPIO programming.

To lower the bar to getting started I decided to purchase a GPIO/electronics kit and repurpose one of my Raspberry Pis. I chose [Sunfounder](https://www.sunfounder.com/), mostly because they seem to be well regarded and their kits get good reviews. I ended up choosing the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1) (AKA Raphael)  because it has a good number of interesting electronic components and associated projects.

One thing I noticed right away is that most GPIO articles assume Python for the programming language. Sunfounder's kits and associated documentation support a number of languages including Python and C, but they lean towards Python and more projects are available in Python and not so many in C. I'm primarily a Go developer so I'm also interested in implementing GPIO projects in Go.

Some of the Sunfounder kit reviews mention that the project documentation is a little sparse and short on explaining how things work. I'm just getting started and I've seen a little of this, but the first project, [Sunfounder Blinking LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/play_with_c.html), is pretty well documented. I only got hung up in a couple of spots and it didn't take long for me to figure out what I did wrong. The second project, [RGB LED](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.2_rgb_led_c.html) was a bit different. It's more complicated and much less detailed regarding explaining how and why things work. If all you want to do is follow a cookbook to implement the project it works well enough. But it's lacking if you want to understand how and why it works in more detail. I spent quite a while working on this one trying to fill in the information missing from the project documentation. The point of this series is to fill that gap, from both coding and electronic perspectives.

The focus of this article is the [Sunfounder Blinking LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/play_with_c.html). It's the first project in a [set of projects targeting the Ultimate Starter/Raphael kit](https://docs.sunfounder.com/projects/raphael-kit/en/latest/introduction.html).

## Prerequisites

If you don't have one, you'll need a Raspberry Pi. I used a Raspberry Pi 3B+ with the 'stretch' version of the Raspbian OS. Given that the Sunfounder Ultimate Starter Kit is advertised to work with a Raspberry Pi 4, I would expect the 4 series to work as well. I'm less sure about other Raspberry Pi versions, especially versions with 26 vs. 40 GPIO pins.

Next you'll need is a [breadboard](https://www.amazon.com/dp/B082KBF7MM/ref=sspa_dk_detail_4?psc=1&pd_rd_i=B082KBF7MM&pd_rd_w=1tGTV&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=fX8JB&pf_rd_r=44DE0RS1E9FD42RBYC7R&pd_rd_r=47cbdc7f-7834-455f-9429-ef74a438bd45&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFVVkdZVUZRNUw3ODkmZW5jcnlwdGVkSWQ9QTA4MzI4MzYyU0VLNzBJM0cxRUVMJmVuY3J5cHRlZEFkSWQ9QTA0Mjk1NTMzSzNSWlNFUjU0NURBJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), some [jumper wires](https://www.amazon.com/dp/B08HZ26ZLF/ref=syn_sd_onsite_desktop_19?psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUExRFpLWElCRjg1MUNMJmVuY3J5cHRlZElkPUEwMjMyMTE1M01aOFE3U1BQS09YSiZlbmNyeXB0ZWRBZElkPUEwODE5NTMxMktEMTlZRjEyQjBJNiZ3aWRnZXROYW1lPXNkX29uc2l0ZV9kZXNrdG9wJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==), [a 220 Ohm resistor, and an LED](https://www.amazon.com/ELEGOO-Electronics-Component-resistors-Potentiometer/dp/B01ERPXFZK/ref=sr_1_7_sspa?crid=3EJQNCOWP00IF&dchild=1&keywords=resistors&qid=1631478270&s=industrial&sprefix=resis%2Cindustrial%2C219&sr=1-7-spons&psc=1&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEzTktVSzNYMkxMSDlKJmVuY3J5cHRlZElkPUEwMjAzMDY0NVRERkFLVjVRTUFWJmVuY3J5cHRlZEFkSWQ9QTA5MjM2NjUxUFZYQUlETVAzRDA3JndpZGdldE5hbWU9c3BfbXRmJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). You should also consider getting a [40 pin female to female with a T-Type adapter](https://www.amazon.com/dp/B082PRVRYR/ref=sspa_dk_detail_2?psc=1&pd_rd_i=B082PRVRYR&pd_rd_w=8mKhr&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=e9psa&pf_rd_r=S09F37DF2G5FW8B8GX4B&pd_rd_r=c065c120-e60b-45e9-b93b-f581f048cf46&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUFCMzhUQ09COFI2VlMmZW5jcnlwdGVkSWQ9QTA5NjU2ODUxRDkxNEYwSTYwV09KJmVuY3J5cHRlZEFkSWQ9QTAxOTg1MTUyRUhEUlc2VzQ2VDQ4JndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==) to attach the GPIO outputs to the breadboard. You can use only jumper wires, but the cable will make things easier and will help prevent damage to the GPIO pins on the Raspberry Pi. If you elect not to buy the 40 pin cable with T-Type adapter you'll need to buy [male-to-female jumper wires](https://www.amazon.com/SinLoon-Breadboard-Arduino-Circuit-40-Pack/dp/B08M3QLL3Q/ref=pd_sbs_7/143-0445142-7950409?pd_rd_w=sVLrc&pf_rd_p=8b76d7a7-ab83-4ddc-a92d-e3e33bfdbf03&pf_rd_r=CDM5TGJT03VKF0ZFB577&pd_rd_r=8e58fd82-8503-41cf-b8f2-c78eaeb78d25&pd_rd_wg=tT1U0&pd_rd_i=B08M3QLL3Q&psc=1). Buying all these things separately will cost more than a kit however. [Here's a simple kit that has all of the above](https://www.amazon.com/dp/B06WP7169Y/ref=sspa_dk_detail_5?psc=1&pd_rd_i=B06WP7169Y&pd_rd_w=OZVyf&pf_rd_p=887084a2-5c34-4113-a4f8-b7947847c308&pd_rd_wg=0V0IH&pf_rd_r=623YJTBQ2CN2B2GYXQG5&pd_rd_r=faa61f0f-3aec-4cf0-8e7e-d44eb1b3b92f&spLa=ZW5jcnlwdGVkUXVhbGlmaWVyPUEyUVlDQzMzVVZBMFYxJmVuY3J5cHRlZElkPUEwMzExNzk4MUhGSjFSS0VKTlBROCZlbmNyeXB0ZWRBZElkPUEwMzYwNjg2UUdMRU44N0YzNzIwJndpZGdldE5hbWU9c3BfZGV0YWlsJmFjdGlvbj1jbGlja1JlZGlyZWN0JmRvTm90TG9nQ2xpY2s9dHJ1ZQ==). If you expect to follow this series I recommend buying the [Sunfounder Raspberry Pi Ultimate Starter Kit](https://www.amazon.com/gp/product/B09BMVT4CB/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1).

You will also need some basic Go programming knowledge as well as familiarity with logging on to a Raspberry Pi terminal, or into the desktop GUI that comes with some OS versions. Depending on the approach you take, you may need to connect a keyboard and monitor to the Raspberry Pi. I simply SSH into the Pi. You'll also need familiarity with how to use an editor like Vi or nano.

I chose not to download the code from the Sunfounder site, preferring to write my own instead, even if all I did was copy directly from the project documentation. Due to this I created my own location to create the code. In fact, [my code is in Github](https://github.com/youngkin/gpio). If you do like downloading code you have the option of downloading, cloning, or forking it from my Github repository. As an added bonus, the project code written in Go is also located there.

If you're interested in Go development on a Raspberry Pi you'll need to install the development environment onto the Raspberry Pi. [Here's a simple source](https://www.jeremymorgan.com/tutorials/raspberry-pi/install-go-raspberry-pi/) that explains how to accomplish this. This source is a little dated, but the only significant issue is with the version of Go to install. The source shows installing Go __1.14.4.linux-arm64.tar.gz__ and __1.14.4.linuxarmv6l.tar.gz__. The current versions are __1.17.1.linux-arm64.tar.gz__ and __1.17.1.linuxarmv6l.tar.gz__. For the Raspberry Pi 3B+ the correct choice will be __1.17.1.linuxarmv6l.tar.gz__. The other is intended for 64 bit systems like the Raspberry Pi 4 series.

Finally, I'm assuming a basic knowledge of Linux if you want to veer away from the cookbook style of the Sunfounder docs. For example, I won't be explaining what __root privileges__ are.

## Blinking LED in C

As mentioned above, this article is about the [Blinking LED project](https://docs.sunfounder.com/projects/raphael-kit/en/latest/1.1.1_blinking_led_c.html). The Sunfounder documentation for this project is quite good. You should start with the [Introduction](https://docs.sunfounder.com/projects/raphael-kit/en/latest/introduction.html) and work your way through the following sections up to and including the [Play with C](https://docs.sunfounder.com/projects/raphael-kit/en/latest/play_with_c.html) section.

Since the Sunfounder project documentation is so good, I only have a couple of comments.

### Information that would have been helpful

The documentation for the project states "_Therefore, to turn on an LED, we need to make GPIO17 low (0V) level._". What it doesn't explain is why. The reason is that the 220 Ohm resistor is connected on one side to a 3.3 volt power source and the LED anode (positive terminal) on the other. And GPIO17 is connected to the LED's cathode (negative terminal). LEDs only illuminate when current flows from the anode to the cathode. GPIO pins on the Raspberry Pi, e.g., GPIO17, generally have a default state of HIGH (3.3 volts). With the connection of the 3.3 volt power source to the anode and the 3.3 volt GPIO17 pin on the cathode side, no current will flow through the circuit. So the LED will not turn on. Setting GPIO17 to LOW, 0 volts, allows the current to flow, lighting the LED.

This is a minor point, but the program does not need to be run with root (sudo) privileges. Simply running `./BlinkingLed`, vs. `sudo ./BlinkingLed`, will work. This is not always true for GPIO programs. I'll point that out when running a program that does require root privileges.

## Blinking LED in Go

This version of Blinking LED will work with the same breadboard setup as the C version.

First things first, we need a Go library to drive the GPIO interface. I'm using [go-rpio](https://github.com/stianeikeland/go-rpio) for several reasons:

1. It seems to be in fairly wide use
2. It seems to be fairly complete
3. It's relatively active
4. It comes with example code and good documentation

You can just use the [rpio-go blinker.go example](https://github.com/stianeikeland/go-rpio/blob/master/examples/blinker/blinker.go). It uses `rpio.Pin.Toggle()`. I created a [Github respository](https://github.com/youngkin/gpio) that contains examples in both C and Go. My Go version of `blinker` uses direct writes instead of `rpio.Pin.Toggle`. I thought showing an optional way to do this would be helpful, especially since later projects will use direct writes. My [gpio respository](https://github.com/youngkin/gpio) uses Go's module system which will automatically download the `rpio-go` library when built. Here's my version of `blinker`. I won't be explaining Go sytax as I'm assuming familiarity with Go.

```go
  1 package main
  2
  3 // Run using 'go run blinkingled.go'
  4
  5 // The Go version of this project uses the go-rpio library to
  6 // control the GPIO pins.
  7 //
  8 // The Go module system is used to choose the correct version of
  9 // this library. See the file '../go.mod' for details.
 10
 11 import (
 12     "fmt"
 13     "os"
 14     "time"
 15
 16     "github.com/stianeikeland/go-rpio/v4"
 17 )
 18
 19 func main() {
 20     // Initialize the go-rpio library. By default it uses BCM pin numbering.
 21     if err := rpio.Open(); err != nil {
 22         fmt.Println(err)
 23         os.Exit(1)
 24     }
 25
 26     // Release resources held by the go-rpio library obtained above after
 27     // 'main()' exits.
 28     defer rpio.Close()
 29
 30     // Select the GPIO pin to use, BCM pin 17
 31     pin := rpio.Pin(17)
 32
 33     // Set the pin (BCM pin 17) to OUTPUT mode to allow writes to the pin,
 34     // e.g., set the pin to LOW or HIGH
 35     pin.Output()
 36
 37     for i := 0; i < 5; i++ {
 38         // Setting the GPIO pin to LOW allows current to flow from the power source thru
 39         // the anode to cathode turning on the LED.
 40         pin.Low()
 41         //        pin.Write(rpio.Low)
 42         fmt.Printf("LED on, Pin value should be 0: %d\n", pin.Read())
 43         time.Sleep(time.Millisecond * 500)
 44         pin.High()
 45         //        pin.Write(rpio.High)
 46         fmt.Printf("\tLED off, Pin value should be 1: %d\n", pin.Read())
 47         time.Sleep(time.Millisecond * 500)
 48     }
 49
 50     // Turn off the LED
 51     pin.High()
 52 }
```

Line 16 imports the `rpio-go` library

```go
16     "github.com/stianeikeland/go-rpio/v4"
```

The rest of the program is explained through embedded comments. The program can be run typing `go run blinkingled.go` at the command prompt and hitting enter.

## Summary

This article has shown how to configure a breadboard with an LED that can be controlled by programs written in both C and Go. Feel free to suggest changes or ask questions in the comments section below.