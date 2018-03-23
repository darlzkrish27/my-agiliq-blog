---
layout: post
title:  "On Captcha"
date:   2009-07-16 14:05:35+05:30
tags: tips
author: shabda
---
When building public facing websites, spam is a real problem. Captcha has been teated as the first line of defence aginst this problem. If you must use captcha, here are some best practices working with them.

### Can you do without one?
A lot of places captcha's are put to filter spam in user generated comment. One of the largest sources of UGC is comments on wordpress blogs. They do not use a captcha, and instead pass all comments through Akismet to verify which comments are spam, and reject the spams. In many cases, such a system would work for you.

### Can you use a simpler alternative?

Unless you run a site with millions of monthly users, the spammers are not going to write a bot specifically for youur site. So if you have a text question asking "what is 2 + 2", the bots are not going to get past that, as the question is unique to your site. Of course, gmail/yahoo can not use this approach, as bots get written for them specifically, but for you site it might.

This proposed approach has been called [SAPTCHA](http://dmytry.pandromeda.com/texts/captcha_and_saptcha.html) and anecdotal evidence suggests that it works very well in practice.

### Prefer recaptcha

[Recaptcha](http://recaptcha.net/) is an existing implementation of captcha, which allows you to plug in a captcha system, to your existing pages. Using this over home grown captcha's offers advantages like

1. Its is battle tested, in being both secure against bots and having a level of usability.
2. It automatically provides and alternate means of validation. Which brings us to our third point.

### Provide alternate means of validation to visually impaired people.

This might be in the form of an audio captcha, an option to email the administrator.

### Do captcha validation only once

This seems so obvious, and yet many sites require you to complete captcha multiple times, in case of any error in the form. Once an user has completed captcha, the form must be displayed without captcha in case of errors.

--------

If you use Django, these are already very easy to use with existing apps.

1. [Django recaotcha](http://www.djangosnippets.org/snippets/433/)
2. [Django akismet](http://www.djangosnippets.org/snippets/1255/)
3. [Python defensio](http://defensio.com/downloads/python/)

---------

You should follow me on twitter [here](http://twitter.com/uswaretech)[.](http://dustincurtis.com/you_should_follow_me_on_twitter.html)

