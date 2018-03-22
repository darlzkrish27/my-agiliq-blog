---
layout: post
title:  "Five Things I Hate About Django."
date:   2008-04-18 11:48:38+05:30
categories:	python
author: shabda
---
The [five](http://www.jacobian.org/writing/2007/mar/04/hate-python/) [things](http://ivory.idyll.org/blog/mar-07/five-things-I-hate-about-python) [I hate](http://use.perl.org/~brian_d_foy/journal/32556?from=rss) [about *](http://adam.gomaa.us/blog/five-things-i-hate-about-django/) meme seems have died down, and memes, should not be allowed to die.

 Of course I love Django, and have bet very heavily on it. But we do not know a topic, until we know it warts, so here you go. The listing is in no particular order, so sorry no numbering.

### Ajax with Django is hard:

 Most of the Django Community has decided that bundling Javascript helpers with a python framework is bad idea. Though I can understand the reasoning, (argued [here](http://www.b-list.org/weblog/2006/jul/02/django-and-ajax/) and [rebuttal](http://feh.holsman.net/articles/2006/07/04/what-should-your-framework-do-for-you)), that Javascript is so basic that you can not be expected to not know it, I can not agree with it. SQL is as basic as Javascript, and yet we have ORM for abstracting away the common and the tedious.

Of Course, with [simplejson](http://djangoapi.matee.net/django.utils.simplejson-module.html), and a [good Javascript library](http://jquery.com), you can build Ajax apps fast and with only a minimal amout of fuss. And yet switching between Python and Javascript, twice every hour is a huge time drain. Eg. I put commas after the last element in Python arrays, with JS this would work in FF, but fail with weird errors in IE.

### Lack of [identity map](http://martinfowler.com/eaaCatalog/identityMap.html):

If you get the same row from the DB twice using Model.objects.get, you will get two different objects. Apart from the performance problems of two DB queries, when only one should have done, when you update one of them, the other does not get updated, and you will have *interesting* things happening in your application. And if you update both of them, you might write two inconsistent changes to the DB.

Look at this code for example.

<!-- <pre lang="python">
See this code
In [2]: from django.contrib.auth.models import User
In [3]: usr1 = User.objects.create_user('ram', 'demo@demo.com', 'demo')
In [4]: usr2 = User.objects.get(username='ram')
In [5]: usr3 = User.objects.get(username='ram')
In [6]: user2 == user3
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
...
In [7]: usr2 == usr3
Out[7]: True
In [8]: usr3.username = 'not_ram'
In [9]: usr3.save()
In [10]: usr2.username
Out[10]: u'ram'
In [11]: us3.username
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
...
In [12]: usr3.username
Out[12]: 'not_ram'
In [13]: usr2 == usr3
Out[13]: True
</pre> -->

### Whether Sessions are browser length/persistent are set sitewide:

You can set whether you want sessions to be browser length/persistent using `SESSION_EXPIRE_AT_BROWSER_CLOSE` in `settings.py`. But you can not set them per user, [without mucking with django internal](http://code.djangoproject.com/wiki/CookBookDualSessionMiddleware). This might seem a minor annoyance, yet this is something which you need to do for every app, as the remember me, function will not work without this.

### Newforms is very limited:

Let us say you want the Form to contain a varible number of fields. How can you define the `NewForms` class to do your biddings.

<pre lang="python">
from django import newforms as forms
class MyForm(forms.Form):
    foo = froms.CharField()
	bar = froms.CharField()
</pre>

This can only create a form with a fixed number of fields. While there are ways to generate forms with variable number of fields, (generate the Form class programatically), they are not easy or well documented. (Remind me to write such tutorial sometime.)

Bonus question: How can you generate a form with same form elements multiple (and variable number) times, ala what happens with `edit_inline`?

### Settings mixes application configuration which should be public and passwords, which should be private:

If I am distributing an app `MIDDLEWARE_CLASSES` is something which I would assume users would not (generally) modify. Similarly, in most of the cases, `INSTALLED_APPS`, would also be something which users would not change, (unless you are distributing standalone_apps). This means, I want to source control `settings.py`. But `settings.py` also contain my DB setiings, and `SECRET_KEY`, which means, I cannot source control `settings.py`.

And while we are at it, can we refactor `settings.py`, so it works without

<pre lang="python">
os.environ['DJANGO_SETTINGS_MODULE'] = 'settings'
</pre>

### Bonus:

Two things which used to bug me but no more.
1. You cannot extend Models - Well now you can if you use queryset-refactor, or [soon](http://groups.google.com/group/django-users/browse_thread/thread/6a275999abab2e66) can if you are on trunc.
2. Url configuration using regexes. - [Now they have two problems.](http://regex.info/blog/2006-09-15/247) joke notwithstanding, mapping URLs to views is one problem where regexes fit the problem beautifully. With less that 50 lines of code, you can manage a large number of views, and Url patterns.

[Now stay tuned for Five things I love about Django](http://42topics.com/blog/feed/)

