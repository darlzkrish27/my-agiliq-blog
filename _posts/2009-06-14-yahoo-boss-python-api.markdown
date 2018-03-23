---
layout: post
title:  "Yahoo BOSS python api"
date:   2009-06-14 21:47:51+05:30
tags: yahoo
author: lakshman
---
Yahoo has a search api with generous limits, [BOSS](http://developer.yahoo.com/search/boss/). There are a few [python](http://www.google.co.in/search?q=boss+python+api) [apis](http://pysearch.sourceforge.net/) around it. But we wanted a lighter api, and one which has the same interface as out Bing Python api. So here is the updated [bingapi](http://pypi.python.org/pypi/bingapi/0.02).(Now with bossapi.py as well). Or svn it from [here](https://svn.uswaretech.com/bingapi/)

-------------

### Usage

    Usage is mostly compatible with bingapi
    
    In [2]: from bingapi import bossapi
    
    In [3]: api = bossapi.Boss('<appid>')
    
    In [4]: api.do_web_search('Uswaretech')
    Out[4]: ....
    
    In [5]: api.do_news_search('salsa')
    Out[5]: ...
    
    In [6]: api.do_siteexplorer_search('http://uswaretech.com')
    Out[6]: .....






