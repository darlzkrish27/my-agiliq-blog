---
layout: post
title:  "An interview with Michael Trier"
date:   2008-05-06 17:34:45+05:30
categories: marketing
author: shabda
---
<img src="http://uswaretech.com/blog/wp-content/uploads/2008/05/mtrier-small.jpg" alt="" title="mtrier-small" width="154" height="205" class="right frame" />
[Michael Trier](http://blog.michaeltrier.com/) is  a long time Django user and evangelist. He has worked with a number of technologies including Rails and .net. His insights on marketing Django to traditionally *Enterprisy* areas were extremely informative. He produces <acronym title="This Week in Django">TWiD</acronym>, along with [Brian Rosner](http://oebfare.com/) which is great to keep abreast of the latest happenings in the Django community. He graciously agreed to be interviewed by the [42topics blog](http://42topics.com/blog/).

--------------------------------------

**Shabda:**  Would you tell a little about yourself, how did you get started with Django, what other projects have you used or are associated with?

**Michael:** Well, I've been programming ever since I can remember, probably around 11 years old.  I grew up in Silicon Valley, and that whole story is a pretty interesting one.  I did the usual thing of starting out with languages like ASM, C, C++ , Pascal.  Moved on to things like Delphi, VB, and most recently I spent quite a bit of time with Ruby, Rails, and within the past year and a half dabbling with Django.
I came to Django for a particular reason.  I was focussed on building a high content push type of site and it just seemed like Django was a much better fit for that than Rails.  Obviously I could have done it with either language, but I believe in using the right tool for the job.

**Shabda:** You are using Django for your next venture. What are the specific areas where using Django been a better way to develop for you, compared to any other choice you might have made, say Rails?

**Michael:** I hate to bring up the scaling issue, but that was certainly something I was seeing as a problem with Rails.  The type of site I'm building I hope to see some pretty high traffic (don't we all).  So that was one thing.  The other thing was that it seems like if you're doing a large content driven site, Django just makes that very easy.  Additionally, as most people say, one big win for Django is the built in Admin.  It has allowed me to focus on the front end of the site, while giving the other people involved a way to immediately start working on the content part.  Right away this helps us see where we may need to enhance functionality or perhaps we've built in too much flexibility.
That kind of feedback comes back to us quickly and so that is invaluable.

**Shabda:** How do you compare Rails to Django? In what areas is Django better than Rails (Apart from scaling/efficiency)? What does Django still need to learn from Rails?

