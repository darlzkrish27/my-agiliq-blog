---
layout: post
title:  "Tastypie with ForeignKey"
date:   2015-03-29 13:06:51+05:30
tags: django-tastypie
author: akshar
---
## Tastypie with ForeignKeys

This is a followup post on <a href="http://agiliq.com/blog/2015/03/getting-started-with-django-tastypie/" target="_blank">Getting started with tastypie</a>. We will use the same project setup as used in the last post.

This post will cover:

* Fetch ForeignKey data in GET calls
* Create an object with ForeignKeys using POST calls

### Setup the application

Let's add the capability to categorise the expenses

Add a model called ExpenseCategory

	class ExpenseCategory(models.Model):
		name = models.CharField(max_length=100)
		description = models.TextField()

Add a FK from Expense to ExpenseCategory

	class Expense(models.Model):
		description = models.CharField(max_length=100)
		amount = models.IntegerField()
		user = models.ForeignKey(User, null=True)
		category = models.ForeignKey(ExpenseCategory, null=True)

There already exists some Expense in db without an associated category, so make ExpenseCategory as nullable.

Create and apply migrations

	python manage.py makemigrations
	python manage.py migrate

Let's create an expensecategory from shell and associate it with an expense of user Sheryl.

	u = User.objects.get(username='sheryl')
	ec = ExpenseCategory.objects.create(name='Misc', description='Miscellaneous expenses')
	e = Expense.objects.create(description='Went to Stockholm', amount='5000', user=u, category=ec)

### Get FK fields in response too.

We want category in Expense GET endpoint too.

Our first approach would be adding 'category' to ExpenseCategory.Meta.fields. Try it

	fields = ['description', 'amount', 'category']

Try the expense GET endpoint for Sheryl

	http://localhost:8000/api/expense/?username=sheryl&api_key=1a23&format=json

Still don't see category in response. We need something more than this.

#### Adding fields.ForeignKey on ExpenseResource

There is no easy way to achieve this without adding a resource for ExpenseCategory.

We need to create an ExpenseCategoryResource similar to ExpenseResource

Add ExpenseCategoryResource to expenses/api.py

	class ExpenseCategoryResource(ModelResource):
		class Meta:
			queryset = ExpenseCategory.objects.all()
			resource_name = 'expensecategory'

Add proper url pattern for ExpenseCategoryResource in expenses/api.py

	expense_category_resource = ExpenseCategoryResource()

	urlpatterns = patterns('',
		url(r'^admin/', include(admin.site.urls)),
		url(r'^api/', include(expense_resource.urls)),
		url(r'^api/', include(expense_category_resource.urls)),
	)

Verify things are properly setup for ExpenseCategoryResource by accessing

	http://localhost:8000/api/expensecategory/?format=json

Add the following to ExpenseCategory

	category = fields.ForeignKey(ExpenseCategoryResource, attribute='category', null=True)

Try

	http://localhost:8000/api/expense/?username=sheryl&api_key=1a23&format=json

After this you'll be able to see category in response

This will return resource_uri of ExpenseCategory by default

#### Using full=True

Probably you want to see the name and description of category in the response 

Make the following modification

	category = fields.ForeignKey(ExpenseCategoryResource, attribute='category', null=True, full=True)
	
Try the GET endpoint again

	http://localhost:8000/api/expense/?username=sheryl&api_key=1a23&format=json

### POST data with FK

There are several ways in which we can set category on expense while making POST call to create expenses.

#### Post with resource_uri of FK

We already have one ExpenseCategory in the db and the resource_uri for that expensecategory is '/api/expensecategory/1/'

We want to create an expense and set the category as our earlier created expensecategory.

	post_data = {'description': 'Bought a phone for testing', 'amount': 2200, 'category': '/api/expensecategory/1/'}
	post_url = 'http://localhost:8000/api/expense/?username=sheryl&api_key=1a23'
	r = requests.post(post_url, data=json.dumps(post_data), headers=headers)

#### Posting entire data of FK

