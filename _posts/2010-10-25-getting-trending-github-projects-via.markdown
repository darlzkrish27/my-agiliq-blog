---
layout: post
title:  "Getting trending Github projects via YQL"
date:   2010-10-25 01:57:33+05:30
tags: API
author: shabda
---
[Github.com/explore](http://github.com/explore) allows you to see the trending Github topics. They use [repopular.com](http://repopular.com/) for this,
which use twitter retweets to find out the popular Github repos.

Since neither Repopular, nor Github have a RSS of these trending repos, I wanted to get a list of these. Here is how easy it is with 
[YQL](http://developer.yahoo.com/yql/).

<script src="http://gist.github.com/643959.js?file=repopular_projects.py"></script>


How we do it
------------------------

* Go to [YQL console](http://developer.yahoo.com/yql/console/?q=use%20'http://yqlblog.net/samples/data.html.cssselect.xml'%20as%20data.html.cssselect;%20select%20*%20from%20data.html.cssselect%20where%20url%3D%22www.yahoo.com%22%20and%20css%3D%22%23news%20a%22#h=use%20%27http%3A//yqlblog.net/samples/data.html.cssselect.xml%27%20as%20data.html.cssselect%3B%20select%20*%20from%20data.html.cssselect%20where%20url%3D%22repopular.com%22%20and%20css%3D%22div.bd%20a%22). Give the SQL query to get the data from the webpage. 

* `where url="repopular.com" and css="div.pad a"` is the magic which select the webpage, and also what DOM elemenst we are interested in.

* We get this data in JSON format which is munged to get the list of links.

* This list of links is passed via `is_github_project` which gets me just the Github projects.

* And we are done.


<script src="http://gist.github.com/643970.js?file=gistfile1.txt"></script>



PS. [YQL](http://developer.yahoo.com/yql/) is amazing.

.

