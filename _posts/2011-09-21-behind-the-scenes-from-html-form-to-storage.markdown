---
layout: post
title: "From HTML Form to Storage"
date:   2011-09-21 17:00:00
author:   thejaswi
tags:   upload
---

In this post, we are going to see what happens behind the scenes when a
file is uploaded to a django powered web application.

[![File Upload Flow](http://agiliq.com/dumps/images/20110921/file_storage.gif){.align-center
width="543px"
height="792px"}](http://agiliq.com/dumps/images/20110921/file_storage.gif)

An HTML form with a file input (atleast one) and encoding set to
[multipart/form-data](https://docs.djangoproject.com/en/dev/ref/forms/api/#binding-uploaded-files)
is submitted. The
[MultiPartParser](https://code.djangoproject.com/browser/django/trunk/django/http/multipartparser.py#L31)
parses the POST request and returns a tuple of the POST and FILES data
`(request.POST, request.FILES)`. The MultiPartParser processes the
uploaded data using the [File Upload
Handlers](https://docs.djangoproject.com/en/dev/topics/http/file-uploads/#upload-handlers)
objects (through the
[new\_file](https://code.djangoproject.com/browser/django/trunk/django/core/files/uploadhandler.py#L87),
[receive\_data\_chunk](https://code.djangoproject.com/browser/django/trunk/django/core/files/uploadhandler.py#L100)
and
[upload\_complete](https://code.djangoproject.com/browser/django/trunk/django/core/files/uploadhandler.py#L116)
methods). The `request.FILES` values are a sequence of instances of
[UploadedFile](https://docs.djangoproject.com/en/dev/topics/http/file-uploads/#uploadedfile-objects).

In the django form, we pass the `request.FILES` MultiValueDict. These
UploadedFile instances are validated by the `full_clean` method on the
[Form](https://docs.djangoproject.com/en/dev/ref/forms/api/). The
[full\_clean](https://code.djangoproject.com/browser/django/trunk/django/forms/forms.py#L254)
method in turn calls the
[\_clean\_fields](https://code.djangoproject.com/browser/django/trunk/django/forms/forms.py#L273)
method which calls the
[clean](https://code.djangoproject.com/browser/django/trunk/django/forms/fields.py#L493)
method on the `forms.FileField` and checks if the data is empty.

After the form is successfully validated, we might assign and save the
`cleaned_data` of the form to a model instance (or by saving the
`ModelForm`). When the `save` method on the model instance is called,
the `save_base` method calls a `pre_save` method on each field of the
model. This
[pre\_save](https://code.djangoproject.com/browser/django/trunk/django/db/models/fields/files.py#L244)
method returns the value of the file instance bound to that FileField
and calls it\'s `save` method. This
[save](https://code.djangoproject.com/browser/django/trunk/django/db/models/fields/files.py#L84)
method (on the models.FileField) in turn calls the [save
(Storage)](https://code.djangoproject.com/browser/django/trunk/django/core/files/storage.py#L34)
method on the `Storage` which is either picked up from the arguments
passed to the FileField (`FileField(storage=...)`) or the
[DEFAULT\_FILE\_STORAGE](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-DEFAULT_FILE_STORAGE)
settings attribute.

This is the flow from the HTML Form all the way upto the File Storage.
Hope you liked it.