You find that the expense you want to create doesn't fit in any of the existing categories. You want to create a new expensecategory while making POST data to expense endpoint.

So we want to creating ExpenseCategory and Expense together.

You need to post the following data in such case.

	post_data = {'description': 'Went to paris to attend a conference', 'amount': 9000, 'category': {'name': 'Travel', 'description': 'Expenses incurred on travelling'}}

No category exists for Travel yet.

Check the count of ExpenseCategory currently so that later you can verify that a new ExpenseCategory is created.

	ExpenseCategory.objects.count()
	1   #output

POST the data to expense endpoint

	r = requests.post(post_url, data=json.dumps(post_data), headers=headers)
	print r.status_code    #401

##### Why you got 401

Even though you tried creating an Expense on expense post endpoint, tastypie internally tries creating an expensecategory because of structure of post_data. But tastypie finds that ExpenseCategoryResource doesn't have **authorization** to allow POST yet.

So we need to add proper authorization to ExpenseCategory before this post call can succeed.

Add the following to ExpenseCategoryResource.Meta

	authorization = Authorization()

#### POSTing again

Try the post call again.

	r = requests.post(post_url, data=json.dumps(post_data), headers=headers)

This would have worked well and a new ExpenseCategory should have been created.

	ExpenseCategory.objects.count()
	2    #output

Also the new expense would have got associated with the newly created ExpenseCategory.

#### POST with id of FK

	post_data = {'description': 'Bought second Disworld book', 'amount': 399, 'category': {'pk': 1}}

	r = requests.post(post_url, json.dumps(post_data), headers=headers)

### Optimizing FK field calculation

If you have full=True on FK resource then a database call will happen for each FK of each row.

eg: 

	category = fields.ForeignKey(ExpenseCategoryResource, attribute='category', null=True, full=True)

Here if you are hitting expense GET list, and suppose you are getting 20 expenses.

For each expense, it's category needs to be fetched from db and the categorie's full representation needs to be calculated. So 20 extra db calls will happen.

Fixing it. Set ExpenseResource.Meta.queryset to

	queryset = Expense.objects.all().select_related("category")

Now the extra 20 calls will not be made.

### Getting category_id without ExpenseCategoryResource

Suppose you are only interested in getting category_id in GET calls and don't care about name and description of category.

Comment out everything about ExpenseCategoryResource and relation to ExpenseCategoryResource from ExpenseResource

And add the following line to ExpenseResource

	category_id = fields.IntegerField(attribute='category_id', null=True)

After this you will find **category_id** in GET responses.

#### Handling POST

Similarly you want to be able to POST without worrying about ExpenseCategoryResource.

You want to send the category_id in post data and using that id, expense should be associated with correct expensecategory.

	data = {'description': 'Bought iphone', 'amount': 6500, 'category_id': 1}
	r = requests.post("http://localhost:8000/api/expense/?username=sheryl&api_key=1a23", data=json.dumps(data), headers=headers)

### Gotchas

We have a FK to ExpenseCategory from Expense and it is nullable. Similarly, we have an en FK from ExpenseResource to ExpenseCategoryResource which is nullable.

Mark ExpenseCategoryResource non-null for now, i.e remove null=True from it.

So it looks like

	category = fields.ForeignKey(ExpenseCategoryResource, attribute='category', full=True)

Check the count of expenses in the db.

	In [20]: Expense.objects.count()
	Out[20]: 23

Make a POST request to expense endpoint but don't pass any associated expensecategory.

	post_url = 'http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23'
	post_data = {'description': 'Bought second Disworld book', 'amount': 399}
	r = requests.post(post_url, json.dumps(post_data), headers=headers)
	print r.status_code
	400 #output

Status code 400 means that request wasn't successul and so expense shouldn't have been created.

But actually it was created even though tastypie returned us a 400 status code. You can check it using count.

	In [22]: Expense.objects.count()
	Out[22]: 24

This happened because at the model/db level ExpenseCategory allows null. So save() on Expense was successful. But at api/tastypie level, ExpenseCategoryResource is not nullable, so tastypie raised an error.

