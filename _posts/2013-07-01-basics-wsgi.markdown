---
layout: post
title:  "Basics of WSGI"
date:   2013-07-01 11:45:22+05:30
tags: python
author: akshar
---
In this post we will write a web app which will serve few urls. We will not use any Python framework for it. It's only to illustrate how things work under the hood.

Before we start, let's clarify few terms we will be using.

* **Web Server**: When we say web server, we mean the software, and not the machine that stores your code. This server recieves the request from a client(Web browser) and returns a response. Web server doesn't create the response, it only returns the response. So, a server needs to talk with a **Web Application** which can create a response.

* **Web Application**: **Web Server** will get the response from it. It is the job of the Web Application to create a response based on the url and pass this response back to the **Web Server**. Web Server's job is only to return this response to the client.

* **WSGI**: WSGI is an interface. It is just a specification or a set of rules. WSGI is not a software. WSGI is not a library or a framework. WSGI is not something you can install via pip.

WSGI comes into picture because the Web Server needs to communicate with the Web Application. WSGI specifies the rules which needs to be implemented by the Web Application side and the Web Server side so that they can interact with each other. So, a WSGI compliant server will be able to communicate with a WSGI compliant Web Application.

In WSGI architecture, **WSGI Application** has to be a callable and it needs to be given to the **Web Server**, so the Web Server can call the Web Application whenever the server recieves a request.

For understanding more on why WSGI came into existence, read about [WSGI on wikipedia](http://en.wikipedia.org/wiki/Web_Server_Gateway_Interface).

Writing our Web Application. Just copy it for now and we will go through each line to see what exactly is happening.

    #web_application.py
    from wsgiref.simple_server import make_server

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

    httpd = make_server(
        '127.0.0.1', 8051, application)

    httpd.serve_forever()

Run this file `python web_application.py`. Visit `http://127.0.0.1:8051/` and `http://127.0.0.1:8051/abcd`. You will see first page with response **Index** and second page with response **Hello**.

Stepping through the code.

* Python provides a function called `make_server`. We can use this function to create a WSGI compliant server provided by Python.
* We created a callable called `application`. You can think of this callable as **Web Application**.
* `make_server()` creates an instance of WSGI compliant server. So, `httpd` is the **Web Server** in our case.
* The first and second argument to `make_server` specifies the host and port on which the server will listen for requests.
* Third argument to `make_server` passes the **Web Application** which the **Web Server** would use to get the response.
* In the last line we start the web server using `serve_forever`.

Whenever a request comes, Web Server running on port 8051 will call the Web Application which in our case is the callable **application**.

Web Application code details.

* To interact with WSGI Web Server, Web Application needs to be WSGI compliant as well. 
* Server will call it with two arguments. So, **it must accept two arguments**. This is one condition for `application` to be WSGI compliant.
* First argument passed to it will be a variable containing various information about the request. In our example, we used it to read the request path.
* Second argument passed to it will be a callable. **`application` must use this callable** to notify the server of the status of response and for setting various headers. This is second condition for `application` to be WSGI compliant.
* We satisfied both the conditions required for our application to be WSGI compliant.
* Finally a response is returned by `application` to the WSGI server.
* And server finally relays this response back to the client.

###Edit:

I overlooked one thing in the **application** code. It was pointed by **defnull** on reddit.

The last line of **application** should not be **return response_body**. Instead, it should be **return [response_body]**. Reason being:

* WSGI server expects the return from application to be an iterable and sends each element of this iterable to the client in an unbufferred fashion.
* If we keep `return response_body`, response_body is a string and hence an iterable and so our code still works. But every character of the response_body will be sent one by one.
* If we keep `return [response_body]`, the iterable is a list here, containing only one element which is the string `response_body`. So, in this case entire response_body will be sent at once.
* That's why we should replace `return response_body` with `return [response_body]`. Edited the code for this change.


###More Edit:

By now probably you would have read why WSGI came into existence.

Before WSGI, one Python web application would work with some web server and not work with other web server. WSGI was written to fix this. With WSGI, any wsgi compliant web application would work with any wsgi compliant web server.

Python provided make_server() gives a WSGI compliant web server. Other wsgi compliant web servers are Gunicorn, uWSGI etc.

Let's run our web application with gunicorn instead of make_server.

In web_application.py, comment the lines which correspond to make_server. So comment these two lines.

    #httpd = make_server(
    #    '127.0.0.1', 8051, application)

    #httpd.serve_forever()

Install gunicorn.

	pip install gunicorn

Run your web application with gunicorn

	gunicorn web_application:application --bind=localhost:8051

You should be able to access http://localhost:8051/.

So it was very easy to switch from one wsgi compliant web server, i.e make_server, to another wsgi compliant web server, i.e gunicorn.

#### Gunicorn details

* As with make_server(), we need to tell gunicorn the **application** callable it has to use.
* Also we need to tell gunicorn the host and port on which it has to run.
* And our **application** callable is in file web_application.py. So we used "web_application:application".

You can read more about Gunicorn configurations <a href="http://agiliq.com/blog/2014/06/minimal-gunicorn-configuration/" target="_blank">here</a>.

#### Switching to another wsgi compliant web server.

Let's now switch from gunicorn to uWSGI. uWSGI is another WSGI compliant web server.

Stop gunicorn using "Ctrl+c"

Install uwsgi.

	pip install uwsgi

Use uwsgi to serve your web application.

	uwsgi --http :8051 --wsgi-file web_application.py

You should be able to access http://localhost:8051/.

So it was very easy to switch from one wsgi compliant web server, i.e gunicorn, to another wsgi compliant web server, i.e uWSGI.

You should read [this](http://webpython.codepoint.net/wsgi_tutorial) to get more idea of WSGI.

