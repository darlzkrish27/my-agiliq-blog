---
layout: post
title:  "Writing an e-mail application with Lamson - I"
date:   2011-04-01 09:08:15
author:   thejaswi
tags:   e-mail
--- 
Shabda suggested we open a Off late, we\'ve been slightly busy with a \'lot\' of new developments
on our end and we\'ve not been able to devote attention to the blog. In
these busy periods, we tend to forget things faster. One such thing that
Shabda recently pointed out that a couple of our applications (client
and open source project) were raising exception. Because of the low
frequency of these errors, they went unnoticed for quite a few days. We
do check e-mail often but we are mostly drowned in our project
management tool, [Unfuddle](http://www.unfuddle.com/). Shabda suggested
we open a ticket for every error (with duplicates taken care off) and
this way we wouldn\'t have to distract ourselves scouring multiple
sources like mail, Unfuddle etc.

This was a fairly straightforward requirement. Unfuddle has an API and
all we have to do is wrap the [500 error
handler](http://docs.djangoproject.com/en/dev/topics/http/views/#the-500-server-error-view).
We could write the error handler in a separate app and then add it to
the `INSTALLED_APPS` setting and error handler to all the projects.
Sounds fine but we should not miss out a single project! It didn\'t
sound like a scalable solution and so we decided to add an e-mail
address to the `MAIL_ADMINS` setting that would open a ticket invoking
the Unfuddle API. Not a big difference over the previous solution. This
method just saves 1 line of typing over the previous. But the idea of
having to learn and use [Lamson](http://lamsonproject.org/) got us
excited!

Lamson is a Python SMTP server written by [Zed
Shaw](http://zedshaw.com/) that makes e-mail application development not
suck! If we hadn\'t heard of Lamson, we would\'ve probably used Postfix
to drop mail to fetchmail and then run a python script to parse the mail
and redirect it accordingly. This solution has way too many dependencies
and is a configuration hell!

In the coming blog posts, let\'s write a Lamson powered e-mail app that
will open a ticket in the unfuddle project management system through
their API. Before we start, I will give away a little secret. You don\'t
have to be a sys-admin or a python expert to write an e-mail app powered
by Lamson. It\'s as easy as writing a simple web app in django.

Installation of Lamson
----------------------

The Lamson documentation covers how to
[install](http://lamsonproject.org/docs/getting_started.html) Lamson.
But we are not going to discuss this very deeply. We\'ll just use a
[virtualenv](http://pypi.python.org/pypi/virtualenv) to install it.:

    $ virtualenv lamson_env
    $ cd lamson_env
    $ source bin/activate
    $ pip install lamson

Now let\'s create a lamson app named `ticket-creator` that will house
the e-mail application.:

    $ lamson gen -project ticket-creator                                                                                                                                                        
    $ cd ticket-creator                                                                                                                                                                         
    $ ls ticket-creator                                                                                                                                                                         
    app  config  logs  muttrc  README  run  tests                                                                                                                                               

Lamson creates files for us to make code organization very easy. Let us
next look at the functionality of the files:

    $ ls app/
    data  handlers  __init__.py  model  templates

The `app` directory holds all the logic for the e-mail application.
Right from the models (models), the views (templates) and controllers
(handlers), it houses them all! Doesn\'t this method of organizing the
code look similar? Yes, it is an MVC based e-mail app framework!:

    $ ls config
    boot.py  __init__.py  logging.conf  settings.py  testing.py  test_logging.conf

The `boot.py` contains the config attributes that is loaded when lamson
starts. Most likely, you wouldn\'t have to touch this file. The
`settings.py` holds all the configuration attributes. This file is
analogous to the django settings and is just a plain python file. The
`testing.py` is equivalent to `boot.py` but is used in test mode. The
`logging.conf` and `test_logging.conf` hold all the logging related
attributes.

The `logs` directory holds the files where the logging outputs are
redirected. To debug your server, this is the place you\'ve to start
off. The `muttrc` file holds the mutt configuration directives to use
lamson to send mail and open a local mailbox. The `run` directory holds
the pid and other runtime specific files. The `tests` directory houses
all the test cases you might write while developing your e-mail
application.

For this session, let\'s close with starting the lamson server. In the
e-mail app:

    $ lamson start
    $ ls run
    smtp.pid undeliverable
    $ ps aux|grep lamson
    theju 10155 0.2 0.1 21768 7532 ? Sl 19:19 0:00 /home/theju/lamson_env/bin/python /home/theju/lamson_env/bin/lamson start

To see on which port lamson is running, or to modify the port, we can
change the port of the `receiver_config` attribute in the
`config/settings.py` file. To stop lamson, just type:

    $ lamson stop
    Stopping processes with the following PID files: ['./run/smtp.pid']
    Attempting to stop lamson at pid 10155

This is the end of the first part where we just dipped our finger nails
into this amazing project called `Lamson`. In the coming post, we are
going to see how to write the logic for the application and in the final
post on how to deploy this application.
