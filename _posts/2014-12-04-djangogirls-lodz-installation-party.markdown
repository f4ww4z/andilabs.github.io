---
layout: post
title:  "Djangogirls: Installation Party notes for Windows users"
date:   2014-12-04 12:49:17
categories: django djangogirls
---

Installation party
==================


True to be told, when I've heard that I will as coach guide people how to start with Python and Django using Windows I was scared a bit! I've thought installing dev stuff on Windows will be ass-pain, but I have to admit it was only a bit harder than on Linux, but anyway way easier than on OS X.
----------------

To be installed:
----------------

PowerShell (if not already installed)
==================

Python 3.4.2
==================
	https://www.python.org/downloads/

SQLite (nothing to install)
==================
	"By default, the configuration uses SQLite. If you’re new to databases,
	or you’re just interested in trying Django, this is the easiest choice.
	SQLite is included in Python, so you won’t need to install
	anything else to support your database."
	FROM: https://docs.djangoproject.com/en/1.7/intro/tutorial01/



SublimeText
===========
	http://www.sublimetext.com/3
	https://sublime.wbond.net/installation
	shift + ctrl + P -> install packages
		djaneiro
		pep8

GIT
===
	http://git-scm.com/download/win
	IMPORTANT! installation mode as shown here:
	http://tech.pro/tutorial/1840/setting-up-git-and-github-from-the-windows-command-prompt


[HINT] PowerShell COPY and PASTE
================================
	To select text in PowerShell with the mouse, just select it as usual.
	To copy the selected text to the clipboard you have to either hit Enter, or right-click.
	To paste into the Powershell window, right click
	FROM: http://superuser.com/a/405858/216659



PowerShell WORKOUT
==================
	navigate in directories structure, access history with keys down/up -arrows
	basic commands: ls (dir), pwd, cd, mv, cp, mkdir, cat, >, <, clear, tree,


Setting up:
-----------

VIRTUALENVS
===========
Install Virtualenv

	pip install virtualenv

PIP
===
PIP - is cool! You can install useful python libraries with it.

	If you know that for ceratin project it is neccesary to have certain libs,
	just create plain text file with listing them, e.g:
		Django==1.7.1
		Pillow==2.6.1
	we call this file `requirements.txt` by convention

INSTALL MULTIPLE LIBRARIES AT ONCE

and you know what is beautiful? You install all listed there librariers (for bigger project it can be dozens of them!!) with runing just:

	pip install -r requirments.txt

#Wait a moment... what IF I develope multiple projects with completly different libraries, or would like to play with different python versions?

### The way to go is using a isolated environments called virtualens!


BASIC VIRTUALENV WORKFLOW:
==========================

CREATE

	go to C:\ create directory `virtualenvs`, and go there
	create new virtualenv named `wirtuoz`:
		C:\Python34\python -m venv wirtuoz

ACTIVATE

activate the created virtualenv:

	C:\virtualenvs\wirtuoz\Scripts\activate

DEACTIVATE

just:

	deactivate

WINDOWS PowerShell DISCLOSURE

Due to powershell security restrictions issue (setup only once):

	Run PowerShell as admin !!
	Execute this command:
		Set-ExecutionPolicy Unrestricted -Scope CurrentUser

WHAT'S INSIDE CURRENT VIRTUALENV?

do it by listing installed libraries:

	pip freeze

by the way if you want to create `reuqirements.txt` for your project just:

	pip freeze > reuqirements.txt

GIT + Github.com
================

#### GIT != github.com

Git is advanced revision control system allowing teams easy parallel development. Github is a kind of cool online hosting with nice GUI functionality and especially features making work of distributed teams on code in mostly open source projects easier.

####Setup github repo on github.com
	Login
	Create new repo
	Edit README.md with some content.
	copy SSH clone URL, it should look something like: git@github.com:github_username/repo_name.git

####Clone the repo

1) Before we clone, we need to generate on our computers SSH keys

	PLEASE use `id_rsa` name, it will be then used
	as default and will allow us to skip bit more sophisticated configuration
	add our public key to github on https://github.com/settings/ssh
	all that is demonstrated in this tutorial:
	http://tech.pro/tutorial/1840/setting-up-git-and-github-from-the-windows-command-prompt

2) If everything was setup right, we should succed with running:

	git clone git@github.com:github_username/repo_name.git

####Basic commands demo:

	pull, push, commit, diff, status, checkout

####If still some time and power remains:

	- branches
	- merging


