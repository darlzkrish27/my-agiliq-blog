---
layout: post
title:  "Google Appengine - First Impressions"
date:   2008-04-09 14:32:57+05:30
tags:	python
author: shabda
---
Google launched their EC2 competitor, Appengine yesterday, and all hell broke loose. And in about 24 hours, <a href="http://groups.google.com/group/google-appengine/browse_thread/thread/c5bf4ab38d93910d">the 10,000 accounts were used up</a>. Currently it is tied to only working with python, and Django 0.96.1 works out of the box.
<!- - more - ->

###The Good
- Python powered. Django works out of the box.
- No sysadmining chores.
- Promise of infinite scalability with no configuration. (Ah!)
- Free for now.

###The Bad
- Python powered, if you want to use ruby/java/php, sorry you are out of luck.
- Your code is tied to Google. You might be able to reuse most of your code, but the DB/ORM sepcific code, ah you are out of luck. And if you are building database backed websites, well most of your most complex code talk to ORM.
- Too magical. *Explicit is better than implicit*. On the dev server I do not know where my data is stored. If I change the data model, the changed models are available immediately. But well, how do you do it?
- Free for now, and no way to pay for when your usage out grows the free quota limits.

