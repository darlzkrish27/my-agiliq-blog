---
layout: post
title: Request to Response
date:   2012-01-02 18:00:00
author:   thejaswi
tags:   response

summary:   In the previous installment of \"Behind the Scenes\", we saw how the
    control flows from [Form to File
    Storage](http://agiliq.com/blog/2011/09/behind-the-scenes-from-html-form-to-storage/).
    Today, we are going to see how the application reacts from request
    to response. In this post, we are going to assume that we are using
    django\'s inbuilt `runserver`. The flow doesn\'t change much for
    other WSGI servers available. When you invoke the `runserver`
    management command, the command line options are validated and an
    instance of
    [WSGIServer](https://code.djangoproject.com/browser/django/trunk/django/core/servers/basehttp.py#L113)
    is created and passed the
    [WSGIRequestHandler](https://code.djangoproject.com/browser/django/trunk/django/core/servers/basehttp.py#L130),
    which is used to create the request object
    ([WSGIRequest](https://code.djangoproject.com/browser/django/trunk/django/core/handlers/wsgi.py#L128)).
    After the request object is created and the request
---

In the previous installment of \"Behind the Scenes\", we saw how the
control flows from [Form to File
Storage](http://agiliq.com/blog/2011/09/behind-the-scenes-from-html-form-to-storage/).
Today, we are going to see how the application reacts from request to
response.

In this post, we are going to assume that we are using django\'s inbuilt
`runserver`. The flow doesn\'t change much for other WSGI servers
available.

When you invoke the `runserver` management command, the command line
options are validated and an instance of
[WSGIServer](https://code.djangoproject.com/browser/django/trunk/django/core/servers/basehttp.py#L113)
is created and passed the
[WSGIRequestHandler](https://code.djangoproject.com/browser/django/trunk/django/core/servers/basehttp.py#L130),
which is used to create the request object
([WSGIRequest](https://code.djangoproject.com/browser/django/trunk/django/core/handlers/wsgi.py#L128)).
After the request object is created and the request started signal is
fired, the response is fetched through the
[WSGIRequestHandler.get\_response(request)](https://code.djangoproject.com/browser/django/trunk/django/core/handlers/base.py#L72).

In the `get_response` method of the request handler, first the urlconf
location (by default the `urls.py`) is setup based on the
`settings.ROOT_URLCONF`. Then a
[RegexURLResolver](https://code.djangoproject.com/browser/django/trunk/django/core/urlresolvers.py#L219)
compiles the regular expressions in the urlconf file. Next, the request
middlewares are called in the order specified in the
`settings.MIDDLEWARE_CLASSES` followed by the view middlewares after
matching the view (`callback`) function against the compiled regular
expressions from the urlconf. Then the view (`callback`) is invoked and
verified that it does not return `None` before calling the response
middlewares.

You can see the pictorial representation of the flow below:

[![Request to Response Flow](http://agiliq.com/static/dumps/images/20120102/request_to_response.png){.align-center
width="543px"
height="792px"}](http://agiliq.com/static/dumps/images/20120102/request_to_response.png)
