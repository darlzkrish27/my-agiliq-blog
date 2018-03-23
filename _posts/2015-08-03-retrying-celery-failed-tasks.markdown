---
layout: post
title:  "Retrying celery failed tasks"
date:   2015-08-03 10:53:19+05:30
tags: celery
author: akshar
---
I assume you have a basic understanding of celery. If you want to learn about basics of celery, you can check our <a href="http://agiliq.com/blog/2015/07/getting-started-with-celery-and-redis/" target="_blank">last blog</a>.

### Use case

In one of my projects, I work with Twitter api. I need to fetch a user's tweets. Twitter provides an api endpoint for fetching user's tweets. Fetching of tweets involve network calls and so should happen in background, so we fetch the tweets using a celery task. So I have a celery task which makes one api call to Twitter. If I am able to fetch the tweets I consider the celery task was successful.

But Twitter rate limits this api endpoint and I can't hit this api endpoint indefinite number of times. Twitter allows maximum of 180 calls to this endpoint in a 15 minute window. Any call beyond 180 calls will start failing and Twitter will raise an exception instead of returning tweets. If Twitter raises exception instead of returning tweets, I consider that the celery task has failed.

Assuming a very active Twitter user signs up and we need 200 api calls to fetch all his tweets. So my celery tasks run 200 times. First 180 tasks would succeed but next 20 tasks would fail because of rate limiting.

I do not want to miss any tweet by a user, and so any failed task must be retried after 15 minutes. This is where celery retry functionality comes into picture.

#### Pseudocode for my usecase

	@app.task(bind=True)
	def fetch_tweets(token_details):
		# token_details is user specific User token that needs to be passed to Twitter
		try:
			resp = make_twitter_call()	
			# Till 180 calls to Twitter, we will get `resp`
			# process result
		except TwitterException:
			# Till 180th call, this part will not be executed
			# 181th call onwards, Twitter will raise an error and this part of code will be executed
			# Retry fetch_tweets after 15 mins.

Before fixing this use case, let's play around with some basic examples.

### Basic example

Create a file called add.py with following content

###### add.py

	from celery import Celery

	app = Celery('add', broker='redis://localhost:6379/0')

	@app.task(bind=True)
	def add(self, x, y):
		return x + y

Ensure task defined in this file is running properly by running the worker and trying to run the task from ipython shell.

###### Worker terminal

	celery worker -A add -l info

###### Ipython

	from add import add
	add.delay(3, 4)

After you do this, addition should happen and "7", i.e 3+4, should be printed on worker terminal.

#### Explicity raising error

Setting up Twitter access tokens etc will take effort and time, so instead of setting up Twitter, let's try to replicate a similar scenario where some calls to our task succeed and some calls to task fail.

Let's first replicate a failed task. A failed task means an error happening in the task. Raise an error in your task, and see how it behaves.

	from celery import Celery

	app = Celery('add', broker='redis://localhost:6379/0')

	@app.task(bind=True)
	def add(self, x, y):
		raise Exception()
		return x + y

Restart celery worker

	celery worker -A add -l info

Run the task again

	from add import add
	add.delay(3, 4)

On celery terminal, you will see that an exception is raised.

	[2015-07-31 16:39:13,440: ERROR/MainProcess] Task add.add[fb067fbf-e79e-4fdd-b398-94da6be960e1] raised unexpected: Exception()
	Traceback (most recent call last):
	  File "/Users/akshar/.virtualenvs/hack/lib/python2.7/site-packages/celery/app/trace.py", line 240, in trace_task
		R = retval = fun(*args, **kwargs)
	  File "/Users/akshar/.virtualenvs/hack/lib/python2.7/site-packages/celery/app/trace.py", line 438, in __protected_call__
		return self.run(*args, **kwargs)
	  File "/Users/akshar/Play/Python/hack/add.py", line 7, in add
		raise Exception()
	Exception

Suppose you want to retry the task after 2 seconds when it fails. Make the following code change

	from celery import Celery

	app = Celery('add', broker='redis://localhost:6379/0')

	@app.task(bind=True)
	def add(self, x, y):
		try:
			raise Exception()
		except Exception as e:
			self.retry(countdown=2, exc=e)
		return x + y

