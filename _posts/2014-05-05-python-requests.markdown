---
layout: post
title:  "Python-requests"
date:   2014-05-05 15:28:08+05:30
tags: python
author: Rakesh
---

When we want to fetch internet resources we tend to use urllib2 but when dealing with "HTTP basic Auth" urllib2 is pretty unintuitive and painful.

We can use urllib2.urlopen() to open the URL which can be a string or a request object, but the factor of the matter is that implementing anything beyond that is a bit troublesome.

Wouldn't it be a delightment, if we can use the GET, POST, PUT, DELETE with a single function call with the URL and auth data as parameters?

So Python-requests brings this delightment to life by extending the HTTP capabilities.

Python-requests Module
------------------------------
The Requests module is a an elegant and simple HTTP library for Python. It's HTTP for Humans.

Python-requests take all of the work out of Python HTTP/1.1 making our integaration with web services seamless. There will be no need to manually add query strings to our URLs, or to form-encode our POST data.

Efficiency of Python-requests
-------------------------------------

Python-requests allows us to send HTTP/1.1 requests. We can add headers, form data, multipart files and parameters with simple Python dictionaries and also we can access the response data in the same way. Python-requests is the perfect module if we want to repeatedly interact with a RESTful API.


Example for urllib2 vs requests
------------------------

If we try to fetch data with urllib2 the code would look like:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    import urllib2

    gh_url = 'https://api.github.com'

    req = urllib2.Request(gh_url)

    password_manager = urllib2.HTTPPasswordMgrWithDefaultRealm()
    password_manager.add_password(None, gh_url, 'user', 'pass')

    auth_manager = urllib2.HTTPBasicAuthHandler(password_manager)
    opener = urllib2.build_opener(auth_manager)

    urllib2.install_opener(opener)

    handler = urllib2.urlopen(req)

    print handler.getcode()
    print handler.headers.getheader('content-type')


The same functionality implemented by Python-requests looks like:

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-

    import requests

    r = requests.get('https://api.github.com', auth=('user', 'pass'))

    print r.status_code
    print r.headers['content-type']


The above code shows how simple the code turns to if we use Python-requests.

Module installation:
-----------------

To install requests, simply:

    `pip install requests`

Or, if you absolutely must:

    `easy_install requests`


Procedure:
-----------------------------

* To make a request

Example request to let us get GitHub’s public timeline

    import requests
    r = requests.get('https://github.com/timeline.json')

Response object 'r' helps to get all the information we need from the object.


* To make a POST request:

    
    r = requests.post("http://httpbin.org/post")

We can use the other HTTP request types: PUT, DELETE, HEAD and OPTIONS in the same way:

    r = requests.put("http://httpbin.org/put")
    r = requests.delete("http://httpbin.org/delete")
    r = requests.head("http://httpbin.org/get")
    r = requests.options("http://httpbin.org/get")

* Reading the response content:

Let's consider the GitHub timeline:

    import requests
    r = requests.get('https://github.com/timeline.json')
    r.text
    '[{"repository":{"open_issues":0,"url":"https://github.com/...

Requests will automatically decode content from the server.
We can also find out what encoding Requests, by using the
r.encoding property:

    r.encoding
    'utf-8'

* Passing parameters in Urls:

If you wanted to pass key1=value1 and key2=value2 to httpbin.org/get we can do it with the following code:

    payload = {'key1': 'value1', 'key2': 'value2'}
    r = requests.get("http://httpbin.org/get", params=payload)

* Dealing with Binary Response Content

We can also access the response body as bytes, for non-text requests.

    >>> r.content
    b'[{"repository":{"open_issues":0,"url":"https://github.com/...

* Dealing with JSON Response Content:

There’s also a builtin JSON decoder, in case we are dealing with JSON data:

    >>> import requests
    >>> r = requests.get('https://github.com/timeline.json')
    >>> r.json
    [{u'repository': {u'open_issues': 0, u'url': 'https://github.com/...

* Dealing with Response Headers:

We can view the server’s response headers using a Python dictionary:

    `r.headers`

Authentication
----------------------------------

While trying to fetch resources from webservices many of them expect us to undergo authentication.

There are many types of authentication, but the most common is HTTP Basic Authentication.

* Basic Authentication

Here is an example to make requests with Basic Authentication.

    from requests.auth import HTTPBasicAuth
    requests.get('https://api.github.com/user', auth=HTTPBasicAuth('user', 'pass'))


    # Due to the prevalence of HTTP Basic Auth,
    # requests provides a shorthand for this authentication method:

    requests.get('https://api.github.com/user', auth=('user', 'pass'))

* Digest Authentication:

Another very popular form of HTTP Authentication.

    from requests.auth import HTTPDigestAuth
    url = 'http://httpbin.org/digest-auth/auth/user/pass'
    requests.get(url, auth=HTTPDigestAuth('user', 'pass'))

* OAuth 1 Authentication

A common form of authentication for several web APIs is OAuth. The requests-oauthlib library allows Requests users to easily make OAuth authenticated requests:

    import requests
    from requests_oauthlib import OAuth1

    url = 'https://api.twitter.com/1.1/account/verify_credentials.json'
    auth = OAuth1('YOUR_APP_KEY', 'YOUR_APP_SECRET',
                      'USER_OAUTH_TOKEN', 'USER_OAUTH_TOKEN_SECRET')

    requests.get(url, auth=auth)


Timeouts
----------

We can tell requests to stop waiting after a given number of seconds with timeout parameter:


    `requests.get('http://github.com', timeout=0.001)`

Errors and Exceptions
----------------------------

If there is any network problem (e.g. DNS failure, refused connection, etc), Requests will raise a ConnectionError exception and in the event of the rare invalid HTTP response, Requests will raise an HTTPError exception. If a request times out, a Timeout exception is raised.

If a request exceeds the configured number of maximum redirections, a TooManyRedirects exception is raised.

All exceptions that Requests explicitly raises inherit from "requests.exceptions.RequestException".




