---
layout: post
title:  "Getting started with Django tastypie"
date:   2015-03-23 17:40:53+05:30
tags: django-tastypie
author: akshar
---
Django tastypie is a library to write RESTful apis in Django.

### Why use REST

You have a database backed web application. This application tracks expenses. The application allows the capability to enter your expenses, view all your expenses, delete an expense etc. Essentially this application provides CRUD functionality. Django application has access to database credentials, but they are never seen by the users of the web application. Django application decides what to show to which user. Django application ensures that a particular user only sees the expenses entered by him and not somebody else's expenses.

Now you want to provide a mobile application (Android or iOS) corresponding to this web application. Android application should allow the user to view his expenses, create an expense as well as any other CRUD functionality. But database credentials could not be put in Android code as it is not too hard to decompile an apk and get the db credentials. And we never want a user to get the db credentials else he will be in a position to see everyone's expenses and the entire database. So there has to be another way to allow mobile applications to get things from the database. This is where REST comes into picture.

With REST, we have three components. A database, a Django application and a mobile application. Mobile application never accesses the database directly. It makes a REST api call to Django application. Mobile application also sends a api_key specific to the mobile user. Based on api_key, Django application determines what data to make visible to this particular api_key owner and sends the corresponding data in response.

### Resource

REST stands for Representational State Transfer. It is a standard for transferring the state of a **Resource**, from web to mobile.

#### What do I mean by **state of a Resource**?

An expense could be a resource. A Person could be a resource. A blog post could be a resource. Basically any object or instance your program deals with could be a resource. And a resource's state is maintained in it's attributes. eg: You could have a model called Expense. The state of a expense instance is represented by its attributes.

Any REST library should be able to create and return a representation of such resource, which simply stated means that REST library should be able to tell us the attributes and their values for different model instances. And tastypie is adept at doing this.


### Setting up the application

I am using Django 1.7. Some things might be different for you if you are using different version of Django.

As with all projects, I want to keep things in a virtual environment

	$ mkvirtualenv tastier
	$ workon tastier

Install Django

	$ pip install Django

Start a Django project

	(tastier) $ django-admin.py startproject tastier

	(tastier) $ cd tastier/

Start an app

	(tastier) $ python manage.py startapp expenses

Add this app to INSTALLED_APPS

Run migration

	(tastier)~ $ python manage.py migrate

Runserver

	(tastier)~ $ python manage.py runserver

Check that your are able to access http://localhost:8000/admin/login/

I have pushed the code for this project to <a href="https://github.com/akshar-raaj/tastier" target="_blank">Github</a>. You will be able to checkout at different commits in the project to see specific things.

### Getting started

Install django-tastypie.

	(tastier) $ pip install django-tastypie

Create a file called `expenses/api.py` where you will keep all the tastypie related things.

Suppose your program deals with a resource called Expense. Let's create a model Expense in expenses/models.py

	class Expense(models.Model):
		description = models.CharField(max_length=100)
		amount = models.IntegerField()

Run migrations

	python manage.py makemigrations
	python manage.py migrate

We will later add a ForeignKey(User) to Expense to associate an expense with User. Don't worry about it for now, we will come back to it.

Let's add few Expense instances in the database.

	Expense.objects.create(description='Ate pizza', amount=100)
	Expense.objects.create(description='Went to Cinema', amount=200)

### Handling GET

You want the ability to get the representation of all expenses in your program at url "http://localhost:8000/api/expenses/".

To deal with a resource, tastypie requires a class which overrides **ModelResource**. Let's call our class **ExpenseResource**. Add following to expenses/api.py

	from tastypie.resources import ModelResource

	from .models import Expense

	class ExpenseResource(ModelResource):

		class Meta:
			queryset = Expense.objects.all()
			resource_name = 'expense'

And you need to add the following to tastier/urls.py

	from expenses.api import ExpenseResource

	expense_resource = ExpenseResource()

	urlpatterns = patterns('',
		url(r'^admin/', include(admin.site.urls)),
		url(r'^api/', include(expense_resource.urls)),
	)

