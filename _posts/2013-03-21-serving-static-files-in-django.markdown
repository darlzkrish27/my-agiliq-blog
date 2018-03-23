---
layout: post
title:  "Serving static files in Django"
date:   2013-03-21 10:30:01+05:30
tags: static
author: akshar
---
Serving static files, especially during development, is frequently a pain point. In this post, we will discuss the various settings and directory structure and how they interact with each other. Let's start with development when you have DEBUG = True.

We will create a Django project from scratch so that we are fully aware about what file in what directory we are talking about. So, this blog post will be a little longer than it should be. We will try these things with Django 1.4, but these will work on Django 1.3 as well. I have not tested it with Django 1.2, so not sure about 1.2.

###Setting up the project.
If you think you don't need this section, then skip to the next section titled <a href="#servee">Serving static files</a>. Just make sure to see the directory structure at the end of this section to get an idea of it, so that you will be comfortable in the next section.

We will try everything inside a virtual environment. We name our virtual environment as staticvirt. So the command we need is **virtualenv staticvirt**.

    ~$ virtualenv staticvirt

Next we need to start a Django project inside this virtualenv. Make sure you are inside the virtualenv and have activated it. Also make sure that you install Django inside your virtual environment as we don't want to pollute your system wide site packages.

    ~$ cd staticvirt/
    ~/staticvirt$ source bin/activate
    (staticvirt)~/staticvirt$ pip install django==1.4

Start a Django project.

    django-admin.py startproject test_project

Change to directory containing the project.

    cd test_project/

Let's see the directory structure at this point.

    (staticvirt)~/staticvirt/test_project$ tree
    .
    |-- manage.py
    `-- test_project
        |-- __init__.py
        |-- settings.py
        |-- urls.py
        `-- wsgi.py

    1 directory, 5 files

Check the contents of test_project/settings.py. Search for all the lines which contain the substring **static**. I will list all those lines below.

    STATIC_ROOT = ''

    STATIC_URL = '/static/'

    STATICFILES_DIRS = (
    )

    STATICFILES_FINDERS = ( 
        'django.contrib.staticfiles.finders.FileSystemFinder',
        'django.contrib.staticfiles.finders.AppDirectoriesFinder',
    #    'django.contrib.staticfiles.finders.DefaultStorageFinder',
    )

    INSTALLED_APPS = ( 
        ....
        ....
        'django.contrib.staticfiles',
        ....
    )

Whatever we saw till here is the default setting provided to us by Django. We have not made any modifications.

Let's create an app in which we will have a template and then we will write a static file, which will be a stylesheet, and will use that stylesheet in this template.

    python manage.py startapp some_app

Add **some_app** to INSTALLED_APPS in test_project/settings.py.

We need a urls.py file for some_app where we will define the urls available on some_app. Project's urls.py should include the urls.py of some_app. So, we add the following line to test_project/urls.py.

    url(r'^some_app/', include('some_app.urls'))

We have following line in urls.py of some_app i.e in some_app/urls.py.

    url(r'^home$', direct_to_template, {"template": "some_app/home.html"})

Create a directory called **templates** and add it to TEMPLATE_DIRS. I created this **templates** directory on the same level as manage.py.

Adding **templates** directory to TEMPLATE_DIRS require following changes for me. It will be same if you use the same directory structure as I am using.

    PROJECT_DIR = os.path.dirname(__file__)

    TEMPLATE_DIRS = ( 
        os.path.join(PROJECT_DIR, '../templates'),
    )

We need to create home.html for app some_app and it needs to go into templates. So, we create a template file **templates/some_app/home.html**. We have the following content in the html file.

    <html>
        <body>
            <h1>This is home for some_app</h1>
        </body>
    </html>

