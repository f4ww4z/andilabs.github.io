---
published: true
layout: post
title:  "Spike: using Vagrant for djangogirls workshops as alternative to all the mess with Windows vs Mac vs Linux configurations"
date:   2014-12-07 14:05:00
categories: django djangogirls devops tools vagrant
---

What is Vagrant
===============

[Vagrant](https://www.vagrantup.com/) is tool allowing easy start of virtual machines for development. Mostly we think about Linux. The machine has no GUI only command line. It uses virtualbox as virtualatization backend. It is avaliable for all platforms, including Windows, Mac and Linux.


Motivation for using Vagrant
============================

- We **save time lost on configuring stuff** and learning how to run it on Windows, which anyway for Python/Django won't be the production server never-ever.

- We are in **1:1 Linux World**, where girls hopefully one day will end-up anyway configuring their production servers on some cloud or whererever. For sure this servers will be Linux with only command line interface so getting used to it is worth.

- Thanks to setup of **shared folder** user can **use favorite python Editor e.g SublimeText for Windows or Mac nativly**.

- Thansk to setup of **IP interface** the served application will be visible from native webbrowser of user's operating system.

- **Installing** and configuring all the stuff on Linux is **much easier**.

- The **coaches can support** the workshop participants **much better**, because they are using Linux on daily basis.

- The **groups** can be **heterogenous in terms of operating system** used by the members.

- **Homogenity of Linux version** ensures that if certain configuration, e.g of Heroku Toolbelt tested once it **will work with greatest probability for every workshop participant**.

Drawbacks
=========

- It may be a bit confusing for the first time to get used to idea and usage of virtual machines


Further guidlines for vagrant setup.
====================================

- There could be pre-prepared virtualbox image for fast setup in case of some emergency situation (somebody not present at installation party, or removes by mistake the virtual machine.)

- For performance issues, it is worth consider hosting on some local machine the image of Linux to be download by vagrant (or having it on usb stick), to be Internet connection/speed independent.

- As extension we can create the tutorial how to use [salt](https://docs.vagrantup.com/v2/provisioning/salt.html), which throguh configuration files, and states allows a very fast managment of machine configuration, automates installation, pulling from repos, etc.

- On USB stick it will be worth to have installation packages of VirtualBox and Vagrant for Mac, Windows and Linux.