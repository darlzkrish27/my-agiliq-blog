---
layout: post
title:  "Remote debugging - debugging pesky server only bugs"
date:   2009-07-02 23:37:02+05:30
tags: django
author: shabda
---
Here is a quick tip. (Obvious if you work with Django for any length of time, but I have seen a few people who are not aware)

You can put debug trace `import pdb; pdb.set_trace()` in your code, and put it on the server. When you access this view from your local browser, the debug is still hit and you have a debug shell on your server where you can step through. (Obviously this works only if you ran the code via `manage.py runserver`. But `manage.py runserver` start the server to listen only on local connections. If you want to access remotely you need to run as,

    python manage.py runserver 0.0.0.0:8000

Edit: As [SmileyChris](http://tactful.co.nz/) commented, a faster way is,

    python manage.py runserver 0:8000

The `0.0.0.0` implies that remote connections are possible.

For me, this has been a lifesaver against those pesky bugs which show themselves only on the server, but not on the local machine.

