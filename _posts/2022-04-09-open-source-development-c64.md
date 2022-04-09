---
layout: post
title: C64 open source software development in the 8-bit era
date: 2022-04-09 08:48:42 -05:00
categories: [dev,vintage-computing]
tags: [commodore,c64,vintage,6502,assembly]
image: /assets/images/transactor.jpg
---

The experience of developing and sharing free/open source software for 8-bit machines in the mid-1980s.

<!--more-->

<img src="/assets/images/transactor.jpg" alt="Transactor Magazine" align="right">
In the days of developing software for the Commodore 64 and other 8-bit machines, the web was still a long way off and we didn't have anything like Github for sharing code. We had BBSes of course, but most of those at the time were "warez" sites, dedicated primarily to sharing pirated games. You could find some utilities and applications, but for the most part, when it come to learning about programming and sharing projects, we relied heavily on printed books and magazines like [The Transactor](https://en.wikipedia.org/wiki/The_Transactor).

The Transactor was a key resource for users of the various Commodore machines - PET, CBM, VIC-20, Commodore 64 and 128. Later, the magazine featured content for the Amiga as well. I subscribed to Transactor shortly after getting my first C64 sometime around 1983. The arrival of each month's subscription spawned hours of meticulously typing in source code listings and trying new techniques and algorithms.

In 1987, I submitted one of my own 6502 assembly utilities I had been using for a while and it was published in the March 1988 edition of the magazine. I even got paid for it, making it the first time I officially made money from developing software.

# The article

