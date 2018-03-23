---
layout: post
title:  "I am so starving: Web app in python frameworks."
date:   2010-11-07 21:27:44+05:30
tags: python
author: shabda
---
I have written the same web app in various web frameworks. [Get it from Github.](https://github.com/agiliq/so-starving)

Copied from the [Readme](https://github.com/agiliq/so-starving/blob/master/README.md).
-----------------------------

This is a set of apps which creates the same application in various
Python web micro-frameworks.

The app(s) talks to Facebook, and finds out recent people
who have posted a public status with the text "so starving".

This idea came from reading [Onion](http://www.theonion.com/articles/i-am-so-starving-vs-i-am-so-starving,11541/).

We have the same app in these frameworks.  

### Microframeworks:

* [appengine](http://code.google.com/appengine/)
* [flask](http://flask.pocoo.org/)
* [web.py](http://webpy.org/)
* [juno](https://github.com/breily/juno)
* [bottle](http://bottle.paws.de/docs/dev/index.html)
* [itty](http://toastdriven.com/fresh/itty-sinatra-inspired-micro-framework/)
* [tornado](http://www.tornadoweb.org/)
* [pyroutes](http://www.pyroutes.com/)

### Full stack frameworks:

* [django](http://djangoproject.com/)
* [web2py](http://web2py.com/)
* [pyramid](http://docs.pylonshq.com/faq/pyramid.html)


------------------------

If the framework included template engine and caching, that was used directly.
Otherwise werkzeug cache and Jinja2 templates were used.

Next step: Write a simple blog engine in these various Frameworks.

---------

[Get it from Github.](https://github.com/agiliq/so-starving)