#### GET all expenses

After this you should be able to hit

	http://localhost:8000/api/expense/?format=json

and you will see all the expenses from database in the response.

The response would be:

	{"meta": {"limit": 20, "next": null, "offset": 0, "previous": null, "total_count": 2}, "objects": [{"amount": 100, "description": "Ate pizza", "id": 1, "resource_uri": "/api/expense/1/"}, {"amount": 200, "description": "Went to Cinema", "id": 2, "resource_uri": "/api/expense/2/"}]}

You will find the representation of `expense` instances in **objects** key of response.

#### Get a particular expense

You can get the representation of expense with id 1 at

	http://localhost:8000/api/expense/1/?format=json

See how you are able to hit these two urls without adding them in urlpatterns. These urlpatterns are added by tastypie internally.

#### How these endpoints help and ties with mobile application example?

If the mobile app wants to show all the expenses it could use the url **http://localhost:8000/api/expense/?format=json**, get the response, parse the response and show the result on app.

Right now every user will see all the expenses. As we move forward we will see how only a user's expenses will be returned when a REST call is made from his/her mobile device.

#### Serialization

You must have realized that REST returns you serialized data. You might be wondering why use django-tastypie to achieve it, and not just use json.dumps. You can undoubtedly use json.dumps and not use django-tastypie to provide REST endpoints. But django-tastypie allows the ability to do many more things very easily as you will soon agree. Just hang on.

#### Changing Meta.resource_name

You can change ExpenseResource.Meta.resource_name from `expense` to `expenditure`.

	class ExpenseResource(ModelResource):

		class Meta:
			queryset = Expense.objects.all()
			resource_name = 'expenditure'

And then the old urls will stop working. Your new GET urls in that case will be

	http://localhost:8000/api/expenditure/?format=json
	http://localhost:8000/api/expenditure/1/?format=json

Changing the resource_name changes the urls tastypie makes available to you.

Now change the resource_name back to `expense`.

We have our first commit at this point. You can checkout to this commit to see the code till this point.

	git checkout b6a9c6

#### Meta.fields

Suppose you only want `description` in expense representation, but don't want `amount`. So you can add a fields attribute on ExpenseResource.Meta

	class Meta:
		queryset = Expense.objects.all()
		resource_name = 'expenditure'
		fields = ['description']

Try

	http://localhost:8000/api/expense/?format=json

So if you don't have **fields** attribute on Meta, all the attributes of Model will be sent in response. If you have **fields**, only attributes listed in fields will be sent in response.

Let's add `amount` also to `fields`. Though this gives us the same behaviour as not having ExpenseResource.Meta.fields at all.

	class Meta:
		queryset = Expense.objects.all()
		resource_name = 'expenditure'
		fields = ['description', 'amount']

We have our second commit at this point. You can checkout till this point by doing:

	git checkout 61194c

#### Filtering

Suppose you only want the Expenses where amount exceeds 150.

If we had to do this with Django model we would say:

	Expense.objects.filter(amount__gt=150)

**amount__gt** is the key thing here. This could be appended to our url pattern to get the expenses where amount exceeds 150.

This could be achieved at url

	http://localhost:8000/api/expense/?amount__gt=150&format=json

Try this. You will get an error because we haven't asked tastypie to allow filtering yet.

Add **filtering** attribute to ExpenseResource.Meta

    class Meta:
        queryset = Expense.objects.all()
        resource_name = 'expense'
        fields = ['description', 'amount']
        filtering = {
            'amount': ['gt']
        }

You should be able to use

	http://localhost:8000/api/expense/?amount__gt=150&format=json

This will only return the expenses where amount exceeds 150.

Now we want to get all the expenses on Pizza. We could get pizza expenses in following way from shell.

	Expense.objects.filter(description__icontains='pizza')

