---
layout: post
title:  "An Interview with Adrian Holovaty - Creator of Django"
date:   2008-06-24 03:36:06+05:30
tags: interviews
author: shabda
---
[Adrian Holovaty](http://www.holovaty.com/) is the co-creator of Django, author of the Django book and is currently the BDFL of Django along with Jacob Kaplan-Moss. He studied journalism at [University of Missouriâ€“Columbia](http://www.missouri.edu/about/mufacts.php) and has worked with many news sites including [Lawrence Journal World](http://www2.ljworld.com/) and [Washington Post](http://www.washingtonpost.com/). He currently works at [EveryBlock](http://everyblock.com/), a startup he founded.

-----------------------------------------------------------------------------

**Shabda:** Would you tell a little about yourself? You majored in Journalism, how did you move to Programming?

**Adrian:** Sure! I have a dual background in journalism and computer science. I
intended to get a degree in journalism and a minor in computer
science, but things got a little off track. I heard from a
professional colleague that a certain Web site would probably hire me
if I graduated a semester early, so I hurriedly dropped my CS minor
and got special permission to graduate early -- but once the time came
to ask for that job, the company was in the middle of a hiring freeze
and didn't have a position for me. So, in hindsight, I should've stuck
with the minor, but things ended up working out OK.

Because journalism and computer science don't normally go together,
I've had some success in this silly little niche of employing Web
development in news organizations -- "journalism via computer
programming." Professionally, I've always worked as a Web developer at
news organizations, up until my current gig, which is to run
[EveryBlock](http://everyblock.com). But in its own way, it's a news
organization as well.

Aside from all of that, I'm quite into music, particularly gypsy jazz,
which is the music of Django Reinhardt (hence the name of the Web
framework). I just recently attended a [week-long gypsy jazz guitar
camp in Massachusetts](http://djangoinjune.com) and had a tremendous time
staying up until early hours of the morning jamming with great
musicians from various parts of the world. If money were no object, I
would love to play music full time. In the meantime, I'll settle for
posting [YouTube videos of my guitar playing](http://youtube.com/adrianholovaty), for an audience that seems to be
comprised mostly of 16-year-old boys who constantly pester me for
transcriptions of my arrangements.

I live with my wife in the beautiful city of Chicago.

**Shabda:** What are your current roles and responsibilities at Django?

**Adrian:** Well, I co-created Django back in 2003 with my friend [Simon Willison](http://simonwillison.net/)
while we were working together. Now, along with [Jacob Kaplan-Moss](http://www.jacobian.org/), I'm
Benevolent Dictator For Life of the project -- a title we shamelessly
ripped off from Guido/Python. This means that I have a hand in all
sorts of things, from high-level API design to checking in patches and
fixing bugs. I've touched probably every bit of the framework over the
years, including implementing the initial ORM (back when it was a code
generator!), pair-programming the original Django template system and
URLconf/view framework with Simon and building the original admin
application with design help from Wilson Miner.

One of the things I enjoy the most is writing documentation -- both
because I like technical writing and because I cannot stand bad
open-source documentation.

**Shabda:** Can you describe the philosophy behind Django. What are its overarching goals?

**Adrian:** A couple of years ago, I wrote the "[Design philosophies](http://www.djangoproject.com/documentation/design_philosophies/)" document,
which sums up the main points nicely.

Generally, the goal is to make Web development fast, fun and easy for
the developer, while keeping performance as fast as possible and code
as easy to understand as possible.

**Shabda:** How did the move at WorldOnline from PHP to Python happen? Why did you start from scratch (which finally lead to creation of Django), instead of using something like Zope?

**Adrian:** Simon and I had been big fans of [Mark Pilgrim](http://diveintomark.org/) (and still are!), and
we'd read his online book "[Dive Into Python](http://www.diveintopython.org/ )." This was around the same
time that the PHP "infrastructure" I'd developed had begun to feel
really, really crufty, so...one day we just decided to start using
Python! It's quite nice working at a small organization with a very
loose management structure; our boss, Rob Curley, was cool enough to
let the developers themselves decide which technologies to use, as
long as the work got done. "I don't care how the sausage is made," he
always used to say.

We looked at the existing Python Web frameworks/libraries, but none of
them felt 100% right to us. Simon and I are both quite into best
practices, such as clean URLs, proper use of GET vs. POST, etc., so we
were very picky in our analysis of those existing tools.

**Shabda:** Can you tell us about [Everyblock](http://everyblock.com/)? Why did you choose to create your own mapping engine instead of using something like Google Maps? How hard has it been creating a new mapping engine?

**Adrian:** EveryBlock is the project I lead for my day job. It's funded by a
two-year grant from the [Knight Foundation](http://www.knightfoundation.org/), and the goal is to
experiment with block-specific news -- that is, the site lets you
enter an address (currently only in Chicago, New York City and San
Francisco) and view recent news within a very tight radius of that
address. I'm pretty confident it's the most granular attempt at local
news ever attempted on the Web. Nobody else is crazy enough to do it,
I guess!

We do a TON of work collecting local information, normalizing it and
pulling it together for people in one place. A large part of what we
publish is government data such as crime reports, restaurant
inspections, building permits, business licenses...and even local
movie filmings. Much of this stuff either is buried in "deep Web"
government databases or has never before been available online. A
second part of what we do is detecting addresses and locations in news
articles and blog entries. Plus, we pull in various other geographic
Web stuff like Flickr photos and Yelp reviews.

All in all, our goal is to show you recent, geographically relevant
stuff that you might not have heard about. Say the local newspaper
wrote something about your neighborhood on page B-23 -- would you
really have noticed that article? Say there was an aggravated assault
on your block -- would you really have remembered to check your police
department's crime-reports Web site on a daily basis? That's the basic
philosophy.

Regarding our custom mapping engine...Let's face it: Google Maps is so
passe. As one of the original Google Maps mashup guys (I developed
[chicagocrime.org](http://chicagocrime.org) by reverse-engineering Google's JavaScript, *before*
the API was released), I have all the respect in the world for what
Google has done to invigorate the world of Web maps. But it's time to
take the next step. The Google Maps API doesn't give you any control
over the colors of the map tiles, or change the fonts in street
labels, or disable building footprints, or hide one-way street markers
or subway stations or bus stops or any of the other stuff that's
essentially hard-coded in the map. So, at EveryBlock, we rolled our
own maps so we could have much, much more fine-grained control over
all of these things. Not to mention it's a great way to differentiate
ourselves from the 2.5 million boring, same-old-yellow-blue-orange
Google Maps mashups out there.

Paul Smith from the EveryBlock team has written an article at [A List
Apart with more of the technical specifics](http://www.alistapart.com/articles/takecontrolofyourmaps).

**Shabda:** As per the [Knight grant](http://www.knightfoundation.org/grants/) rules, you would be releasing the code for Everyblock next year. Would you not be giving away your secret sauce? How do you plan to maintain competitive advantage once that code is freely
 avaliable under a permissive license?

**Adrian:** Our competitive advantage is that we're an incredible team, and I'm
sure we'll come up with a way to feed our families.

**Shabda:** What problems would you like Django to tackle after 1.0? What big features would you be most interested in having in Django, after it hits 1.0?

**Adrian:** Generally I'd like to add higher and higher levels of abstraction to
the framework. The Django admin application is a good example of that
-- it's not just an abstraction of an HTTP request; it's an
abstraction of an entire Web application! We should do more of those.

**Shabda:** What is one thing about Django which you absolutely love, and one thing which you think Django should do differently?

**Adrian:** I love the way URLconfs work -- like a table of contents for your Web
app. I also love template inheritance. I don't love the fact that
we're generally slow in keeping up with tickets and feature requests.

**Shabda:** A lot of people these days have to evaluate between Django and ROR. How can they make this decision? When should Adrian Holovaty use ROR? When should DHH use Django?

**Adrian:** My answer is simple: Try out both frameworks and see which one you like better.

**Shabda:** Does Django need to market itself differently? What can Django community do for this?

**Adrian:** I'm comfortable with the amount of attention (or lack thereof,
depending on your perspective) that our project gets. I'm comfortable
with the size of the community. I'm comfortable with the fact that the
right people have found out about it through word of mouth, books,
blogs, [Google App Engine](http://code.google.com/appengine/) or wherever else. If things continue at their
current rate, we'll continue to do just fine, as a healthy open-source
project.

The one thing I'd ask the community to do is to continue staying
civil, polite and approachable.

**Shabda:** What does the phrase 'journalism via computer programming' mean? How can these two divergent fields be tied together?

**Adrian:** "Journalism via computer programming," in my opinion, is when a
journalist writes code to tell a story. Instead of talking on the
phone with sources to gather facts, this could involve screen-scraping
Web sites to gather raw data. Instead of writing a newspaper article,
this could involve building a database-driven Web site.

Journalism has several subdisciplines -- photography, information
graphics, video. I advocate that computer programming should be
another one of those subdisciplines. Just as a newspaper employs
photographers and graphic artists, it should employ programmers who
help gather information and tell stories with it.

**Shabda:** How can traditional journalists do more 'journalism via computer programming'? How can programmers do more 'journalism via computer
programming'? Is it easier for Programmers to move to this field, or for
Journalists?

**Adrian:** I believe it's easier for programmers to become journalists than it is
for journalists to become programmers, but both sides need to gain an
appreciation for the other in order for this sort of thing to happen
more often. Fortunately, some news organizations are starting to hire
developers with this in mind, and some geeks are realizing journalism
is a great, (mostly) pure field that lets you improve the world
through information.

One concrete thing programmers can do is to look for jobs at news
organizations, which desperately need technical talent. Developers,
you will be loved, you will be treated like geniuses, and your
non-techie coworkers will be very easily impressed!

Another route programmers can take is to get training in journalism --
in fact, Northwestern University's journalism school is giving out
full-tuition scholarships for [programmers who want to learn
journalism](http://www.medill.northwestern.edu/admissions/programmers.html).

**Shabda:** Thanks Adrian.

-----------------------------------------------

This was Adrian's interview, the last in django-interviews series. But we have a lot of interesting things coming up, so [stay tuned](http://42topics.com/blog/feed/).

