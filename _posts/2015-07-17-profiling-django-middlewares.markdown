---
layout: post
title:  "Profiling Django Middlewares"
date:   2015-07-17 15:19:47+05:30
tags: middlewares
author: akshar
---
I assume you have a basic understanding of Profiling, what it means and why we use it.

### Why this post

Recently I had to profile a Django application which wasn't performing as fast as it should. This application had several custom middlewares too. So it was possible that custom middlewares were the cause of slow performance.

There are some existing Django libraries to profile Django code. eg: Django Debug Toolbar, django-silk , django_cprofile etc. Most of them can profile view code well but they can't profile other middlewares.

I wanted a way to profile middlewares too.

### Problem with Django Debug Toolbar

I assume you understand middlewares and how the order in which middlewares are defined matter. If you want to get more idea about middlewares, <a href="http://agiliq.com/blog/2015/07/understanding-django-middlewares/" target="_blank">this post</a> might help.

Django debug toolbar is probably designed for profiling the views. It uses process_view() and returns an HttpResponse instace from process_view(). process_request() of all middlewares run before any middleware's process_view(). So using Django debug toolbar, it's not possible to profile what's going on inside process_request() of different middlewares.

And since process_view() of debug toolbar returns HttpResponse, process_view() of other middlewares is bypassed and so we can't profile process_view() of other middlewares.

So I guess it is not possible to profile middleware code using Django debug toolbar.

### django-silk

Django silk seemed better at profiling middlewares too. It looks promising and I will play more with it.

But Django silk also tracks queries executed, inserts the results in db etc. In case you only wanted to know the time it takes to execute different functions and wanted to find out the most time consuming functions, you might not want the overhead of django silk.

### Writing our own middleware

We want to write a simple middleware that just tells the most expensive functions/methods and time it took to execute those functions. We don't want to capture sql queries or anything fancy.

We will use standard Python provided **cProfile** to achieve our goal. <a href="https://docs.python.org/2/library/profile.html" target="_blank">This official doc</a> can help you get familiar with cProfile in 10 mins.

Add the following in any app's middleware.py. Supposing you have an app called books and you add this in books/middleware.py

	import cProfile, pstats, StringIO

	class ProfilerMiddleware(object):
		def process_request(self, request):
			pr = cProfile.Profile()
			pr.enable()
			request._pr = pr

		def process_response(self, request, response):
			request._pr.disable()
			s = StringIO.StringIO()
			sortby = 'cumulative'
			# Sort the output by cumulative time it took in fuctions/methods.
			ps = pstats.Stats(request._pr, stream=s).sort_stats(sortby)
			# Print only 10 most time consuming functions
			ps.print_stats(10)
			print s.getvalue()
			return response

And add books.middleware.ProfileMiddleware at top of your MIDDLEWARE_CLASSES.

	MIDDLEWARE_CLASSES = (
		'books.middleware.ProfilerMiddleware',
		# Assuming you have some custom middlewares here, even they will be profiled
		'django.contrib.sessions.middleware.SessionMiddleware',
		'django.middleware.common.CommonMiddleware',
		'django.middleware.csrf.CsrfViewMiddleware',
		'django.contrib.auth.middleware.AuthenticationMiddleware',
		# This middleware will be profiled too.
		# 'books.middleware.SomeCustomMiddleware',
		'django.contrib.messages.middleware.MessageMiddleware',
		'django.middleware.clickjacking.XFrameOptionsMiddleware',
	)

Try any url and you should see the profiler output on the runserver console.

### Explanation

* We put our middleware at top of MIDDLEWARE_CLASSES.
* So this middleware's process_request() will be executed before any other middleware's process_request(). Also it will be executed before any other middleware's any other function like process_view() etc.
* We enable profiling in process_request() so everything hereafter will be profiled. So process_request() and process_view() of any other middleware will be profiled.
* We disable profiling in process_response() of our middleware. process_response() of this middleware will run at last, i.e after process_response() of all other middlewares have run.
* This way process_response() of all other middlewares get profiled too.