So to achieve this thing in api, we need to make following changes to ExpenseResource.Meta.filtering:

    class Meta:
        queryset = Expense.objects.all()
        resource_name = 'expense'
        fields = ['description', 'amount']
        filtering = {
            'amount': ['gt'],
            'description': ['icontains']
        }

And then following url would give us the pizza expenses

	http://localhost:8000/api/expense/?description__icontains=pizza&format=json

With GET endpoints we were able to do the Read operations. With POST we will be able to do Create operations, as we will see in next section.

### Handling POST

It's hard to do POST from the browser. So we will use **requests** library to achieve this.

Check expense count before doing POST.

	>>> Expense.objects.count()
	2

Tastypie by default doesn't authorize a person to do POST request. The default authorization class is **ReadOnlyAuthorization** which allows GET calls but doesn't allow POST calls. So you will have to disallow authorization checks for the time being. Add the following to ExpenseResource.Meta

	authorization = Authorization()

You'll need to import `Authorization` class for it.

	from tastypie.authorization import Authorization

After this, ExpenseResource would look like:

	class ExpenseResource(ModelResource):

		class Meta:
			queryset = Expense.objects.all()
			resource_name = 'expense'
			fields = ['description', 'amount']
			filtering = {
				'amount': ['gt'],
				'description': ['icontains']
			}
			authorization = Authorization()

Don't get into detail of Authorization for now, I will come back to it.

Let's make a POST request to our rest endpoint which will create an Expense object in the database.

	post_url = 'http://localhost:8000/api/expense/'
	post_data = {'description': 'Bought first Disworld book', 'amount': 399}
	headers = {'Content-type': 'application/json'}
	import requests
	import json
	r = requests.post(post_url, json.dumps(post_data), headers=headers)
	>>> print r.status_code
	201

status_code 201 means that your Expense object was properly created. You can also verify it by checking that Expense count increased by 1.

	>>> Expense.objects.count()
	3

If you hit the GET endpoint from your browser, you will see this new Expense object too in the response. Try

	http://localhost:8000/api/expense/?format=json

We have third commit at this point.

	git checkout 749cf3

#### Explanation of POST

* You need to POST at the same url where you get all the expenses. Compare the two urls.
* One way of posting is to POST json encoded data. So we used json.dumps
* If you are sending json encoded data, you need to send appopriate Content-type header too.

<br/>

#### How this ties in with mobile

Android or iOS has a way to make POST request at a given url with headers. So you tell mobile app about the endpoint where they need to post and the data to post. They will call this rest endpoint, and the posted data will be handled by Django tastypie and proper row will be created in the database table.

### Adding authentication

Currently GET and POST endpoints respond to every request. So even users who aren't registered with the site will be able to see the expenses. Our first step is ensuring that only registered users are able to use the GET endpoints.

#### Api tokens and sessions

In a web application, a user logs in once and then she is able to make any number of web requests without being asked to login every time. eg: User logs in once and then can see her expense list page. After first request she can refresh the page, and can still get response without being asked for her login credentials again. This works because Django uses sessions and cookies to store user state. So browser sends a cookie to Django everytime the user makes a request, and Django app can associate the cookie with a user and shows the data for this particular user.

With mobile apps, there is no concept of sessions, unless the mobile is working with a WebView. The session corresponding thing in a mobile app is Api key. So an api key is associated with a user. Every REST call should include this api key, and then tastypie can use this key to verify whether a logged in user is making the request.

#### Creating user and api token

Let's create an user in our system and a corresponding api token for her.

On a shell

	u = User.objects.create_user(username='sheryl', password='abc', email='sheryl@abc.com')

Tastypie provides a model called ApiKey which allows storing tokens for users. Let's create a token for Sheryl.

	from tastypie.models import ApiKey
	ApiKey.objects.create(key='1a23', user=u)

We are setting the api token for sheryl as '1a23'

You need to ensure tastypie is in INSTALLED_APPS and you have migrated before you could create ApiKey instance.

