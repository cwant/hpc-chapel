---
layout: page
title: Setup
permalink: /setup/
---

There are several pieces of software you will wish to install before the workshop.
Though installation help will be provided at the workshop, 
we recommend that these tools are installed (or at least downloaded) beforehand.

## SSH

All students should have an SSH client installed.
SSH is a tool that allows us to connect to and use a remote computer as our own.
Please follow the directions below to install an SSH client for your system.

**Windows**

Test to see if SSH is installed on your computer by bringing up a command
prompt (type `cmd` from the application search). From the command
prompt, type `ssh` and see if windows finds a program by that name.

If `ssh` is not found, but you have Git Bash installed, run Git Bash and
you should have access to the `ssh` command from there.

If all else fails, you can install MobaXterm from [http://mobaxterm.mobatek.net](http://mobaxterm.mobatek.net).
You will want to get the Home edition (Installer edition).

**MacOS**

MacOS comes with SSH pre-installed, you can run it from the terminal program.

**Linux**

Linux users do not need to install anything, you should be set!

## Connecting to the training cluster

Your instructor will have given you a username and password to our training cluster.
From a prompt on your computer, you can type:

```
ssh YOURUSERNAME@training.uofa.c3.ca
```
{: .bash}