Restart celery, run the task and check the output. Output would be

	2015-07-31 16:59:24,027: INFO/MainProcess] Received task: add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] eta:[2015-07-31 11:29:26.014576+00:00]
	[2015-07-31 16:59:24,030: INFO/MainProcess] Task add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] retry: Retry in 2s: Exception()
	[2015-07-31 16:59:26,075: INFO/MainProcess] Received task: add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] eta:[2015-07-31 11:29:28.065412+00:00]
	[2015-07-31 16:59:26,076: INFO/MainProcess] Task add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] retry: Retry in 2s: Exception()
	[2015-07-31 16:59:28,127: INFO/MainProcess] Received task: add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] eta:[2015-07-31 11:29:30.115907+00:00]
	[2015-07-31 16:59:28,128: INFO/MainProcess] Task add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] retry: Retry in 2s: Exception()
	[2015-07-31 16:59:30,771: ERROR/MainProcess] Task add.add[f9c6a487-9fd7-4bb2-8db6-2c58849809c7] raised unexpected: Exception()
	Traceback (most recent call last):
	  File "/Users/akshar/.virtualenvs/hack/lib/python2.7/site-packages/celery/app/trace.py", line 240, in trace_task
	  ...
	  ....
	  File "/Users/akshar/Play/Python/hack/add.py", line 8, in add
		raise Exception()
	Exception

#### Explanation

* Task ran for first time.
* An error was raised which you caught.
* We retried the task after 2 seconds in case of error.
* Currently our task is written in such a way that every run of task raises error. So first retry of task also raised error.
* Task was retried for 3 times, which is the celery default.
* Third retry also raised an error, but instead of executing the code of "except" block, celery is smart to figure out that a task should only be retried 3 times and should not be retried forever.

#### Retry only once

Suppose you only want to retry a failed task once instead of three which is the celery default.

Change self.retry line, so it looks like

	self.retry(countdown=2, exc=e, max_retries=1)

Restart celery and run the task, output would be

	[2015-07-31 17:36:57,065: INFO/MainProcess] Received task: add.add[d1b099a4-026f-4bff-9bfd-8833a2b742e3]
	[2015-07-31 17:36:57,086: INFO/MainProcess] Received task: add.add[d1b099a4-026f-4bff-9bfd-8833a2b742e3] eta:[2015-07-31 12:06:59.076743+00:00]
	[2015-07-31 17:36:57,087: INFO/MainProcess] Task add.add[d1b099a4-026f-4bff-9bfd-8833a2b742e3] retry: Retry in 2s: Exception()
	[2015-07-31 17:37:00,968: ERROR/MainProcess] Task add.add[d1b099a4-026f-4bff-9bfd-8833a2b742e3] raised unexpected: Exception()
	Traceback (most recent call last):
	  File "/Users/akshar/.virtualenvs/hack/lib/python2.7/site-packages/celery/app/trace.py", line 240, in trace_task
	  ......
	  File "/Users/akshar/Play/Python/hack/add.py", line 8, in add
		raise Exception()
	Exception

This time, the failed task was retried only once.

### Replicating exact Twitter scenario

We want the task to succeed in some cases and fail in other cases. Modify your code, so it looks like:

	import random

	from celery import Celery

	app = Celery('add', broker='redis://localhost:6379/0')

	@app.task(bind=True)
	def add(self, x, y):
		# Get a random number between 1 and 10
		num = random.randint(1, 10)
		print num # To help properly understand output
		try:
			# If number is odd, fail the task
			if num % 2:
				raise Exception()
			# If number is even, succeed the task
			else:
				return x + y
		except Exception as e:
			self.retry(countdown=2, exc=e, max_retries=1)

#### Explanation

In our task, we generate a random number between 1 and 10. If the number is even then the task succeeds and returns the sum of numbers. If the generated random number is odd then the task fails, in which case we retry the task after 2 seconds.

#### Outputs of some random runs of tasks

	[2015-07-31 17:47:13,800: INFO/MainProcess] Received task: add.add[879c089d-54fb-4592-96e4-325f61bdce39]
	[2015-07-31 17:47:13,801: WARNING/Worker-3] 7
	[2015-07-31 17:47:13,822: INFO/MainProcess] Received task: add.add[879c089d-54fb-4592-96e4-325f61bdce39] eta:[2015-07-31 12:17:15.811826+00:00]
	[2015-07-31 17:47:13,823: INFO/MainProcess] Task add.add[879c089d-54fb-4592-96e4-325f61bdce39] retry: Retry in 2s: Exception()
	[2015-07-31 17:47:16,873: WARNING/Worker-1] 8
	[2015-07-31 17:47:16,874: INFO/MainProcess] Task add.add[879c089d-54fb-4592-96e4-325f61bdce39] succeeded in 0.00106367497938s: 7

Generated random number was 7. So task raised an exception and so it was retried in 2 secs. In the retried run, generated random number was 8 which is even and so no exception was raised and instead the sum was calculated and returned.