**Michael:** That's a good question and not something I've given a lot of thought to, but off the top of my head I would say that with something like the Admin, Django definitely wins over Rails.  Now within the Rails community there are a lot of interesting [third-party](http://code.trebex.net/auto-admin) plugins that have attempted to mimic the Django admin, but up until now those that I've looked at have fallen short.  It's a huge effort as we've seen with the amount of work being put into the [NewForms-Admin rewrite](http://code.djangoproject.com/wiki/NewformsAdminBranch).
I also think the middleware stuff in Django is very nicely done, although it could use a bit more options in terms of request / response ordering of the middleware items.  I also think Django templates are just perfect.  I really can't see how to enhance on those at all.
As far as Rails is concerned, there's a lot of nice features in rails that Django could learn from, and most of it just has to do with the maturity of the two projects.
Rails caching takes caching one step further.  Rails has model level validations which are very nice and quite frankly the right way to do it, in my opinion.
[ActiveRecord](http://ar.rubyonrails.com/), Rails' ORM, has things like aggregation support, and supports a lot of flexibility in how you are able to filter the dependency relationships between your models with things like `has_many :through` (basically intermediate models).
I also like the `before_filter` and `after_filter` type of stuff in rails.  It makes it real easy to invalidate cache or do other things in an event driven way.
Oh one thing back on the Django side, I think NewForms are done really well.  It's an elegant solution.  And it is interesting because in the .net world I'm starting to see a copy of that through things like [dynamic data controls](http://quickstarts.asp.net/Futures/dynamicdatacontrols/default.aspx).

**Shabda:** One specific area which I believe Django can learn from Rails is marketing. What would you say to this? With 1.0 release coming soon, how can Django start to market itself better.

**Michael:** I would agree with you on this.  I think the impending 1.0 release will definitely do a lot on its own to market the product.  There are also at least 3 more books coming out in the next several months.  The [Google App Engine](http://code.google.com/appengine/) announcement that heavily featured the name of Django has also helped to bring it focus.
But there's another element that I don't know if you can fabricate and that is that [DHH](http://loudthinking.com) is a charismatic individual.
In terms of real things that can be done are obviously things like more screencasts just featuring the product.  It's interesting to note that consistently the number 1 screencast on iShowU is the Django one that features a simple tutorial on Django.  Although a lot of people have no interest in screencasts, there are a lot of people that do.  We all learn in different ways.
I also think that on the djangoproject.com weblog we can do a much better job of regular blogging on what is going on within the community.
There are times when you look at the [weblog](http://www.djangoproject.com/weblog/) and it's 2 or 3 months old.  From the face of the website you would think that nothing is going.  The last release is over a year old.  Meanwhile those of us that are in the community every day, we see that tons of stuff is happening.  This needs to be communicated better.

**Shabda:** Talking of GAE, what effect do you see of it on Django? How can Django make the GAE-Django integration painless, or even if this should be done, and what efforts the Django community should expend on this instead of say focusing the efforts on 1.0 release?

**Michael:** I think GAE helps Django, but only a little bit.  It helps it from the standpoint of name recognition and will cause a lot of people to say to themselves, "what's this Django thing all about."  I think GAE really helps Python in a real way.
In terms of making the integration painless, that's going to be a very difficult task.  I have a taste of that with the django-sqlalchemy project I'm working on, and I'm just mapping a relational model to another relational model.  With GAE it's quite different.  That said, Google folks are working on on a project to do "some" [mapping](http://code.google.com/p/google-app-engine-django/) of Django to GAE.  I think over time it will expand in its focus.  I think if they are willing they are the best people to do the work.  I'd rather see Django focus on getting to that 1.0 point.
As far as GAE itself is concerned, I'm kind of on the fence on the real value there.  I know that people like [Jaiku](http://jaiku.com/) are porting their stuff over to it, and Kevin Rose said that it would have been a great platform for something like Digg. Personally, I'm not really convinced of that, but it's still early so we'll all have to wait and see what comes of it.

**Shabda:** Your TWiD has been a great help for people to keep track of all the happenings in the community. Would you like to share some interesting tidbits you have learned with TWiD, and point to the more interesting ones?

<img src="http://uswaretech.com/blog/wp-content/uploads/2008/05/twid_small.png" alt="" title="twid_small" width="150" height="150" class="right" />
**Michael:** Thank you, I'm glad that people find it helpful.  I think one of the interesting things is the amount of work that goes into it.   A lot of people are surprised because neither Brian nor I are very professional, so when we're on the mic it sounds like we're just sitting there talking about whatever.  The reality is that it takes quite a bit of work in finding the stuff we want to discuss, attempting to understand enough about the topics to at least sound somewhat intelligent on them, and then there is the recording and post-production.
As far as actual topics the [two](http://blog.michaeltrier.com/2008/4/22/this-week-in-django-19-2008-04-20) [recent](http://blog.michaeltrier.com/2008/4/28/this-week-in-django-20-2008-04-27) episodes on Internationalisation have been a great learning experience for me personally.  Going into it, I really didn't even know enough about it to know what questions to ask. Thankfully [Malcolm Tredinnick](http://pointy-stick.com/blog/) put the whole thing together.  He's been a huge help to the show.
Another show that was fascinating for me was the one on [GeoDjango](http://code.djangoproject.com/wiki/GeoDjango) with [Matthew Wensing](http://djangopeople.net/wensing/).  It's a fascinating topic and I hope to put together more shows on that subject.
I think a lot of people like the interviews, and we do too, but we also don't want to make the show just interviews every week.  So we try to split it up.

**Shabda:** Would you tell a little about the venture you are working on?

**Michael:** Sure.  The project is a hyper-local media site.  We're working on  providing an online place for a mix of general media type of content (stories, events, etc...) with citizen journalism types of offerings.  A lot of people are working in this space, trying to figure out how you bring the big media type of stuff down to a local level.
We then want to mix some of that with interesting datasets, a la [EveryBlock](http://everyblock.com/), as well as provide some level of social interaction.  The cool thing is that once we release we're going to make the entire thing available on a New BSD license.
Frankly, initially it's not going to be that interesting.  A lot of what we'll provide in the way of Classifieds, Marketplace, Aggregators, stories, etc.. will be similar to the types of offerings that you see in things like [Ellington](http://www.ellingtoncms.com/) or just about any newspaper's online presence these days.  Down the road I hope that we can expand it into something quite useful.
The framework we'll be releasing is called ArEyah.  So expect to see something from me on that in the next several months.  Timeframes are tough to nail down at this point because this is a side project in addition to some of the other things I'm involved in.

**Shabda:** For many people, such as me, the question is what business advantage remains for you after you are releasing all your secret sauce under an open license. For example even EveryBlock will have to [release all their source](http://www.everyblock.com/about/faq/) under an open license, after the end of the [Knight Grant](http://www.knightfoundation.org/) expiry period. What would you say to this? What do you hope to achieve with releasing the project under a BSD license?

**Michael:** That's really a good question.  For me it was two-fold.  First I wanted to take the "intellectual value" of it off the table.  In other words, I did not want to be in a position with my partners where I had an extremely unfair advantage.  Secondly I really think this is something that could benefit communities everywhere; of course that remains to be seen.  If that is the case, then I think a lot of individuals would get involved and make it a better product for the benefit of all.
As far as competitive advantage, I really see that being in the execution.  In comes down to how well we serve our community.  If we are not doing a good job of it then someone else should be able to come along and beat us out there.  There were certainly be some things that are specific to the community that we're targeting that don't have a place in the general framework.  Those things will be our own and they will be highly tailored for our use, in order to serve a specific need we have here.

**Shabda:** We see a lot of comparison going on between Django and ROR, or Django and Turbogears. But we do not see enough comparison between Django and other traditionally 'Entrprisy' frameworks. Say Java based frameworks like Struts+Hibername or Asp.net. As you have worked with .net, how would you compare Django with these frameworks. What can Django do to make itself popular in the areas dominated by these frameworks?

**Michael:** The differences have less to do with feature set comparisons.  A lot of what I do every day for corporate clients could be done much more quickly and actually often more robustly with something like Django.  So it's not a thing where you can say feature x is in .net but not in Django.  That said like any framework / language there are edge case types of things where you might say "Java is the right tool for this job."
What it really comes down to is the corporate culture.  .NET and Java own those corporate cultures.  There are real reasons why it makes good business sense to build your corporate infrastructure on Microsoft products.  You can pick up the phone and get three .NET developers tomorrow.  Regardless of whether your company is based in Louisville, Kentucky or White Plains, New York.
In the case with Microsoft, much more than in the Java world, they provide a singular full-stack solution.  No one, in the executive sense, needs to make any more decisions about which reporting tool, which database backend, or which IDE they are going to decide to use.  Often that alone is motivation enough.

**Shabda:** Your last post on your blog was about the benefits of DVCS as compared to Centralized systems. What are the compelling benefits of DVCS over, say, SVN. In particular how can moving to a DVCS help Django?

**Michael:** Well I'm somewhat new to the DVCS world, as are a lot of people, so I'm no expert on the subject.  To me though the benefits of a DVCS are at more of a personal level.  I have seen tremendous personal benefit in being able to commit while sitting in a coffee shop, or being able to branch code locally and then merge that back in.
As far as benefit to a centralized project like Django, I guess it remains to be seen.  I've been watching the shift of Rails to Git with great interest.
So I guess in summary, I just don't know enough at this point to say that it would be beneficial. The shift is probably more psychological. I think, for a lot of projects, it would be just another way to have a centralized repository.

**Shabda:** You and Brian Rosner are working on [django-sqlalchemy](http://code.google.com/p/django-sqlalchemy/). How would a [sqlalchemy](http://www.sqlalchemy.org/) based ORM be better than the current implementation? What is the status of this project?

**Michael:** Yes, Brian and I are probably the primary committers, but we have several other individuals that have pitched in on the project.  I like to think about the benefits in two separate areas.  First, there's the approach where you just plug django-sqlalchemy in and continue to use Django with its filter syntax, etc...
In that case the benefits you gain are not benefits at the ORM level (because you're still using Django's syntax, which doesn't support things like aggregation).  The benefits instead are things provided as a result of having SQLAlchemy as the backend.  So this would be things like multi-database backends, [sharding](http://www.sqlalchemy.org/docs/04/sqlalchemy_orm_shard.html), additional database support like DB2 or Firebird.
The second approach is where you actually need things like aggregation or more complex queries, without resorting to raw sql.  In that case we expose SQLAclhemy's ORM right on your models.  So the full power of SQLAlchemey gets exposed and available for you to use whenever you need.
So I could technically have 90% of my ORM code just using Django's syntax but then realize that I need to do something a little outside of its capabilities.  In that situation I might chose to just use the exposed properties to get what I need.
Finally there's one more thing related to this that we still do not have clear at this point, but will come down the road, and that's actually adding different functionality at the model level.  This might be things like Intermediate Model support being made available through django-sqlalchemy.
As far as status of the project, it has been moving along very well.  We have a few more filters to implement and a handful of management commands, plus lots more testing.  One of the things I've done in the past week is to code up a test application using django-sqlalchemy to see in a "real-world" sense where some of the problems are.  That has been really helpful.

**Shabda:** So if the extra features added by SQLachemy is not needed, then this aims to be backwards compatible with current Django syntax?

**Michael:** Yes, our aim is that you could plug it in and run all your stuff just as is.  In other words with django-sqlalchemy as your backend db (that's how it gets exposed) we should be able to pass all of Django's test.  Once we're able to do that we tag it 1.0 and get some people hammering on it.
We do have one big hurdle that I have not discussed.  Currently we are using multiple inheritance to modify the Django classes.  That doesn't work for the contrib apps or third-party apps because that would require a change to their code base.
We took this approach originally just because we wanted to focus on the mapping issues first and prove the concept.
The eventual plan is to use some class replacement techniques to inject our stuff right into the Django models at evaluation time.  That will make it work across the board.

**Shabda:** Before we leave. Would you like to share a tip, or hard to find information about Django with our readers?

**Michael:** Not really a code tip, because we do those each week on TWiD, but more of an approach tip.  The generic views stuff is extremely powerful, and quite often I see a lot of new users doing stuff in views that could easily be done with a generic view, or with a wrapped generic view.  James Bennett has a [great post](http://www.b-list.org/weblog/2006/nov/16/django-tips-get-most-out-generic-views/) on this, and I suggest everyone check it out.  I think often, especially for people that are new to the community, the power of it is overlooked.
Finally one more thing, spend some time in [IRC](irc://irc.freenode.com/). IRC is a great way to get an education in Django very quickly.  Reading the questions and the responses has been invaluable to me in learning how to use Django more effectively. It's a great community.

**Shabda:** Thanks a ton for this great interview. It was extremely  informative and interesting.

**Michael:** Thank you.

--------------

This was the interview of Michael Trier. This week I am not going to any more interview, but stay tuned for next week, we have even more Django interviews coming.

And of course, the [42topics](http://42topics.com/) is live now. And we have a [Django](http://42topics.com/django/) section. ([How 42topics works](http://42topics.com/blog/how-42topics-works/)?)
So join now, and lets get rolling.

