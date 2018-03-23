---
layout: post
title:  "Using a Postman http client for efficient HTTP testing"
date:   2014-09-08 16:05:43+05:30
tags: http client
author: Rakesh
---

Interacting with the API's have become a significant part of software development and constructing and testing them effectively will always boost up the business firm.

POSTMAN HTTP and REST client is one of its kind which makes the API testing process effortless. It is available as both a Google Packaged App and a Google Chrome in-browser app. It has got clean and intuitive user interface, with all the important features accessible within one click. The performance of the API testing process gets drastically affected by following features it has:

1. Create request quickly
2. Collections
3. Environments
4. Built-in authentication helpers

1. Create request quickly
--------------------------

The Postman interface makes the process of request creation real quick. The four important parts of the HTTP request, i.e. URL, method, headers and the request body are given with effective tools to work with.

URL :

The URL input field stores URL's stored previously and will show an auto-complete drop down as we begin entering our URL.
Sometimes it becomes troublesome while testing complicated URLs which contains a lot of URL parameters, the POSTMAN REST client contains URL params button. Once it is clicked it displays a key value editor in which we can add the URL parameters.

Headers:

There is Headers button which displays a header editor added to an auto-complete drop down of all the common headers. The best part of headers in POSTMAN are we can save commonly used headers together in a header-preset. For example, I work on a project which contains a different Authorization key for local and my Amazon server so if I create a header preset to my local and Amazon by clicking the "Add preset" button the headers can be prefilled and I need not change the header key values manually all the time.

Method:

Changing the method is straightforward. We can select the method like GET, POST, PUT, PATCH, DELETE, etc. In the drop down from the select control. The request body editor area will change depending on whether the method can have a body attached to it or not.


Request body:

Using POSTMAN we can send almost any kind of HTTP request. The Request body contains three areas: form-data, urlencoded, raw. The form-data editor lets us set key/value pairs for our data. A raw request can contain anything, we can also send the formatting type along with the correct header.

2. Collections
-----------------------

We tend to test different requests in a project and it will make our testing process simple if we can organize all the API calls separately basing on the similarities. A collection is a group of individual requests which are accessible on the left sidebar. The requests are further organized into folders to completely mirror our API. To make the request more descriptive we can add the meta data like name and description which makes the searching a needed request a painless job.

We can easily create a collection by clicking on the "new collection" icon on the sidebar. If we have a request that is loaded in the request editor already we can save it to a specific collection by clicking "Add to collection". This feature helps us to replay the same task of testing a request effortlessly.

3. Environments
--------------------

Often while working with APIs we will need to have different setups. For example, our local machine, the development server or the production API. Environments give us the ability to customize requests using variables. We need not worry about remembering all the values once they are in POSTMAN saved in an environment. As an add on there is a capability to download the environments and can be saved as a JSON file. Environments also help you separate sensitive data from your collection like keys and passwords. This feature helps us to switch the context quickly basing on the server we are working on.


4. Built-in authentication helpers
-------------------------

Postman supports Basic Auth, Digest Auth and OAuth 1 helpers. It becomes easier with POSTMAN while dealing with different types authentications like Digest Auth which are bit complicated than Basic Auth due its flexible interface.

Example:

<img src="http://i.imgur.com/ru1RLtc.png" alt="Postman example" style="width:830px">

In the above example I am trying to get the "mentions" of my twitter timeline. I am using Oauth 1.0 to authenticate the API. Once I select the OAuth 1.0 in the "type of authentication" it shows up the Keys and we can fill up the values over there. The next part contains the URL and the URL params. We may tend to change the count value and since_id to test the API, it becomes simple when we try to fill those values in the URL params. The left side bar is my collection, which contains different API requests.


Bonus Stuff:  API management software
------------

An API management software always increases the productivity. Every day an API can receive a thousand or potentially millions of requests. Before the API processes those requests it has to deal with some important functionalities:

1. Identity/ Authentication
2. Traffic controls
3. Rate limiting
4. Performance
5. Security
6. Scalability
7. Filtering
8. Encryption
9. Logging, etc.,

Once the above things are handled perfectly, we can accomplish our goal. To deal with them service providers like APIGEE, 3scale and Mashery came into the picture. The important service they provide is API proxy. An API proxy can protect API's against attack and misuse. They have the capability to translate between JSON and XML. They also can track the API usage and performance.

Let's create a proxy using 3scale to get an idea of using the proxy.

1. Create an account on 3scale. http://www.3scale.net/logmein/
2. Setup the application
3. Declare the API backend, i.e. the endpoint host of our API. The proxy will direct all traffic from sandbox development endpoint after processing authentication, authorization, rate limits and statistics.
4. Turn on the sandbox proxy
5. Go to the Applications tab and get the credentials i.e the user_key.
6. Now we can access the  sandbox endpoint which may look something like this http://api.2445579856672.proxy.3scale.net/ from the dashboard.

Added to that we can also create mapping rules for every request by specifying the Rule and patterns of the request so that we can test a specific request functionality and whether the endpoint is working properly or not form 3scale itself.

The API management software service providers like APIGEE, 3scale also provide different services like Analytics services, API consoles, API modelling, API management etc.,


