---
layout: post
title:  "When and how to use Django ListView"
date:   2017-12-29 11:36:39+05:30
tags: django
author: akshar
---
### When to use ListView?

Django provides several class based generic views to accomplish common tasks. One among them is ListView. Our <a href="http://agiliq.com/blog/2017/12/when-and-how-use-django-templateview/" target="_blank">last post</a> was on TemplateView.

ListView **should be used** when you want to present a list of objects in a html page.

ListView **shouldn't be used** when your page has forms and does creation or update of objects.

TemplateView can achieve everything which ListView can, but ListView avoids a lot of boilerplate code which would be needed with TemplateView.

Let's write a view using base view **View** and then modify it to use TemplateView and then to use ListView. ListView would help us avoid several lines of code and would also provide better separation of concern.

### Vanilla View

Assume there is a model called Book which looks like:

    class Book(models.Model):
        name = models.CharField(max_length=100)
        author_name = models.CharField(max_length=100)

We want to have a page which shows all the books in the database. View would look like:

	class BookListView(View):

		def get(self, request, *args, **kwargs):
			books = Book.objects.all()
			context = {'books': books}
			return render(request, "book-list.html", context=context)

book-list.html looks like the following:

	{% for book in books %}
	  {{book.name}}
	  <br/>
	{% endfor %}

#### By subclassing TemplateView

	class BookListView(TemplateView):
		template_name = 'book-list.html'

		def get_context_data(self, *args, **kwargs):
			context = super(BookListView, self).get_context_data(*args, **kwargs)
			context['books'] = Book.objects.all()
			return context

As discussed in last post on TemplateView, we didn't have to provide a get() implementation and bother with render() while using TemplateView. All that was taken care of by TemplateView.

#### By subclassing ListView

	from django.views.generic.list import ListView

	class BookListView(ListView):
		template_name = 'book-list.html'
		queryset = Book.objects.all()
		context_object_name = 'books'

ListView removes more boilerplate from TemplateView. With ListView we didn't have to bother with get_context_data() implementation. ListView takes care of setting context variable 'books' with the queryset we defined on BookListView.

We can also add filtering in ListView.queryset.

	class BookListView(ListView):
		template_name = 'book-list.html'
		queryset = Book.objects.filter(name='A Feast for Crows')
		context_object_name = 'books'

Had we wanted pagination, we would have had to add several lines of code in TemplateView or vanilla View implementation. ListView provides pagination for free, we don't have to add pagination code.

Pagination can be added to ListView subclasses by setting a variable `paginate_by`

	class BookListView(ListView):
		template_name = 'book-list.html'
		queryset = Book.objects.all()
		context_object_name = 'books'
		paginate_by = 10

After this **/books-list/?page=1** will return first 10 books. **/books-list/?page=2** will return next 10 books and so on.

