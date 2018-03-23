---
layout: post
title:  "Easy client side form validations for Django: Django Parsley"
date:   2013-02-07 11:30:00+05:30
tags: django
author: shabda
---
[Parsleyjs](http://parsleyjs.org/) is a JavaScript library to do client side data validations.
It does this in a non-intrusive way via adding a `data-*` attributes to form fields. Check it out, it's really slick.

Django forms do a great job of server side data validations. By integrating Parsley with Django form, you get good client side data validations as well. Get the [app here](https://github.com/agiliq/django-parsley).

### Some design considerations

When I started writing this app I wanted to make it really easy to use. My first thought was to make this a filter or a template tag which would have allowed you to do:

    {{ form|parsleyfy }}

However, I use Django-crispy-forms in all my projects. If parsley was a filter, it would have made piping to `|crispy` impossible as the render step would already have been completed. Hence `parsleyfy` is class decorator. Django-crispy-forms and family will play well with `parsleyfy`.


Here is the readme.

[![Build Status](https://travis-ci.org/agiliq/Django-parsley.png?branch=master)](https://travis-ci.org/agiliq/Django-parsley)

###What is it?

[Parsleyjs](http://parsleyjs.org/) is a JavaScript library to do client side data validations.
It does this in a non-intrusive way via adding a `data-*` attributes to form fields.

When you define a Django form, you get server side validations for free using
the form field attributes. Django-parsley adds these validations to client side, by tagging your form with `data-*` attributes.

### Installation

Add `parsley` to your `INSTALLED_APPS`.

### Usage

`parsley` provides a single class decorator called `parsleyfy`. Decorate your `Form` with `parsleyfy` to get the validations.

Eg.

    from parsley.decorators import parsleyfy


    @parsleyfy
    class FieldTypeForm(forms.Form):
        name = forms.CharField(min_length=3, max_length=30)
        url = forms.URLField()
        url2 = forms.URLField(required=False)
        email = forms.EmailField()
        email2 = forms.EmailField(required=False)
        age = forms.IntegerField()
        income = forms.DecimalField()

Your rendered form's HTML will look like this

    <p><label for="id_name">Name:</label> <input data-required="true" data-minlength="3" maxlength="30" type="text" data-maxlength="30" id="id_name" name="name" /></p>
    <p><label for="id_url">Url:</label> <input type="text" data-required="true" data-type="url" name="url" id="id_url" /></p>
    <p><label for="id_url2">Url2:</label> <input type="text" data-type="url" name="url2" id="id_url2" /></p>
    <p><label for="id_email">Email:</label> <input type="text" data-required="true" data-type="email" name="email" id="id_email" /></p>
    <p><label for="id_email2">Email2:</label> <input type="text" data-type="email" name="email2" id="id_email2" /></p>
    <p><label for="id_age">Age:</label> <input type="text" data-required="true" data-type="digits" name="age" id="id_age" /></p>
    <p><label for="id_income">Income:</label> <input type="text" data-required="true" data-type="number" name="income" id="id_income" /></p>

Note the `data-*` attributes.

You could also do

FieldTypeForm = parsleyfy(FieldTypeForm)

Which is the same thing.

Put this form inside a

    <form data-validate="parsley">
        {{ form.as_p }}
    </form>

Include the parsleyjs and you are good to go.

###License

3 Clause BSD.

### Bug report and Help

For bug reports open a github ticket. Patches gratefully accepted. Need help? [Contact us here](http://agiliq.com/contactus)


