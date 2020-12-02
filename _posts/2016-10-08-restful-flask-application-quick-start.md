---
layout: post
title: RESTful Flask application quick-start
date: 2016-10-08 22:14:10 -0500
categories: [dev]
tags: [flask, rest, python, sqlalchemy, jwt]
image: /assets/images/flask_icon.png
---

With the rise of the [single-page application](https://en.wikipedia.org/wiki/Single-page_application) (SPA) web front-ends and mobile apps, the backend of many web applications is a collection of RESTful interfaces that provide JSON data rather than generating HTML. The rendering is up to the client side. While there are some drawbacks to this approach (heavier client, slower initial page loads, etc.), there are also a number of advantages, not the least of which is better [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) since the front-end and back-end code are completely independent of one another.
<!--more-->

This [Flask quick-start template](https://github.com/keathmilligan/flask-quickstart) provides a basis for creating such a “REST-only” server. It can be used to serve mobile apps or SPA front-ends built with [Angular](https://angularjs.org/) or some other framework.

### About the Template:

* Written using Python 3.x and based on the [Python Flask microframework](http://flask.pocoo.org/). This template uses the latest version of Flask which features improved command-line interface support and [many other enhancements](/whats-new-in-flask-0-11-x).
* The server only provides RESTful interfaces and generates no HTML at all.
* Supports [JSON Web Token](https://jwt.io/) (JWT) authentication.
* Database model support with [SQLAlchemy](http://www.sqlalchemy.org/). Automatic marshaling of objects is provided using [Marshmallow](https://marshmallow.readthedocs.io/en/latest/).
* Unit-testing with [PyTest](http://docs.pytest.org/en/latest/).
* [Setuptools](https://setuptools.readthedocs.io/en/latest/) installation script.

The template is available on [Github](https://github.com/keathmilligan/flask-quickstart). Check out the README file to get started.
