---
layout: post
title: 5 templates for new Python projects
date: 2022-03-27 20:12:26 -05:00
categories: [dev]
tags: [python,flask,microservices,cli,click,pytest,pylint,pipenv]
image: /assets/images/python-templates.png
---

![Python Templates](/assets/images/python-templates.png)

Kick-start your next Python project with the right template and establish good coding practices from day 1.
<!--more-->

Using a template can help you quickly bootstrap a new project by eliminating the need to write common boilerplate framework/language setup from scratch. Templates are also a good place to include starting points for good practices like linting, unit-testing, documentation, formatting and shared standard configuration. Here are 5 templates that I've evolved for my Python projects that you might find useful for yours.

## Using the templates

Each of these are available on Github as a [template repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template). This makes it easy to create a new project by selecting the template you want to use and clicking the **Use this template** button to create a new repository using the template as a basis. Note that this is not the same as forking a repository (but you can still do that too).

Once you've created your new repository, you can tailor it to your needs by adding your own functionality and stripping out anything you don't need.

## About the templates

I've tried to balance each of these templates by providing enough of a starting point to be useful while not throwing in a bunch you probably won't need. All of the templates include a few standard items that can promote consistency and better coding practices, including:

* An [.editorconfig](https://editorconfig.org/) file. This is a configuration file that is recognized by most popular IDEs and text editors that standardizes things like end-of-line handling, margins, tab styles and other common editor settings. By including this file, everyone who edits the code will automatically inherit the same settings to help promote consistency.
* A [.gitignore](https://git-scm.com/docs/gitignore) file to avoid adding generated and other files to the repository that should not be checked in.
* [PyLint](https://pylint.org/) and a .pylintrc file to encourage clean Python code.
* [Black](https://black.readthedocs.io/en/stable/) for standardized formatting.
* [PyTest](https://docs.pytest.org/en/7.1.x/) and sample unit-tests.

### Packaging and dependency management

The templates use one of two approaches for packaging and dependency management:

* [setuptools](https://setuptools.pypa.io/en/latest/) - for packages that can be published and installed with `pip`.
* [pipenv](https://pipenv.pypa.io/en/latest/) - for applications, utilities and other types of programs.

`setuptools` is best suited for shared package projects such as libraries or frameworks that you intend to publish on [PyPI](https://pypi.org/) or another artifact repository.

`pipenv` is best suited for other types of projects such applications, command-line utilities and scripts that will not be published. It automatically creates your project's virtual environment and tracks installed dependencies so that your app can be installed elsewhere reliably.

See [Pipfile vs. setup.py](https://pipenv.pypa.io/en/latest/advanced/#pipfile-vs-setuppy) for more discussion on these two approaches.

### Documentation

`setuptools`-based templates include a [Sphinx](https://www.sphinx-doc.org/en/master/) documentation scaffold for generating formal package docs.

`pipenv`-based templates simply use markdown for documentation that can be viewed in Github or Gitlab.

## The templates

### 1) Python App Template

![Python App Template](/assets/images/python-app-template.png)

**Github: [https://github.com/keathmilligan/python-app-template](https://github.com/keathmilligan/python-app-template)**

This is a general-purpose application template that features:

* Predictable installation and package management with [Pipenv](https://pipenv.pypa.io/en/latest/).
* PyTest unit-test support
* PyLint and Black
* An [.editorconfig](http://editorconfig.org/) file
* Command-line argument parsing and color output support with [Click](https://click.palletsprojects.com/en/8.0.x/)

### 2) Flask Quick Start Template

![Flask Template](/assets/images/flask-template.png)

**Github: [https://github.com/keathmilligan/flask-quickstart](https://github.com/keathmilligan/flask-quickstart)**

A template for creating a stand-alone Flask-based web application that serves HTML and/or RESTful API endpoints. It features:

* Predictable installation and package management with [Pipenv](https://pipenv.pypa.io/en/latest/).
* Flask 2.x and blueprints
* Jinja2 HTML templates
* SQLAlchemy database models
* Marshmallow for object marshalling
* JSON Web Token (JWT) authentication
* PyTest unit-tests
* Serve in production with waitress
* Pylint and Black

### 3) Flask Service Quick Start Template

![Flask Service Template](/assets/images/flask-service-template.png)

**Github: [https://github.com/keathmilligan/flask-service](https://github.com/keathmilligan/flask-service)**

A template for creating Python microservices based on Flask. Featuring:

* Predictable installation and package management with [Pipenv](https://pipenv.pypa.io/en/latest/).
* Flask 2.x and blueprints
* JSON Web Token (JWT) authentication using public key token validation
* PyTest unit-tests
* Serve in production with waitress
* Pylint and Black
* Sample Docker container

### 4) Python Package Boilerplate

**Github: [https://github.com/keathmilligan/python-boilerplate](https://github.com/keathmilligan/python-boilerplate)**

Boilerplate template for creating installable Python packages. Featuring:

* `setuptools` packaging
* Support for installing development dependencies through setup.py
* PyTest unit-test support
* PyLint
* An [.editorconfig](http://editorconfig.org/) file
* Sphinx documentation generation
* `pyproject.toml` PEP 517/518 support

### 5) Python CLI Package Boilerplate

**Github: [https://github.com/keathmilligan/python-boilerplate-cli](https://github.com/keathmilligan/python-boilerplate-cli)**

A variation of the Python Package Boilerplate template that includes a command-line interface and color output using [Click](https://click.palletsprojects.com/en/8.0.x/).
