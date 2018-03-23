---
layout: post
title:  "Supervisor with Django and Gunicorn"
date:   2014-05-09 10:40:13+05:30
tags: supervisor
author: akshar
---
Supervisor with Django: A starter guide

This post assumes that you have used gunicorn and know what it does. I will try everything inside a virtual environment and hope you do the same.

#### What is supervisor.

Supervisor is a monitoring tool that can monitor your processes. It can restart the process if the process dies or gets killed for some reason.

#### Use of supervisor: Why I started using it.

In production, I use gunicorn as web server. I started a gunicorn process as a daemon and logged out from the server. My site ran as expected for few days. All of a sudden, we started getting '502 Bad Gateway' and I had no idea why. I had to ssh to the server to find out what went wrong. After `ps aux | grep gunicorn`, I found out gunicorn wasn't running anymore. My gunicorn process died on its own, and I had no idea when and why. Had I used supervisor, supervisor would have been controlling the gunicorn process. It must have recieved a signal when gunicorn died and it would have created a new gunicorn process in such scenario. And my site would have kept running as expected.

##### Other scenario

We want to run a process which doesn't allow deamonizing. eg: I wanted to keep `celery`, a Python worker, running on the production server. I could not run it in the foreground because I had to logout from the server. I did not find an easy way to run celery as a daemon. Here too, supervisor came handy.

#### Project setup

I am setting up a new Django project so you will have proper idea of the path and how to use paths in supervisor configuration file.

We will use Django 1.6.

Create a new Django project:

    (PythonEnv)/tmp $ django-admin.py startproject testproj
    (PythonEnv)/tmp $ cd testproj/

Project structure looks like:


    (PythonEnv)/tmp/testproj $ tree
    manage.py
    testproj
        __init__.py
        settings.py
        urls.py
        wsgi.py

