---
layout: post
title:  "Understanding Django Middlewares"
date:   2015-07-17 14:26:11+05:30
tags: middlewares
author: akshar
---
I assume you have read <a href="https://docs.djangoproject.com/en/1.8/topics/http/middleware/" target="_blank">official Django docs on middleware</a>. I will elaborate on things mentioned in the documentation but I assume you are familiar with basics of middleware.

In this post we will discuss the following.

* What is a middleware
* When to use middleware
* Things to remember when writing middleware
* Writing some middlewares to understand how order of middleware matters

### What is a middleware

Middlewares are hooks to modify Django **request** or **response** object. Putting the definition of middleware from Django [docs](https://docs.djangoproject.com/en/1.8/topics/http/middleware/).

	Middleware is a framework of hooks into Django’s request/response processing. It’s a light, low-level “plugin” system for globally altering Django’s input or output.

### When to use middleware

You can use middleware if you want to modify the **request** i.e **HttpRequest** object which is sent to the view. Or you might want to modify the **HttpResponse** object returned from the view. Both these can be achieved by using a middleware.

You might want to perform an operation before the view executes. In such case you would use a middleware.

Django provides some default middleware. eg: AuthenticationMiddleware

Very often you would have used **request.user** inside the view. Django wants **user** attribute to be set on `request` before any view executes. Django takes a middleware approach to accomplish this. So Django provides an AuthenticationMiddleware which can modify the request object.

And Django modifies the request object like:

	https://github.com/django/django/blob/master/django/contrib/auth/middleware.py#L22

Similarly you might have an application which works with users of different timezones. You want to use the user's timezone while showing any page to the user. You want access to user's timezone in all the views. It makes sense to add it in session in such case. So you can add a middleware like this:

	class TimezoneMiddleware(object):
		def process_request(self, request):
			# Assuming user has a OneToOneField to a model called Profile
			# And Profile stores the timezone of the User.
			request.session['timezone'] = request.user.profile.timezone

TimezoneMiddleware is dependent on `request.user`. And `request.user` is populated in **AuthenticationMiddleware**. So TimezoneMiddleware written by us must come after Django provided AuthenticationMiddleware in the tuple settings.MIDDLEWARE_CLASSES.

We will get more idea about order of middlewares in coming examples.

### Things to remember when using middleware

* Order of middlewares is important.
* A middleware only need to extend from class **object**.
* A middleware is free to implement some of the methods and not implement other methods.
* A middleware may implement **process_request** but may not implement **process_response** and **process_view**. Infact it is very common and lot of Django provided middlewares do it.
* A middleware may implement **process_response** but not implement **process_request**.

AuthenticationMiddleware only implements process_request and doesn't implement process_response. You can check it <a href="https://github.com/django/django/blob/master/django/contrib/auth/middleware.py#L14" target="_blank">here</a>

GZipMiddleware only implements process_response and doesn't implement process_request or process_view. You can see it <a href="https://github.com/django/django/blob/master/django/middleware/gzip.py#L9" target="_blank">here</a>

### Writing some middlewares

Make sure you have a Django project with a url and a view, and that you are able to access that view. Since we will try several things with `request.user`, make sure authentication is properly set for you and that `request.user` prints the right thing in this view.

Create a file middleware.py in any of your app.

I have an app called `books` and so I am writing this in books/middleware.py

	class BookMiddleware(object):
		def process_request(self, request):
			print "Middleware executed"

Add this middleware in MIDDLEWARE_CLASSES

	MIDDLEWARE_CLASSES = (
		'books.middleware.BookMiddleware',
		'django.contrib.sessions.middleware.SessionMiddleware',
		'django.middleware.common.CommonMiddleware',
		'django.middleware.csrf.CsrfViewMiddleware',
		'django.contrib.auth.middleware.AuthenticationMiddleware',
		'django.contrib.messages.middleware.MessageMiddleware',
		'django.middleware.clickjacking.XFrameOptionsMiddleware',
	)

Make request to any url. This should get printed on `runserver` console

	Middleware executed

Modify BookMiddleware.process_request so it looks like

	class BookMiddleware(object):
		def process_request(self, request):
			print "Middleware executed"
			print request.user

Make request to a url again. This will raise an error.

	'WSGIRequest' object has no attribute 'user'

This happened because attribute `user` hasn't been set on `request` yet.

Now change the order of middlewares so BookMiddleware comes after AuthenticationMiddleware

	MIDDLEWARE_CLASSES = (
		'django.contrib.sessions.middleware.SessionMiddleware',
		'django.middleware.common.CommonMiddleware',
		'django.middleware.csrf.CsrfViewMiddleware',
		'django.contrib.auth.middleware.AuthenticationMiddleware',
		'books.middleware.BookMiddleware',
		'django.contrib.messages.middleware.MessageMiddleware',
		'django.middleware.clickjacking.XFrameOptionsMiddleware',
	)

Make request to any url. This should get printed on `runserver` console

	Middleware executed
	<username>

This tells that `process_request` is executed on the middlewares in the order in which they are listed in settings.MIDDLEWARE_CLASSES

You can verify it further. Add another middleware in your middleware.py

	class AnotherMiddleware(object):
		def process_request(self, request):
			print "Another middleware executed"

Add this middleware in MIDDLEWARE_CLASSES too.

	MIDDLEWARE_CLASSES = (
		'django.contrib.sessions.middleware.SessionMiddleware',
		'django.middleware.common.CommonMiddleware',
		'django.middleware.csrf.CsrfViewMiddleware',
		'django.contrib.auth.middleware.AuthenticationMiddleware',
		'books.middleware.BookMiddleware',
		'books.middleware.AnotherMiddleware',
		'django.contrib.messages.middleware.MessageMiddleware',
		'django.middleware.clickjacking.XFrameOptionsMiddleware',
	)

Now output would be:

	Middleware executed
	<username>
	Another middleware executed

#### How returning HttpResponse from process_request changes things

Modify BookMiddleware so it looks like

	class BookMiddleware(object):
		def process_request(self, request):
			print "Middleware executed"
			print request.user
			return HttpResponse("some response")


Try any url now and your output would be:

	Middleware executed
	<username>

You will notice two things:

* Your view will no more be executed and no matter which url you try, you will see "some response".
* AnotherMiddleware.process_request will not be executed anymore.

So if a Middleware's process_request() returns a HttpResponse object then process_request of any subsequent middlewares is bypassed. Also view execution is bypassed. You would rarely do this or require this in your projects.

Comment "return HttpResponse("some response")" so process_request of both middlewares keep executing.

#### Working with process_response

Add method process_response to both the middlewares

	class AnotherMiddleware(object):
		def process_request(self, request):
			print "Another middleware executed"

		def process_response(self, request, response):
			print "AnotherMiddleware process_response executed"
			return response

	class BookMiddleware(object):
		def process_request(self, request):
			print "Middleware executed"
			print request.user
			return HttpResponse("some response")
			#self._start = time.time()

		def process_response(self, request, response):
			print "BookMiddleware process_response executed"
			return response

Try some url. Output would be

	Middleware executed
	<username>
	Another middleware executed
	AnotherMiddleware process_response executed
	BookMiddleware process_response executed

AnotherMiddleware.process_response() is executed before BookMiddleware.process_response() while AnotherMiddleware.process_request() executes after BookMiddleware.process_request(). So process_response() follows the reverse of what happens with process_request. process_response() is executed for last middleware then second last middleware and so on till the first middleware.

#### process_view

Django applies middleware's process_view() in the order it’s defined in MIDDLEWARE_CLASSES, top-down. This is similar to the order followed for process_request().

Also if any process_view() returns an HttpResponse object, then subsequent process_view() calls are bypassed and not executed.

Check our <a href="http://agiliq.com/blog/2015/07/profiling-django-middlewares/" target="_blank">next post</a> to see a practical use of middleware.

