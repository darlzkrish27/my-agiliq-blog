---
layout: post
title:  "Three underutilized python commands"
date:   2014-05-01 17:05:18+05:30
tags: Python
author: shabda
---
Today's post are three simple Python commands which should be known more widely, but are not.

#### Start a simple web server to serve files from current directory.

    ~$ python -m SimpleHTTPServer
    Serving HTTP on 0.0.0.0 port 8000 ...

This is neither secure, nor scalable, but handy if you are in a directory and want to quickly test something.

#### Testing emails

    ~$ python -m smtpd -n -c DebuggingServer


Starts a fake SMTP server on port 1025. Useful if you are testing emails on a system which doesn't allow [pluggable emails backends.](https://docs.djangoproject.com/en/dev/topics/email/#email-backends)

#### Pretty print JSON

    echo '{"2legs":"Better", "4Legs": "Good", "6Legs": "Spider"}' | python -m json.tool


* Inspired by [a tweet from Adrian Holovaty](https://twitter.com/adrianholovaty/status/461699628967411713)
* The -m switch was added in [PEP-338](http://legacy.python.org/dev/peps/pep-0338/)
* Try these too `python -m unittest discover .`, `python -m timeit -s 'import random; 10*random.randint(10, 20)'`






