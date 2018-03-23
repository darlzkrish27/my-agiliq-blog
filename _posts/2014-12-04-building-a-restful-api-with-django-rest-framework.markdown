---
layout: post
title:  "Building a RESTful API with Django-rest-framework"
date:   2014-12-04 21:26:02+05:30
tags: API
author: Rakesh
---
API's turned to be the heart of every application in our time. With the rise of social media, API's are being developed at a faster pace and gaining a lot of attention. Gone are the days where RPC architectures like CORBA and XML-RPC are used to enable the exchange of information and REST has taken its place making its mark in getting the things pretty straight forward.

We use APIs in our everyday life. Take an instance booking a ticket for a movie through some website. Here is the process that takes place in the background. When dealing with the payment for the ticket, the website connects to the bank and sends your credit card details to the remote application and it gets verified. And once, the payment is confirmed by the remote application, it sends a confirmation response to the movie booking website to issue the tickets. The payment stuff turned possible here as the bank facilitates an API through which a website can connect to it and deal with the payments which resulted us an effective and seamless transaction.

In this post we will get to know how to build a Restful API for an application using a Django-rest-framework. Before dipping ourselves in the API building stuff, let us acquire a picture on REST.

REST
-------

In simple words REST is a web architecture which lives on top of HTTP. It's a way in which we should use HTTP where it acts as a set of guiding principles behind the given application. It allows the clients and servers to communicate with each other in all the possible ways like getting the resource, posting the resource, deleting etc. REST is a resource based on which we use an HTTP verb to dictate what operation we want to do on a URI which can be GET, POST and so on. In REST we represent the resource data in JSON and XML format. An API is called a Restful API if it adheres to all the interaction constraints of REST.

Django-rest-framework
-----------------------
Django-rest-framework makes the process of building web API's simple and flexible. With its batteries included it won't be a tedious task to create an API. Before creating our own API let us look into some vital parts of the REST framework
The important parts of Django-rest-framework are:

* Serialization and Deserialization: 
The first part in the process of building an API is to provide a way to serialize and deserialize the instances into representations. Serialization is the process of making a streamable representation of the data which will help in the data transfer over the network. Deserialization is its reverse process. In our process of building an API we render data into JSON format. To achieve this Django-rest-framework provides JSONRenderer and JSONParser. 
JSONRenderer renders the request data into JSON, using utf-8 encoding and JSONParser parses the JSON request content.

Without dipping inside the technicalities let us see how it takes place.

Serializing data with JSONRenderer:

A JSONRenderer converts the request data into JSON using utf-8 encoding. Normally we use this when we want our data to be streamable.

    >>> data
    {'cal_id': u'2', 'username': u'tester'}
    >>> content = JSONRenderer().render(data)
    >>> content
    '{"cal_id": "2", "username": "tester"}'


Deserializing data with JSONParser:

A JSONParser parses the JSON request content. Firstly, we parse a stream into Python native datatypes with BytesIO and then we will restore those native datatypes into to a fully populated object instance afterwards. We use JSONParser to desrialize the data.

    >>> content
    '{"cal_id": "2", "username": "tester"}'
    >>> stream = BytesIO(content)
    >>> data = JSONParser().parse(stream)
    >>> data
    {u'username': u'tester', u'cal_id': u'2'}

* Requests and Responses

Requests and responses are the essential parts of Django-rest-framework which provides flexibility while parsing.

Request objects:

The request object in REST framework has more abilities. The attribute request.DATA in Request object is similar to Request.POST added with the capabilities of handling arbitrary data which works for 'POST', 'PUT' and 'PATCH' methods. And also we can use the different attributes like request.FILES, request.QUERY_PARAMS, request.parsers, request.stream which will help in dealing with the request a hassle free task.

Response objects:

The Response object helps to render the correct content type as requested by the client unlike the normal HttpResponse.

Syntax: Response(data, status=None, template_name=None, headers=None, content_type=None)

Status Codes: 
Instead of using the normal HTTP status codes, we will make use of explicit identifiers for every status code to make the process of reading the status code much simple. Here are some status codes that we represent in a REST framework we use normally.

    HTTP_200_OK
    HTTP_201_CREATED
    HTTP_400_BAD_REQUEST
    HTTP_401_UNAUTHORIZED
    HTTP_403_FORBIDDEN

