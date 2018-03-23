---
layout: post
title:  "Getting started with webapp2 and GAE"
date:   2017-09-04 12:45:27+05:30
tags: appengine
author: akshar
---
Deploying a webapp2 app on Google app engine

We want to setup our dev environment to write a web application using webapp2 which we will deploy to Google app engine.

### Agenda

* About GAE(Google App Engine)
* Setting up dev environment for GAE
* Running webapp2 locally
* Deploying it on app engine

### About GAE

They key points around GAE are:

* It provides Platform as a Service. GAE is not Infrastructure as a service.
* Fully managed environment. Your runtime, libraries and frameworks are managed, installed and updated behind the scene. You don't have to do sysadmin task. You don't have to provision the server.
* GAE provides shell access when needed. <a href="https://cloud.google.com/appengine/">App engine docs</a> puts it as "Infrastructure when you need it."

### Setting up dev environment for GAE

GAE is part of Google Cloud Platform(GCP), so first we need GCP SDK.

GCP SDK can be installed from <a href="https://cloud.google.com/sdk/docs/">here</a>. Once GCP is installed, you should be able to run command **gcloud**.

If you are on Mac, you should see a directory called "google-cloud-sdk" in ~/Downloads. If you are on Ubuntu, you will find this directory "google-cloud-sdk" in whichever folder your downloads go.

Once GCP is installed, we need App engine extension for Python. App engine extension can be downloaded from <a href="https://cloud.google.com/appengine/docs/standard/python/download" target="_blank">here</a>. The most important thing is:

	gcloud components install app-engine-python

### Running webapp2 locally

Once app engine extension for python is installed, a directory called `google_appengine` is added to **platform** directory of Google Cloud SDK installation path.

So you should be able to see ~/Downloads/google-cloud-sdk/platform/google_appengine/. This directory has a file called **dev_appserver.py** which we need for running the app locallly.

Let's write some code now. Create a file main.py with following content

	import webapp2

	class HelloWebapp2(webapp2.RequestHandler):
		def get(self):
			self.response.write('Hello, webapp2!')

	app = webapp2.WSGIApplication([
		('/', HelloWebapp2),
	], debug=True)

	def main():
		from paste import httpserver
		httpserver.serve(app, host='127.0.0.1', port='8080')

	if __name__ == '__main__':
		main()
		
We need a yaml file called app.yaml which tells to app engine about runtime environment and other provisioning information. Add a file app.yaml at same level as main.py with following content.

	runtime: python27
	api_version: 1
	threadsafe: true

	handlers:
	- url: /.**
	  script: main.app

Start the dev server by issuing following command

	./<path-to-dev_appserver.py> app.yaml

### Deploying to GAE

Create or choose an exisiting **project** on GCP console from <a href="https://console.cloud.google.com/start">here</a>.

I have a project with id "the-pentameter-845". Using `gcloud config`, I set this as my current project so that I can deploy my code under this project.

	gcloud config set account akshar@agiliq.com
	gcloud config set project the-pentameter-845

And then deploy your code using following command

	gcloud app deploy

gcloud would prompt and ask you if you want to deploy to https://the-pentameter-845.appspot.com. Your url would be different depending on your project id.

With this deployment should be done and you should be able to view your project at https://<your-project-id>.appspot.com.

### Adding more routes

Add one more class.

	class NedStark(webapp2.RequestHandler):
		def get(self):
			self.response.write('Hi! I am Ned. People considered me very brave.')

And add it as a route so that this class gets used when someone accesses url /ned

	app = webapp2.WSGIApplication([
		('/', HelloWebapp2),
		('/ned', NedStark),
	], debug=True)

After this you should be able to access localhost:8080/ned. No change is needed in app.yaml

Deploy the app again.

	gcloud app deploy app.yaml

Now you should be able to access https://<your-project-id>.appspot.com/ned

### Installing libraries with pip

GAE runtime environment provides some third party libraries, so these need not be installed with pip. Similarly during development, dev_appserver.py provides these same libraries which need not be installed with pip. eg: One such library is webapp2 itself. You can see complete list of third party libraries available in runtime environment <a href="https://cloud.google.com/appengine/docs/standard/python/tools/built-in-libraries-27" target="_blank">here</a>

If you want to use a library which isn't provided by app engine runtime, you need to follow the steps mentioned <a href="https://cloud.google.com/appengine/docs/standard/python/tools/using-libraries-python-27#installing_a_third-party_library target="_blank">here</a>

Assuming you want to write a handler which has to fetch data from some database. Suppose you want to use SQLAlchemy for fethcing data, so we need SQLAlchemy in our application.

Add line `import sqlalchemy` in main.py and try to access the url. Your development server wouldn't respond because you don't have sqlalchemy. To add SQLAlchemy do the following.

	mkdir lib
	pip install -t lib/ SQLAlchemy

Add a file appengine_config.py with following content

	from google.appengine.ext import vendor

	vendor.add('lib')

Now access any url, and development server would again start serving the urls properly.


