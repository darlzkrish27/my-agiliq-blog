---
layout: post
title:  "Python Wrapper on Bing API"
date:   2009-06-10 12:09:33+05:30
tags: API
author: lakshman
---
The newly launched search engine [Bing](http://www.bing.com/) has a simple restful API. We have created a thin Python wrapper over this API, which allows to query the Bing servers in a very pythonic way.

Installing this is as easy as `easy_install bingapi`.

[Or get it [here](https://svn.uswaretech.com/bingapi/) or [here](http://pypi.python.org/pypi/bingapi/0.01) ]

Using

    from bingapi import bingapi
    bing = bingapi.Bing(<appid>)
    bing.do_web_search('Usware Technologies')

----------------------------------

The README from the project is posted below, which provides more details on using this.


bingapi.py is a very thin python wrapper over the Bing API.
Bing provides a very simple Restful interface to their search engine
and provides results in JSON and XML interface.

With bingapi.py, we query the rest urls to get the JSON response,
which is parsed with simplejson.py. bingapi.py just adds simple niceties
like logging and error handling.

### Installing bingapi.py

    easy_install bingapi

or Svn it from `https://svn.uswaretech.com/bingapi/`

### Using bingapi.py

To use this library, you need to have an Appid from Bing. You can get it from
http://www.bing.com/developers/createapp.aspx

The meat of bingapi.py is `talk_to_bing` function. It takes a query(eg. salsa)
and a sources(eg. web) and other optional parameters in extra_args
(eg. {'web.offset': 40}) and returns the dictionary of Bing's response.

eg
    
    In [2]: from bingapi import bingapi
    
    In [3]: bing = bingapi.Bing('<Your Appid>')
    
    In [4]: bing.talk_to_bing('salsa', sources='web')
    Out[4]: 
    ..........
    
    In [5]: bing.talk_to_bing('salsa', sources='web', extra_args={'web.offset':40})
    Out[5]: 
    ..........

Class `Bing` also provides utility functions to do the various types of seraches
so you dont have to remember the source type.

The various functions available are,

    do_web_search
    do_image_search
    do_news_search
    do_spell_search
    do_related_search
    do_phonebook_search
    do_answers_search

All of them are used similarly.

    In [6]: bing.do_web_search('salsa')
    Out[6]: 
    ..........
    
    In [7]: bing.do_image_search('salsa')
    Out[7]: 
    ..............
    
    In [8]: bing.do_answers_search('what is salsa')
    Out[8]: 
    ...............

If you want to use multiple sources in one call you can use `talk_to_bing`
directly as

    In [10]: bing.talk_to_bing(query='salsa', sources='web news')
    Out[10]:
    ............

But, but.. Why bing? you ask. Well, other than the fact that bing search is indeed better in some areas, particularly multimedia, bing has fairer [usage quota and API restrictions](http://www.bing.com/community/blogs/developer/archive/2009/05/28/announcing-the-new-live-search-api-version-2-0-beta.aspx). 

### Resources

* [http://www.bing.com/developers/](http://www.bing.com/developers/)
* [http://pypi.python.org/pypi/bingapi/0.01](http://pypi.python.org/pypi/bingapi/0.01)
    

---------------

Developing a search mashup? Bing API? [Yahoo](http://uswaretech.com/blog/2008/04/new-tutorial-building-a-search-engine-with-appengine-and-yahoo/), Google APIs? [We can help](http://uswaretech.com/contact/).

t/).

