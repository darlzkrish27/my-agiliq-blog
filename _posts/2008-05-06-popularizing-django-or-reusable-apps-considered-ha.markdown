---
layout: post
title:  "Popularizing Django -- Or Reusable apps considered harmful."
date:   2008-05-06 10:44:54+05:30
categories: marketing
author: shabda
---
For all its technical merits, Django is still a very [niche technology](http://google.com/trends?q=php%2C+django%2C+). It is my belief that the thing which is holding Django back the most, is due to one of its strengths.

Making reusable apps is easy and simple in Django. In Django this is the [correct](http://www.b-list.org/weblog/2007/mar/27/reusable-django-apps/) way to do things. You take a few apps, mix them together in your project, and deploy to start your site.

Compare the installation steps of Wordpress and an imaginary blog software better than Wordpress called Djangopress.

#### Wordpress
1. FTP wordpress to webserver.
2. Point browser to site.com/blog
3. Next-Next-Next done.

#### Djangopress
1. Svn checkout Djangopress
2. Svn checkout [django-registration](http://code.google.com/p/django-registration/)
3. Svn checkout other Django apps Djangopress depends on. Maybe [django-mptt](http://code.google.com/p/django-mptt/), [django-threadedcomments](http://code.google.com/p/django-threadedcomments/) or a few others.
4. Edit your settings.py to add all these apps to `INSTALLED_APPS`.
5. Add database settings, and other changes if needed.
6. Telnet to your server and do `syncdb`
7. Create templates. Done.

This does not take into account the extra hoops Apache makes  you jump through, compared to using a PHP app.

### How I got started with web programming.

I wanted to run a forum. [PhpBB](http://www.phpbb.com/) was free, and seemed most widely used. Installed it, and wanted to tinker with it, so learnt Php. If there was a different forum software, which was technically superior, but which asked me to write templates for it before I could start a forum, guess which one I would have chosen?

### So how to popularize Django.

In my [interview of James Bennett](http://42topics.com/blog/2008/04/interview-with-james-bennett-django-release-manager/), I asked what is Django's killer app. And he said there need not be a Killer app for Django, reusable apps will do. I guess I will have to disagree. Even internet needed a [killer app](http://en.wikipedia.org/wiki/Webmail) to get breakthrough popularity. Let's see what a Killer app gives you.

1. It fills a big niche, so people are forced to learn your language/framework.
2. It forces the Hosting company to support your language/framework.
3. If a large number of places use it, it gives your framework name recognition.

So to popularize Django, I propose setting up DjangoPackagedApps.com to distribute packaged Django apps, to complement reusable Django apps. A packaged Django app, must have these properties.
1. All dependencies must be included.
2. Beautiful templates must be included out of the box.
3. Users must not need to modify anything in settings.py apart from the database settings.

And installing the PackagedApp must be no more than the number of steps needed in Wordpress.

1. Svn checkout/FTP DjangoPackagedApp
2. Only thing to edit in settings.py is database settings.
3. Do syncdb. done.

-----------------------------------

Do yo use [Django](http://42topics.com/Django/)? Do you [program](http://42topics.com/programming/)? Find things which YOU will love reading at [42topics.com](http://42topics.com/register/).

