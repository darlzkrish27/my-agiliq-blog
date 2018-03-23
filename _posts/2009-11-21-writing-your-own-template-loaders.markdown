---
layout: post
title:  "Writing your own template loaders"
date:   2009-11-21 17:11:17+05:30
tags: tutorial
author: shabda
---
Django has three builtin template loaders which are used to get the templates for rendering.

    TEMPLATE_LOADERS = (
        'django.template.loaders.filesystem.load_template_source',
        'django.template.loaders.app_directories.load_template_source',
    #     'django.template.loaders.eggs.load_template_source',
    )
    
Writing your template loader is a awfuly easy. It is a callable which

1. Returns a tuple of (openfile, filename) if it can find the template.
2. Raise TemplateDoesNotExist if the templates cannot be found.

The simplest template loader can be
    
    from django.template import TemplateDoesNotExist
    def load_template_source(template_name, template_dirs=None):
            try:
                return open(template_name).read(), template_name
            except IOError:
                raise TemplateDoesNotExist, template_name

if you put this to your template loaders directory,

    TEMPLATE_LOADERS = (
        'django.template.loaders.filesystem.load_template_source',
        'django.template.loaders.app_directories.load_template_source',
        'dgrid.load_template_source'
    #     'django.template.loaders.eggs.load_template_source',
    
You can do access a template using an absolute path.
    
    In [6]: get_template('/home/shabda/abc.txt')
    Out[6]: <django.template.Template object at 0x91c860c>
    
    In [7]: temp = get_template('/home/shabda/abc.txt')
    
    In [8]: temp
    Out[8]: <django.template.Template object at 0x9358a0c>
    
    In [9]: temp.render
    Out[9]: <bound method Template.render of <django.template.Template object at 0x9358a0c>>


---------------

This is the first in the series of `short and sweet` Django posts we are going to do. You are interested, right. Do follow us on [twitter](http://twitter.com/uswaretech) or [subscribe to our feed](http://feeds.feedburner.com/uswarearticles).

-----

We build *Amazing We Apps*. [Talk to us](http://uswaretech.com/contact/) or email us at sales@uswaretech.com .

