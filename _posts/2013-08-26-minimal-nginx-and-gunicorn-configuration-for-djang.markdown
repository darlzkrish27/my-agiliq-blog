---
layout: post
title:  "Minimal Nginx and Gunicorn configuration for Django projects"
date:   2013-08-26 18:05:50+05:30
tags: nginx
author: akshar
---
We will see how to deploy a Django project with nginx and gunicorn.

It was easier for me to understand nginx and gunicorn on development machine and then move to a publicly accessible server. So, we will cover this post in a similar sequence:

####On development machine

* Use django's `runserver` to serve the site.
* Stop `runserver` and start using gunicorn instead.
* Add nginx to this configuration.

####On server

* Deploy the project on a publicly accessible server using same stack.

<br/>

###Use runserver

We will use a minimal project with two apps and few static files. You can view this project at <a href="https://github.com/akshar-raaj/static-files-in-django" target="_blank">github</a>.

Project structure:

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
    |-- requirements.txt
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
    |-- test.db
    `-- test_project
        |-- __init__.py
        |-- settings.py
        |-- test_settings.py
        |-- urls.py
        `-- wsgi.py

This project was started using **django-admin.py startproject test_project**, so this structure is inside folder `test_project`.

Cloning and trying it out would help you follow along, <a href="https://github.com/akshar-raaj/static-files-in-django" target="_blank">so go ahead and clone this project</a>.

Our assumption is that we are working from the same level as `manage.py`.

Runserver and verify that you can access

    http://localhost:8000/some_app/home
    http://localhost:8000/other_app/home

First url should have a red background and the second one should have a blue background. 

This project uses some static resources, you should check our post on [serving static file](http://agiliq.com/blog/2013/03/serving-static-files-in-django/) if you are confused about serving static files during development.

###Stop runserver and start using Gunicorn

Stop the Django development server.

Make sure you have gunicorn installed, else install it with 

    pip install gunicorn

Gunicorn is a WSGI compliant server and you need to pass your **application** object to it. You should read about [Basics of WSGI](http://agiliq.com/blog/2013/07/basics-wsgi/) if you are not comfortable with wsgi.

Django provides an **application** object. The default structure provided by `django-admin startproject` gives a `wsgi.py` file which contains the `application`. You need to pass this `application` to gunicorn.

Run gunicorn passing it the `application`:

    gunicorn test_project.wsgi:application

Note: In the recent django versions the wsgi file extension has been changed it is a python module(ex: test_project_wsgi.py) and so you can serve it with just module name like below

    gunicorn test_project:application

Refresh your page at **http://localhost:8000/some_app/home**. Your page should be served, however the red background must have vanished.

Red background vanished because the static files(stylesheets) are not being served anymore. They are not being served because we do not have any url pattern for `/static/` in urls.py. Earlier they were served because we were using Django development server, i.e runserver, which does some magical things behind the scence to serve static files.

###Add nginx to this configuration

Gunicorn is meant to serve dynamic content, it should not be used to serve static files. We will add nginx to serve static files.

We want to serve static files from port 8000 and so it is required that gunicorn listens on some different port. Stop gunicorn and run it on port 8001.

    gunicorn test_project.wsgi:application --bind=127.0.0.1:8001

Now you will not be able to see your page at **http://localhost:8000/some_app/home/**. It will be available on port 8001 at **http://localhost:8001/some_app/home**.

Make sure you have nginx installed or install it with:

    sudo apt-get install nginx

Nginx acts as a reverse proxy here. All our request will initially come to nginx. It's for nginx to decide what requests it wants to serve and what requests it wants to pass to some other server.

In our case, we want nginx to serve requests for static resources and to pass any other request to gunicorn.

We need to tell nginx the location of our static resources and for that we need to make sure all our static resources are at a single location. Run `python manage.py collectstatic` to collect all the static resources at location specified by STATIC_ROOT.

Set STATIC_ROOT in settings.py:

    STATIC_ROOT = os.path.join(PROJECT_DIR, '../staticfiles/')

Run collectstatic:

    python manage.py collectstatic

You should see that a directory `staticfiles` gets created at the same level as manage.py. All your static files must have got copied to this directory. Get the path of this directory, we need it in the `alias` directive of nginx cofiguration.

Create a file `/etc/nginx/sites-enabled/example` and add following content to it.

    server {
        listen localhost:8000;

        location / {
            proxy_pass http://127.0.0.1:8001;
        }

        location /static/ {
            autoindex on;
            alias /home/akshar/staticvirt/test_project/staticfiles/;
        }
    }

Your `alias` directive will differ from mine.

Make sure nginx is running:

    sudo service nginx restart

Nginx is listening for requests now.

Refresh page at `http://localhost:8000/some_app/home/`, the page should be visible with red background which conforms that static files are being served properly.

####Explanation:
* `listen` directive tells that nginx is listening for any request that comes at `localhost:8000`.
* There are two `location` directives.
* Bottom `location` can overide top `location`. Bottom will have preference over top.
* When a request starting with `/` comes, its being passed to port 8001.

eg: If request comes for `http://localhost:8000/some_app/home`, nginx tries to match it with one of the `location` defined in the configuration file. In this case, it matches with first location. Nginx sees that a `proxy_pass` is defined in this case so it passes this request to the proxy_pass which is http://127.0.0.1:8001. Gunicorn is listening at port 8001, so gunicorn will repond to this request.

* When a request starting with `/static/` comes, bottom `location` is used as it has preference over the top `location`.

When a request comes for `http://localhost:8000/static/some_app/styles.css`, Nginx looks into the directory pointed to by `alias` which is **staticfiles**. It tries to find **some_app/styles.css** inside this directory and if this file is available then serves the file.

Now we are comfortable with serving django sites with nginx and gunicorn.

###Deploy on a publicly accessible server.

I will use domain `pythoninternals.com` for illustration. We need to do the following things on the server where A record of domain `pythoninternals.com` points.

We don't need any change for gunicorn and can run it in the same way:

    gunicorn test_project.wsgi:application --bind=127.0.0.1:8001

Create file `/etc/nginx/sites-enabled/example` on the server and add content:

    server {
      listen 80; 
      server_name pythoninternals.com;
      location / { 
        proxy_pass http://127.0.0.1:8001;
      }

      location /static/ {
        alias /home/ubuntu/staticvirt/test_project/staticfiles/;
      }
    }

This file is almost similar to the nginx conf we had on development machine.

Make sure that you have collected the static files in directory `staticfiles`.

Some differences in this nginx conf and dev machine's nginx conf are:

* It's listening on port 80. When a request is made for domain `pythoninternal.com`, it comes on the default port which is 80. So, nginx must be listening for any request on port 80.
* `server_name` makes sure that this configuration file will only be used for `pythoninternals.com`.
* There might be another domain called `abcde.com` being served from this same server, we don't want this configuration to be used for `abcde.com`. That's why we specify the `server_name`.

After this a request for `pythoninternals.com/some_app/home` would be served properly from this server.

Running gunicorn the way we did, will keep it in the foreground and we will have to stop gunicorn to exit from the server. So, we need to run it as a daemon.

Run gunicorn as daemon:

    gunicorn test_project.wsgi:application --bind=127.0.0.1:8001 --daemon

With this, gunicorn runs as a background process and we can quit from the server without affecting gunicorn.

Till now, we have set various configurations for gunicorn, like --bind and --daemon, on the terminal. The suggested way to do it is using configuration file. You can read about it at <a href="http://agiliq.com/blog/2014/06/minimal-gunicorn-configuration/" target="_blank">another post we wrote</a> and should move these configurations to a separate file.


