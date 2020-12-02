---
layout: post
title: 'Build Notes: Android AOSP for Raspberry Pi 3B+ with 7" LCD Touchpanel'
date: 2019-09-14 23:02:26 -0500
categories: [tech]
tags: [android aosp raspberry-pi]
image: /assets/images/aosp-rpi3-touch.jpg
---

![Android AOSP on Raspberry Pi 3B+](/assets/images/aosp-rpi3-touch.jpg)

How get the 7" HDMI touch display working with [brobwind's RPI 3 Android Pie build](https://github.com/brobwind/pie-device-brobwind-rpi3).
<!--more-->

# Build Notes

Before building brobwind's AOSP image, you might want to check your system for a few issues not covered in the standard Android build environment setup guide:

You will need a few additional packages to complete the build and prepare your SD:
```
sudo apt-get install python-pip libyaml-dev gettext libgettextpo-dev gdisk 
sudo pip install prettytable Mako pyaml dateutils --upgrade
```

Check the version of `dosfstools` on your system if you have it installed. On Debian 9 and other later Debian-based distributions, the standard version of `dosfstools` available via apt (4.x) is incompatible with the Android build and you will need to replace it with an older version:

```
git clone git://github.com/dosfstools/dosfstools.git
cd dosfstools/
git checkout v3.0.28
make
sudo apt install checkinstall
sudo mkdir /usr/local/share/doc
sudo checkinstall --fstrans=no
sudo apt install ./dosfstools_20190901-1_amd64.deb
```

# Flashing

If you use `fastboot` to flash invidual partition images and you are getting errors when flashing large images, try adding the `-S` option. For example:

```
fastboot -s udp:192.168.0.107 -S 114512K flash system_a out/target/product/rpi3/system.img
```

# Testing

This Android build is hard-coded to 720p resolution (1280x720). Your display must be able to accept this resolution or it will not work.

# Booting with the 7" LCD Display

The display used here is a 7" 800x480 capacitive touch display sold under various brand names. This particular unit is a [Velleman VMP402](https://www.velleman.eu/products/view/?id=439070) available from many sources. There are other similar displays in different sizes and resolutions - it may possible to modify these instructions to get those to work as well.

Also note that this display draws power from the RPi's USB port, so be sure your power supply is up to the task.

## Method 1 - Rebuild the rpiboot image

Edit `device/brobwind/rpi3/boot/config.txt` and add the following lines:

```
hdmi_force_hotplug=1
max_usb_current=1
hdmi_group=2
hdmi_mode=1
hdmi_mode=87
hdmi_cvt 800 480 60 6 0 0 0
dtoverlay=ads7846,cs=1,penirq=25,penirq_pull=2,speed=50000,keep_vref_on=0,swapxy=0,pmax=255,xohms=150,xmin=200,xmax=3900,ymin=200,ymax=3900
```

`hdmi_force_hotplug=1` is key, otherwise the display will not activate.

Rebuild the images with `m -j` from the top of the build tree.

## Method 2 - Edit the images

If you don't want to rebuild the images, you can modify the rpiboot image directly:

```
sudo mkdir /mnt/rpiboot
sudo mount out/target/product/rpi3/rpiboot.img /mnt/rpiboot
```

Edit `/mnt/rpiboot/config.txt` (as root) and add the lines above.

```
sudo umount /mnt/rpiboot
```

Reflash the image with fastboot or some other method.

![RPI Android AOSP](/assets/images/rpi-android.gif)
