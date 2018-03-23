---
layout: post
title:  "When and how to use Django TemplateView"
date:   2017-12-29 11:30:27+05:30
tags: django
author: akshar
---
### When to use Template View?

Django provides several class based generic views to accomplish common tasks. Simplest among them is TemplateView.

TemplateView **should be used** when you want to present some information in a html page.

TemplateView **shouldn't be used** when your page has forms and does creation or update of objects.

TemplateView is most suitable in following cases:

* Showing 'about us' like pages which are static and hardly needs any context. Though it is easy to use context variables with TemplateView.
* Showing pages which work with GET requests and don't have forms in it.

Let's write a view using base view **View** and then modify it to use TemplateView. TemplateView would help us avoid several lines of code.

#### Vanilla view

An 'about us' page with **View**.

	from django.views.generic.base import View
	from django.shortcuts import render

	class AboutUs(View):
		def get(self, request, *args, **kwargs):
			return render(request, "about-us.html")

With vanilla View we need to provide a get() implementation and must return a HttpResponse() object from get().

#### TemplateView

The same functionality can be achieved with **TemplateView** in following way:

	from django.views.generic.base import TemplateView

	class AboutUs(TemplateView):
		template_name = 'about-us.html'

As you can notice, we didn't have to provide a get() implementation while using TemplateView. TemplateView has it's own get(). TemplateView.get() also encapsulates the creation of HttpResponse object and returning it.

We only had to specify the template_name while using TemplateView.

An 'about us' page with `context` using a vanilla **View**.

	class AboutUs(View):
		def get(self, request, *args, **kwargs):
			context = {'name': 'Gryffindor'}
			return render(request, "about-us.html", context=context)

#### TemplateView with context variables

An 'about us' page with `context` using a **TemplateView**.

	class AboutUs(TemplateView):
		template_name = 'about-us.html'

		def get_context_data(self, *args, **kwargs):
			context = super(AboutUs, self).get_context_data(*args, **kwargs)
			context['name'] = 'Gryffindor'
			return context

TemplateView has a better separation of defining context variables and defining template name.

Essentially a TemplateView helps you avoid boilerplate code like:

* providing a GET() implementation.
* creating a returning a HttpResponse() or a subclass of HttpResponse() object.


