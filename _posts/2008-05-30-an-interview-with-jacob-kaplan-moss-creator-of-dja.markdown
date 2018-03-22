---
layout: post
title:  "An Interview with Jacob Kaplan-Moss - Creator of Django"
date:   2008-05-30 12:03:18+05:30
categories: python
author: shabda
---
<img src="http://uswaretech.com/blog/wp-content/uploads/2008/05/jacob_headshot2.png" alt="Jacob Kaplan-Moss" title="jacob_headshot2" width="225" height="232" class="right frame" />
[Jacob Kaplan-Moss](http://djangopeople.net/jacobian/) is the co-creator of Django along with Adrian Holovaty, as well as the author of [the Django Book](http://www.amazon.com/Definitive-Guide-Django-Development-Right/dp/1590597257). He has been involved with Django since before it was called Django. He is currently employed at Whiskey Media where his job is hacking at Django. He blogs on [Jacobian.org](http://www.jacobian.org/). He graciously agreed to be interviewed at the 42topics blog.

--------------------------------

**Shabda:** Would you tell a little about yourself? How did you get started with Django? What other software/applications have you worked with. (Both OSS and otherwise)?

**Jacob:** So a bit about me: I grew up in Silicon Valley, so like many other geeks I got started with computers really early; I was programming professionally before I graduated high school. I didn't start out doing web development; in college I worked on video surveillance systems for airports, harbors, marinas, and highways.   That's where I found Python: I rewrote a Java-based camera controller in Python and haven't looked back since.
I started doing web development pretty seriously when I moved to New York and took a job for a design firm there. The job was pretty terrible, and the technologies were worse: the good sites were PHP, and there was a bunch of WebLogic crap that was absolute hell to maintain.
So in 2004 I saw the [job opening](http://www.holovaty.com/blog/archive/2004/06/29/1135) that [Simon](http://simonwillison.net/) and [Adrian](http://www.holovaty.com/) posted on their blogs  -- and jumped on it. Web development in Python seemed like a dream job, and it really was.
So that's how I ended up in Kansas working for the local paper. At that point I suppose I was using Django, though we didn't have a name for it yet: we just called it "The CMS".
I'd been getting more into Open Source all along, though I'd barely contributed (I think I got a single line into Python at one point years ago -- that was my largest contribution).

**Shabda:** What areas of Django have you worked most in? What are you current areas of focus?

**Jacob:** Well, at this point I've touched pretty much every part of Django at some point weather in the form of improvements, refactoring, or just small patches and bug fixes. I'm far from an expert in any particular area, though, so I'd say my main role is more holistic: I'm most concerned with making sure that Django "feels" correct and that APIs and conventions match across the framework.
Right now my current focus is documentation: I've been working on improving the structure and organization of the documentation so that it's easier to find what you're looking for. We've got something like 40,000 lines of documentation so organization and metadata is critical.
Now that I'm lucky enough to get to work on Django at work, I'll probably be able to take on some more big tasks like that in the future.

**Shabda:** One of the most loved things about Django is its [comprehensive documentation](http://www.djangoproject.com/documentation/). What is the motivation behind refactoring this? What is going to be the new organization? How is this progressing?

**Jacob:** Thanks -- I've always thought that Django's documentation set us apart from most Open Source projects, and I've been really proud of what we've done there. However, over the past year or so as the size of the documentation (and Django itself) grew I think we've slipped from "outstanding" to merely "above average." To us (the core maintainers and I) that's not acceptable.
The best place to read up on the project and goals is in a [post](http://groups.google.com/group/django-developers/msg/d31fa312da5a9ec3) I made to [Django-dev](http://groups.google.com/group/django-developers/) a couple of months ago. In a nutshell, I'm breaking up the documentation into smaller, more manageable chunks -- the current DB API is almost sixty printed pages! I'm also separating more high-level how-to's and topical guides from the detailed API references that get in the way when you're just learning.
There's some tool improvements going on under the hood that'll make the documentation easier to write, edit, maintain, and contribute to; hopefully that'll help decrease the number of undocumented features.
The work's pretty close to done, actually. I was trying to finish before I went on vacation a month ago, but didn't quite get there. So that means I need to roll in a month's worth of documentation improvements that happened to the current docs while I was gone, but that shouldn't take much time. You can expect to see the new docs rolled out online very soon.

**Shabda:** What does Whiskey Media, the startup you are currently associated with do? Is your role there developing Django, or are you associated with other day to day activities as well?

**Jacob:** I think I'll have to be a bit coy and not say very much about Whiskey Media; we'd rather let our actions speak for themselves than try to build some sort of artificial hype. I'll just say that we're trying to solve some of the big problems in web publishing; you can probably see why Django's the best technical bet for a company trying to be the on the cutting edge of content publishing.
Most of my time at Whiskey is devoted to working on Django -- coding, but also community management, evangelism, and organization. I also have some internal responsibilities, of course, but those are more nebulous: mostly I just help out wherever I'm needed.

**Shabda:** As you said "we'd rather let our actions speak for themselves than try to build some sort of artificial hype". Do you see Django taking a potential hit in marketing due to similar belief of Django community? Does Django need to market itself differently than it has been doing till now? What can Django community do for this?

**Jacob:** Huh, that's a tricky one. Well, let me break down the question a bit and look at a few different angles.
So first, I've always been a bit uncomfortable with the idea -- pervasive in Open Source -- that this is some sort of popularity contest. People are always comparing traffic figures, or numbers of job postings, or numbers of Google results, or whatever. I think the idea's supposed to be that if Project Foo has more users than Project Bar that Foo somehow "wins". But I don't see it that way at all.
I mean, I moved from New York City to Lawrence, KS. New York City has a population of, what, 8 million or so? Lawrence has a population of around 80,000. Does that mean New York City is "better"? To most of those 8 million, sure... but to me? Hell no.
As in most areas, there's no accounting for taste: one of my good friends here in Lawrence is a die-hard Perl fan, and though it makes him a bit twisted it doesn't mean that I'm somehow "better" than him -- we just have different needs and different ways of thinking.
All that said, though, there is undeniably a value in popularity.
Only a tiny number of people who use a given piece of Open Source software become involved in the community, and only a tiny fraction of those contribute back to the project, and only a tiny fraction of those contributors become long-term committers. So more users translates directly to more contributors, and more contributors brings more "value" to the project.
(By "value" I'm referring to code, good ideas, etc.)
So as you can probably tell this is something we struggle with. On one hand I think Django's great, and it's in the project's best interest to persuade others to give it a shot. But on the other hand there's a huge amount of bullshitting that goes on where software is concerned, and I try my best to not add to the pile.
It's hard to tell when you've crossed the line from evangelism to hype, and hype can be toxic. Witness the recent backlash against Ruby on Rails: do you think they'd get such violent vitriol if they'd been more modest in their promotion?

**Shabda:** Talking of <acronym title="Ruby on Rails">ROR</acronym>, a lot of people these days have to evaluate between Django and ROR. What questions should they ask themselves to answer this question? (Well apart from "do you know Python better or Ruby better?"). To make this more interesting when would YOU choose ROR over Django, if you knew both Ruby and Python equally well?

**Jacob:** So first let me back up a step -- I'll answer directly in a moment -- and make a quick point about web development in general. The past couple-three years have seen a radical improvement in the quality of tools available to web developers. It really wasn't that long ago that CGI represented the state-of-the-art in web development.
Today, though, there's really some fantastic tools available. [Rails](http://rubyonrails.com/), [Django](http://djangoproject.com/), [Symfony](http://www.symfony-project.org/), [Pylons](http://pylonshq.com/), [TurboGears](http://turbogears.org/), [Seaside](http://www.seaside.st/)... any of these tools represent a major improvement over the CGI/PHP model of web development. If you're still writing web sites the way you did five years ago, you're missing out.
When it comes to which of these tools to choose, of course, we're back to that question of taste. Each tool comes from a different world, and has a different "attitude" towards web development. The cool part, though, is that most new web development tools emphasize a quick start as a key feature: you could probably evaluate a dozen web development frameworks in just a couple of days. So my best advice is to to try a few and see what "clicks".
It's probably important to also pay attention to the language that the framework uses. One of the real pleasures of writing Django apps is that you get to take advantage of the awesome Python community. I've got a set of libraries in Python that I can't imagine developing web apps without; many of those libraries don't have analogues in any other environment.
So if I knew Python and Ruby equally well -- I don't -- I'd still probably lean towards Python, and towards Django. However, there's one place I can think of where Rails is far superior: Rails runs on the JVM. This is a big deal: there's any number of large corporate environments where the JVM is the only game in town. And obviously if I had to choose between Java and Ruby I'd choose Ruby!
I'll mention, though, that [Jython](http://www.jython.org/) (Python on the JVM) is improving by leaps and bounds, and that getting Django working perfectly on the JVM is one of the [Google Summer of Code projects](http://code.google.com/soc/2008/psf/appinfo.html?csaid=DA6AC3DE94E157E) the Jython team is sponsoring.

**Shabda:** Coming back to the marketing question.. For most of non hackers choosing the framework to use is a big question, and the decision they will make depend on various non-tech factors, such as availability of capable skilled people. You mentioned that you would choose Django over Rails due to the awesome Python libraries. For the long term survival and growth of Django, do you not think that an early capture of mindshare in developer community is important? For example I have pitched Django to a fair number of people, and I always have to start with "Django, what?" as compared to "Yeah, we are evaluating Rails for our requirements."

**Jacob:** Sure, there's definitely a value of having Django be familiar to your friendly local Pointy Haired Boss, so in that context a certain amount of advertising is important. There's nothing worse than being forced by management to use the wrong tool for the job. Keep in mind, though, that this cuts both ways: I'd feel pretty unhappy if there was a team using Django because someone read about it in CTO Monthly. I'd agree that we could be doing more in terms of "brand awareness" or something, but all in all I'm pretty happy with the size and quality of Django's community.

**Shabda:** There is not much information about the early days of Django, so a little about that. How did the move at [WorldOnline](http://www2.ljworld.com/) from PHP to Python happen? Why did you create a new framework, instead of reusing something like Zope?

**Jacob:** I wasn't yet at World Online when we moved from PHP to Python, but from what I understand it was a pretty typical change. Adrian and Simon got fed up with the pain and suffering wrought by PHP, and wanted something cleaner and -- most importantly -- something that would be easy to maintain. Python really shines here.
The main reason we ended up building our own framework was that we didn't know we were building a framework. We just wanted to "build cool shit" and, over time, we built tools to help us do that. It wasn't until we started showing it off that we realized we had something that could be used by other people.
This, by the way, is one of the reasons I think Django turned out so great. If you sit down one day and say, "I'm going to develop a framework!" you're almost certainly going to become an Architecture Astronaut, and if you ever actually finish the thing'll be so over designed nobody will want to use it. If, on the other hand, you simply try to solve real-world problems in a clean, obvious way, you'll eventually end up with a great tool.
Look at Rails, TurboGears, even PHP; they all started as simple libraries written by frustrated programmers just trying to get the job done.

**Shabda:** What are the overarching goals of Django? Again to make this more interesting, Here is a [quote from you](http://www.linkedin.com/in/jacobian). "My work as a core developer of Django focuses on giving anyone -- even (especially) non-programmers -- the tools to create dynamic, content-driven websites." Should not that be the job of something like [Wordpress](http://wordpress.org/), while Django should aim to give "programmers, but not non-programmers -- the tools to create dynamic, content-driven websites."

**Jacob:** Wordpress is great if you want to publish a blog, but what if you want a website to track a book collection, or sell tickets to concerts, or organize a local farmer's market? There's a great deal that Wordpress (and other single-purpose tools) can do, but the amazing thing about the web is just how wide-open the possibilities are. There's a whole world of possibilities out there, and the end-goal is to help anyone self-publish anything they can dream up.
One of the most fascinating aspects of the history of communication is how intertwined literacy is with social controls. For most of the history of humankind, literacy was strictly available to the elite. The Web created the greatest democratization of publishing ability in history, and has almost immediately turned into a battleground between traditional, centralized publishing and decentralized democratic publishing.
As a programmer, we don't personally play much of a role in this sea change, but I do see the lowering of the barrier to self-publishing as something we ought to continuously think about.

**Shabda:** If Django's goal is "and the end-goal is to help anyone self-publish ", does not that mean Django is trying to fill the niche of an Extensible CMS like [Drupal](http://drupal.org/), as compared to filling the gap left by PHP? Are non programmers really using Django, or are they using apps built with Django?

**Jacob:** No, you're right that Django's lower-level than something like Drupal. Django's not trying to be a CMS but to be a tool you could use to build your own CMS. It's easier to design your own content models than to shoehorn your publishing into a CMS limited by the ideas of its developers.
There are indeed quite a few non-programmers using Django, though in the long run I think the interesting trend is that more and more computer users are learning a little bit of programming -- enough to develop a site with Django, say.

**Shabda:** (Since I ask this to everyone!) What is one thing about Django which you absolutely love, and one thing which you think Django should do differently?

**Jacob:** I only get to choose one thing I love?
I think my favorite bit of Django is the [URLconf](http://www.djangoproject.com/documentation/url_dispatch/) system. I love that Django forces me to think about URL design as part of my application instead of some byproduct; I've always hated web tools that try to someone pretend that you're writing a desktop app. I'll admit to obsession over my URL design from time-to-time, but I really enjoy clean, semantic URLs.
As for something we should do differently: the assumption that you'll only use a [single database](http://code.djangoproject.com/wiki/MultipleDatabaseSupport) is a bad one, and needs to be fixed. Unfortunately, it's an assumption that we made really early on, which means that fixing it is going to be tricky. There's some smart people working on it right now so I've got high hopes, but I wish we'd not made that assumption to begin with.


**Shabda:** Django started as a in house project and was later open sourced. At your
current startup your major job is with Django itself. The Django book which
you wrote was released under a permissive license. How difficult is it to
convince people outside the hacker culture of the business value of open
source? What are the best ways to do this? How difficult was doing this at
World online?

**Jacob:**
These days, thankfully, the business value of Open Source is pretty
well-established. There really hasn't been a lot of "convincing"
necessary. For example, Apress didn't just agree to release the Django
Book under a permissive license: they actively encouraged it. You'd
have to ask them if it was "worth it" in terms of sales, but I'm sure
that being the first Google hit for "django book" didn't hurt!
Releasing Django at World Online is actually an interesting story. We
decided at the 2005 PyCon that we should Open Source some of our
software, and started building a business case for doing so. We
prepared a series of arguments -- open source will increase our
visibility and lead to easier sales and easier hiring, open source
will improve the quality of our software, etc. -- and took them to our
management. All of those things turnned out to be true, by the way,
but o our surprise, the argument that was the most effective was
actually a "moral" one. We talked about how Open Source had helped our
business (Apache, Linux, Python, etc.) and argued that it was time to
"give back" to the community. The World Company has always been a
company that's tried to be a conscientious part of our local community
here in Lawrence, and they really jumped at the chance to participate
in the global Open Source community.
I like to tell this story, by the way, because it really makes me
hopeful that this "hacker culture" is in fact compatible with business
culture. Fact is that most businesses *are* in fact concerned with
doing the "right thing" -- they just often don't know what that is
when it comes to technology. Of course the business case for Open
Source needs to be there, too -- and it is -- but I think there are a
lot of companies that'll jump on the chance to "give back."

**Shabda:** Before we close would you like to share any tips with us?
**Jacob:** If you don't read the [Django community aggregator](http://www.djangoproject.com/community/) you really should: there are some incredibly smart people blogging about Django and you'll learn something new from all of them.

---------------------
This was the interview of [Jacob Kaplan-Moss](http://www.jacobian.org/). We have a few more Django interviews coming, before we close this series of Django interviews, so [stay tuned](http://42topics.com/blog/feed/).

).

