---
layout: post
title:  "The Unfuddle Tutorial"
date:   2010-12-04 20:55:21+05:30
tags: tips
author: shabda
---
[Unfuddle](http://unfuddle.com/) is the tool we use for our non open source development. 
I have found it to be the best tool for Software Project management, in particular 
I think it is superior to Basecamp and Assembla. 

When you start using Unfuddle, the number of things can seem overwhelming. This tutorial should help you *Unfuddle the Unfuddle*.

------------------------------


##The Unfuddle Glossary

Unfuddle has,

* Projects: Top level Things which need to be done.
* User: People who are working on a given project.
* Ticket: What a `User` works on.
* Milestone: A timed list of `ticket`s which should be completed before this given time.
* Messages: A broadcast for all `user`s.
* Notebook: A wiki for storing documentation.
* Repository: Git or SVN for storing the code.
* Ticket Report: Reports about the 

## The tips

#### Work from a list.

All action item items should be a ticket for somebody. Everything which a `user` needs to work should be in his work queue.

#### Make it easy to see everyone's work items.

When we start a project we create project reports with every `user` ticket queue.

#### Manage cognitive overload

People should have less than 10 tickets in their queue. More than this leads to a paradox of choice, 
with making it hard to decide to work on. 5 is a good number of open tickets per person, with proving choices on what to work on next, 
without being overwhelming.

#### Prefer Unfuddle messages to email.

Unfuddle message threads are easier to refer to in conversation in tickets and other places, having a meaningful url.

#### Commit all your code.

All code should be in a central repo, which can be refered to from tickets and messages.

#### Commit messages should be meaningful.

This is a good commit message.

	Makes the color of visited links on Buy now page to Cyan
	Anonymous users can do a checkout using Paypal
	Refs #441, fixes #372

This will automatically attach this commit to the given tickets, making it easier to track why a change was introduced, and how did a bug get fixed.

#### Get all stakeholders on board.

Your project probably has many people who are acuustomed to keeping things on email. For them using a tool like Unfuddle would be a big change.
You need to get them on board to use Unfuddle. 


-------------------------------


This was first in a series of tutorials we would be publishing for using Unfuddle. Stay tuned.

d.

