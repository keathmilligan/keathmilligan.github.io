---
layout: post
title: Cross-platform native 3D rendering engine for Windows, Linux, macOS, iOS and Android.
date: 2016-10-13 00:00:00 -0500
categories: [dev]
tags: [opengl, graphics, c, windows, linux, macos, ios, android]
featured-img: sg3-legacy.png
---

![sg3-legacy](/assets/images/sg3-legacy.png)

This is an old project that provides OpenGL scene and object management functionality. The goal was to provide a native core graphics library that would run on Windows, Linux, macOS, Android and iOS. It is written in C and provides compatibility wrappers around the OpenGL APIs to account for platform differences.
<!--more-->

The library includes:

* 3D math functions (written in C) - vectors, matrix manipulation, quaternians, etc.
* 3D model loading
* Camera positioning and manipulation
* 3D object management
* Skyboxes
* Lensflare effects
* 2D billboards
* HUD overlays (text images, shapes/lines, etc.)

### Source Code

<https://github.com/keathmilligan/sg3-legacy>
