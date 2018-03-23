---
layout: post
title:  "Rails and Django commands : comparison  and conversion"
date:   2010-03-20 18:35:30+05:30
tags: Uncategorized
author: shabda
---
The most commonly used Rails commands and their Django equivalents

Rails                   | Django                         
-------------------------|--------------------
rails console            | manage.py shell                   
rails server             | manage.py runserver                  
rake                     | None                              
rails generate           | None                              
rails dbconsole          | manage.py dbshell                 
rails app_name           | django-admin.py startproject/manage.py startapp            
rake db:create           | manage.py syncdb      


The salient points to note are,

1. Django has all commands via `manage.py`, Rails has it broken into `rails` and `rake`.
2. Overall there are more Rails+Rake commands available than Django commands
3. There is no one to one mapping between Rails and Django commands.
Eg. There are no equivalent to rake doc:* or rake notes in Django.

Similarly there is no equivalent to `manage.py createsuperuser` in rails.

#### References

[http://docs.djangoproject.com/en/dev/ref/django-admin/](http://docs.djangoproject.com/en/dev/ref/django-admin/)
[http://guides.rails.info/command_line.html](http://guides.rails.info/command_line.html)
[http://github.com/uswaretech/Acts-as-Django/blob/master/commands.markdown](http://github.com/uswaretech/Acts-as-Django/blob/master/commands.markdown)




