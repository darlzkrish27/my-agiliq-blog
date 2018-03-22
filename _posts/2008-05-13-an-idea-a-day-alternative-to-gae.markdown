---
layout: post
title:  "An idea a day - Alternative to GAE"
date:   2008-05-13 20:55:39+05:30
categories: startup
author: shabda
---
This is an article in the **Five Startup Ideas** series at [the 42topics blog](http://42topics.com/). In his essay, [Ideas for Startups](http://www.paulgraham.com/ideas.html), Paul Graham argues that ideas are not a critical factor for success of startups. Although I do not believe that ideas are worthless, as many people do, I believe that they are not any where near as important as execution. So to prove my point, I am giving away 5 startup ideas in next five days. All of them describe a problem, its solution, the technology involved, the competition and market size. If you are not a hacker, and want to build any of these things may I suggest [Uswaretech](http://uswaretech.com/).

------------------

### Title:

Building a scalable alternative to Google App Engine.

### The problem:

Scalability is Hard, lets go shopping. -[Consultant Barbie](http://reddit.com/r/programming/user/consultant_barbie/)

There are many companies trying to take the pain out of scaling. [Amazon EC2](http://www.amazon.com/gp/browse.html?node=201590011) and [GAE](http://code.google.com/appengine/) make this much easier.

However there are still big [problems](http://42topics.com/blog/2008/04/google-appengine-first-impressions/) with GAE. For example not being able to run cron jobs make this instantly unusable for many people. Many of these shortcomings would be removed in time, still I believe that when a site can pay for itself, many people would like to move it on their infrastructure, with much more freedom to do things.

### The solution:

Create a drop in replacement for GAE. People must be able to just drop in their GAE applications and start using it, without modification.

### Technologies involved:

Of course this is a hard problem. You essentially want to build a super scalable system. But this is possible if you mix some open source components well. For hardware, host all your system on EC2 instances. Use [Hadoop](http://hadoop.apache.org/core/) for getting Mapreduce functionality. Use [Hbase](http://wiki.apache.org/hadoop/Hbase) or [Hypertable](http://hypertable.org/) instead of Bigtable. Use Django to talk to them.

### Existing Competition:

This is such a new area there are no existing competition in this area. But if you want to take other systems promising infinite scalability as competition then there is [Heroku](http://heroku.com/). Of course GAE is an competitor as well.

### Market Size:
Heroku raised [3 Million USD](http://www.techcrunch.com/2008/05/08/ruby-on-rails-startup-heroku-gets-3-million/) recently.

### Others:

It would not be a lot of extra work to build massively scalable solutions for Django. You play with the same stack, but write two database API, one which mimics Django, and another which mimics GAE.

-----------------------

This was part 4 of the series of 5 startup ideas. For next five days we will publish a new idea a day. If you want to read all of them, please [subscribe](http://42topics.com/blog/feed/). Oh and have you seen the [42topics startup](http://42topics.com/startups/)  section? Or if you want you can [create](http://42topics.com/create/) your own topic.