The default authentication class provided by tastypie is **Authentication** which allows anyone to make GET requests. We need to set ExpenseResource.Meta.authentication to ensure that only users who provide valid api key are able to get response from GET endpoints.

Add the following on ExpensesResource.Meta.

	authentication = ApiKeyAuthentication()

You need to import ApiKeyAuthentication.

	from tastypie.authentication import ApiKeyAuthentication

Try the GET endpoint to get the list of expenses

	http://localhost:8000/api/expense/?format=json

You will not see anything in response. If you see your runserver terminal, you'll notice that status code 401 is raised.

Api key should be sent in the request to get proper response.

Try the following url

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23

With this Sheryl will be able to get proper api response.

Try sending wrong api_key for sheryl and you will not see proper response.

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a2

With these we ensure that only registered users of the system with proper api key will be able to make GET requests.

Fourth commit at this point

	git checkout 48725f

#### How this ties in with mobile app

When user installs the app, he logs in using his username and password for first time. These credentials are sent to Django server using a REST call. Django server returns the api key corresponding to this user to the mobile app. Mobile app stores this api token on mobile end and then uses this token for every subsequent REST call. User doesn't have to provide the credentials anymore.

#### Making unauthenticated POST requests

Unauthenticated POST requests will not work anymore

Try creating an Expense without passing any api key.

	post_data = {'description': 'Bought Two scoops of Django', 'amount': 399}
	headers = {'Content-type': 'application/json'}
	r = requests.post("http://localhost:8000/api/expense/", data=json.dumps(post_data), headers=headers)
	print r.status_code       #This will give 401

Check that Expense count isn't increased

#### Making authenticated POST requests

You only need to change the url to include username and api_key in the url. This will make the request authenticated.

	r = requests.post("http://localhost:8000/api/expense/?username=sheryl&api_key=1a23", data=json.dumps(post_data), headers=headers)

This should have worked and Expense count should have increased.

Try with wrong api_key and it will fail.

### Getting only User's expense

Till now we aren't associating Expense to User. Let's add a ForeignKey to User from Expense.

Expense model becomes:

	from django.db import models
	from django.contrib.auth.models import User


	class Expense(models.Model):
		description = models.CharField(max_length=100)
		amount = models.IntegerField()
		user = models.ForeignKey(User, null=True)

Since we already have some Expenses in db which aren't associated with a User, so we kept User as a nullable field.

Make and run migrations

	python manage.py makemigrations
	python manage.py migrate

Right now our **authorization** class is set to **Authorization**. With this every user is **authorized** to see every expense. We will have to add a custom authorization class to enforce that users see only their expenses.

Add the following to expenses/api.py

	class ExpenseAuthorization(Authorization):

		def read_list(self, object_list, bundle):
			return object_list.filter(user=bundle.request.user)

And change **authorization** on ExpenseResource.Meta so it becomes:

	class ExpenseResource(ModelResource):

		class Meta:
			queryset = Expense.objects.all()
			resource_name = 'expense'
			fields = ['description', 'amount']
			filtering = {
				'amount': ['gt'],
				'description': ['icontains']
			}
			authorization = ExpenseAuthorization()
			authentication = ApiKeyAuthentication()

#### Explanation of ExpenseAuthorization

* When GET endpoint is called for expense list, object_list is created which gives all the expenses.
* After this, authorization is checked where further filtering could be done.
* In case of GET on list endpoint, authorization class' read_list() method is called. object_list is passed to read_list.
* In tastypie there is a variable called bundle. And bundle has access to request using bundle.request
* When authentication is used properly, bundle.request.user is populated with correct user.

Try expense list endpoint for Sheryl

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23

You will not get any expense after adding ExpenseAuthorization

	{"meta": {"limit": 20, "next": null, "offset": 0, "previous": null, "total_count": 0}, "objects": []}

This happenned because at this point no expense is associated with Sheryl.

#### Create an expense for Sheryl and try the GET endpoint

