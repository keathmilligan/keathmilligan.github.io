---
layout: post
title: Creating a web development environment for Windows that doesn’t suck
date: 2016-09-29 17:32:23 -0500
categories: [dev]
tags: [windows, vscode, cmder, tools, web, javascript]
featured-img: 
---

While it’s true that many open source projects consider Windows a second class citizen, the reality is that it continues to be the operating system of choice for most business environments and many developers find themselves with no alternative. But this doesn’t mean you have to settle for a poor experience.
<!--more-->

As far as code editing goes, you won’t have a problem. Fortunately, there’s no shortage of great web editors/IDEs to choose from for Windows. [Atom](https://atom.io/), [SublimeText](https://www.sublimetext.com/), [WebStorm](https://www.jetbrains.com/webstorm/) and others both free and non are all great choices each with their own pros and cons. One notable relative newcomer and my personal editor of choice, is [Visual Studio Code](https://code.visualstudio.com/) from Microsoft – a free editor/IDE with a focus on web and script development.

![Visual Studio Code](/assets/images/vscode-old.png)

VSCode strikes a great balance between functionality and performance. It comes with great web development capabilities built-in, including syntax highlighting and codesense support for Javascript and Typescript. A quickly growing growing ecosystem of extensions allow you to add more support as needed (Python, integrated Chrome debugging, linters and more).

Most (all?) web development workflows and build systems are fundamentally built around the command-line. Even though your IDE may have built-in support for webpack or some other build system, eventually you will need to do something at the command-prompt. While you could probably limp along with Windows’ abysmally-antiquated cmd.exe, there are much better alternatives available.

One of the first that comes to mind for many people is Cygwin. Cygwin has been around for ages and provides a very complete Unix-like environment. I don’t actually recommend it as a web development environment though. Cygwn is designed to provide an execution environment that provides as much compatibility with a Unix system as possible, making it easier to port Unix applications to Windows. This introduces a number of differences in the way it deals with paths and other things that can lead to issues when working with non-Cygwin applications. It’s really overkill if you just need a better command prompt and a few Unix-like utilities.

Enter [cmder](http://cmder.net/):
![Cmder](/assets/images/cmder.png)

Cmder is a full-featured terminal-like replacement for the Windows command-prompt that adds tabs, better scrollback, better copy/paste support, Git integration, node integration, Unix utilities and much more. While cmder is actually an aggregation of several utilities like ConEmu and Clink that have been around for a while, it brings them together nicely in a well-polished package that is easy to install and ready to use out of the box. It’s become the first thing I install on any new Windows machine.

### Other Tools

* If you’re going to use VSCode and you will be using Git for source control, then you’ll definitely want to install the official [Git](https://git-scm.com/) Windows package. This will enable VSCode/Git integration and allow you to perform all of the common source control operations from within the editor.
* Sadly [nvm](https://github.com/creationix/nvm) does not support Windows (at least not readily), so if you use NodeJS and you need to work with multiple versions at the same time, check out [nvm-windows](https://github.com/coreybutler/nvm-windows). It’s not a direct port, but serves the same basic purpose.