Let's check our project directory structure, so as to remove any confusion.

    ~/staticvirt/test_project$ tree -I *.pyc
    .
    |-- manage.py
    |-- some_app
    |   |-- __init__.py
    |   |-- models.py
    |   |-- tests.py
    |   |-- urls.py
    |   `-- views.py
    |-- templates
    |   `-- some_app
    |       `-- home.html
    `-- test_project
        |-- __init__.py
        |-- settings.py
        |-- urls.py
        `-- wsgi.py

    4 directories, 11 files

We did not want .pyc files to be shown and so used a switch with *tree* to ignore .pyc files.

Start the server. Make sure you have added your database settings.

    (staticvirt)~/staticvirt/test_project$ python manage.py runserver

Navigate to **http://127.0.0.1:8000/some_app/home**. Hereafter, we will call this page as home page of some_app. You should see the html we just wrote.

<a name="servee"></a>
###Serving static files.

Let's edit home.html of some_app and add a stylesheet to it. There doesn't exist any stylesheet yet, we will create it after editing html.

    <html>
        <head>
            <link href="{{STATIC_URL}}styles.css" rel="stylesheet" type="text/css">
        </head>
        <body>
            <h1>This is home for some_app</h1>
        </body>
    </html>

Refresh home page of some_app. You will not see any changes yet because we have not created the stylesheet file.

Also navigate to **http://127.0.0.1:8000/static/styles.css**, you will see a 404 page.

**Static files should be created inside the static/ subdirectory of apps**

Let's create the stylesheet file. Since we want to use this stylesheet in the template of some_app, we will create it inside the **static/** subdirectory of some_app.
So, let's create a file some_app/static/styles.css. Add following contents to this file.

    body
    {
        background-color: red;
    }

Refresh home page of some_app. You should see that the background of this page becomes red. Also, refresh page at **http://127.0.0.1:8000/static/styles.css**. You will see that it's no more a 404 page and you will see the contents of your stylesheet at this url. If you can't see these changes, make sure that **some_app** is added to INSTALLED_APPS and restart the server.

####Points to be noted

* We did not make any changes to any of the static settings provided by Django to us. We left the static settings as they were in default settings.py provided by Django.
* You don't need any change in your urls.py for serving your static files in development. You don't need to add staticfiles_urlpatterns(). Many a times, I got confused with this.
* You do not need **python manage.py collectstatic** for serving static files in development.
<br/>
####How it works internally

*   First, check all the static settings we have in settings.py
*   They are STATIC_URL, STATIC_ROOT, STATICFILES_FINDERS, STATICFILES_DIRS.
*   Also, we have 'django.contrib.staticfiles' in INSTALLED_APPS.
*   For now, forget about STATIC_ROOT and STATICFILES_DIRS. Even if you comment it or delete it from settings, your project will continue to work as it is working currently.
*   We need 'django.contrib.staticfiles' in our INSTALLED_APPS if we want Django's default server to serve static files.
*   By Django's default server, we mean **python manage.py runserver**, that's provided with Django.
*   Django's default server will serve the static files at STATIC_URL. Notice that STATIC_URL is set to '/static/'. 

    That's why we have our static file, ie stylesheet, getting served at `http://127.0.0.1:8000/static/styles.css`. If you try to get it at `http://127.0.0.1:8000/static_changed/styles.css`, you will not be able to, you will get a 404 instead.

    If you want to get the stylesheet at `http://127.0.0.1:8000/static_changed/styles.css`, make STATIC_URL='/static_changed/'. Go ahead make this change and check. With this change, static files will get served at /static_changed/.
    
    This change was only to illustrate what is the purpose of STATIC_URL. Change it back to its default, i.e STATIC_URL='/static/'. 

*   Next question is how does Django know from where to read the static files or where to find the static files. That's where STATICFILES_FINDERS comes into picture.

    Check that we have two entries in STATICFILES_FINDERS, i.e 'django.contrib.staticfiles.finders.FileSystemFinder' and 'django.contrib.staticfiles.finders.AppDirectoriesFinder'. You can ignore FileSystemFinder for now, if you wish you can go ahead and comment the FileSystemFinder line.

    Appdirectoriesfinder tells Django to look into static/ subdirectory of each app in INSTALLED_APPS while looking for static files. Remember, we have styles.css under static/ subdirectory of some_app. That's why Django was able to find it and it got served properly by Django server. 

    If you change the name of this subdirectory from 'static/' to something else, your static resources will not be served. Go ahead, try it.

    Try commenting Appdirectoriesfinder line and your css file at http://127.0.0.1:8000/static/styles.css will not be served anymore. Ok, uncomment it back after trying this out.

Now, we are fine with what STATIC_URL and STATICFILES_FINDERS does. We still do not need STATIC_ROOT and STATICFILES_DIRS.

To understand few other things about serving static files, we need one more app.

Let's create it.

    python manage.py startapp other_app

Edit project urls.py to include the url for other_app. Now project's urls.py i.e test_project/urls.py contains two lines.

    url(r'^some_app/', include('some_app.urls')),
    url(r'^other_app/', include('other_app.urls')),

We need following line in other_app's urls.py i.e in other_app/urls.py

    url(r'^home$', direct_to_template, {"template": "other_app/home.html"})

Let's create other_app/home.html inside templates/ directory.

    <html>
        <body>
            <h1>This is home for other_app</h1>
        </body>
    </html>

