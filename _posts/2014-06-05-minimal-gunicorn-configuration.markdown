---
layout: post
title:  "Minimal Gunicorn configuration"
date:   2014-06-05 11:17:33+05:30
tags: gunicorn
author: akshar
---
<a href="http://gunicorn.org/" target="_blank">Gunicorn</a> is an http server written in Python. It is a WSGI compliant server and can serve web applications which are compliant with WSGI. <a href="http://agiliq.com/blog/2013/07/basics-wsgi/" target="_blank">This would help</a> if you aren't comfortable with WSGI.

We will serve a basic web application using Gunicorn. And see the minimal and must configuration settings you should set while using gunicorn. We will specify the configuration variables in an external file and will direct gunicorn to use that configuration file. 

### Web application

We are working in directory /tmp and have activated a virtual environment called PythonEnv

    (PythonEnv)/tmp $

Let's write the web application which we will serve using gunicorn.

    (PythonEnv)/tmp $ vim web_application.py 

    #web_application.py
    def application(environ, start_response):
        path = environ.get('PATH_INFO')
        if path == '/':
            response_body = "Index"
        else:
            response_body = "Hello"
        status = "200 OK"
        response_headers = [("Content-Length", str(len(response_body)))]
        start_response(status, response_headers)
        return [response_body]

### Serve application with gunicorn

Make sure to install gunicorn.

    (PythonEnv)/tmp $ pip install gunicorn

Serve this application.

    (PythonEnv)/tmp $ gunicorn -b localhost:8001 web_application:application

Make sure you can access http://localhost:8001/

With this, gunicorn is serving the application on localhost port 8001. We had to use `-b` flag to instruct gunicorn to **bind** on a particular port and host. There are tons of other options that can be set. Setting all of them on command line is a tedious task. So, we should have a configuration file.

### Adding a configuration file

Let's create a configuration file.

    (PythonEnv)/tmp $ vim gunicorn_cfg.py

#### Bind on host and port

And add the following content in file gunicorn_cfg.py

    bind='localhost:8001'

Now run gunicorn using this configuration file. Make sure to `Ctrl+c` the running gunicorn before proceeding with this.

    (PythonEnv)/tmp $ gunicorn -c gunicorn_cfg.py web_application:application

Still gunicorn is running in the foreground, which we usually would not like on a production server. So, make it daemonized. Make sure to `Ctrl+c` the existing gunicorn process

But at some point we would like to kill the existing gunicorn process and restart gunicorn. Since it would be daemonized, we wouldn't be able to kill with `Ctrl+c`. We would require the process id of running gunicorn process for killing it. So let's keep track of process id of gunicorn before we make it daemonized.

#### Keep track of process id

Add following to gunicorn_cfg.py

    pidfile='gunicorn_pid'

Run gunicorn

    (PythonEnv)/tmp $ gunicorn -c gunicorn_cfg.py web_application:application

Open another terminal and **ls**. You'll see that a file callled gunicorn_pid is created. **cat gunicorn_pid** and you will see a number. This number is the process id of gunicorn. Kill gunicorn with `Ctrl+c` again.

#### Make it daemonized

Add the following to gunicorn_cfg.py
    
    daemon = True

Run gunicorn

    (PythonEnv)/tmp $ gunicorn -c gunicorn_cfg.py web_application:application

Gunicorn is running as a daemon now. Make sure you can access `http://localhost:8001/`

Check process id of gunicorn

    (PythonEnv)/tmp $ cat gunicorn_pid 
    19558

Kill existing gunicorn process and restart gunicorn.

    (PythonEnv)/tmp $ kill 19558
    (PythonEnv)/tmp $ gunicorn -c gunicorn_cfg.py web_application:application

This would establish that gunicorn_pid is working properly.

#### Add access loggging

At some point of time, your site will start behaving strangely and you would like to verify that requests are infact coming to gunicorn. So, there should be a file to log the requests.

Edit gunicorn_cfg.py and add:

    accesslog='gunicorn_access.log'

Restart gunicorn for this change to take effect. You'll have to read the gunicorn_pid file to read the process id of gunicorn for killing it, as we did earlier. Once you restart gunicorn, you will find a file named `gunicorn_access.log` created in the same directory as gunicorn_config.py

As you make requests to `http://localhost:8001/`, you will see more lines being logged to file gunicorn_access.log. Read about Linux **tail** command to know how to monitor a log file.

#### Add error logging

Let's insert an error in our code. Add `a = a + 1` as first line of function `application` in file `web_application.py`. 'a' is an undefined variable being used.

Restart gunicorn and try to access localhost:8001/

You get 'Internal Server Error' and you don't have a way of knowing which line caused the error.

Add following line to gunicorn_cfg.py

    errorlog='gunicorn_error.log'

Restart gunicorn for this change to take effect. You'll find a file named `gunicorn_error.log` is added at the same level as gunicorn_cfg.py

We'll again try to access `localhost:8001/`. But before that, let's monitor the errorlog file using tail.

On terminal.

    (PythonEnv)/tmp $ tail -f gunicorn_error.log

Now access http://localhost:8001/. Look at the terminal where `tail` is running. You'll notice error logged on the terminal.

    File "/tmp/web_application.py", line 3, in application
        a = a + 1
    UnboundLocalError: local variable 'a' referenced before assignment

So with an errorlog file, you can easily find out the cause of error. Undo the error we injected in code and restart gunicorn. Your application should start behaving properly again.

Read <a href="http://gunicorn-docs.readthedocs.org/en/latest/settings.html#config-file" target="_blank">Gunicorn config file docs</a> to know about more config options.


