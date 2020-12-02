---
layout: post
title: Python, LDAP and macOS
date: 2016-10-11 20:13:00 -0500
categories: [dev]
tags: [python, ldap, macos]
image: /assets/images/
---


The Python LDAP packages (python-ldap and pyldap) mostly work on macOS, but if you try to use some options and APIs, you will run into trouble.
<!--more-->

For example:

```
Python 3.5.2 (default, Sep 15 2016, 07:38:42) 
[GCC 4.2.1 Compatible Apple LLVM 7.3.0 (clang-703.0.31)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import ldap
>>> ldap.set_option(ldap.OPT_X_TLS_CACERTFILE, 'cacert.pem')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/kmilligan/workspace/python-ldap-test-2/.virtualenv/lib/python3.5/site-packages/ldap/functions.py", line 137, in set_option
    return _ldap_function_call(None,_ldap.set_option,option,invalue)
  File "/Users/kmilligan/workspace/python-ldap-test-2/.virtualenv/lib/python3.5/site-packages/ldap/functions.py", line 64, in _ldap_function_call
    result = func(*args,**kwargs)
ValueError: option error
>>> 
```

The reason for this is that macOS ships with an outdated version of the openldap libraries. To fix it, you will need to build and install a newer version of openldap on your system (as an alternative) and build a version of the Python LDAP libraries that are linked to it.

This is covered for previous versions of macOS in various StackOverflow questions and elsewhere (see links below), but some of the answers may lead you down a bit of a rabbit hole and have you taking unnecessary steps. The steps below will get you going with Python & LDAP with a minimum of hassle.

What you will need:

* [Homebrew](http://brew.sh/)
* Python 3 (or 2.7.x) installed with brew. For this example, I am using Python 3.5.2, but the same steps should work with 2.7.x. Don’t try to do this with macOS’s stock Python.

I’ll be using the pyldap package for this instead of python-ldap. Pyldap is a fork of python-ldap and provides a compatible API with some additional enhancements. The same basic process should work for python-ldap though.

Install OpenLDAP 2.4.x with Homebrew:

`brew install openldap`

Homebrew will install this as a “keg-only” package so it does not conflict with the system version of OpenLDAP.

Clone or download the [pyldap sources](https://github.com/pyldap/pyldap) from Github. E.g.:

`git clone git@github.com:pyldap/pyldap.git`

cd into the pyldap directory and create a virtual environment (with the version of Python you want to use). For example:

`pyvenv .virtualenv`

or:

`python3 -m venv .virtualenv`

Activate this virtual environment:

`source .virtualenv/bin/activate`

Edit the setup.cfg file and add the newer OpenLDAP library and include paths:

```
# Define extra include and library dirs if needed
library_dirs = /usr/local/opt/openldap/lib /usr/lib /usr/lib64 /usr/local/lib /usr/local/lib64
include_dirs = /usr/local/opt/openldap/include /usr/include /usr/include/sasl /usr/local/include /usr/local/include/sasl
```

Build the package:

`python setup.py build`

At this point, this directory will now contain an updated version of pyldap that you can install in your project’s virtual environment. Just specify the path to it with the pip command:

`pip install <path-to-pyldap-sources>`

(with the target Python environment active)

(Optional) Build a Wheel

If you don’t want to keep the LDAP package source code around, you can build a wheel. A Python wheel is a self-contained installation file that includes everything needed for the package you want to install. You could for example, keep this wheel with your project as a resource.

To do this, install the “wheel” package in your virtual environment:

`pip install wheel`

Then build the wheel with:

`python setup bdist_wheel`

Look for a “.whl” file in the “dist” directory. You can install this wheel with pip. For example:

`pip install pyldap-2.4.25.1-cp35-cp35m-macosx_10_11_x86_64.whl`

(in the target virtual environment)
 
When you are done building the new pyldap library, remember to deactivate the virtual environment:

`deactivate`
 