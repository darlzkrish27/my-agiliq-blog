---
layout: post
title:  "Not exactly, not exactly tim the enchanter"
date:   2012-11-07 11:30:00+05:30
tags: forms
author: Javed
---
I've been putting off writing about my experience with django form wizard
for some time now. I came across [this blog post by Kenneth Love](http://brack3t.com/not-exactly-tim-the-enchanter.html)
which finally compelled me to write this down.


About Django Form Wizard:
-------------------------

Form Wizard can be handy when you have a huge form, and you want to make it less
intimidating to the user by splitting it into multiple steps.

If you have glued multiple forms together with `sessions` or whatnot and it
turned out to be a mess, you now what I'm talking about :)

Think surveys, checkouts, lengthy registrations etc.

The class `WizardView` implements most of functionality while subclasses
`SessionWizardView`, `CookieWizardView` etc. specify which backend to use.

Typically, the view takes a list of forms and presents them one by one.
After each step, the form is validated and the raw form data is saved in the backend
(session or cookie for POST data and `file_storage` for files). After the last step,
each form is filled with raw data from the backend and validated again.

The list of fully-filled up form instances is then passed to the `done` handler
where the forms can be processed.

I guess the key bit of info here is: Nothing is written to the database until all
the steps are completed and the forms are saved manually.


I know that feel:
-----------------

I'm not a fan of putting more stuff than necessary in `urls` either, and I think
this can be easily remedied the way Kenneth wants it.

For example, to specify `forms_list` and `url_name` in the `views` instead of `urls` one could do:


    from holygrail.forms import NameForm, QuestForm, ColorForm


    class MerlinWizard(NamedUrlSessionWizardView):

        ...
        ...

        @classonlymethod
        def as_view(cls, *args, **kwargs):
            kwargs.update({
                'form_list': [
                    NameForm,
                    QuestForm,
                    ColorForm,
                ],
                'url_name': 'merlin_wizard'
            })
            return super(MerlinWizard, cls).as_view(*args, **kwargs)

        ...
        ...

and voilà, you no longer need to pass any args to `MerlinWizard`


ModelForms/Formsets:
--------------------

I think modelforms/formsets are perfectly fine for usage in the form wizard,
although, you'll need to dive into a few details:

Save a modelform:

We can use the `process_step` method here. This methods takes the form instance
and returns the data to be saved in the storage for the current step. By default,
it returns the raw form data.

    def process_step(self, form):
        """
        you can save the form here,
        since it's guaranteed to be valid
        """
        if self.steps.current == '2':
            instance = form.save()
        return super(MerlinWizard, self).process_step(form)

If you need to pass around the instance to another step, you can add it to the step's storage data e.g:

Let's say we need to save a model in step 2 and create a formset based on its instance
in step 3:

    def process_step(self, form):
        """
        you can save the current form here,
        since it's guaranteed to be valid
        """
        step_data = super(MerlinWizard, self).process_step(form).copy()
        if self.steps.current == '2':
            instance = form.save()
            step_data['instance_pk'] = instance.pk
        return step_data

then you can retrieve the `instance_pk` like this:

    def get_form_kwargs(self, step=None):
        """
        in step 3, use the instance from step 2
        as a kwarg to the formset
        """
        kwargs = super(MerlinWizard, self).get_form_kwargs(step=step)
        if step == '3':
            step_data = self.storage.get_step_data('2')
            instance_pk = step_data['instance_pk']
            instance = MyModel.objects.get(pk=instance_pk)
            kwargs.update({
                'instance': instance
            })
        return kwargs


Defaults:
---------

I agree with Kenneth that storages should just work™ like they work with forms/modelforms.

But, it's handy when you want to keep the wizard media away for the rest of the user media.

One workaround is to just set the file storage to the default storage engine:

    from django.core.files.storage import default_storage

    ...
    ...

    class MerlinWizard(NamedUrlSessionWizardView):

        file_storage = default_storage
        ...
        ...


Are we `done`, yet?:
--------------------

You can handle all the forms you want to save to the database in `done` method.

AFAIK, there's no need to do:

    del self.request.session["wizard_key_here"]

The wizard cleans up both when it's started and when it's `done`.
Also, relying on `request.session` breaks the storage abstraction,
so you're better off using `self.storage` if your *really* want to mess with the storage.
This way, you can switch to a different wizard backend without worrying too much.


Conclusion:
-----------

Having spent a good deal of time with the form wizard over the past few weeks, I feel that
form wizard does a very good job at handling the use case it was designed for.

Even though the documentation is not perfect, the implementation is clear, concise and flexible
enough to handle a variety of scenarios.



