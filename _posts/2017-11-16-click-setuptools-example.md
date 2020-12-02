---
layout: post
title: Click setup tools example
date: 2017-11-16 00:00:00 -0500
categories: [dev]
tags: [python]
image: /assets/images/
---

The [Click](https://click.palletsprojects.com/en/7.x/) Python library supports creating command-line applications written in Python with robust argument and option parsing and many other useful features tha go above and beyond what is offered by other packages that attempt to fill the same need. This is a very simple example of a Python command-line utility using Click with a setuptools-based installer.
<!--more-->

The goal here is to create a command-line tool written in Python that can be installed on a user's system using pip then invoked by name rather than running it as a Python script. That is:

`myawesome-app [options] <args>`

Instead of:

`python <path-to-script>/myawesome-app.py [options] <args>`

_These examples assume Python 3.x._

# Create the App

Crate a new directory for your app project.

Create a subdirectory for your app's main package called `mypkg` and in this directory, create an `__init__.py` file:

```python
import click

@click.command()
def cli():
    print('Hello World!')
```

# Create the setup.py file

Now go back to the root of the project and create a file called `setup.py`:

```python
from setuptools import setup, find_packages

setup(
    name='vscode-python-test',
    version='0.1.0',
    packages=find_packages(exclude=['tests']),
    install_requires=['click'],
    entry_points="""
        [console_scripts]
        myscript=mypkg:cli
    """
)
```

This tells setuptools about your application's dependencies (click) and other back information needed to create an installable package. They key for a command-line utility is declaring an entry point in the `[console_scripts]` section. In this case, the `cli` object that is created the app's main package.

Setuptools will automatically create a wrapper script called "myscript" that correctly invokes your app.

# Create a virtualenv for testing

For testing, you should also create and activate [a virtual environment](https://docs.python.org/3/tutorial/venv.html) so you aren't installing into your global Python:

`python -m venv .virtualenv`

This creates a virtual environment in the project directory. You could place it elsewhere, this is simply a convention I like to use.

Then activate the virtualenv with

`source .virtualenv/bin/activate`  (Linux/MacOS/Unix)

or

`.virtualenv\Scripts\activate.bat`  (Windows)

# Install & Test

Now install your app and its dependencies in your local virtual environment with:

`pip install -e .`

Now the "mympkg" command will be available in your path:

```
$ myscript
Hello, World!
```

This extremely minimal setuptools configuration provides only the bare minimum to install a package. A configuration for a real application will likely require significantly more. See [A "shrink-wrap" Python project template](a-shrink-wrap-python-project-template-and-development-pattern).

### Source Code

<https://github.com/keathmilligan/click-setuptools-example>