Run syncdb and runserver. Hereafter, we will run all our commands from directory **/tmp/testproj/** with **PythonEnv** activated.

    python manage.py syncdb
    python manage.py runserver

And then access the '/admin/'. You should be able to see django admin.

Let's use `gunicorn` instead of using `runserver`.

    pip install gunicorn

    gunicorn testproj.wsgi:application --bind 127.0.0.1:8000

Make sure you can still see `/admin/`. You will not be able to view static files now, because we aren't using Django development server anymore. Read [this post](http://agiliq.com/blog/2013/08/minimal-nginx-and-gunicorn-configuration-for-djang/) if you want to know how to serve static files using nginx. Though you don't need to do this now, our motive is not to see static files.

Let's run gunicorn as a daemon. Also, we will tell gunicorn to use a file to keep track of process id, so that we can use that process id to kill gunicorn whenever we want.

    gunicorn testproj.wsgi:application --bind 127.0.0.1:8000 --pid /tmp/gunicorn.pid --daemon

The problem with this approach is, gunicorn might die at any moment and you will not know about it immediately. Since we are working on dev environment, it is acceptable but gunicorn dying on a server is not acceptable. Your site will loose too many users/customers.

Kill this gunicorn process, we will let supervisor handle gunicorn hereafter.

    (PythonEnv)/tmp/testproj $ cat /tmp/gunicorn.pid 
    19363
    (PythonEnv)/tmp/testproj $ kill -9 19363

#### Using supervisor

Let's install supervisor inside our virtual environment.

    (PythonEnv)/tmp/testproj $ pip install supervisor

With this, you should have a `echo_supervisord_conf` command available. Get a stub supervisor file in your current directory using it.

    echo_supervisord_conf > ./supervisord.conf

 
With this, your directory structure should look like:


    db.sqlite3
    manage.py
    supervisord.conf
    testproj
        __init__.py
        settings.py
        urls.py
        wsgi.py

You can remove most of the sections of this supervisor configuration file if you want. Initially we can only keep:

    [supervisord]
    logfile=/tmp/supervisord.log ; (main log file;default $CWD/supervisord.log)
    logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
    logfile_backups=10           ; (num of main logfile rotation backups;default 10) 
    loglevel=info                ; (log level;default info; others: debug,warn,trace)
    pidfile=/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
    nodaemon=true               ; (start in foreground if true;default false)
    minfds=1024                  ; (min. avail startup file descriptors;default 1024)
    minprocs=200                 ; (min. avail process descriptors;default 200)

From [supervisor docs](http://supervisord.org/running.html):

Before supervisord will do anything useful for you, you’ll need to add at least one **program** section to its configuration. The program section will define a program that is run and managed when you invoke the supervisord command. To add a program, you’ll need to edit the supervisord.conf file.

So, we'll add a **program** section for gunicorn to supervisord.conf. Add it after the [supervisord] section.

    [program:gunicorn]
    command=/home/akshar/.virtualenvs/PythonEnv/bin/gunicorn testproj.wsgi:application --bind 127.0.0.1:8000 --pid /tmp/gunicorn.pid ;
    directory=/tmp/testproj/ ;

You need to change **command** and **directory** depending on your environment. My gunicorn executable resides at `/home/akshar/.virtualenvs/PythonEnv/bin/gunicorn`. Find where your gunicorn executable is and use that in `command` section of `program`.

We passed `testproject.wsgi:application` as application object to gunicorn, so `testproject` need to be available on the PATH for this to work. And for something to be available on the path, you need to be in correct directory. That's why we need the `directory` setting inside `program` section.

Make sure neither a gunicorn nor a supervisor process exists till here:

    (PythonEnv)/tmp/testproj $ ps aux | grep supervisord
    (PythonEnv)/tmp/testproj $ ps aux | grep gunicorn

Run `supervisord`

    (PythonEnv)/tmp/testproj $ supervisord

Now a supervisor process as well as a gunicorn process should be running. Even though you did not explicitly start a gunicorn process, supervisor started gunicorn for you. Verify that supervisor as well as gunicorn process exists. You need to check these on a different terminal, because `supervisord` must be running on your first terminal in the foreground, since it wasn't daemonized.

    (PythonEnv)/tmp/testproj $ ps aux | grep supervisord
    (PythonEnv)/tmp/testproj $ ps aux | grep gunicorn

Access `http://localhost:8000/admin/` to make sure your site is running fine.

Kill `supervisord` process(Ctrl+c), we will deamonize it now. Change **nodaemon=true** in your supervisord.conf to **nodaemon=false**.

Run supervisord again

    (PythonEnv)/tmp/testproj $ supervisord

Supervisor should no more be running in foreground. Verify that you have a gunicorn process running too, gunicorn was started by supervisor.

    (PythonEnv)/tmp/testproj $ ps aux | grep gunicorn

Also, match the process id given by 'ps aux' with pid shown in /tmp/gunicorn.pid

Verify that `/admin/` is still accessible.

#### Supervisor client

Supervisor also provides a client using which you can see currently managed process by this supervisor instance. Add the following sections between `supervisord` and `program` section.

    [inet_http_server]
    port=127.0.0.1:9001   ;

    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

    [supervisorctl]
    serverurl=http://127.0.0.1:9001 ;

You need to restart supervisor to see these changes taking effect. Kill the existing supervisor process by checking its id from supervisor pid file. Notice that you have a `pidfile` option under your `supervisord` section, which tells the file which stores process id of supervisor.

    (PythonEnv)/tmp/testproj $ cat /tmp/supervisord.pid 
    9820
    (PythonEnv)/tmp/testproj $ kill -9 9820

This killed the exising supervisor process.

Also, you need to kill the running gunicorn process. Else supervisor will fail to start another gunicorn process because the current gunicorn process has already occupied port 8000.

    (PythonEnv)/tmp/testproj $ cat /tmp/gunicorn.pid 
    9824
    (PythonEnv)/tmp/testproj $ kill -9 9824

Run `supervisord` again 

    (PythonEnv)/tmp/testproj $ supervisord

Again verify that gunicorn process was started by supervisor.

Start supervisor client by issuing following command:

    (PythonEnv)/tmp/testproj $ supervisorctl

It should tell you that a process for gunicorn is being managed by supervisor. Output should be something like:

    gunicorn                         RUNNING    pid 9950, uptime 0:01:01
    supervisor>

Suppose you made some change in your Django code and you need to restart gunicorn for that change to take effect. You can easily do it from supervisor client.

    supervisor> restart gunicorn

Output should be:

    gunicorn: stopped
    gunicorn: started

You can verify that the pid of gunicorn changes everytime you restart gunicorn from supervisorctl.

supervisorctl provides a host of other functionalities too. Use command `help` on supervisorctl to know more.

##### Surprise

You have a running gunicorn process. Kill it.

    (PythonEnv)/tmp/testproj $ cat /tmp/gunicorn.pid 
    10137
    (PythonEnv)/tmp/testproj $ kill -9 10137

Now gunicorn should be killed and you should not be able to access `/admin/`. But you will be able to access it, try it. Surprised? Check the pid of gunicorn now.

    (PythonEnv)/tmp/testproj $ cat /tmp/gunicorn.pid 
    10301

So, somehow a new gunicorn process was created as soon as we killed gunicorn, and the process id for that gunicorn process was written in /tmp/gunicorn.pid.

###### Why did it happen.

When you killed gunicorn, supervisor came to know that gunicorn process died. And it's supervisor's duty to restart any process if it dies, if supervisor controls that process. Since gunicorn was being controlled by supervisor, supervisor made sure to restart gunicorn. So, be assured that if you are using supervisor and let it control gunicorn, then supervisor will make sure to always keep gunicorn running. And your site will never be down.

This is how we used supervisor on our development environment. You can similarly use supervisor on a production server.

It is worth mentioning that supervisor conf can have multiple `program` section. So, you can use single supervisor instance to control gunicorn, celery and any other process you want.

