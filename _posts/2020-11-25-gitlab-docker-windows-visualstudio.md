---
layout: post
title: Deploying and using Windows containers with Gitlab CI
date: 2020-12-03 18:44:35 -06:00
categories: [dev]
tags: [gitlab, docker, windows, visual studio, ci]
image: /assets/images/gitlab-docker-windows-visualstudio.png
---

![gitlab-docker-windows-visualstudio](/assets/images/gitlab-docker-windows-visualstudio-row.svg)

Configuring a Linux-based Gitlab runner to support Docker-based builds is relatively straight-forward and well-documented. Doing the same with Windows is a bit less so. Here's how to configure a Windows Server 2019 VM to host Docker-based builds with Visual Studio or other Windows-based tools.

<!--more-->

# Plan Your Setup

Before we begin, it's worth spending a little time to plan your setup - there are a couple of issues that may impact how you proceed:

## Container Compatibility

The first issue is Windows [container compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-20H2%2Cwindows-10-20H2). If you're just getting started with Windows containers, you might be surprised to get an error that looks something like this when you try to run a Windows container:

![Windows container verson error](/assets/images/windows-container-version-error.png)

This is because a Windows container host can only run a container image that is of the _same OS version or older_. For example, if you are running Windows 10 or Server version 1909, you will not be able to run a version 2004 or newer image.

Fortunately, Microsoft provides most of their base container images in a variety of versions, but you should research which base images you are going to need and plan for a container host that will support them.

## Process Isolation vs. Hyper-V Isolation

Windows containers support two [isolation modes](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container) - **process isolation**, which is very similar to how containers normally work in a Linux environment and **Hyper-V isolation** that runs the container in a highly-optimized virtual machine using Windows Hyper-V. There are pros and cons to both:

* Process isolation is the "traditional" container isolation model and is more efficient because the container host and container both share the same OS/kernel.
* In order to use process isolation, **_the host and container image must be the same version_**. Otherwise you will get the version mismatch error above.
* While Hyper-V isolation offers better security and a wider range of compatibility, containers running Hyper-V isolation will be somewhat slower and consume more resources than process-isolation containers.

## Gitlab / Gitlab Runner Version

When Gitlab runs a Docker-based CI job, it uses a "helper" Docker image to bootstrap the execution environmnent for the build. In the case of Windows and due to the container compatibilty constraints above, this helper image must match the Windows version of your container. This means the Gitlab team has to create an updated helper image for each new version of Windows and this helper image is included in the Gitlab release.

As of this blog post date, 13.6 is the latest official release of Gitlab and it supports Windows hosts/containers up to version 1909. [Check the documentation](https://docs.gitlab.com/13.6/runner/executors/docker.html) for your version of Gitlab to get the latest supported Windows container version.

> Normally you will be limited to whatever version of Windows container host your version of Gitlab supports. However, if you _really_ want to run later container versions, there is a simple, but somewhat ugly work-around. See [this issue comment](https://gitlab.com/gitlab-org/gitlab-runner/-/issues/26420#note_374547809) by Cédric Menzi. Basically, Cédric's work-around allows you to use a Windows 2004 (or later) container host by fooling Gitlab it into thinking the server is an older version. Ugly but effective.
{: .danger }

# Server Setup

## Virtual/Physical Machine Configuration

If you plan to use Hyper-V isolation, you will need to enable virtualization features in your BIOS settings.

## OS Install

I'm using a headless Windows Server 2019 VM. Windows 10 will also work but there are Microsoft [licensing restrictions](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/faq) to be aware of for production use.

> We're going to be doing everything from the **command-line** using Powershell. _Be sure your command prompt window or remote session is running as Adminstrator_.
{: .terminal }

## Install OpenSSH

Unless you are working with a physical machine or prefer or have some other remote access solution already in place, you will need some method for remotely Administering your server to install and configure the runner. While you can use [Powershell remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands), OpenSSH is simpler and more flexible for me.

Follow [these instructions](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse) to install the OpenSSH option on your server and perform the initial configuration.

### Public key auth for Adminstrator logins

If you don't want to enter the Administrator password each time you open an SSH session to the server, add your SSH public key to `C:\ProgramData\ssh\administrators_authorized_keys` (note that `ProgramData` is a hidden directory). If you login as another user, you will add your public key to `%USERPROFILE%\.ssh\authorized_keys`.

## Install Docker

For a server installation, you will not be installing the regular Docker Desktop release for docker.com, instead follow the Microsoft instructions for [adding the Docker Windows component](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/set-up-environment?tabs=Windows-Server) from the command-line.

> If you want to test out your Docker installation, remember the constraints above - by default, Docker for Windows will attempt to run containers using process isolation, which means that the container image version must match your server version.
{: .warning }

### Install Hyper-V

Unless you plan to only run containers with process isolation, you will need to install the Hyper-V service. Check to see if it is already installed with:

```
Get-WindowsFeature -Name Hyper-V
```

If it is not installed, install it with:

```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

And reboot.

### Enable Hyper-V isolation in Docker config

If you plan to use Hyper-V isolation, you will need to tell Docker to use it by default instead of process isolation. Edit (or create if it does not exist) `C:\ProgramData\docker\config\daemon.json` and add:

```json
{
    "exec-opts": [
        "isolation=hyperv"
    ]
}
```

Restart the Docker service with:

```
restart-service docker
```

## Install & Register Gitlab Runner

Now you are ready to [install and register the Gitlab runner](https://docs.gitlab.com/runner/install/). Be sure to select the `docker-windows` executor.

Depending on your setup, you may need to edit `config.toml` to configure additional options.

# Resources

* [Windows Containers Documentation](https://docs.microsoft.com/en-us/virtualization/windowscontainers/)
* [Gitlab CI/CD Documentation](https://docs.gitlab.com/ee/ci/README.html)
* [Gitlab Runners Documentation](https://docs.gitlab.com/runner/)
