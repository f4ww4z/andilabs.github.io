---
layout: post
published: true
title:  "TEAMOCIL - automatically create windows and panes in TMUX with YAML files!"
date:   2015-02-07 04:35:00
categories: utility os-x mac-os-x terminal iterm
---

Motivation:
-----------

In previous post I mentioned how to [configure iTerm using profiles, to open right apps, activate virtualenv, run development server to save some time](http://andilabs.github.io/utility/os-x/mac-os-x/terminal/iterm/2015/02/02/iTerm-start-fast-developing.html). 

Cool but not perfect. It was not utilizing any terminal multiplexer like screen, tmux, or byobu..
It was sufficient for small projects like jekyll blog, but not for some serious development, where you need to run and monitor multiple things, e.g:

- GIT status
- Project directory tree
- application server (django)
- interactive shell (ipdb, shell_plus)
- coffee compiler
- sass compiler
- postgres shell
- mongo shell

Pretty much huh? It will be cool to have in seperate panes opened few things, and maybe splited among few windows.
Doing it manually everyday is pretty tedious. Sometimes multiplexers remember their state, but not always and then you have to do it once more manually...

It will be cool to have some file with sppecified what we neeed. Thats allows us [TEAMOCIL](http://www.teamocil.com/)

Install 
-------

    gem install teamocil
    mkdir ~/.teamocil
    nano my_project.yml


Configuration file
------------------

{% highlight yaml %}
windows:
  - name: sample-three-panes
    root: ~/mbf
    layout: main-vertical
    panes:
      - commands:
        - source ~/.virtualenvs/spots/bin/activate
        - python manage.py shell_plus --settings=mbf.settings.andi
      - commands:
        - git fetch
        - git status
      - commands:
        - source ~/.virtualenvs/spots/bin/activate
        - python manage.py runserver --settings=mbf.settings.andi

  - name: frontend
    root: ~/mbf/www/static
    layout: main-vertical
    panes:
      - commands:
        - cd js
        - tree
      - commands:
        - coffee -o js/ -cw coffee/
      - commands:
        - sass --watch styles:css

  - name: databases
    root: ~/mbf
    layout: main-vertical
    panes:
      - commands:
        - psql
      - commands:
        - mongod
      - commands:
        - mongo
{% endhighlight %}

Starting
--------

    tmux
    teamocil my_project

Result
------

Results in following windows:
![django](/assets/teamocil-demo/django.png)

![databases](/assets/teamocil-demo/databases.png)

![frontend](/assets/teamocil-demo/frontend.png)


Basic navigation:
-----------------

Switch between Panes:
=====================

    Ctrl-b o

Switch between Windows:
=======================

    Ctrl-b n  # next
    Ctrl-b p  # previous

More commands can be found here: [http://fideloper.com/mac-vim-tmux](http://fideloper.com/mac-vim-tmux)