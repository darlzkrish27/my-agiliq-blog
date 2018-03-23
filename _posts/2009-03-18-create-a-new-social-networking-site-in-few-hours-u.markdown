---
layout: post
title:  "Create a new social networking site in few hours using pinax platform (django)."
date:   2009-03-18 17:03:20+05:30
tags: pinax
author: rama
---
if you are excited to see how such a social networking site would like.Just follow the below link

[http://www.xfgx.info/](http://www.xfgx.info/)

We made the above site on pinax platform (Please note pinax has done most of things we have done very little).A platform for rapidly developing websites.

In this blogpost we will try to understand what pinax is? How it helps a developer to do things quickly? How to install & deploy?  and various other things related to pinax.

##  What is Pinax?

In the present day world every website require  components  like regisration,openid support,some kind groups/ tribes,user profiles etc.....Almost every site has to code logic for these components. As they are reusable across many sites what if we have a platform (means a django project) which provides all these reusable components outof box and asks the developer to built on top of it.Such a platform  help the developers to rapidly build the websites , it helps them to focus on core aspects of their site.

Pinax is such a (django) platform which provides a collection of integrated reusable django apps.Out of box pinax provides several reusable components/apps  like

1. openid support

2. contact import (from vCard, Google or Yahoo)

3. notification framework etc...

More details at [http://pinaxproject.com/](http://pinaxproject.com/)

If you still haven't understood please look at the talks given by James Tauber (creator of piinax) at

1. <object width="425" height="344"><param name="movie" value="http://www.youtube.com/v/1J91Ownq-7g&hl=en&fs=1"></param><param name="allowFullScreen" value="true"></param><param name="allowscriptaccess" value="always"></param><embed src="http://www.youtube.com/v/1J91Ownq-7g&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed></object>

2. [http://thisweekindjango.com/twid/episode/24/this-week-in-django-24/](http://thisweekindjango.com/twid/episode/24/this-week-in-django-24/)

Let us dig into more details like how to install and deploy pinax,

## How to install pinax?

     Get the latest version of pinax
     1.svn checkout http://svn.pinaxproject.com/pinax/trunk/   pinax
     2.cp pinax/projects/complete_project    ../../complete_project
     3.mv compete_project myprojectname

1. First check out the latest version of pinax.
2. Then get into the pinax/projects directory where you will find several projects like
complete_project , basic_project.

* Complete_project : provides all reusable apps listed on pinaxproject website.You need to build on top of this project if the application which you are going to develop requires all or many of these.

* basic_project : it provides only very basic reusable apps  like registration,login,openid support etc.You need to build on top of this project if you just requires
only registrationa and login support.

etc....

3.Take out the project on which you want to built (it can be complete project or basic project or others depending on your needs) to the another directory where your  django projects are there.

####  Note : After step 3 if you are installing stable version  [pinax0.5.1](http://downloads.pinaxproject.com/pinax-0.5.1.tar.gz) instead of latest version at trunk , then you should follow the installation instructions given at the bottom of this blog.
<br />

    4.sudo easy_install pip
    5.sudo easy_install virtualenv
    6.sudo apt-get install git-core
    7.sudo apt-get install bzr
    8.sudo pip install -r pinax/requirements/external_apps.txt

In previous versions pinax is using svn:externals to bring third party apps which  merge into the pinax source code.But now pinax team has moved from svn:externals to pip.

More details : [http://code.google.com/p/django-hotclub/wiki/MovingToDistutils](http://code.google.com/p/django-hotclub/wiki/MovingToDistutils)

### What is pip?
pip is similar to easy_install.easy_install allows you to install several python pakages.Similarly pip allows you to install third party apps and also python packages.

4.First install pip,virtualevn (isolated Python environments)

Please refer this url [http://blog.ianbicking.org/2008/12/14/a-few-corrections-to-on-packaging/](http://blog.ianbicking.org/2008/12/14/a-few-corrections-to-on-packaging/)  for more info on pip and virtualenv

5,6,7.You also need to install git and bzr repositories.

Please refer this url for more info [http://iraniweb.com/blog/?cat=17](http://iraniweb.com/blog/?cat=17)

8.using pip install all the third party external apps which are listed in pinax/requirements/external_apps.txt file.

    10 cd myprojectname (which you created in step 3)
    11.change PINAX_ROOT & ROOT_URLCONF in setting.py
    12 python manage.py syncdb
    13.python manage.py runserver

11.change myprojectname.settings.py as follows

1. PINAX_ROOT = "path to pinax directory which was created in step 1".
2. For ROOT_URLCONF please change it from complete_project.urls to urls.

Now type http://localhost:8000/ you will see pinax project screens, lots of common functionality is being provided out of box by pinax platform.

## How to remove an application from pinax?
Apps are independent of one another.It is the project which integrates all this django reusable applications.You can remove an app
by first removing the respective app urls in  urls.py and changing several templates to reflect this.

## How to add another application to pinax?
Just put it on the python path and add in  settings.INSTALLED_APPS tuple.

More details about installation,deployment at
[http://pinaxproject.com/docs/0.5.1/deployment.html](http://pinaxproject.com/docs/0.5.1/deployment.html)

## How to deploy?
 Deploying a pinax application  is also very easy if you are using mod_python just add the following to your httpd.conf file

    <Location "/">;
    SetHandler python-program
    PythonHandler myprojectname.deploy.modpython
    SetEnv DJANGO_SETTINGS_MODULE complete_project.settings
    PythonDebug On
    PythonPath "['path to directory which contains your project (created in step3)','path to pinax project which     you   
    made in step 3"] + sys.path"
    <Location>

* Note that  PthyonHandler is myprojectname.deploy.modpython instead of django.core.handlers.modpython

* Please note that portions of media is being served from  myprojectname (created in step 3) and from the pinax directory(Created in step 1, myprojectname has reference to that).Please take a note of this while creating symlinks for the media.

Have you every used pinax? if yes then please let us know your experience on pinax in comments.And also point me out if there are any corrections or mistakes in the above article.

### Update (Installation of pinax 0.5.1 stable version)
After 3rd step you should do the following 

    4.cd myprojectname 
    5.change PINAX_ROOT in manage.py 
    PINAX_ROOT =  'â€œpath to pinax directory which was created in step 1'   
    6.change ROOT_URLCONF in settings.py     
     ROOT_URLCONF = 'For ROOTURLCONF please change it from completeproject.urls to urls'.
    7. continue from  step 12  of above installation (whch is for trunk).

#### As this stable version is using svn:externals there is no need to install third party applications via pip.