On the shell

	u = User.objects.get(username='sheryl')
	Expense.objects.create(description='Paid for the servers', amount=1000, user=u)
	Expense.objects.create(description='Paid for CI server', amount=500, user=u)

Try expense list endpoint for Sheryl again

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23

You should be able to see all of Sheryl's expenses.

Fifth commit here.

	git checkout 26f7c1

#### How mobile app will use it.

When Sheryl installs the app, she will be asked to login for the first time. There will be a REST endpoint which takes the username and password for a user and if the credentials are right, returns the api key for the user. Sheryl's api key will be returned to the mobile app which will store it in local storage. And when Sheryl wants to see her expenses, this REST call will be made to Django server.

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23

This will only return Sheryl's expenses.

### POST and create Sheryl's expense

Till now if a POST request is made, even if with Sheryl's api key, expense is created in db but is not associated with Sheryl.

We want to add functionality where if POST request is made from Sheryl's device then expense is associated with Sheryl. If POST request is made from Mark's device then expense should be associated with Mark.

Tastypie provides several hookpoints. We will use one such hookpoint. ModelResource provides a method called **hydrate** which we need to override. Add the following method to ExpenseResource.

    def hydrate(self, bundle):
        bundle.obj.user = bundle.request.user
        return bundle

* This method is called during POST/PUT calls.
* bundle.obj is an Expense instance about to be saved in the database.
* So we set **user** on bundle.obj by reading it from bundle.request. We have already discussed how bundle.request is populated during authentication flow.

Make a POST request now with Sheryl's api_key.

	post_data = {'description': 'Paid for iDoneThis', 'amount': 700}
	r = requests.post("http://localhost:8000/api/expense/?username=sheryl&api_key=1a23", data=json.dumps(post_data), headers=headers)

Verify that the latest expense instance gets associated with Sheryl. You can also verify it by seeing that this object gets returned in GET expense list endpoint.

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23
	
Sixth commit at this point

	git checkout 17b932

#### Try on your own

* Create one more user in database, from shell.
* Create api key for this user.
* POST to REST endpoint with this new user's api_key and username and verify that the expense gets associated with this new user.
* Check GET expense list for this new user and verify that only expense created for this user is in the response.

Now is a good time to dig deeper into django-tastypie and understand about following:

* Dehydrate cycle. It is used during GET calls.
* Hydrate cycle. It is used during POST/PUT calls. Once you read about hydrate cycle, you will understand when method **hydrate()** is called.
* More about authorization and different methods available on **Authorization** which could be overridden by you.


### Want more?

I am still trying few things with tastypie. Hereafter I will not have much explanation, but I will point to the commit where I attain certain functionality change.

### Authorization on detail endpoint.

Expense with id 1 is not associated with any user. But Sheryl is still able to see it at:

	http://localhost:8000/api/expense/1/?format=json&username=sheryl&api_key=1a23

She shouldn't be able to see it as it is not her expense.

So add the following to ExpenseAuthorization

    def read_detail(self, object_list, bundle):
        obj = object_list[0]
        return obj.user == bundle.request.user

After this Sheryl will not be able to see detail endpoint of any expense which doesn't belong to her. Try it

	http://localhost:8000/api/expense/1/?format=json&username=sheryl&api_key=1a23

Commit id for this:

	e650f3
	git show e650f3

### PUT endpoint

Expense with id 5 belongs to Sheryl. She wants to **update** this expense, she essentially want to change the description.

Current thing is:

	http://localhost:8000/api/expense/5/?format=json&username=sheryl&api_key=1a23

Make PUT request

	put_url = "http://localhost:8000/api/expense/5/?username=sheryl&api_key=1a23"
	put_data = {'description': 'Paid for Travis'}
	headers = {'Content-type': 'application/json'}
	r = requests.put(put_url, data=json.dumps(put_data), headers=headers)