Check the directory structure at this point.

    ~/staticvirt/test_project$ tree -I *.pyc
    .
    |-- manage.py
    |-- other_app
    |   |-- __init__.py
    |   |-- models.py
    |   |-- tests.py
    |   |-- urls.py
    |   `-- views.py
    |-- some_app
    |   |-- __init__.py
    |   |-- models.py
    |   |-- static
    |   |   `-- styles.css
    |   |-- tests.py
    |   |-- urls.py
    |   `-- views.py
    |-- templates
    |   |-- other_app
    |   |   `-- home.html
    |   `-- some_app
    |       `-- home.html
    `-- test_project
        |-- __init__.py
        |-- settings.py
        |-- urls.py
        `-- wsgi.py

Add other_app to INSTALLED_APPS.

Now check the url **http://127.0.0.1:8000/other_app/home**.

Let's add a stylesheet to home of other_app. Let's assume we want to make background color of other_app's home as blue. For this we create the stylesheet other_app/static/other_style.css

    body
    {
        background-color: blue;
    }

Now add this stylesheet to other_app's home template. So, we edit templates/other_app/home.html and change its content to

    <html>
        <head>
            <link href="{{STATIC_URL}}other_style.css" rel="stylesheet" type="text/css">
        </head>
        <body>
            <h1>This is home for other_app</h1>
        </body>
    </html>

Refresh your page at `http://127.0.0.1:8000/other_app/home`. You will see a blue background on this page. You might have to restart your server to see this change. Also, you will be able to see the stylesheet for this page at `http://127.0.0.1:8000/static/other_style.css`.

Also, verify that home of some_app still has a red background at `http://127.0.0.1:8000/some_app/home`.

####What happened here.

When we made a request for /static/other_style.css, Django knows that STATIC_URL is set to '/static/' which matches the first fragment of the url we provide and hence infers that we want it to serve a static file. So, it starts looking into static/ subdirectory of all the apps. It looks into 'static/' subdirectory of all apps because STATICFILES_FINDERS contains 'django.contrib.staticfiles.finders.AppDirectoriesFinder'. Since it finds a file called other_style.css in the static/ subdirectory of other_app, it serves it.

However, there is one problem with this approach. You must have noticed that we named the stylesheet of other_app as other_style.css. What if we wanted to name this stylesheet as styles.css. Let's try doing it.

    mv other_app/static/other_style.css other_app/static/styles.css

Once we make this change, we will also have to change the html of homepage of other_app to include this file. So, edit templates/other_app/home.html and make this change. We have to do this because we renamed other_style.css to styles.css. Html of other_app changes to:

    <html>
        <head>
            <link href="{{STATIC_URL}}styles.css" rel="stylesheet" type="text/css">
        </head>
        <body>
            <h1>This is home for other_app</h1>
        </body>
    </html>

Now see both the html pages, we have created.

`http://127.0.0.1:8000/some_app/home`

`http://127.0.0.1:8000/other_app/home`

You will find that the background of both pages is shown as red now. It depends on what order you have the apps listed in INSTALLED_APPS. If some_app appears before other_app in INSTALLED_APPS, both the pages will have red background. If other_app appears before some_app, both the pages will have blue background. In my INSTALLED_APPS, some_app appears before other_app, so both pages have red background for me.

####Why this happened:
Both these pages try to access a static file named styles.css. Django tries to find this file in the static/ subdirectory of each app listed in INSTALLED_APPS. Once it finds it in static/ subdirectory of some_app, it serves it and does not try to find it in static/ subdirectory of other_app. Since, static/styles.css for some_app says that the background should be red, so the background is shown red for both of these pages. 

####How do we prevent it:
So, we want to name stylesheet in both apps as styles.css. For this, we need one extra level of directory inside static/ subdirectory of each app. Let's do the following

    mkdir some_app/static/some_app
    mv some_app/static/styles.css some_app/static/some_app

    mkdir other_app/static/other_app
    mv other_app/static/styles.css  other_app/static/other_app/

So, we created a directory inside static/ subdirectory of each app. The name of this directory is the same as the app name. Then we moved the static resource i.e stylesheet inside the new directory.

We will also have to change the path to stylesheet in the templates.

Edit templates/some_app/home.html and change the path of stylesheet. So, the new content becomes:

    <html>
        <head>
            <link href="{{STATIC_URL}}some_app/styles.css" rel="stylesheet" type="text/css">
        </head>
        <body>
            <h1>This is home for some_app</h1>
        </body>
    </html>

Make similar change to templates/other_app/home.html, so that new content becomes:

    <html>
        <head>
            <link href="{{STATIC_URL}}other_app/styles.css" rel="stylesheet" type="text/css">
        </head>
        <body>
            <h1>This is home for other_app</h1>
        </body>
    </html>

Now see both the html pages again

`http://127.0.0.1:8000/some_app/home`

`http://127.0.0.1:8000/other_app/home`

You should find red background for some_app and blue background for other_app. 