Another output

	[2015-07-31 17:47:33,681: INFO/MainProcess] Received task: add.add[64e43f81-0f80-49ac-a068-4c84c689ea16]
	[2015-07-31 17:47:33,681: WARNING/Worker-2] 7
	[2015-07-31 17:47:33,703: INFO/MainProcess] Received task: add.add[64e43f81-0f80-49ac-a068-4c84c689ea16] eta:[2015-07-31 12:17:35.692557+00:00]
	[2015-07-31 17:47:33,704: INFO/MainProcess] Task add.add[64e43f81-0f80-49ac-a068-4c84c689ea16] retry: Retry in 2s: Exception()
	[2015-07-31 17:47:36,890: WARNING/Worker-4] 3
	[2015-07-31 17:47:36,894: ERROR/MainProcess] Task add.add[64e43f81-0f80-49ac-a068-4c84c689ea16] raised unexpected: Exception()
	Traceback (most recent call last):
	  File "/Users/akshar/.virtualenvs/hack/lib/python2.7/site-packages/celery/app/trace.py", line 240, in trace_task
      ...
	  File "/Users/akshar/Play/Python/hack/add.py", line 15, in add
		raise Exception()
	Exception

Generated random number was 7. So task raised an exception and so it was retried in 2 secs. In the retried run, generated random number was 3 which is again odd, so retried task also raised an exception. And since we have set `max_retries` as 1, no furter retry was done.

#### Setting a datetime instead of countdown

Till now we have been working with **countdown** which tells the number of seconds in which failed task should be retried. We can also set a datetime at which task should be retried.

Let's retry the failed task after 2 seconds but using datetime instead of countdown.

	import random
	import datetime
	import pytz

	from celery import Celery

	app = Celery('add', broker='redis://localhost:6379/0')

	@app.task(bind=True)
	def add(self, x, y):
		# Get a random number between 1 and 10
		num = random.randint(1, 10)
		print num
		try:
			# If number is odd, fail the task
			if num % 2:
				raise Exception()
			# If number is even, succeed the task
			else:
				return x + y
		except Exception as e:
			dt = datetime.datetime.now(pytz.utc) + datetime.timedelta(seconds=1)
			self.retry(eta=dt, exc=e, max_retries=1)

Output

	[2015-07-31 18:59:17,177: INFO/MainProcess] Received task: add.add[b5f95e50-da3a-4295-871a-fc940bef5c46]
	[2015-07-31 18:59:17,178: WARNING/Worker-3] 1
	[2015-07-31 18:59:17,200: INFO/MainProcess] Received task: add.add[b5f95e50-da3a-4295-871a-fc940bef5c46] eta:[2015-07-31 13:29:19.178833+00:00]
	[2015-07-31 18:59:17,202: INFO/MainProcess] Task add.add[b5f95e50-da3a-4295-871a-fc940bef5c46] retry: Retry at 2015-07-31 13:29:19.178833+00:00: Exception()
	[2015-07-31 18:59:19,690: WARNING/Worker-1] 10
	[2015-07-31 18:59:19,691: INFO/MainProcess] Task add.add[b5f95e50-da3a-4295-871a-fc940bef5c46] succeeded in 0.00107540999306s: 7

### Retrying failed Twitter tasks

I assume you have the idea of <a href="https://pypi.python.org/pypi/twitter" target="_blank">Python twitter library.</a>. My code before using retry looked like.

	import twitter

	@app.task(bind=True)
	def fetch_tweets(self)
		auth = twitter.OAuth(oauth_token, oauth_token_secret, key, secret)
		client = twitter.Twitter(auth=auth)
		params = {'user_id': user_id, 'count': 2}
		response = client.favorites.list(**params)
		return response

Till first 180 calls, tasks were succeeding. 181st call onwards the tasks started failing. So I changed it to.

	import twitter

	@app.task(bind=True)
	def fetch_tweets(self)
		auth = twitter.OAuth(oauth_token, oauth_token_secret, key, secret)
		client = twitter.Twitter(auth=auth)
		params = {'user_id': user_id, 'count': 2}
		try:
			response = client.favorites.list(**params)
		except twitter.TwitterError as e:
			if e.e.code == 429:
				# 429: rate limit exceeded
				# Ask Twitter to tell at what time rate limit window gets reset
				rate_limit_status = self.client.application.rate_limit_status(resources="statuses")
				reset = rate_limit_status["resources"]["statuses"]["/statuses/user_timeline"]["reset"]
				dt = datetime.datetime.utcfromtimestamp(reset)
				raise self.retry(eta=dt, exc=e, max_retries=1)
		return response

After this, any failed tasks were retried when Twitter rate limit window gets reset.

