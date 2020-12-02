---
layout: post
title: "DevMock: Network Server Mocking Tool"
date: 2019-08-26 18:23:07 -0500
categories: [dev, testing, devops]
tags: [devops, testing, mocking, java, http, ssh, telnet, sockets]
image: /assets/images/devmock.png
---

<img src="/assets/images/devmock.png" align="right">**DevMock** is a network test tool that allows you to create "mock" servers with HTTP/HTTPS, SSH or telnet/socket interfaces that will provide responses to requests or commands that you define.
<!--more-->

DevMock is inspired by (and makes use of) [WireMock](http://wiremock.org/) which allows you to mock HTTP-based APIs. It adds to this by providing a similar ability to mock commands executed over SSH, telnet and plain socket interfaces and allows to you to define multiple servers with any combination of interfaces. It can be useful for testing in a variety of scenarios that involve working with multiple servers or network devices.

# Configuration

Devmock uses a JSON configuration file. by default, it will look for a file named "devmock.json" in the current directory. This file defines your mock servers and their interfaces. For example:

```json
{
  "journalDir": "journal",
  "devices": [
    {
      "name": "device1",
      "ip": "127.0.0.1",
      "info": {
        "os_type": "Linux",
        "version": "4.9.0-3-amd64 #1 SMP Debian 4.9.30-2+deb9u5 (2017-09-19) x86_64 GNU/Linux"
      },
      "interfaces": [
        {
          "type": "web",
          "serverName": "Mock Server",
          "http": 8080,
          "https": 8443,
          "mappingsDir": "mappings",
          "mappings": [
            "favicon.json",
            "test.json"
          ]
        },
        {
          "type": "ssh",
          "port": 2022,
          "mappingsDir": "mappings",
          "mappings": [
            "cli.json"
          ]
        },
        {
          "type": "tty",
          "port": 2023,
          "mappingsDir": "mappings",
          "mappings": [
            "cli.json"
          ]
        }
      ]
    }
  ]
}
```

This example defines a single server that binds to `127.0.0.1` and provides an HTTP/S, SSH and Telnet interfaces.

# Mappings

The `mappings` sections in the interface definitions specify the requests or commands that will be mocked.

## Mocking HTTP/S Requests

HTTP/S support is provided by WireMock. For example, `test.json` in the `mappings` directory defines a simple response to a RESTful API GET request:

```json
{
  "request": {
    "url": "/api/test",
    "method": "GET"
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json"
    },
    "jsonBody": {
      "result": {
        "field1": 1,
        "field2": 2
      }
    }
  }
}
```

For more information on mocking HTTP/S request/responses, refer to the [WireMock documentation](http://wiremock.org/docs/).

## Mocking Command-Line Interfaces

DevMock provides the ability to mock simple command/response scenarios with SSH, Telnet and plain sockets. For an example, see `cli.json` in the `mappings` directory:

```json
{
  "welcomeFile": "welcome.txt",
  "eol": "\r\n",
  "prompt": "$ ",
  "commands": [
    {
      "command": "test",
      "response": "This is a test command"
    },
    {
      "command": "uname -a",
      "response": "{{info.os_type}} {{name}} {{info.version}}"
    }
  ],
  "notFound": "-bash: {{command}}: command not found"
}
```

# Multiple Mock Servers

DevMock can run multiple mock servers at once, each bound to a different IP address. You can configure additional IP addresses on your system and configure mock servers on each address. For example on MacOS, you can add a new loopback IP address with:

```
sudo ifconfig lo0 alias 127.0.0.2 up
```

Then you can create a new device entry in the `devmock.json` file bound to this address.


### Source Code

<https://github.com/keathmilligan/devmock>
