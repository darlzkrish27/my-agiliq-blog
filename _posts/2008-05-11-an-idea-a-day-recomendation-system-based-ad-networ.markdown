---
layout: post
title:  "An idea a day - Recomendation system based ad network"
date:   2008-05-11 20:40:51+05:30
categories: startup
author: shabda
---
This is an article in the **Five Startup Ideas** series at [the 42topics blog](http://42topics.com/). In his essay, [Ideas for Startups](http://www.paulgraham.com/ideas.html), Paul Graham argues that ideas are not a critical factor for success of startups. Although I do not believe that ideas are worthless, as many people do, I believe that they are not any where near as important as execution. So to prove my point, I am giving away 5 startup ideas in next five days. All of them describe a problem, its solution, the technology involved, the competition and market size. If you are not a hacker, and want to build any of these things may I suggest [Uswaretech](http://uswaretech.com/).


### Title:

Ad network which takes into account user feedback.

### The problem:

Today Adsense is the default Ad network. If a person has a website they just slap an adsense ad unit on the page, and try to get a few bucks.

Adsense is a contextual advertising solution. It read the page and tries to find the area the page is about. For example, try [this site](http://www.frihost.com/forums/). From reading this site, google decided that this is about web hosting, and showed ads about web hosting.

The problem with this is that this site is about free web hosting. So no person on that site is clicking on paid web hosting ads, and the site owner in not making any money. Assume now that the demographic of this site is gamers. So if you showed ads about games, the ad revenue for the site owner can be potentially much higher.

Of course you need to do this algorithmically. So you need to take users feedback into account. See solution for how this can be done.

### The solution:

There are a few ways to find out what ads will convert for a given webpage. When the ad unit is put up on the page, show a random collection of ads. After a few clicks have happened, you can pin down with reasonable accuracy the niches from which these clicks happened. Now start showing ads from that niche, and keep track of which subniches convert best. Soon you can find out which niches, and ads are best performing and show them.

When you are just starting you would not have large inventory of ads to show on all pages. So you need to show ads from other networks as well. [Amazon](http://www.amazon.com/E-Commerce-Service-AWS-home-page/b?ie=UTF8&node=12738641) has an API, so does [Ebay](http://developer.ebay.com/common/api/), [Adsense](http://code.google.com/apis/adsense/) has one too, though I am not sure it has what you would be looking for. You can use these to show ads your members would be interested in.

### Technologies involved:

Use any server side technology ROR, [Django](http://uswaretech.com/), or if your feeling adventurous J2EE. All of the services mentioned above have SOAP or REST api. So any programming language won't be a problem.

### Existing Competition:

You are essentially trying to mix Recommendation System with Ad networks. There is no ad network which I know which does this. There have been a few ad networks which have used the Ebay shopping API to create such an ad network, most notably [Shopping Ads](http://shoppingads.com/), but you still need to tell the system which area your site targets, and then it shows ads from this area.
This system of asking which ads work best for a site cab be automated away, or can be used  to make a first guess, but an automated system can perform much better than asking humans for each page in site about this.

### Market Size:

As in my previous post, I am unable to find the exact size of market here, but Google's financial information is [here](http://finance.google.com/finance?fstype=ii&q=NASDAQ:GOOG). Also the whole internet is based on advertising, so the market size is big.

### Others:

None

------------

This was part 2 of the series of 5 startup ideas. For next five days we will publish a new idea a day. If you want to read all of them, please [subscribe](http://42topics.com/blog/feed/). Oh and have you seen the [42topics startup](http://42topics.com/startups/)  section? Or if you want you can [create](http://42topics.com/create/) your own topic.

