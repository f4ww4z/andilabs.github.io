---
layout: post
published: true
title:  "click it and go make yourself coffee; Let iTerm open right apps, activate virtualenv; run development server for you. Stay DRY everyday!"
date:   2015-02-02 22:05:00
categories: utility os-x mac-os-x terminal iterm
---
![iterm](/assets/iterm.png)

I wonder that for so long time I was doing it manually

1. Open terminal
2. navigate to projects directory
3. open project in SublimeText
4. activate virtualenv
5. run development server
6. open Chrome to watch whats happens

2-6 is just about pasting all commands you need seperated by `&&` or `|` or `|&` or `;` [what should I choose? Read here](http://stackoverflow.com/a/5130889/953553)

	cd mbf && open -a Sublime\ Text . && source ~/.virtualenvs/spots/bin/activate  && open -a Google\ Chrome http://127.0.0.1:8000 && python manage.py runserver --settings=mbf.settings.andi

to execute into `Send text at start`:

![iterm smart](/assets/iterm_smart.png)

what is quite nice you can build multiple such profiles. You can start them in new windows or new tabs of iTerm.

to be extended
--------------

using some terminal multiplexer

DONE
====
	
	READ: [TEAMOCIL - automatically create windows and panes in TMUX with configuration from YAML files!](http://andilabs.github.io/utility/os-x/mac-os-x/terminal/iterm/2015/02/07/osx-tmux-configuration-fiels-with-setting-magical-teamocil.html)