####What happened here
* Template of some_app references `http://127.0.0.1:8000/static/some_app/styles.css`.
* Django sees that this url starts with /static/, which is the STATIC_URL and infers that it needs to serve the static file some_app/styles.css
* It starts looking for file some_app/styles.css in static/ subdirectory of apps.
* It finds this file in static/ of some_app and serves it.
* Template of other_app references `http://127.0.0.1:8000/static/other_app/styles.css`.
* Django starts looking for file other_app/styles.css in static/ subdirectory of apps.
* It finds it in static/ of other_app and serves it.

Hope STATIC_URL, STATICFILES_FINDERS and how static files are served makes more sense now.

###About STATICFILES_DIRS
Till now our assumption was that we need separate styling on templates of some_app and other_app. So, we wrote different stylesheet for them.

Assume we want some style which should be common on all the pages across the project and they are not specific to a particular app. In that case, we do not keep such stylesheet in the static/ subdirectory of any particular app. Instead we create a directory at the same level as manage.py and put project wide static resources in that directory.

Let's see it in action.

Create a directory called **project_static** at the same level as manage.py.

    mkdir project_static

Let's create a file named base.css inside it.

    touch project_static/base.css

Edit this file project_static/base.css and add the following content to it.

    h1
    {
        font-style: italic;
    }

So, we want to make h1 as italic across entire project.

Django is still not aware about this file and will not serve it. For making Django aware about this file we need to add the directory containing this file to STATICFILES_DIRS. So edit test_project/settings.py and add the required directory to STATICFILES_DIRS.

    STATICFILES_DIRS = ( 
        os.path.join(PROJECT_DIR, '../project_static'),
    )

Try to access url `127.0.0.1:8000/static/base.css`. You should see the contents of the stylesheet you just wrote. Make sure that you have 'django.contrib.staticfiles.finders.FileSystemFinder' in STATICFILES_FINDERS, otherwise this url will give you a 404 page.

####What happened here:

* Django server sees that a request comes for a static resource, since it is for a url that starts with '/static/'.
* It tries to find the static file i.e base.css in any of the directory defined in STATICFILES_DIRS.
* Since we have only one directory defined in STATICFILES_DIRS, which is project_static, Django server tries to find this file inside this directory. It finds the file it is searching for in this directory and serves it.
* Had it not found this file in any of the directory defined in STATICFILES_DIRS, it would have tried to find the file in static/ subdirectory of each of the apps defined in INSTALLED_APPS.
* Notice that we still didn't need to add staticfiles_urlpatterns().

For using this file in templates, we need to reference this stylesheet. Add the following line to both of the templates we defined.

    <link href="{{STATIC_URL}}base.css" rel="stylesheet" type="text/css">

Refresh the urls for both the pages. You will see that h1 appears in italic on these pages.

Let's check the final directory structure which will help in case you have some issues.

    (staticvirt)~/staticvirt/test_project$ tree -I *.pyc
    .
    |-- manage.py
    |-- other_app
    |   |-- __init__.py
    |   |-- models.py
    |   |-- static
    |   |   `-- other_app
    |   |       `-- styles.css
    |   |-- tests.py
    |   |-- urls.py
    |   `-- views.py
    |-- project_static
    |   `-- base.css
    |-- some_app
    |   |-- __init__.py
    |   |-- models.py
    |   |-- static
    |   |   `-- some_app
    |   |       `-- styles.css
    |   |-- tests.py
    |   |-- urls.py
    |   `-- views.py
    |-- templates
    |   |-- other_app
    |   |   `-- home.html
    |   `-- some_app
    |       `-- home.html
    `-- test_project
        |-- __init__.py
        |-- settings.py
        |-- urls.py
        `-- wsgi.py

    11 directories, 20 files

###About STATIC_ROOT

* You should never require STATIC_ROOT in development if you are using Django's runserver.
* Once you go to production, you can use it on the server. Django provides a management command called **collectstatic**, which will collect all the static resources, i.e the resources found in STATICFILES_DIRS and the resources found in static/ subdirectory of apps, into a single location defined by STATIC_ROOT.
* STATIC_ROOT only comes into picture if you want to use **collectstatic** management command provided by Django.

Let's try it out.
Create a directory named static_resources.

    mkdir static_resources

Edit test_project/settings.py and add the following line:

    STATIC_ROOT = os.path.join(PROJECT_DIR, '../static_resources')

Now run the following command:

    python manage.py collectstatic

You will be asked for confirmation. Type 'yes', and you will see all your static resources are collected into the directory defined by STATIC_ROOT.

And on your production server you can configure your server to serve all the request for static files from the directory defined by STATIC_ROOT.

Again, this STATIC_ROOT part was only for illustration purpose. You should not need it during development.

