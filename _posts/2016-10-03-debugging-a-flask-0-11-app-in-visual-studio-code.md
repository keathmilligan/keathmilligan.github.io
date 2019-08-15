---
layout: post
title: Debugging a Flask 0.11 app in Visual Studio Code
date: 2016-10-03 19:14:43 -0500
categories: [dev]
tags: [python, flask, debugging]
featured-img: debug_flask_vscode.png
---

![Debugging Flask with VSCode](/assets/images/debug_flask_vscode.png)

*Update July 30, 2017*: a new simplified method that works for both Linux/MacOS and Windows with no changes to your Flask project required.

Visual Studio Code with the the Python extension makes for a great Python development environment – especially if you work on blended Python/Javascript web apps. Here’s how to debug a Flask 0.11.x (or later) app without having to add files or modify your project code. See “Solution 2” here for debugging an older version of Flask.
<!--more-->

First, be sure you have your virtual environment configured in VSCode (you are using a virtual environment, right?). Select Preferences > Workspace Settings from the menu. Your .vscode/settings.json file should have a line something like this:

```
"python.pythonPath": "/Users/kmilligan/.virtualenvs/flask/bin/python"
```
In the .vscode directory, create a file named launch.json if it does not already exist. Change the “Flask” entry as follows:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Flask",
      "type": "python",
      "request": "launch",
      "stopOnEntry": false,
      "pythonPath": "${config:python.pythonPath}",
      "module": "flask.cli",
      "cwd": "${workspaceRoot}",
      "env": {
        "FLASK_APP": "sample",
        "LC_ALL": "en_US.utf-8",
        "LANG": "en_US.utf-8"
      },
      "args": [
        "run",
        "--no-debugger",
        "--no-reload"
      ],
      "envFile": "${workspaceRoot}/.env",
      "debugOptions": [
        "WaitOnAbnormalExit",
        "WaitOnNormalExit",
        "RedirectOutput"
      ]
    }
  ]
}
```

* be sure to change the LC_ALL and LANG values to your something appropriate for your locale.

Notes:

* Previously, you would need to add a “run.py” script to your project for Windows, but that is no longer needed.
* Unless you really need to debug app startup, set stopOnEntry to false so it won’t break in library code.
* Change program to the path to your “flask” command in the virtual environment. You can get this from the command line (with the virtual environment active) by typing “which flask”.
* Change the `FLASK_APP` environment variable to the name of your app’s bootstrap file.
* Set `--no-debugger` to avoid any potential conflicts with the Werkzueg debugger.
* Set `--no-reload`. The Python debugger doesn’t support module reloading.
* Now set some breakpoints and start debugging!

A note about the `LC_ALL` and `LANG` environment variables:

When using Python 3 in some environments without these settings, you might see a message like:

```
RuntimeError: Click will abort further execution because Python 3 was
  configured to use ASCII as encoding for the environment.
```

The problem is that Click (the library used by Flask for command-line processing) isn’t able to determine what locale settings to use. The `LC_ALL` and `LANG` environment variables set the locale explicitly. Refer to the [Click documentation](http://click.pocoo.org/5/python3/) for more info.
