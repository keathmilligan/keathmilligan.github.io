---
layout: post
title: Debugging pytest in VSCode (without adding files to your project)
date: 2016-10-05 22:34:10 -0500
categories: [dev]
tags: [python, pytest, debugging, vscode]
image: /assets/images/
---

For debugging pytest executions, the official VSCode Python extension documentation recommends creating an additional file in your project and setting up a launcher to start the debugger against it. While this is simple, I really don’t like having to modify my project’s code or add source files just to satisfy my editor/IDE.
<!--more-->

*Update July 29, 2017: updated to work on Windows and simplify configuration.*

So, to debug pytest without having to create an additional source file, setup a launcher configuration with the “program” option pointing to the “pytest” script itself, for instance:

```json
JavaScript
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "PyTest",
            "type": "python",
            "request": "launch",
            "stopOnEntry": false,
            "pythonPath": "${config:python.pythonPath}",
            "module": "pytest",
            "args": [
                "-sv"
            ],
            "cwd": "${workspaceRoot}",
            "env": {},
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

In my case, I have a virtual environment setup in the root of the project (.virtualenv). Alter this path as appropriate for the location of your virtual environment.

Also, be sure to set “cwd” to the root of your project or where ever you would normally run pytest from so it will properly discover your tests.