Description of Expense 5 is updated as you can verify by trying the detail endpoint again.

	http://localhost:8000/api/expense/5/?format=json&username=sheryl&api_key=1a23

Notice that amount remains unchanged. So PUT changes whatever data you provide in the api call and lets everything else remain as it is.

### DELETE endpoint

First check all of Sheryl's expenses

	http://localhost:8000/api/expense/?format=json&username=sheryl&api_key=1a23

Sheryl wants to delete her expense with id 5. After this is done she will have one less expense in db.

	delete_url = "http://localhost:8000/api/expense/5/?username=sheryl&api_key=1a23"
	r = requests.delete(delete_url)

Verify that this expense got deleted.

So we were able to do Create, Read, Update and Delete with REST api calls.

### Restrict POST request to certain users only

Suppose we want the users to be able to create expenses from web end but don't want to allow creating expense from mobile using the api. Yeah, weird requiremtn.

Also we don't want to disallow POST for all users. We still want Sheryl to be able to POST.

To try this we first need a new user with api key in our system. Create it from Django shell.

	u = User.objects.create_user(username='mark', password='def', email='mark@abc.com')
	ApiKey.objects.create(key='2b34', user=u)

#### Restricting POST for everyone except Sheryl

Add following method to ExpenseAuthorization

    def create_detail(self, object_list, bundle):
        user = bundle.request.user
        # Return True if current user is Sheryl else return False
        return user.username == "sheryl"

Try making POST request as Mark and see you will not be able to do it. If you want you can see the expense count at this point.

	post_data = {'description': 'Petty expense', 'amount': 3000}
	r = requests.post("http://localhost:8000/api/expense/?username=mark&api_key=2b34", data=json.dumps(post_data), headers=headers)
	print r.status_code #Should have got 401

Also you can check the expense count again to verify that expense isn't created.

Status code 401 tells that you aren't authorized to do this operation.

#### Verify that Sheryl is still able to create expense

Try posting the same post_data as Sheryl

	r = requests.post("http://localhost:8000/api/expense/?username=sheryl&api_key=1a23", data=json.dumps(post_data), headers=headers)
	print r.status_code

Status code must be 201 in this case which means expense is created. You will be able to see this expense at Sheryl's GET expense list endpoint.

#### Verify that Mark is still able to do GET

Mark or any other user should still be able to make GET request even if he isn't able to make POST request.

	http://localhost:8000/api/expense/?username=mark&api_key=2b34&format=json

Since Mark doesn't have any expense in db, so no object is there is **objects** key of response. Try creating an expense for this user from shell and then try the GET endpoint again.

### Explicitly defining fields and customising them

ModelResource.Meta.fields can be dealt with in a similar way to ModelForm.Meta.fields. If you add the field to ModelResource.Meta.fields then it gets sane default behavriour. But it can be customised by adding the field explicitly on the ModelResource.

Let's try customising **description** field.

Add it explicitly on ExpenseResource

	description = fields.CharField()

You will need to import the following for it to work.

	from tastypie import fields

Try the GET endpoint

	http://localhost:8000/api/expense/?username=sheryl&api_key=1a23&format=json

You will see description as null for every expense. Because we committed a mistake.

While explicitly defining a field, we also need to tell the expense attribute that needs to be used for this particular field.

So we need to say

	description = fields.CharField(attribute='description')

Now GET endpoint will work as it used to work earlier.

But we did not achieve anything by explicitly adding the field. So what's the point of explicitly adding it.

Suppose you want the description in the detail endpoint but don't want it on list endpoint. This can be achieved in following way

	description = fields.CharField(attribute='description', use_in='detail')

Try the list and detail endpoint now and notice the difference.

	http://localhost:8000/api/expense/?username=sheryl&api_key=1a23&format=json
	http://localhost:8000/api/expense/8/?username=sheryl&api_key=1a23&format=json

**use_in** is documented <a href="http://django-tastypie.readthedocs.org/en/latest/fields.html#use-in" target="_blank">here</a>

