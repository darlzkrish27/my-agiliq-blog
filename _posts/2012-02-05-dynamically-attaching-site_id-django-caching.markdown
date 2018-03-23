---
layout: post
title:  " Dynamically attaching SITE_ID to Django Caching"
date:   2012-02-05 23:56:52+05:30
tags: SITE_ID prefix
author: anoop
---
It would be useful and convenient, if you have an automatic way to add the SITE_ID, especially, when you have multiple sites running on the same deployment. Django provides a cache prefix function KEY_FUNCTION in settings which can be used to achieve this.

Just follow the following steps, and your cache, automatically prepends SITE_ID to the cache key, making it unique across multiple sites.

1. Put the following into the settings file.

             CACHES = {

             'default': {

              'BACKEND': 'django.core.cache.backends.db.DatabaseCache',

               'LOCATION': 'cache_table',

               KEY_FUNCTION = ‘projectname.appname.modulename.functionname’,


                        }

                 }

2. Write a function to get current site id, say, get_current_site(), which returns current SITE_ID.

3. Add a function like below, as functionname at the same path as specified in KEY_FUNCTION.

        from django.utils.encoding import smart_str

        def prefix_site_id(key, key_prefix, version):

           site_id = get_current_site()

           return ':'.join([str(site_id), str(version), smart_str(key)])

That’s it. You have successfully added an automatic, dynamic, function based cache prefix for django.


