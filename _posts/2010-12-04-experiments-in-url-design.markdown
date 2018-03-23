---
layout: post
title:  "Experiments in URL design."
date:   2010-12-04 15:41:47+05:30
tags: apps
author: shabda
---
Keeping url structure sane plays a major role in user experience, by making it easy to go to specific pages
and navigate the site. Some real world urls and how they could have been better follow.   

----------------------

## The Bad Urls:

* http://news.ycombinator.com/item?id=1967820
* http://area51.phpbb.com/phpBB/viewforum.php?f=99
* http://msdn.microsoft.com/en-us/aa336522.aspx

#### The Good

* Easy to develop for. The id referred is the Primary key

#### The Bad

* Hard for users to select the correct page from a url bar dropdown
* Bad for SEO

#### Neutral: 

* Easy to guess next and previous pages.

----------------------

## The fairly good Urls.

* http://www.reddit.com/r/django/comments/efh2v/djangoeasymaps_app_that_makes_it_easy_to_display/
* http://stackoverflow.com/questions/4352854/jquery-handle-keypress-for-list

#### The Good

* Includes title in the url which makes it easy to select in browser dropdown and history.
* Good for SEO.

#### The Bad

* Includes redundant info in Urls, which make it unnecessarily long.
* Urls are not unique. http://stackoverflow.com/questions/4352854/anything will work.

#### What I would like

* http://stackoverflow.com/jquery-handle-keypress-for-list
* http://www.reddit.com/djangoeasymaps_app_that_makes_it_easy_to_display/


----------------------


## The almost good urls.

* http://www.linkedin.com/in/shabda/
* http://www.reddit.com/r/india
* http://www.reddit.com/user/shabda/



#### The Good

* Short
* Includes only important info in the url.

#### Cons

* Quite good, but still includes some unneeded info.

#### What I would like 

* http://www.linkedin.com/shabda/
* http://www.reddit.com/india
* http://www.reddit.com/u/shabda/


----------------------


## The prefect urls.

* https://github.com/Spacelog
* https://github.com/facebook/tornado
* http://twitter.com/#!/shabda
* http://twitter.com/#!/shabda/kiddos
* http://techcrunch.com/2010/12/03/angry-birds-android-1-million-advertising/
* https://mail.google.com/a/agiliq.com/#starred/12c84752671e87c3
* http://code.google.com/appengine/docs/whatisgoogleappengine.html


#### The Good

* No unneeded info is included in the urls. 
* Each part of the url can be removed to get the logical page for it. 
Eg. [0](http://code.google.com/appengine/docs/whatisgoogleappengine.html) [1](http://code.google.com/appengine/docs/), [2](http://code.google.com/appengine/), 3[http://code.google.com/]

----------------------

Suggestion

Make urls case insensitive

* http://news.ycombinator.com/item?id=1967820
* http://area51.phpbb.com/phpBB/viewforum.php?f=99

Work.

* http://news.ycombinator.com/Item?id=1967820
* http://area51.phpbb.com/phpBB/Viewforum.php?f=99

Do not work. (We changed capitalization.)

While

All of these work.

* http://twitter.com/#!/shabda
* http://twitter.com/#!/Shabda
* https://github.com/Facebook/tornado
* https://github.com/facebook/tornado




