---
layout: post
title:  "Djangogirls: Installation Party notes for Windows users"
date:   2014-12-04 12:49:17
categories: django djangogirls
---

Installation party
==================


True to be told, when I've heard that I will as coach @[djangogirls.org] guiding people how to start with Python and Django using Windows I was scared a bit! I've thought installing dev stuff on Windows will be ass-pain, but I have to admit it was only a bit harder than on Linux, but anyway way easier than on OS X.

[djangogirls.org]: http://djangogirls.org/lodz

So, let's go!

To be installed:
----------------

PowerShell (if not already installed)
==================

Python 3.4.2
==================
Install 3.4.2

    https://www.python.org/downloads/

Open powershell and run following commands:

    [System.Environment]::SetEnvironmentVariable("PATH", $Env:Path + ";C:\Python34")
    [System.Environment]::SetEnvironmentVariable("PATH", $Env:Path + ";C:\Python34\Scripts")

If you have choosen another version, the number in above command will just differ. Then open once more PowerShell and check if you see python idle prompt `>>>` after entering `python`.

SQLite (nothing to install)
==================
    "By default, the configuration uses SQLite. If you’re new to databases,
    or you’re just interested in trying Django, this is the easiest choice.
    SQLite is included in Python, so you won’t need to install
    anything else to support your database."
    FROM: https://docs.djangoproject.com/en/1.7/intro/tutorial01/



SublimeText
===========

* editor:

    http://www.sublimetext.com/3

* enable installation of 3-rd party packages: View -> Show console and copy-paste piece of script from:
    https://sublime.wbond.net/installation to the console. Then `shift + ctrl + P` and start typing: install packages and install

        djaneiro
        SublimeLinter-pep8

GIT
===
    http://git-scm.com/download/win

IMPORTANT! Choose installation mode exactly as shown in screenhots here: [http://tech.pro/tutorial/1840/setting-up-git-and-github-from-the-windows-command-prompt](http://tech.pro/tutorial/1840/setting-up-git-and-github-from-the-windows-command-prompt)


[HINT] PowerShell COPY and PASTE
================================
* To SELECT text in PowerShell with the mouse, just select it as usual.
* To COPY the selected text to the clipboard you have to either **hit Enter**, or **right-click**.
* To PASTE into the Powershell window, **right click**

FROM: [http://superuser.com/a/405858/216659](http://superuser.com/a/405858/216659)



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
PIP - is cool! You can install useful python libraries with it. If you know that for ceratin project it is neccesary to have certain libs, just create plain text file with listing them, e.g:

    Django==1.7.1
    Pillow==2.6.1

we call this file `requirements.txt` by convention

INSTALL MULTIPLE LIBRARIES AT ONCE

and you know what is beautiful? You install all listed there librariers (for bigger project it can be dozens of them!!) with runing just:

    pip install -r requirments.txt

### Wait a moment... what IF I develope multiple projects with completly different libraries, or would like to play with different python versions?

### The way to go is using a isolated environments called **virtualenvs**!


BASIC VIRTUALENV WORKFLOW:
==========================

* CREATE

go to `C:\` create directory `virtualenvs`, and navigate there

create new virtualenv named `wirtuoz` by running:

    C:\Python34\python -m venv wirtuoz


* ACTIVATE

activate the created virtualenv:

    C:\virtualenvs\wirtuoz\Scripts\activate


* INSTALATION

while being in acitve desired virtualenv givn denoted in prompt like `(wirtuoz)` you install libraries by:

    pip install django

* DEACTIVATE

just:

    deactivate

* **WINDOWS PowerShell DISCLOSURE**

    Due to powershell security restrictions issue (setup only once):

    * Run PowerShell as admin !!
    * Execute this command:

        `Set-ExecutionPolicy Unrestricted -Scope CurrentUser`


* WHAT IS INSIDE MY CURRENT VIRTUALENV?

check it by listing installed libraries:

    pip freeze

by the way if you want to create `reuqirements.txt` for your project just:

    pip freeze > reuqirements.txt

GIT + Github.com
================

#### GIT != github.com

Git is advanced revision control system allowing teams easy parallel development. Github is a kind of cool online hosting with nice GUI functionality and especially features making work of distributed teams on code in mostly open source projects easier.

####Setup github repo on github.com
* SignUp if not yet
* Login
* Create new repo
* In web browser on github.com edit README.md of the repo with some content (just to know this are changes made by you). Save(commit).
* copy SSH clone URL, it should look something like: git@github.com:github_username/repo_name.git

####Clone the repo

1) Before we clone, we need to generate on our computers SSH keys

Navigate to your home directory: `C:\Users\your_user_name`

    mkdir .ssh
    cd .ssh
    ssh-keygen -t rsa -C "your.email@example.com"

PLEASE use `id_rsa` name, by typing it explicitly. It will be then used
as default key and will allow us to skip bit more sophisticated configuration

Display value of publick key:

    cat id_rsa.pub

copy it co clipboard and add it to github on [https://github.com/settings/ssh](https://github.com/settings/ssh)

If you get lost, have a look here: [http://tech.pro/tutorial/1840/setting-up-git-and-github-from-the-windows-command-prompt](http://tech.pro/tutorial/1840/setting-up-git-and-github-from-the-windows-command-prompt)

2) Now time for cloning:

    git clone git@github.com:github_username/repo_name.git

### OK, I lied to you a little bit, to motivate you to generate all that stuff, but you can clone all PUBLIC repos from github just by running git clone wihtout any keys. But it won't work for your PRIVATE repos. And anyway we will need it to made (push) any changes we made localy to remote version of repository, so it was worth anyway ;)

3) The definitive test (if we have really access to made some changes to remote version):

    notepad.exe README.md

Made some changes and save file. See the output off:

    git status

Commit the changes you made to repo:

    git commit -am "some memo about changes"

Push it:

    git push origin master

Go to [https://github.com/](https://github.com/) and check if your commit was really pushed.

####Basic commands demo:

    pull, push, commit, diff, status, checkout

####If still some time and power remains:

    - branches
    - merging


