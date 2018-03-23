---
layout: post
title:  "Haml for Django developers"
date:   2011-12-25 11:54:11+05:30
tags: django
author: shabda
---
Haml is taking the Ruby(and Rails) world by storm. Its not used as heavily by Python(and Django) developers as the Python solutions aren't as mature. I *finally* gave Haml a try and was pleasantly surprised how easy it was.

The most mature Python implementation of [Haml is hamlpy](https://github.com/jessemiller/HamlPy), which converts the hamlpy code to Django templates.
Others are [shpaml](http://shpaml.webfactional.com/) and [GHRML](https://github.com/derdon/ghrml)


Lets look at some templates from [Django tutorial](https://docs.djangoproject.com/en/1.2/intro/tutorial03/)

    {% if latest_poll_list %}
        <ul>
        {% for poll in latest_poll_list %}
            <li><a href="/polls/{{ poll.id }}/">{{ poll.question }}</a></li>
        {% endfor %}
        </ul>
    {% endif %}
    
    
Here is this template converted to Haml

    -if latest_poll_list
     %ul 
      -for poll in latest_poll_list 
        %li
            %a{'href':'/polls/{{ poll.id }}'}= poll.question 
    
You can see that this lost a lot of noise from the Django template. Being a Django programmer,
significant whitespace is very pleasing.

Here is another one from the same tutorial.

    <h1>{{ poll.question }}</h1>
    <ul>
    {% for choice in poll.choice_set.all %}
        <li>{{ choice.choice }}</li>
    {% endfor %}
    </ul>

And converted to hamlpy

    %h1
        = poll.question
    %ul
        -for choice in poll.choice_set.all
            %li
                = choice.choice



Again, I lose a lot of noise from the templates, and the signifiant whitespace improve the chance that the out put would be valid html.

[Here are a few more templates](https://github.com/shabda/django-haml-examples).

In particular see [this](https://github.com/shabda/django-haml-examples/blob/master/6.haml) which is haml for [this Pinax file](https://github.com/pinax/pinaxproject.com/blob/master/pinaxsite_project/templates/biblion/blog_base.html)

----
Resource

1. [Haml](http://haml-lang.com/)
2. [Hamlpy](https://github.com/jessemiller/HamlPy)