[![Transactor Article](/assets/images/transactor-article.png)](https://archive.org/details/transactor-magazines-v8-i05/page/n45/mode/2up)

My article was titled "A Better Syntax For Kernel Device I/O" - catchy right?

You can read the archive of the full article at the [Internet Archives](https://archive.org/details/transactor-magazines-v8-i05/page/n45/mode/2up).

## A disk drive usability issue

The problem I was solving had to do with the hardware architecture of the Commodore VIC-20 and Commodore 64 versus the previous generation of Commodore computers like the PET and CBM 80xx machines. Earlier Commodore computers featured an [IEEE-488](https://en.wikipedia.org/wiki/IEEE-488) bus used to interface to peripherals like disk drives and printers. IEEE-488 was fast (for the time), but it was expensive and the connectors and cables were large and bulky. It was not well suited to low-cost machines intended for home use.

<img src="/assets/images/1541.png" align="right">For the VIC-20 and C64, Commodore introduced a new peripheral architecture based on a simplified, lower-cost serial bus. Along with the new computers, they introduced the [1541 disk drive](https://en.wikipedia.org/wiki/Commodore_1541) and the MPS801 printer that used this bus with a few other devices to follow. The serial bus was infamously slow, but it worked and helped keep the cost of the new machines low which was a key factor in making the Commodore 64 [the best selling home computer of all time](https://en.wikipedia.org/wiki/Commodore_64).

<div style="clear:both"></div>
<img src="/assets/images/8250.png" align="left" style="width:200px">For their more expensive business machines, Commodore offered different disk drive models, including dual drive units like the [8050 and 8250](https://en.wikipedia.org/wiki/Commodore_8050). Popular software applications supported the dual drive units and allowed you to effectively use both drives - typically the first drive would be used for the program disk and the second drive would be used to store user data.

<div style="clear:both"></div>
Unfortunately, Commodore never introduced a dual drive version of the 1541. You could add a second 1541 - this involved opening the case of the second drive and cutting a circuit board trace to change its device address from the default (8) so it could be addressed as a separate device. Given that the 1541 cost as much as the C64 itself and modifying circuit boards was not something casual users were comfortable with, dual drive C64 systems never became the norm. This led to few software applications supporting a second drive, which meant lots of disk swapping.

## The program

The solution was implemented as what was referred to as a "wedge" in those days. It was somewhat akin to a plugin or extension in modern software. A wedge would typically intercept some system function and redirect it to alternative implementation that added new functionality or behavior.

Commodore anticipated the need for such applications and included a vector table (or jump table) in RAM to critical system functions like opening files, reading data, writing data, etc. Normally these vectors would point to the standard implementation of these functions in ROM, but a program could change them to point to a custom implementation.

My utility wedge intercepted all of the system functions that worked with the files to add the option of specifying device numbers in the filename itself. The utility would parse out the drive number from the filename and redirect the operation to the correct device on the serial bus. This allowed you to make use of the second drive in any application that prompted for a filename even if that app wasn't designed specifically to support a second drive. This wouldn't work with games that either used or wiped the entire memory of the machine, but it worked with most applications and some games and eliminated a lot of disk swapping.

## Sharing the code

[![6502 PAL assembly source](/assets/images/transactor-article-pal-src.png)](https://archive.org/details/transactor-magazines-v8-i05/page/n47/mode/2up)
*The PAL 6502 assembly source code*

Anyone who wanted to use the program had a couple of options - if they happened to have the PAL assembler, they could type in the assembly source code and assemble it for themselves or they could type in a short BASIC program that would load the binary machine code data directly into memory:

![BASIC loader](/assets/images/transactor-article-loader.png)

This was a pretty standard approach for code you'd find in books and magazines. Obviously, typing in long strings of numbers is error-prone so Transactor implemented a utility (another wedge) called "Verifizer" that displayed a two character checksum in the top-left corner of your display for each line that you type. The correct checksum for each line is included in the printed listing.

# 8-bit development

By the time I put this submission together in 1987, it was the twilight on the 8-bit era. The PC and its clones were rising to dominance, the Mac was gaining a following and many C64 users were upgrading to the Amiga. These machines were still out of reach for many though and the Commodore 64 remained a very popular platform into the early 1990s (for games especially). Like a lot of people, I was shifting my attention to other machines (the PC primarily), but it would be a while before I gave up active development on the C64.

Developing on the C64 could be challenging. Its display was limited to 40 columns by 25 rows, it had an odd keyboard with no number pad and the disk drive was painfully slow. It was also completely unforgiving when it came to coding errors - that usually meant a lock-up that required a power cycle.

## Dual machine development

It wasn't long before I had a second C64. Due the painful experience of coding on a single machine, like a lot of Commodore developers at the time, I built a user-port interface cable and wrote a loader that allowed me to use one machine to edit and compile code and another to run and test it. This saved a lot of time and improved the development workflow immensely. Very similar to [this setup on the Vintage Volts blog](http://www.vintagevolts.com/cross-assembling-on-a-commodore-64/).

### PC Interface

Most commercial software development was not done on the C64 itself. Early on, tools were created that allowed game developers and others to use a PC or another more powerful machine to edit and cross-assemble code to the Commodore. Really advanced developers used 6502 in-circuit emulators that enabled debugging and other sophisticated capabilities. The only people actually editing code on the C64 at the time were poor people like me. Eventually though, I did get a PC clone at some point and while learning DOS and 808x development, I could still go back and write software for the C64 with the help of a cross-assembler that allowed me to edit 6502 assembly source code on the faster PC with an 80-column display and a hard drive and build binaries for the C64.

An ICE was out of the question for me, so to load code onto the C64 from the PC, I built this abomination:

![PC interface](/assets/images/pc-c64-parallel-interface.jpg)

This interface plugged into the C64 user port and linked to the PC with Centronics parallel printer cable. Beyond that, the method to transfer code and send simple commands to the slave machine was essentially the same as the Commodore-to-Commodore approach.

I didn't get a chance to do much more Commodore development, though. Within the next year or so I was to get my first "real" development job for [Dr. Gary Kildall](https://en.wikipedia.org/wiki/Gary_Kildall), founder of Digital Research, Inc. and inventor of the CP/M operating system but that's another story.

![C64](/assets/images/commodore64-2.jpg)

---

If you're interested in learning more about vintage Commodore computers, you don't need to buy one on eBay. You can run software for the C64 and other Commodore machines on your PC using the [VICE]() emulator.

- Full archives of The Transactor are available on the [Internet Archive](https://archive.org/details/transactor-magazines).
- You can find a D64 image of the PAL assembler the [C64 Scene Database](https://archive.org/details/transactor-magazines.)