* Views:
APIView is a view provided by Django-rest-framework which subclasses the Django's view class. If we use this view the requests passed to the handler methods will no more Django's HttpRequest instances and they will be REST framework's Request instances.
And while dealing with the responses they work to set the correct renderer on the response. The process of authentication and the permissions stuff also will be dealt with it.

Creating an API with Django-rest-framework
-------------------------

Using all the above stuff lets us build a simple API which gives the details of a user. The procedure goes the following way:

We will be using Django's built in user in this example. We will create a serializers.py file where we create serializers which are similar to Django forms. Just like model forms we have got model serializers here, which will stop replication of code. And we create a view which lists all the users, creates a new user, retrieve a user and update a user.

1. Here we go with the installation with virtual environment activated:
    
        pip install djangorestframework

2. Create a Django project and an app. I created it with a name rest_example and restapp

        django-admin.py startproject rest_example
        cd rest_example
        python manage.py startapp restapp

3. Add rest_framework and the app name to the installed apps. 

        INSTALLED_APPS = (
            ...
            'rest_framework',
            'restapp')

4. Run syncdb command

        python manage.py syncdb

5. Create a restapp/serializers.py which should look like the below way.

        from django.contrib.auth.models import User
        
        from rest_framework import serializers
        
        class UserSerializer(serializers.ModelSerializer):
            class Meta:
                model = User
                fields = ('id', 'username', 'first_name', 'last_name', 'email')

6. Create a restapp/views.py file for listing all users, creating a new user and so on.
    
        from django.contrib.auth.models import User
        from django.http import Http404
        
        from restapp.serializers import UserSerializer
        from rest_framework.views import APIView
        from rest_framework.response import Response
        from rest_framework import status
        
        class UserList(APIView):
        """
        List all users, or create a new user.
        """
            def get(self, request, format=None):
                users = User.objects.all()
                serializer = UserSerializer(users, many=True)
                return Response(serializer.data)
        
            def post(self, request, format=None):
                serializer = UserSerializer(data=request.DATA)
                if serializer.is_valid():
                    serializer.save()
                    return Response(serializer.data, status=status.HTTP_201_CREATED)
                return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
            
            def delete(self, request, pk, format=None):
                user = self.get_object(pk)
                user.delete()
                return Response(status=status.HTTP_204_NO_CONTENT)

        class UserDetail(APIView):
            """
            Retrieve, update or delete a user instance.
            """
            def get_object(self, pk):
                try:
                    return User.objects.get(pk=pk)
                except User.DoesNotExist:
                    raise Http404
        
            def get(self, request, pk, format=None):
                user = self.get_object(pk)
                user = UserSerializer(user)
                return Response(user.data)
            
            def put(self, request, pk, format=None):
                user = self.get_object(pk)
                serializer = UserSerializer(user, data=request.DATA)
                if serializer.is_valid():
                    serializer.save()
                    return Response(serializer.data)
                return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
            
            def delete(self, request, pk, format=None):
                user = self.get_object(pk)
                user.delete()
                return Response(status=status.HTTP_204_NO_CONTENT)
                
7. Update the root urls.py

        from django.conf.urls import patterns, include, url
        from django.contrib import admin
        
        from restapp import views
        
        admin.autodiscover()
        
            urlpatterns = patterns('',
            url(r'^admin/', include(admin.site.urls)),
            url(r'^users/', views.UserList.as_view()),
            url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),)
        
Voila!! we are done creating our API. 

Let us test our API now.
        
    python manage.py runserver

Now go to your browser and try localhost:8000/users/1/ or use curl on your command line

    curl http://127.0.0.1:8000/users/1/

You can get the user details what you have filled while creating the super user. which would look like this.

    {"id": 1, "username": "restapp", "first_name": "", "last_name": "", "email": "rakesh@agiliq.com"}
Now we can try posting data:

    curl -X POST http://127.0.0.1:8000/users/ -d '{"username":"rakhi", "email":"rakhi@agiliq.com"}' -H "Content-Type: application/json"

And it creates a new user with the username "rakhi" and email "rakhi@agiliq.com"

        {"id": 6, "username": "rakhi", "first_name": "", "last_name": "", "email": "rakhi@gmail.com"}

You can check out the example application on [github](https://github.com/krvc/rest_example)


