---
layout: post
title:  "Book Review: Pragmatic Guide to JavaScript"
date:   2010-12-26 11:40:11+05:30
tags: reviews
author: shabda
---
[Pragmatic Guide to JavaScript](http://www.pragprog.com/titles/pg_js/pragmatic-guide-to-javascript) is the [Pragprog's book](http://www.pragprog.com/) by [Christophe Porteneuve](http://www.thebungeebook.net/) which I have
wanted to read for a while.

I went into this book expecting to read actionable and task focussed book
on modern JavaScript, and this book does not disappoint on that count. Using
35 tasks which you are *going* to need in your browser based JS development,
this books teaches required JS techniques.

However I went into the book expecting a framework agnostic approach. This book however
is very heavily focussed on [Pototype](http://www.prototypejs.org/). Initial chapters start as framework agnostic,
but they soon turn into Prototype based. [1]

[Chapter wise it is](http://media.pragprog.com/titles/pg_js/toc.pdf):

* Bread and butter: pure JavaScript
* The DOM, events, and timers
* UI tricks
* Form-fu
* Talking with the server side
* Making mashups


### Bread and butter: pure JavaScript

Has some basic tasks around managing namespace pollution and named and optional parameters.
Framework agnostic.

### The DOM, events, and timers

Tasks around getting and manipulating Dom elements. Has examples is Prototype, Jquery, YUI, Dojo
and MooTools.

### UI tricks

Task around common UI tasks like tooltips, lightboxes and unobtrusive popups. Becomes Prototype based
from this point onwards.

### Form-fu

Tasks around form validation, conditionally disabling elements and auto-complete.

### Talking with the server side

Task around understanding Ajax and cross domain Ajax using JSON-P and other techniques.

### Making mashups

Tasks around making mashups using Twitter and Flickr API. Also has discussion and history of various
common JS frameworks.

This chapter has sub-chapter on debugging JavaScript using Firebug and other browser specific tools. This is
the best text I have read on debugging JavaScript, and if you are working with browser specific JS bugs, 
this chapter is a must read.

-------------------------------

Overall this book is good read if

1. You work with prototype.
2. Work with other libraries, but would like to know about debugging JS in various browsers.

It is a pass if

1. You work with Jquery/Other libraries and want a book tailored to that.
2. Want a language agnostic book.

[Get it here](http://www.pragprog.com/titles/pg_js/pragmatic-guide-to-javascript)

------------------------------------

[1] It tries to get around this problem by putting code on Github and asking people to fork and create other framework translations. [https://github.com/tdd/pragmatic-javascript](https://github.com/tdd/pragmatic-javascript)



