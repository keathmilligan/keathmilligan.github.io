---
layout: post
title: Python pip gotchya: 32-bit wheels selected on 64-bit systems if you have Visual Studio command-line environment enabled
date: 2020-12-12 20:20:37 -06:00
categories: [dev]
tags: [python, pip, windows, visual studio]
image: /assets/images/python.png
---

This pip gotchya can cause difficult-to-troubleshoot errors.
<!--more-->

[This little pip bug](https://bugs.python.org/issue38989) causes pip to install 32-bit versions of wheels that contain binary components for 64-bit Python installs and virtual environments if you have the Visual Studio command-line build environment active (vcvarsall.bat, vsdevcmd.bat, etc.). This can lead to import errors and other problems the cause of which may not be immediately obvious. 
