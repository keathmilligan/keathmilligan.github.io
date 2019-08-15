---
layout: post
title: Vintage C64 computing, OpenCBM and Raspberry Pi
date: 2017-04-30 23:17:00 -0500
categories: [tech, vintage-computing]
tags: [commodore, raspberry-pi]
featured-img: commodore64.jpg
---

![Commodore 64 BSS](/assets/images/commodore64.jpg)

Get your C64 online with the help of the Raspberry Pi, OpenCBM and the ZoomFloppy.

Getting your C64 online and the OpenCBM working with Raspberry Pi is fairly straightforward, but there is a lot outdated info out there that might make it a bit confusing. This guide rolls up the basics in one place.
<!--more-->

# Setting up the Raspberry Pi

This guide assumes a Raspberry Pi 3 running the standard Raspbian image.

> ***Unplug the ZoomFloppy USB cable if it is plugged in***

# Required Software

You will need the following packages:

* libusb-dev
* libncurses5-dev
* tcpser

Install these using `sudo apt-get install <package>`.

## Build OpenCBM

<img src="/assets/images/cbmlogo.png" align="right">Unfortunately, there is no pre-built Raspbian package for OpenCBM, so you will need to build it.

Download the [OpenCBM sources](http://www.trikaliotis.net/Download/opencbm-0.4.99.98/opencbm-0.4.99.98.tar.bz2) and extract into a work directory.

Build with:

`make -f LINUX/Makefile opencbm plugin-xum1541`

This will build only the parts you will actually use with the Raspberry Pi.

Now install with:

`make -f LINUX/Makefile install install-plugin-xum1541`

# Test

Now plug in the ZoomFloppy USB cable and turn on your 1541 (or other Commodore floppy drive). Run the following command:

`sudo cbmctrl detect`

If all is working, you should see your drive detected. For example:

```
pi@raspberrypi:~ $ sudo cbmctrl detect
 8: 1540 or 1541 
```

See http://spiro.trikaliotis.net/opencbm-alpha for more info.

# Getting Online

Using tcpser, you can connect your C64 to a variety of BBSes still up and running on the internet and accessible via telnet.

## Required Hardware

You will need to build or buy a Commodore user-port to USB interface. There are several options for this. One simple option is the [“Strike-Link”](https://1200baud.wordpress.com/2012/10/14/build-your-own-c64-2400-baud-usb-device-for-less-than-15/) cable using the CH430G device. Another simple (but slow) option is the [Sparkfun RS-232 level shifter](https://www.sparkfun.com/products/449) in conjunction with an RS-232 to USB adapter.

Either way you go, you will need a user port edge connector. You can get these on EBay. The wiring for the Sparkfun level shifter is pretty simple:

![Sparkfun C64 Cable](/assets/images/sparkfun-cbm.jpg)

1. Connect user port pin 2 to VCC
2. Connect user port pins A and N to GND
3. Connect user port pins B and C to TX
4. Connect user port pin M to RX

## Required Software

You will need a terminal program on the C64. [Striketerm](https://1200baud.wordpress.com/category/striketerm/) (which is a modified version of Novaterm) works well and supports Commodore color graphics. You can [download it here](http://csdb.dk/release/?id=130807).

Depending on what cable you have, you may need to tweak tcpser’s settings. For the Sparkfun cable above, the following settings work well for me:

`sudo tcpser -s 2400 -d /dev/ttyUSB0 -tsSiI -l 7 -i "&k0"`

In the terminal, you can “dial” an internet-connected BBS by entering its address:port in place of the phone number. Striketerm’s phonebook supports telnet BBSes natively.

