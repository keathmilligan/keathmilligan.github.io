---
layout: post
title: "Python pip gotchya: 32-bit wheels selected on 64-bit systems if you have Visual Studio command-line environment enabled"
date: 2020-12-12 20:20:37 -06:00
categories: [dev]
tags: [python, pip, windows, visual studio]
image: /assets/images/python.png
---

![pip win32 bug](/assets/images/pip-win32-bug.png)

This pip gotchya can cause difficult-to-troubleshoot errors.

<!--more-->

[This little pip bug](https://bugs.python.org/issue38989) causes pip to install 32-bit versions of wheels that contain binary components for 64-bit Python installs and virtual environments if you have the Visual Studio command-line build environment active (vcvarsall.bat, vsdevcmd.bat, etc.).

This can lead to import errors and other problems the cause of which may not be immediately obvious. Notably with [Pillow](https://pillow.readthedocs.io/en/stable/), you will get something like:

```python
from PIL import Image
```
```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "C:\Users\keath\Workspace\KMX\test\.virtualenv\lib\site-packages\PIL\Image.py", line 94, in <module>
    from . import _imaging as core
ImportError: cannot import name '_imaging' from 'PIL' (C:\Users\keath\Workspace\KMX\test\.virtualenv\lib\site-packages\PIL\__init__.py)
```

The only fix for now is "don't do that" - make sure you aren't doing pip installs in a Visual Studio build tools environment.
