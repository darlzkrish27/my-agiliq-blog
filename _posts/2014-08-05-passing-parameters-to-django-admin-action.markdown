---
layout: post
title:  "Passing parameters to Django admin action"
date:   2014-08-05 21:03:06+05:30
tags: django
author: akshar
---
This post will show how to pass parameters to custom Django admin actions.

This post assumes that you have an idea of <a href="https://docs.djangoproject.com/en/dev/ref/contrib/admin/actions/" target="_blank">Django actions.</a>

Django provides a default admin action which is 'Delete selected rows'. Changelist page for all the models registered with admin has this action available. This action allows to select multiple rows and delete them in a single POST request.

### Problem statement

We want to allow following Django action from admin.

* There is a model called **Instrument**. It has a field **price**
* Allow selecting multiple rows for this model and update the value of `price` for selected rows.
* Price should not be hardcoded. Admin should be able to enter the price.

We should be able to achieve something like:

<img src="http://agiliq.com/dumps/images/20140806/final-action.png" style="width: 443px;"/>

### Solution

Setup the project first.

#### Setup project

Start a Django project inside virtual environment

	/tmp $ mkvirtualenv custom-actions

	(custom-actions)/tmp $ django-admin.py startproject actions
	(custom-actions)/tmp/actions $ python manage.py startapp instruments

Writing the model in instruments/models.py

	class Instrument(models.Model):
		name = models.CharField(max_length=100)
		price = models.IntegerField()

		def __unicode__(self):
			return self.name

Add **instruments** to INSTALLED_APPS

Run syncdb

	(custom-actions)/tmp/actions $ python manage.py syncdb

Register this model with admin in instruments/admin.py

	from django.contrib import admin

	from .models import Instrument

	admin.site.register(Instrument)


You should be able to see Instrument changelist page at <a href="http://localhost:8000/admin/instruments/instrument/" target="_blank">http://localhost:8000/admin/instruments/instrument/</a>

Create few instrument instances. You can also use the fixture provided at the <a href="https://github.com/akshar-raaj/django-custom-actions" target="_blank">github repo</a> accompanying this blog.

	python manage.py loaddata instruments/fixtures/instruments.json

You should be able to see Action **Delete selected instruments** in dropdown of Instrument changelist page.

<img src="http://agiliq.com/dumps/images/20140806/delete-selected.png" alt="Default delete selected instruments" style="width: 443px;"/>

#### Writing our custom action.

We want to pass a parameter which will be used to set price on selected rows. This requires showing a Form field.

When we <a href="https://docs.djangoproject.com/en/dev/ref/contrib/admin/" target="_blank">customize the admin </a>, like adding list_display, list_filter, inlines etc, we write a class which extends from ModelAdmin. ModelAdmin has an attribute called **action_form** which determines the form that gets used for action selection dropdown.

	See https://github.com/django/django/blob/1.6/django/contrib/admin/options.py#L396

action_form is django.contrib.admin.helpers.ActionForm.

Instead of using Django provided **ActionForm**, we will use a custom form. This form should extend from Django provided ActionForm because we want to retain the functionality provided by Django while adding an additional field of our own.

This form would be:

	from django.contrib.admin.helpers import ActionForm

	class UpdateActionForm(ActionForm):
		price = forms.IntegerField()

Django provides a default action 'Delete selected' and this same form would be used for the default action too. So, we should be keeping the **price** field as optional so 'Delete selected' functionality keeps working even if we don't input anything in **price** field.

And we should be setting this form as **action_form** attribute of our ModelAdmin subclass.

	class InstrumentAdmin(admin.ModelAdmin):
		action_form = UpdateActionForm

And this InstrumentAdmin should be used while registering model Instrument with admin.

	admin.site.register(Instrument, InstrumentAdmin)

Final content of instrument/admin.py becomes

	from django.contrib import admin
	from django.contrib.admin.helpers import ActionForm
	from django import forms

	from .models import Instrument

	class UpdateActionForm(ActionForm):
		price = forms.IntegerField(required=False)

	class InstrumentAdmin(admin.ModelAdmin):
		action_form = UpdateActionForm

	admin.site.register(Instrument, InstrumentAdmin)

With this your action form would contain a **Price** field. Verify it on Instrument changelist page.

<img src="http://agiliq.com/dumps/images/20140806/added-price-field.png" style="width: 443px;"/>

Confirm that you are still able to delete rows using 'Delete selected instruments' action without the Price fields interfering with the existing functionality.

If you have written Django actions earlier, writing the function to handle our action should be a no brainer.

	def update_price(modeladmin, request, queryset):
		price = request.POST['price']
		price = int(price)
		queryset.update(price=price)
	update_price.short_description = 'Update price of selected rows'

And add it as **actions** of InstrumentAdmin.

	class InstrumentAdmin(admin.ModelAdmin):
		action_form = UpdateActionForm
		actions = [update_price]

With this you should get 'Update price of selected rows' in actions dropdown.

<img src="http://agiliq.com/dumps/images/20140806/update-price-in-dropdown.png" style="width: 443px;"/>

You might argue, why did we use request.POST to get the price and not use action_form.cleaned_data. Have some patience, I will answer that.

Now try selecting some rows and input a price in the form and verify if your code works properly. It must work.

<img src="http://agiliq.com/dumps/images/20140806/input-update-price.png" style="width: 443px;"/>

We used request.POST and not action_form.cleaned_data because we don't have access to the instance of action_form, i.e instance of UpdateActionForm, in any way inside function update_price. If we try modeladmin.action_form, it will return us class UpdateActionForm and not instance of UpdateActionForm

We can make update_price better by showing a message after the instances' price has been updated.

	def update_price(modeladmin, request, queryset):
		price = request.POST['price']
		price = int(price)
		queryset.update(price=price)
		modeladmin.message_user(request, ("Successfully updated price for %d rows") % (queryset.count(),), messages.SUCCESS)

For message to display, Django message framework should be enabled. It is enabled by default, so unless you have played with settings.py, message will just work.

