---
layout: post
title:  "Interview with James Bennett - Django release manager"
date:   2008-04-27 11:54:11+05:30
categories: interviews
author: shabda
---
<img src="http://uswaretech.com/blog/wp-content/uploads/2008/04/james_photo-20080210.jpg" alt="" title="james_photo-20080210" width="250" height="188" class="frame right" />
[James Bennett](http://www.b-list.org/about/) is the release manager of Django, and a long time contributor. He works on [Ellington](http://www.ellingtoncms.com/), a <acronym title="content management system">CMS</acronym> designed for news organizations. His book, [Practical Django Projects](http://www.amazon.com/Practical-Django-Projects-Pratical/dp/1590599969 ), is being published by Apress, and is scheduled to hit bookshelves in June 2008. He graciously agreed to be interviewed at the 42topics.com blog. His blog, The B-List, can be found [here](http://www.b-list.org/).

**Shabda**: Would you tell something about yourself, how did you get started with Django, and what other OSS projects are you involved with?

**James**: I got into Django fairly soon after the initial public release; I'd been doing PHP and Perl work (mostly [Textpattern](http://textpattern.com/) on the PHP side, [Scoop](http://scoop.kuro5hin.org/) on the Perl side), and I was working on teaching myself Ruby and Rails because it looked interesting. But I'd always liked Python; it was just that there weren't a whole lot of good options for Python web development at that point. You could do [Zope](http://www.zope.org/), or you could do [Twisted](http://twistedmatrix.com/trac/), but they both had pretty steep learning curves when compared to the type of work I was doing, so it just wasn't worth it.
Django changed that; I think I did the [tutorial](http://www.djangoproject.com/documentation/tutorial01/) the day it was released, and I just fell in love.
These days most of my open-source time is devoted to Django, and to various [Django-based apps](http://code.google.com/u/ubernostrum/) I've written and released. Back in the day I used to do the occasional bit of PHP hacking, and I wrote some plugins for blogging engines, but none of that stuff's maintained anymore.

**Shabda**: What are your responsibilities as the release manager of Django? Who are the other core contributors of Django, and what are their current areas of focus?

**James**: My responsibility is largely bureaucracy. Within the project, I keep an eye on the various branches and their progress, try to stay on top of active/problem areas in the ticket tracker, and maintain a list of things that have to happen before 1.0.
Outside the project itself, I also stay in touch with people who are distributing Django -- Linux distros which package it, etc. -- and watch their Django-related bug reports. When there's a security fix, I also send out an advance notice to them that we're going to roll a release, and do the initial disclosure so they have some lead time to respond to that.
Outside of that, I also contribute to the documentation, as well as the occasional code patch, and I maintain the 0.91-bugfixes branch, which provides legacy support and security updates for some of the really ancient Django installs out there (our policy is to support the current release and two prior releases with security updates, which means 0.91, 0.95 and 0.96 right now).
The rest of the core team is just people everybody knows if they watch the developers' list or the Trac timeline: [Adrian](http://www.holovaty.com/) and [Jacob](http://www.jacobian.org/) are the lead guys, obviously. Then there's [Malcolm](http://www.pointy-stick.com/blog/), who you interviewed the other day, and who's carved out a pretty big niche for himself: he did the bulk of the Unicode work and he's doing queryset-refactor, he does a lot of maintenance on the i18n system, and he somehow still finds time to have a day job. I don't know how he does that.
Then there's [Russell Keith-Magee](http://djangopeople.net/freakboy3742/), (russelm in Trac), who's contributed all sorts of useful stuff. He's the go-to guy for testing and serialization, and just an all-around brilliant guy who fixes things left and right.
[Gary Wilson](http://gdub.wordpress.com/) and [Joseph Kocherhans](http://www.linkedin.com/in/jkocherhans) are also both big names you'll recognize from commit messages; Joseph used to work at the next desk over at World Online, but now he's up in Chicago with Adrian, working on [EveryBlock](http://www.everyblock.com/). He's done a lot of the grunt work on newforms and on laying groundwork for newforms-admin.
Ian Kelly and Matt Boersma keep our [Oracle support](http://www.djangoproject.com/documentation/databases/#oracle-notes) working, and do an amazing job of helping to track down and solve obscure problems with that.
And rounding out the core commit bits are Simon Willison and Luke Plant, who aren't as active these days but can still be seen popping up every so often.
Out on the branches, we've had a lot of work done on newforms-admin lately by Brian Rosner, who's been stepping up and helping to really get that sorted out, and there's been a lot of unsung UI work by Christian Metts, who's a designer at World Online but also a not-bad JavaScript programmer and can flaunt some Python when he feels like it.
And then there's the "gis" branch, [GeoDjango](http://code.djangoproject.com/wiki/GeoDjango), which is adding support for GIS -- spatial querying -- to the Django <acronym title="Object-relational mapping">ORM</acronym>.
Justin Bronn and Jeremy Dunck really kick-started that, Justin gave a nice demo of some of their work at PyCon this year.
And recently we've also been giving commit access to some of the translators, so they don't have to go through as much bureaucracy to submit updated translation files; that's sped some things up on the i18n front, because we're blessed with a large number of people who are willing to pitch in and do that work.
Oh, and I can't forget Wilson Miner; he doesn't flaunt it that often, but he's the guy who originally designed the Django admin, and he's been known to make the occasional tweak to fix CSS stuff.
(hope I didn't forget anybody there)

**Shabda**: Would GeoDjango be sometime merged with the trunk, or is it forever going to be a parallel branch to trunk?

**James**: The goal is that the <acronym title=" geographic information systems">GIS</acronym> branch will merge, sometime after queryset-refactor lands, and it's going to provide an application -- django.contrib.gis -- that you can use to enable spatial queries for your models. There's a pretty good writeup on the wiki page of how that works, and they've been tracking queryset-refactor because it helps make some of the custom query construction easier.
I don't know for certain if it'll hit trunk before Django 1.0, but it will not be a branch forever; they've put in a ton of work on that, and I'm looking forward to getting to use it :)

**Shabda**: Once Django hits 1.0, would 0.91 be end of lifed, but support for .96 and .95 continue?

**James**: Like I said, the policy is we provide security fixes for the current release plus the previous two, so 0.91 would sort of fall off there after Django 1.0. But there are a lot of legacy installs out there that are perfectly happy for now, and I wouldn't be surprised to see people unofficially continuing to submit patches and keep that maintained for as long as there are people who want to use 0.91.

**Shabda**: A little about you. You have majored in philosophy. I can think of one other, [Paul Graham](http://www.paulgraham.com/articles.html). Would you say, what you learnt from philosophy help with programming?

**James**: Well, I wouldn't say there's anything specific necessarily. But I think there's a big place for people with liberal-arts backgrounds to come to programming, and I think philosophy's a good path to do that.
If you look at a typical philosophy program, you're doing a lot of logic, a lot of critical analysis, a lot of abstract reasoning.
You have to get comfortable sooner or later with all sorts of formalisms that don't necessarily have any practical meaning, and that's very similar in a lot of ways to programming :)
And when you get right down to it, as programmers, about 90% of what we're paid to do is think: our job is to take a problem, analyze it, break it down into pieces and solve them.
And that's not terribly different from what you spend four years doing in a philosophy program.
I've actually joked about that a bit with some of my former professors, that I still get to argue as much as when I was doing philosophy, but the programming pays a lot better.
I do think, though, that there's a big need for that sort of thing; we don't really teach critical thinking anymore, and while it's a vital skill to have no matter what you do for a living, it's absolutely crucial to programming. So if you can get a good liberal-arts background where you've been taught how to look at things and pick them apart and analyze them, you can definitely do well as a programmer. Though it'd also be a good idea to take at least a few elective math courses...

**Shabda**: You have been working on Ellington, how is Ellington <acronym title="content management system">CMS</acronym> different from, say, a customized version of Drupal?

**James**: Well, they have some things in common: both are modular <acronym title="content management system">CMS</acronym>-style products, both are meant to be extensible.
But Ellington is really targeted from the ground up at news operations. There's all sorts of specialized stuff in there that's really optimized for the way a newsroom works, most of it culled from our experience as a newspaper, and of working with other papers.
So where with Drupal you'd really have to do a lot of customization because nobody's really done this kind of "news all the way through" version of Drupal, with Ellington it's there out of the box, and you hit the ground running.
I think we also have a very unique position because we are a news company, we've got a bunch of newspapers, some periodicals, TV news, etc.
And we work with those folks every day, we see how they do stuff, we hear about it when they run into problems, and so we're in a good spot to see just what a newsroom staff really needs out of their online platform.

**Shabda**: Wordpress is PHP's killer app, arguably Basecamp is <acronym title="Ruby on Rails">ROR</acronym>'s. What would you say Django's killer app is?

**James**: Honestly I don't know right now; I think the thing that Django wins on is that there doesn't have to be the One Big App that everybody uses, instead there's this huge blooming ecosystem of applications.
In a way, it's like asking what the "killer library" of the Python stdlib is; the killer feature is that you've got all that stuff available.
Though there are definitely some cool Django apps out there right now, and a lot more on the way. I think there's a different mentality, though, because in general Python people seem to keep their heads down and just get stuff done. So it may be you don't hear about some project until maybe they decide to do an open-space talk at PyCon or OSCON, and then you just get blown away.
There was a guy at PyCon who came up and showed me an app he's been developing, and I won't spoil it and give away what it is, but it made my jaw drop.
He'd taken something that's a really common software niche that's been dominated by these abominable products because it's not really a sexy thing to be doing, and just absolutely nailed it. Guy's probably gonna make millions.
But I don't know if there is a general-purpose "killer app" for Django right now, and I'm not really sure I want there to be one; people can see that sort of thing and think "oh, that's all it does". They look at, say, Ellington, and think "oh, this is only good for a newspaper-style <acronym title="content management system">CMS</acronym>", or they see Rails and Basecamp and think "oh, this is no good for me, I'm not doing a Web 2.0 thing". So in a way I'm kind of glad we don't have a "killer app" hanging over us and pigeonholing Django.
Though I should definitely give a shout out to [Review Board](http://code.google.com/p/reviewboard/); of the public Django apps I've seen, it's probably the coolest, and again takes something that's not usually sexy in terms of software and really nails it.

**Shabda**: As you said, "Python people seem to keep their heads down and just get stuff done", do you think Django needs to a better job marketing itself. For example, I have pitched Django to a fair number of people, and I always have to start with with "Django what?", as compared to say <acronym title="Ruby on Rails">ROR</acronym>, or PHP which seems to have a good brand recall.

**James**: Well, to some extent Django hasn't done a whole lot of explicit marketing because we're not at 1.0 yet, and I expect that after 1.0 it'll both be a lot easier to do marketing and that there will be more of it going on.
But at the same time, Django's doing pretty well as it is; Google's doing their [App Engine](http://code.google.com/appengine/) stuff with Django bundled, there are startups using it to build the next big thing, and it's even been quietly sneaking its way into some huge corporations.
Plus it seems like every time I turn around there's another book coming out.
I saw one yesterday at the bookstore downtown; I hadn't heard about it until that moment, but I think that makes five or six books that'll be out by the end of this year.
So I think that'll help. [Digital Web magazine](http://www.digital-web.com/articles/intro_to_django_helping_perfectionists_with_deadlines/) did a feature article on Django just recently, and I did an [article for Sitepoint](http://www.sitepoint.com/article/build-to-do-list-30-minutes) a while back as well.
I like to view this as the phase where we build up momentum until eventually Django is an unstoppable juggernaut and everybody's listening to gypsy jazz music ;).

**Shabda**: Everyone is pretty excited about your [coming book](http://www.amazon.com/Practical-Django-Projects-Pratical/dp/1590599969). When can we have it in book stores? Would you give a brief overviews of what's in it?

<img src="http://uswaretech.com/blog/wp-content/uploads/2008/04/practical_django_proj.jpg" alt="" title="practical_django_proj" width="240" height="240" class="right" />
**James**: The book will, I think (and hope) be shipping around the end of June.
It's very much a hands-on introduction to Django: walking through building three applications, picking up progressively more advanced bits of Django as you go, and seeing some best practices in action.
So you start out with just simple stuff, using the contrib apps and learning the basics of getting Django running. Then you do some simple customizations of admin templates, then start building some models and views, then on into building full-on applications.
There's not room to cover every single thing you can do with Django, but I think it handles a pretty good spectrum of techniques from basics up to some advanced things that let you poke around and really get a feel for how stuff works.
And of course, you get periodic bouts of me up on my soapbox yelling at people about how to write reusable applications, because that's [what I do](http://www.b-list.org/weblog/2007/mar/27/reusable-django-apps/).

**Shabda**: You talk a little about App Engine in [Batteries sold separately](http://www.b-list.org/weblog/2008/apr/08/batteries-sold-separately/). What effect do you see of App Engine on python hosting ecosystem, on Python web applications, and on Django?

**James**: I honestly don't know what to expect from App Engine. It's a very different kind of thing from what anybody outside Google is used to, and it'll probably be a while before it's really shaken out and we get an idea of the impact it'll have. I fully expect that Google's going to start supporting other languages, so there won't be this effect where you'll always have to use Python to use App Engine. And the way they've sandboxed it is going to make it feel weird to Python people, I think. And that's on top of getting used to the fact that you're not using an RDBMS; I've been watching the blog flamewars about that, and that seems to be the big takeaway.
If I had to make a prediction now, I'd say that App Engine will bring some people to Python, but probably not in hordes, and that its big long-term effect is going to be to point out to a lot of folks that they're not really using the "<acronym title="relational">R</acronym>" in "<acronym title="Relational database management system">RDBMS</acronym>", and so maybe it's OK to think about their applications in a different way. And that's not Python-specific at all.

**Shabda**: What were the focus areas of the [newforms-admin](http://code.djangoproject.com/wiki/NewformsAdminBranch) branch. How far have they been achieved. About when would the newforms-admin branch be merged? How backwards incompatible is this going to be?

**James**: Well, there are a couple goals running parallel.
The first thing, obviously, is that with the [oldforms](http://www.djangoproject.com/documentation/forms/) package deprecated we've got to get stuff migrated to [newforms](http://www.djangoproject.com/documentation/newforms/), and the admin's got to make that jump.
And because the admin does a lot of tricky and complex stuff with forms, it's also been a fertile proving ground for designing some advanced newforms features that'll find their way back into trunk and make it easier to do that tricky stuff on your own.
Another big thing is cleaning out the coupling issue where you declare a class inside a model to activate the admin interface; there's no reason for that to happen, so instead there's a class you'll subclass and override stuff on to customize the behavior, and then you'll register a model to have the admin interface you've set up that way.
Finally, there's been a lot of stuff abstracted in newforms-admin; while I don't think it's really an official design goal, there's been a lot of opportunity to provide hooks where people can go in and customize stuff. I looked at the code a couple months ago, and it was about 95% of the way to being able to run completely without django.contrib.auth, for example, because there are enough hooks where you can override something and replace defaults to make it use something else.
And that's a huge deal, because it means you could do stuff like run the admin on HTTP auth, or run it in a corporate environment against an <acronym title="Lightweight Directory Access Protocol">LDAP</acronym> database, and you barely have to touch anything; you can already do an auth backend that handles most of the work, then you tweak a few things for the admin and you're good to go with no hacking directly on Django code.
Plus, a lot of people will be happy to see that the methods you can go in and override all get the HttpRequest object as an argument, so even though it's bad workflow a lot of the time you'll be able to have the admin do stuff like automatically fill in a foreign key with the current user.
And along the way there've been huge numbers of bug fixes; there's a lot of stuff that's always just been really fragile in the admin, like edit_inline, and newforms-admin is a good opportunity to just yank out all the things that made them troublesome and do it right.

**Shabda**: Time for a meme. What is the one think you absolutely love about Django, and one thing which Django should have done differently?

**James**: I love [contrib.localflavor](http://code.djangoproject.com/browser/django/trunk/django/contrib/localflavor), and I want people to go look at it and adore it and use it. There's so much useful stuff in there, you can validate checksums on Scandinavian social-security numbers, you can get lists of all the state in Brazil, it's just this great gigantic resource for localizing your apps, and it kicks so much ass.
Something to do differently... I'd probably rewrite the tutorial to not do the apps-inside-projects thing. That trips a lot of people up because they come away thinking they have to do it that way, and so they really miss out on the benefits of being able to write an app once and just use it over and over and over.

**Shabda**: Before we leave, would you like to give a quick tip, or hard to find information about Django to our readers?

**James**: The best tip I can give is not to be afraid to look at [code](http://code.djangoproject.com/browser/django/trunk/django/). We do our best to document all the useful stuff, but that's a pretty huge area to cover, so there are always going to be neat little things hidden away that you either won't see unless you read the code, or won't see until some date in the far future when somebody gets time to document it.
Plus, if you find something cool you can write up a little tutorial and post it, and then people will start reading your blog. That's pretty much what happened to me, where my blog was for a long time just me writing up these little articles to remind myself of stuff I'd learned.
And especially if you're learning Python as you go, reading a significant piece of code like Django can really help to improve your understanding of the language. There's really nothing better than that for assimilating the way a language works, and I think Django's around the right size -- it's small enough you can carry a solid understanding of it around in your head, but it's big enough that there's lots of stuff, and lots of different kinds of stuff, that you can look at and learn from.

**Shabda**: Thanks a ton for this interview, and for sharing these useful information.

**James**: Sure.

-------------------------------
This was James Bennett's interview. I plan to be interviewing a few more leaders from Django community, so if you would like me to ask any question, write them in the comments.

