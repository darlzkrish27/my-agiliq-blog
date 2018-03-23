---
layout: post
title:  "Two Scoops of Django: Review"
date:   2013-02-11 11:30:00+05:30
tags: book
author: shabda
---
![Django best practices](https://s3.amazonaws.com/twoscoops/img/tsd-cover.png)

[Two scoops of Django](https://django.2scoops.org/) is the new book on Django best practices by Daniel Greenfeld and Audrey Roy. I just finished reading it and found it extremely useful.

The book is a collection of tips, their justification and code organized in logical areas. I have been using Django since 2008, but I still found a few tips which were new to me and many which served as good reminder. At about 200 pages, its comprehensive but not overwhelming. If you are an advanced Djangonaut, you can read it in a weekend (And you probably are using all its recommendations anyway). If you are just getting started, putting all the tips to use will keep you busy for month.

A random sampling of tips (The book has more than 50 of them.)

* Use pip, virtualenv and virtualenvwrapper
* Don't keep your virtualenv inside the project folder
* Apps should be small and do one thing well
* Use relative modules, prefer `from .models import foo`
* All settings file should be versioned - local_settings.py is an anti-pattern
* Every settings module should have a corresponding requirements.txt
* Don't hardcode your paths
* SingleTable model inheritance is good, multi table model inheritance is bad.
* Create custom managers, but don't reset  default manager.
* If you use CBV, ensure logic doesn't go to urls
* Use Meta.fields and Never Use Meta.excludes


I especially like that the book mentions alternate ways of doing things. Eg. it recommends CBV as the new and shiny, but advises that FBV are just as reasonable choice.

If you use Django, I highly recommend this book. At just $12, it would be hard to go wrong.





