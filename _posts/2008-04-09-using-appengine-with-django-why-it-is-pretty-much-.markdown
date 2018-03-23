---
layout: post
title:  "Using Appengine with Django, why it is pretty much unusable"
date:   2008-04-09 15:02:36+05:30
tags:	python
author: shabda
---
We are hard at work building [42topics.com](http://www.42topics.com/), and are looking at the best places to deploy this. So when I heard about Appengine, with out of box Django support(gasp!) I was delighted. After using it for a day, [and posting a tutorial](http://www.42topics.com/dumps/appengine/doc.html), I am so disappointed.

###Peeves in no particular order.

- Appengine is a very constrained environment, so out goes any chance to run background jobs.
- The ORM-API is very similar to Django, but yet the Django API is much better. `modelobj.filter('attr =', value1).filter('attr2 =', value2)` or `modelobj.filter(attr = value1, attr2 = value2)`. Putting entity level operations on `modelobj.manager` is much cleaner that making them classmethods, as argued [here](http://www.b-list.org/weblog/2008/feb/25/managers/).
- Very half baked documentation. Releasing without sufficient testing on windows. If you follow the *Getting started* guide, and are on windows, I hope you like [debugging regexes](http://groups.google.com/group/google-appengine/search?q=unbalanced+parenthesis&)
- NO JOINS? Ok so I can use obj.master to simlute this. Umm can I get GROUP BY? What about UNION? Admittedly Bigtable is not relational, but are you telling me Google built all their search without a way to simulate GROUP BY.
- PDB does not work on dev server. With Django's dev server, I put breakpoints with PDB all the time, and it works perfectly. With Appengine devserver, pdb will give you BDBQuit exceptions. I hope you enjoy debugging by reading logfiles.
- Modules used by [django](http://www.google.com/search?q=django+%22from+imp+import%22+site%3Acode.djangoproject.com&sourceid=navclient-ff&ie=UTF-8&rlz=1B3GGGL_enIN217IN217) are [empty](http://code.google.com/appengine/docs/python/purepython.html).
- You can not start a dbshell on the dev server. (And django-admin framework does not work). When I am coding, I write the data insertion views first, and then the other views. Whether the Insert views are working or not can be checked immediately, from the db shell, or from admin. Here until I write the other views, I can not check on my create view.

The other flaws I can live with, but a dbshell not working and pdb breakpoints raising exceptions make this unusable for me. I guess I will stick with Django on a normal web host, and look at EC2, when we really need to scale.

