---
layout: post
title:  "Using bpython shell with django (and some Ipython features you should know)"
date:   2009-12-12 22:47:37+05:30
tags: tutorial
author: lakshman
---
###What is bpython?

> bpython is a fancy interface to the Python interpreter for Unix-like operating system.

says the [bpython](http://bpython-interpreter.org/) home page. It provides syntax highlighting, auto completion, auto-indentation and such stuff.

Unlike [iPython](http://ipython.scipy.org/moin/), which implements then entire shell functions and emulates the standard python shell, and adds enhancements, bpython just adds features on top of the existing python shell functionality, using the curses module. 

<img src="http://i.imgur.com/cqky1.png" alt="bpython" />

The "killer feature" of bpython is, as you can see from the image above, the IDE like prompting of the function parameters and the doc string of the function dynamically. I have always thought, what [IntellijIDEA](http://www.jetbrains.com/idea/) is to Java, IPython is to Python*. But bpython makes it more so, in terms of the code completion, which adds a lot of value to a _ramping up developer_.

The only Python IDE that provides Source Assistant and Go-To-Source functionality conveniently, is the commercial one, [Wing IDE Professional](http://www.wingide.com/wingide). Even with that, since all the defined models are replaced (by using meta-classes) in the runtime, that source assistant is not perfect. Newer versions of Wing seems to claim to obtain data dynamically at the runtime, but its not always comfortable to write code, keeping the IDE in a breakpoint. But hey, bpython provides all these for free!

###But I already use Ipython

You do? So do I. In fact, I am a Power Ipython user, I use all kinds of [%magic](http://ipython.scipy.org/doc/stable/html/interactive/reference.html#magic-command-system) functions: %whos, [%logstart](http://ipython.scipy.org/doc/stable/html/interactive/reference.html#session-logging-and-restoring), %bookmark, %ed  and sometimes use Ipython as an alternative even to bash: %cd, %ls, %ll. And even for doc testing: [%doctest_mode](http://ipython.scipy.org/doc/stable/html/interactive/reference.html#pasting-of-code-starting-with-or), copy-pasting %hist -n or running some code module in the interpreter namespace: %run -i and searching code %psearch. You can also give any arbitrary bash(or your default shell) command by escaping it with !. Oh, ?, ?? and / are of course the starters.

In fact did you know, you could get to ipython shell within any arbitrary place within your django code? Just use the following:

    from IPython.Shell import IPShellEmbed
    ipython = IPShellEmbed()
 
and then call ipython() anywhere within your view, model, forms and you will be dropped to the shell. Its [like ipdb.set_trace()](http://aymanh.com/python-debugging-techniques) only better (unless you want to step through, that is).

###The Python readline bug

The Python version 2.6.4 (the version that is shipped with Ubuntu 9.10), introduced a [readline bug](http://bugs.python.org/issue5833) that adds spaces after tab completion. This bug also affects the Ipython, as it uses the same readline module. If you spend a lot of time on the shell (if you use python, you must, right?), it is very annoying to backspace after each tab, all the time.

Although the bug got fixed pretty soon, it hasn't yet made it to any release that ubuntu updates to. There are ways to workaround this problem, by fixing it at the python level and at the Ipython level, many of them are discussed on the corresponding [Ipython bug](https://bugs.launchpad.net/ipython/+bug/470824) 

###Using bpython with django

Now that you want to use bpython with django, either because you like the auto completion, or because you find the read line bug annoying (and don't want/care-enough to patch your python locally), or you just want to try it, how to do it?

`django manage.py shell` unfortunately, drops only into the ipython shell (if you have ipython installed), and there is no straight forward way to get it to drop to bpython.

But there is still a way to use bpython with django. Just modify your ~/.bashrc to define a python startup environment variable

    export PYTHONSTARTUP=~/.pythonrc

And within it, setup the django environment, that is, do it here manually the thing that `python manage.py shell` would do for you:

    #.pythonrc
    try:
    	from django.core.management import setup_environ
    	import settings
    	setup_environ(settings)
    	print 'imported django settings'
    except:
        pass

This way, `bpython` or even just `python` on the command line, should import the django environment for you.

###Importing all models automatically into the shell

But then, if you are possibly already used to `python manage.py shell_plus` (If you are not, you should be.) that is defined by [django_command_extensions](http://wiki.github.com/django-extensions/django-extensions/current-command-extensions).

So while we are at setting up the django environment, lets just also import all the models into the shell namespace, so that it is convenient to work. Following is some ugly quick hack to get it done.

<script src="http://gist.github.com/231878.js?file=.pythonrc.py"></script>

This can of course be improved upon. If you do it, just leave it in the comments! In fact it would be good if it also includes `from startup.py import *` in a try catch, so that, you can put some extra code into startup.py. Saving into startup.py from within a bpython shell is just a `Ctrl+s` anyway. That way each time you get to the shell, you can have the same expected environment; and it is quite easy to change that file.

*I know, I know, IPython doesn't refactor code, nor build, nor does a million things that Intellij does, but [Python is not Java](http://dirtsimple.org/2004/12/python-is-not-java.html) and basically both intend to enhance developer productivity. And I spend a lot of time on the IPython shell and find it to be a tool just like Intellij is a tool, for Java.


