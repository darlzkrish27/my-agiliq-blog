---
layout: post
title:  "How to build a Facebook app in Django"
date:   2009-02-20 14:05:14+05:30
tags: facebook
author: shabda
---
Facebook has a simple and robust API which allows users to build applications for the Facebook platform.
We being the intelligent developers will use the superb Django framework to build our application.

The completed code for this is available at [https://svn.uswaretech.com/favlang/](https://svn.uswaretech.com/favlang/). The completed
application is deployed to facebook [here](http://apps.facebook.com/1673458a9d3ddaa8c6f888d7150da256/).

How does a Facebook application work?
--------------------------------------

<img src="http://uswaretech.com/blog/wp-content/uploads/2009/02/facebook.jpg" alt="" title="facebook" class="aligncenter size-full wp-image-129" width="478" height="372">

1. The users make a request to apps.facebook.com/yourapp/page1/
2. The Facebook servers make a request to yourserver.com/page1/
3. You get the requests and can make calls to Facebook API, and use FQL to query the Facebook tables.
4. You send back a page written in FBML to FB servers.
5. FB servers render your FBML to HTML.
6. FB servers send a HTML request to the user.

What tools does FB provide you?
==================================

1. FQL: Facebook Query Language. This is a language similar to SQL. Sample this

`SELECT name, pic FROM user WHERE uid=211031 OR uid=4801660`
    
2. Facebook API: This is a set of RESTful urls which can be used to get data from Facebook and
to do actions on the behalf of the logged in users.

3. FBML: Facebook Markup Language. This is a set of tags, similar to Html tags, which allow you to
get your page rendered with extra information on the facebook servers.

`<fb:name uid="211031">`
    
would show the name of the user with the id.

What else do you need?
------------------------

1. A server, which can run Django. Duh.
2. We are using Python, so we can use FB rest API directly. But somebody has already done the hard work and written
the awesome Pyfacebook library, which makes talking to Facebook a breeze.


What will we build?
---------------------

We will build a application which allows you to store your favorite programming language at Facebook.
This has a single page with single input box where you can store your favorite programming language.

The completed code for this is available at [https://svn.uswaretech.com/favlang/](https://svn.uswaretech.com/favlang/). The completed
application is deployed to facebook [here](http://apps.facebook.com/1673458a9d3ddaa8c6f888d7150da256/).

Getting Started
-----------------

1. Go to http://www.facebook.com/developers/
2. Create a new app, give it a name.
3. Get your application api key, and application secret.
4. Map callback url to the base of your Server.
5. Download the Pyfacebook library, and put it where Python can find it. The following import statements should work.
`import facebook` and `import facebook.djangofb`

Setting up your App
---------------------

    

1. Edit settings.py and add the following settings.

    `FACEBOOK_API_KEY = '1673458a9d3ddaa8c6f888d7150da256'`   
    `FACEBOOK_SECRET_KEY = '666197caab406752474bd0c6695a53f6'`  
    
    
2. Add `facebook.djangofb.FacebookMiddleware` to `MIDDLEWARE_CLASSES`

3.  Create an app named Favlang to hold our Code.

Our App
===========

Its a simple Django app with Models, views and Urls. The only difference with a normal Django app is that,
a. The templates are in FBML.
b. We will use FQL and FBAPI to talk to FB.

Urls.py
-------------
    
    from django.conf.urls.defaults import *
    
    urlpatterns = patterns('favlang.views',
        (r'^$', 'canvas'),
    )
    
A basic Urlpatter, mapping patterns to views, nothing FB specific to see here.

Models.py
-------------
    
    from django.db import models
    
    class FacebookUser(models.Model):
        """A simple User model for Facebook users."""
    
        # We use the user's UID as the primary key in our database.
        id = models.IntegerField(primary_key=True)
        language = models.CharField(max_length=64, default='Python')
        
A normal models.py. Nothing FB specific to see here. (Instead of an autoincrementing PK, we have a PK which we will set manually to FB uid.)

Views.py
---------------

    from django.http import HttpResponse
    from django.views.generic.simple import direct_to_template
    
    import facebook.djangofb as facebook
    
    from favlang.models import FacebookUser
    
    @facebook.require_login()
    def canvas(request):
        # Get the User object for the currently logged in user
        user, created = FacebookUser.objects.get_or_create(id = request.facebook.uid)
    
        # Check if we were POSTed the user's new language of choice
        if 'language' in request.POST:
            user.language = request.POST['language'][:64]
            user.save()
    
        # User is guaranteed to be logged in, so pass canvas.fbml
        # an extra 'fbuser' parameter that is the User object for
        # the currently logged in user.
        return direct_to_template(request, 'favlang/canvas.fbml', extra_context={'fbuser': user})
        
Ok. SO finally something FB specific. Lets see what is happening behind the scenes.

1. We are putting our view behind a `@facebook.require_login()` decorator. This is similar to the `login_required`
decorator. It makes sure that a valid FB user is logged in before it allows access to our view.

2. The Middleware `facebook.djangofb.FacebookMiddleware` attaches a `facebook` object to the request, which provides
access to the logged in user and some other relevant data.

3. We created a FacebookUser for the currently logged in user, in our database, with this line.

user, created = FacebookUser.objects.get_or_create(id = request.facebook.uid)

4. We returned the FBML with our data. The FBML is parsed by the Facebook servers and HTML returned to the user.

The template
----------------

    <fb:header>
      {% comment %}
        We can use {{ fbuser }} to get at the current user.
        {{ fbuser.id }} will be the user's UID, and {{ fbuser.language }}
        is his/her favorite language (Python :-).
      {% endcomment %}
      Welcome, <fb:name uid="{{ fbuser.id }}" firstnameonly="true" useyou="false">!
    </fb:name>
    
    <div class="clearfix" style="border: 1px solid rgb(216, 223, 234); padding: 10px; float: left; margin-left: 30px; margin-bottom: 30px; width: 500px;">
      Your favorite language is {{ fbuser.language }}.
      <br><br>
    
      <div class="grayheader clearfix">
        <br><br>
    
        <form action="." method="post">
          <input name="language" value="{{ fbuser.language }}" type="text">
          <input value="Change" type="submit">
        </form>
      </div>
    </div>
    
This template is written in FBML. FBML is a superset of HTML. The tags which start with `<fb:` are="" facebook="" specific="" tags.="" for="" example="" the="" tag="" fb:name="" `=""><fb:name uid="{{ fbuser.id }}" firstnameonly="true" useyou="false">` renders the
name of the user whose uid is passed in uid.

There is also a form which allows user to change their Favorite language. In views.py this is handled as

    if 'language' in request.POST:
        user.language = request.POST['language'][:64]
        user.save()
        
Resources.

1. [Facebook developers](http://developers.facebook.com/).
2. [Facebook developers wiki](http://wiki.developers.facebook.com/).
3. [Facebook developers forum](http://forum.developers.facebook.com/).
4. [Pyfacebook]
5. [FBAPI Documentation](http://wiki.developers.facebook.com/index.php/API)
6. [FQL Documentation](http://wiki.developers.facebook.com/index.php/FQL)
7. [FBML Documentation](http://wiki.developers.facebook.com/index.php/FBML)

8. [**Code listing**](https://svn.uswaretech.com/favlang/)
9. [**Live Application**](http://apps.facebook.com/1673458a9d3ddaa8c6f888d7150da256/)

</fb:name></fb:`></fb:header></fb:name>


