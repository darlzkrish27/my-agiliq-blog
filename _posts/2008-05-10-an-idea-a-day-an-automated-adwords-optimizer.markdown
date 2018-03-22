---
layout: post
title:  "An idea a day - An automated Adwords optimizer"
date:   2008-05-10 20:27:49+05:30
categories: startup
author: shabda
---
This is an article in the **Five Startup Ideas** series at [the 42topics blog](http://42topics.com/). In his essay, [Ideas for Startups](http://www.paulgraham.com/ideas.html), Paul Graham argues that ideas are not a critical factor for success of startups. Although I do not believe that ideas are worthless, as many people do, I believe that they are not any where near as important as execution. So to prove my point, I am giving away 5 startup ideas in next five days. All of them describe a problem, its solution, the technology involved, the competition and market size. If you are not a hacker, and want to build any of these things may I suggest [Uswaretech](http://uswaretech.com/).

-----------------------

### Title:

<img src="http://uswaretech.com/blog/wp-content/uploads/2008/05/photo-adwords.jpg" alt="" title="photo-adwords" width="264" height="204" class="right" />
An automated Adwords optimizer

### The problem:

A lot of the advertisers are using Adwords for SEM. They have to manually keep track of the keywords they are optimizing on, the ad sales copy, the ROI on each keyword+Ad sales combination. For a lot of advertisers, who have a large inventory finding the keywords they want to advertise on, and keep track of is a big challenge. This manual process is very inefficient, drudgery filled and can be automated away.

### The solution:

Take the example of Google search for [Buy books](http://www.google.com/search?q=buy+books). A lot of advertisers are bidding on this keyword. Now take the results for [Buy harry potter](http://www.google.com/search?q=buy+harry+potter) no one is advertising on this keyword. Most of the advertisers would have Harry Potter in their stock, but can not afford to advertise on this as keeping track of each keyword and measuring ROI is unfeasible. So the Software need to do things,

1. Allow to easily track all items in advertisers inventory. Mix this with other commercial intent keywords and bid on them. For example if the advertisers inventory is ['Harry Potter', 'LOTR', 'Lord of the Flies', 'Kamsutra', ... 'Iacocca'] then with one click the advertiser bids on ['Buy Harry Potter', 'Buy LOTR', 'Buy Lord of the Flies', 'Buy Kamsutra', ... 'Buy Iacocca']. The inventory is pulled from the advertisers database.
2. Track the ROI on each keyword. For example Buy Harry Potter is profitable to the advertiser, but for some reason Buy LOTR is not. Automatically remove the keywords which do not perform. The costs are pulled from the advertisers database. The conversion ratio can be calculated by adding a javascript to 'Thanks you for the purchase' page.
3. There are a lot of other places where automated optimization can be done. For example people who are advertising on [buy books](http://www.google.com/search?q=buy+books) would also want to advertise on [buy boks](http://www.google.com/search?q=buy+boks), a misspelling, yet very few are. Automatically advertise on misspellings and track the ROI as in 2.

### Technologies involved:

You would want this to be a web based hosted service. So you can use any server side technology you want. ROR, [Django](http://www.uswaretech.com) any would do. The interesting part is that [Adwords has an API](http://www.google.com/apis/adwords/) using which you can interact with an Adwords account, and most things are possible. using the API is not free, but has a [liberal pricing](http://www.google.com/apis/adwords/quota.html#sheet), so this will not be a barrier.

### Existing Competition:

There are a few software in this area such as [Adgooroo](http://www.adgooroo.com/products/sem_insight.php), but it still requires a lot of manual intervention. There is still a lot of place for optimization and automation which can be done in this space.

### Market Size:

I am unable to get an exact breakdown of the market size of Adwords, but Google's financial information is [here](http://finance.google.com/finance?fstype=ii&q=NASDAQ:GOOG). Of the  5,186.04  million revenue for 3 month period a significant percentage are from Adwords program. (If you can find a better source for market size please let me know.)

### Others:

There is significant risk that if you build your software around the Adwords API, and your software leads to losses for Google, the API may be changed or removed. You need to find a way to make this software win-win for Google and the advertisers. This is possible if you find new areas the advertisers can advertise on.

-----------
This was part 1 of the series of 5 startup ideas. For next five days we will publish a new idea a day. If you want to read all of them, please [subscribe](http://42topics.com/blog/feed/). Oh and have you seen the [42topics startup](http://42topics.com/startups/) section? Or if you want you can [create](http://42topics.com/create/) your own topic.

And if have a question about this, or think this idea sucks leave a comment and I will reply to you queries.

