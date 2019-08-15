---
layout: post
title: A “shrink-wrap” Python project template and development pattern
date: 2016-10-14 20:14:00 -0500
categories: [dev]
tags: [python]
featured-img: python.png
---

<img src="/assets/images/python.png" align="right">Javascript development certainly has its ups and downs, but one of the good things about its ecosystem is the ability to easily share projects and quickly get other developers up and running. With most projects, it is simply a matter of cloning the repo and running “npm install” and you are ready to go without needing anything else pre-installed on your system besides NodeJS/NPM. With python, it’s usually not quite that simple.

<!--more-->

Fortunately, Python does include tools that can get us closer to the “shrink-wrap” model (not to be confused with the [shrinkwrap package](https://pypi.python.org/pypi/shrinkwrap/) which does something different). I’ve created a Python boiler-plate project ([available on github](https://github.com/keathmilligan/python-boilerplate)) that can be used as the basis for your projects that provides this capability.

The goal is reduce the amount of manual steps the developer needs to take to setup the project while at the same time not requiring a lot of pre-installed tools (other than Python itself) and not limiting their ability to use alternative Python interpreters, etc. Using this template, a project’s quick start steps look something like this:

1. Clone the repo
2. Create a Python virtual environment and activate it
3. Run `pip install -e .[dev]`

And then the developer is ready to run the app or its unit-tests.

The Virtual Environment

Other than the need to create the virtual environment, the process is very similar to getting started with a Javascript package. As with development Javascript/NodeJS development, the best practice for Python development is to install the packages needed for a given project into a dedicated and isolated execution environment. A Python [virtual environment](https://docs.python.org/3/library/venv.html) works much like the “node_modules” directory in a Javascript project – it contains isolated installations of the packages your project depends on.

While a virtual environment could be shared between many projects, I prefer to create one dedicated to each project in the project’s root directory itself. For instance:

`pyvenv .virtualenv`

Depending on your Python installation, you might not have a “pyvenv” command. In this case, run:

`python -m venv .virtualenv`

Then activate it using:

`source .virtualenv/bin/activate`

(Linux/Unix/macOS)

or

`.virtualenv\Scripts\activate.bat`

(Windows)

Keeping the virtual environment in the project directory and dedicated to that specific projects helps us keep track of it and avoids conflicts with other projects. Using this project template, you can easily remove and rebuild it. Be sure to add “.virtualenv” to your .gitignore file – you don’t want to check it in.

Project Structure

Let’s look at the project layout. It closely follows what [Kenneth Reitz](http://www.kennethreitz.org/essays/repository-structure-and-python) (author of the popular [Requests](http://docs.python-requests.org/en/master/) package) recommends:

```
sample/
sample/__init__.py
sample/main.py
docs/
docs/conf.py
docs/index.rst
tests/
tests/test_sample.py
README.md
LICENSE
.editorconfig
.flake8
.gitignore
setup.py
```

Rename this to whatever you want. This directory contains your actual application or library code. Other than __init__.py, the contents of this directory are up to you.

If you are writing a small program or library with only a single module, replace the “sample” directory with a single python module (e.g., “sample.py”).

You can create additional top-level packages if needed. If you do this, these need to be listed in setup.py (see below). However, it is probably best to organize additional packages under just the one top-level package.

`docs/`

This directory will contain your Sphinx documentation sources. It includes a skeleton index and configuration file.

`tests/`

Unit-tests. Includes a sample unit test module using pytest.

`README.md`

The obligatory README file. You can change this to a restructured text file if you like (.rst).

`LICENSE`

Replace the text of this file with your project’s license information.

`.editorconfig` (optional)

`.editconfig` is a standard editor/IDE configuration file that allows you to specify certain editor configurations that should be set one way or another for your project (for example tabs vs. space or line-endings). This allows you to better enforce coding conventions and maintain consistency among your developers. You can delete this file if you don’t wish to use it.

`.gitignore`

Don’t pollute the repo

`.flake8` (optional)

Customize as needed to enforce any/all PEP8 and other Flake8 linter warnings.

`setup.py`

This is the setuptools configuration script.

Edit the definitions at the beginning of the file to suit your project:

```
NAME = 'python-boilerplate'
VERSION = '0.1'
AUTHOR = 'Keath Milligan'
REQUIRED_PYTHON_VERSION = (2,7)
PACKAGES = ['sample']
INSTALL_DEPENDENCIES = []
SETUP_DEPENDENCIES = [
    'Sphinx'
]
TEST_DEPENDENCIES = [
    'pytest'
]
EXTRA_DEPENDENCIES = {
    'dev': [
        'pytest',
        'flake8',
        'Sphinx'
    ]
}
```

Here is a description of each of these variables:
* `NAME` – this project name
* `VERSION` – project version
* `AUTHOR` – you
* `REQUIRED_PYTHON_VERSION` – a tuple that defines the minimum required Python version (e.g., “(2,7)” for Python 2.7.x)
* `PACKAGES` – your project’s top-level packages
* `INSTALL_DEPENDENCIES` – additional packages required by this project at runtime if any
* `SETUP_DEPENDENCIES` – packages temporarily needed to run the setup script
* `TEST_DEPENDENCIES` – packages needed to run unit-tests
* `EXTRA_DEPENDENCIES[‘dev’]` – defines packages needed for development (these will not be required to install the package in production). See setuptools documentation for additional information on “extra” requirements.

Refer to the [github repo](https://github.com/keathmilligan/python-boilerplate) for more information on getting started with the template.
