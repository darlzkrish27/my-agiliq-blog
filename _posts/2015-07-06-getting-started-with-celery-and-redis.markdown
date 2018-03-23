---
layout: post
title:  "Getting started with Celery and Redis"
date:   2015-07-06 10:54:44+05:30
tags: redis
author: akshar
---
### Agenda

* When to use Celery.
* Why to use Celery.
* A simple celery program.
* Having a slow script and making it faster using celery.
* Celery configuration and code in different files.
* Using celery with tasks spanned across multiple modules
* Using celery with a package.
* Redis and celery on separate machine
* Web-application/script and celery on separate machines.

### When to use Celery

Celery is a task processing system. It is useful in a lot of web applications.

It can be used in following scenarios.

To do any network call in a request-response cycle. Server should respond immediately to any web request it receives. If some network call is required during a request-response cycle, it should be done outside of request-response cycle. eg: An activation email needs to be sent when user signs up on a site. Sending the email is a network call and might take 2-3 seconds. User should not be made to wait for these 2-3 seconds. So sending activation email should be done outside of request-response cycle. It can be achieved using celery.

Breaking a large task consisting of several independent parts into smaller tasks. eg: Consider you want to read a user's FB timeline. FB provides different endpoints to get different kind of things. FB provides one endpoint to get pictures on a user's timelines, another endpoint to get posts on a user's timelines, another endpoint to get likes of a user etc. If you write a single function to sequentially hit 5 endpoints provided by FB and if network calls take 2 seconds at an average, then your function will take 10 seconds to complete. So you can split your work in 5 individual tasks(it's very easy to do as we will soon see), and let Celery handle the tasks. Celery can hit these 5 endpoints parallely and you can get the response from all the endpoints within first 2 seconds.

### Why to use Celery

We want web responses to be fast. So on user signup, server should send the response immediately and the actual job of sending the email should be sent to celery. Celery would be running in background, outside of request-response cycle and it can send the actual email.

We can use celery to make our scripts faster and to make better utilization of cpu. In the FB example I described earlier, we can go from 10 seconds to 2 seconds and also our cpu utilization would be higher if we use celery.

We can use celery to make our tasks more manageable. In our FB example, if everything were in a single function being executed sequentially and if an error occurred during fetching the second url, then other 3 urls wouldn't be hit. If all 5 urls were being executed in a different process, then getting an error in one process, wouldn't affect others. So tasks become more manageable if we use celery properly.

### Simple celery example

Suppose we have a function which gets a list of urls and it has to get response from all the urls.

#### Without celery

	import requests
	import time

	def func(urls):
		start = time.time()
		for url in urls:
			resp = requests.get(url)
			print resp.status_code
		print "It took", time.time() - start, "seconds"

	if __name__ == "__main__":
		func(["http://google.com", "https://amazon.in", "https://facebook.com", "https://twitter.com", "https://alexa.com"])

#### Run this program

	python celery_blog.py

Output is

	(hack)~/Play/Python/hack $ python celery_blog.py
	200
	200
	200
	200
	200
	It took 7.58989787102 seconds

#### With celery

The main component of a celery enabled program or a celery setup is the **celery worker**.

In our web app signup example, `celery worker` would do the job of sending the emails.

In our FB example, `celery worker` would do the job of fetching the different urls.

Similary in our `celery_blog.py` example, `celery worker` would do the job of fetching the urls.

`Celery worker` and your application/script are different processes and run independent of each other. So your application/script and celery need some way to communicate with each other. That's where a message queue comes into picture.

Application code needs to put the task somewhere from where celery worker can fetch it and execute. Application code puts the task on a message queue. Celery worker fetches the task from message queue and exectues the task. We will use redis as the message queue.

Make sure you have redis installed and you are able to run `redis-server`

Make sure you have celery installed.

Change your file celery_blog.py, so it looks like:

	from celery import Celery

	app = Celery('celery_blog', broker='redis://localhost:6379/0')

	@app.task
	def fetch_url(url):
		resp = requests.get(url)
		print resp.status_code

	def func(urls):
		for url in urls:
			fetch_url.delay(url)

	if __name__ == "__main__":
		func(["http://google.com", "https://amazon.in", "https://facebook.com", "https://twitter.com", "https://alexa.com"])

##### Explanation of code

We need a celery instace for proper celery setup. We created a celery instance called **app**. 

Quoting celery docs from <a href="http://celery.readthedocs.org/en/latest/getting-started/first-steps-with-celery.html#application" target="_blank">here</a>.

	The first argument to Celery is the name of the current module, this is needed so that names can be automatically generated, the second argument is the broker keyword argument which specifies the URL of the message broker you want to use

Message queue and message broker are synonymous term for our basic discussion.

A celery worker can run multiple processes parallely. We want to hit all our urls parallely and not sequentially. So we need a function which can act on one url and we will run 5 of these functions parallely. So we wrote a **celery task** called **fetch_url** and this task can work with a single url. A celery task is just a function with decorator "app.task" applied to it.

From our old function, we called the task 5 times, each time passing a different url.

When we say "fetch_url.delay(url)", the code is serialized and put in the message queue, which in our case is redis. Celery worker when running will read the serialized thing from queue, then deserialize it and then execute it.

##### Start three terminals

* On first terminal, run redis using **redis-server**.
* On second terminal, run celery worker using **celery worker -A celery_blog -l info -c 5**. By seeing the output, you will be able to tell that celery is running.
* On third terminal, run your script, **python celery_blog.py**.

Unlike last execution of your script, you will not see any output on "python celery_blog.py" terminal. It is because the actual work of hitting the url isn't being done by your script anymore, it will be done by celery.

Switch to the terminal where "celery worker" is running. You would see output lines like 

	[2015-07-05 12:57:44,705: INFO/Worker-2] Starting new HTTPS connection (1): facebook.com
	[2015-07-05 12:57:44,711: INFO/Worker-4] Starting new HTTPS connection (1): twitter.com
	[2015-07-05 12:57:44,716: INFO/Worker-3] Starting new HTTPS connection (1): alexa.com
	[2015-07-05 12:57:44,791: INFO/Worker-1] Starting new HTTP connection (1): www.google.co.in

	[2015-07-05 12:57:45,063: WARNING/Worker-1] 200

	[2015-07-05 12:57:45,376: INFO/Worker-5] Starting new HTTPS connection (1): www.amazon.in

	[2015-07-05 12:57:46,179: WARNING/Worker-2] 200

	[2015-07-05 12:57:46,185: INFO/MainProcess] Task celery_blog.fetch_url[2809a803-00b2-44c7-85e5-3f6f71d3f5e3] succeeded in 1.48678409203s: None
	[2015-07-05 12:57:46,218: INFO/MainProcess] Task celery_blog.fetch_url[9d011563-67f9-4961-a61f-19956bf0cf0a] succeeded in 1.50805259595s: None
	....
	.....

Your output might not match this.

First thing to notice is the entire output of celery would have been printed in much less than 8 seconds. Earlier it took around 8 seconds to fetch 5 urls. With celery, it would have taken around 3 seconds or even lesser.

##### Understanding celery worker -A celery_blog -l info -c 5

* "-c 5" means that we set the concurrency as 5. So celery can run 5 parallel sub-processes. Each sub-process can act on a single task.
* "-l info" means we want celery to be verbose with its output.
* "-A celery_blog" tells that celery configuration, which includes the **app** and the tasks celery worker should be aware of, is kept in module celery_blog.py

##### Understanding the output
* Celery worker is running 5 sub-processes simulataneously which it calls Worker-1, Worker-2 and so on.
* It's not necessary that tasks' will be fetched in exactly the same order as they were in list.
* When we ran **python celery_blog.py**, tasks were created and put in the message queue i.e redis.
* **celery worker** running on another terminal, talked with redis and fetched the tasks from queue.
* celery worker deserialized each individual task and made each individual task run within a sub-process.
* celery worker did not wait for first task/sub-process to finish before acting on second task.
* While first task is still being executed in a sub-process, celery worker fetched second task, deserialized it and gave it to another sub-process.
* That's why our output is mixed up, i.e four tasks have started. But before 5th task could start, we got the result from 1st task, i.e the "200" you are seeing.

### Keeping celery code and configuration in different files.

In last example, we only wrote one celery task. Your project might span multiple modules and you might want to have different tasks in different modules. So let's move our celery configuration to a separate file.

Create a file celery_config.py

	from celery import Celery

	app = Celery('celery_config', broker='redis://localhost:6379/0', include=['celery_blog'])

Modify celery_blog.py so it looks like

	import requests
	from celery_config import app

	@app.task
	def fetch_url(url):
		resp = requests.get(url)
		print resp.status_code

	def func(urls):
		for url in urls:
			fetch_url.delay(url)

	if __name__ == "__main__":
		func(["http://google.com", "https://amazon.in", "https://facebook.com", "https://twitter.com", "https://alexa.com"])

Stop old celery worker, and run "celery worker -A celery_config -l info -c 5"

Start ipython and issue "func"

	from celery_blog import func

	func(['https://google.com', 'https://facebook.com'])

##### Output

	[2015-07-05 14:52:02,522: INFO/Worker-1] Starting new HTTPS connection (1): google.com
	[2015-07-05 14:52:02,522: INFO/Worker-5] Starting new HTTPS connection (1): facebook.com
	[2015-07-05 14:52:03,168: INFO/Worker-1] Starting new HTTPS connection (1): www.google.co.in
	[2015-07-05 14:52:03,959: INFO/Worker-5] Starting new HTTPS connection (1): www.facebook.com
	[2015-07-05 14:52:03,966: WARNING/Worker-1] 200
	[2015-07-05 14:52:03,972: INFO/MainProcess] Task celery_blog.fetch_url[7dbf6870-987b-460e-b5f1-ca17af88bc0a] succeeded in 1.45625397097s: None
	[2015-07-05 14:52:04,915: WARNING/Worker-5] 200
	[2015-07-05 14:52:04,922: INFO/MainProcess] Task celery_blog.fetch_url[d836a878-823f-4ca2-b918-a6ab0622a157] succeeded in 2.40576425701s: None

##### Adding another task in a different file

You can add another module and define a task in that module.

Create a module celery_add.py with following content.

	from celery_config import app

	@app.task
	def add(a, b):
		return a + b

Change celery_config.py to include the new module celery_add.py too. So celery_config.py becomes.

	from celery import Celery

	app = Celery('celery_config', broker='redis://localhost:6379/0', include=['celery_blog', 'celery_add'])

Use the new task add

	from celery_add import add

	add.delay(4, 5)

##### Output

	[2015-07-05 15:06:28,533: INFO/MainProcess] Received task: celery_add.add[0e8752a6-1d2f-4f8f-b003-656311beadd9]
	[2015-07-05 15:06:28,537: INFO/MainProcess] Task celery_add.add[0e8752a6-1d2f-4f8f-b003-656311beadd9] succeeded in 0.00138387701008s: 9

### Using celery with a package.

We will keep working with celery_config.py. Consider the folder containing celery_config.py is the root directory of your project.

Create a package called `pack` at the same level as celery_config.py. Since you are creating a package make sure there is a pack/__init__.py file.

Create a file pack/celery_fetch.py with following content.

	import requests
	from celery_config import app

	@app.task
	def fetch_url(url):
		resp = requests.get(url)
		print resp.status_code

	def func(urls):
		for url in urls:
			fetch_url.delay(url)

Change celery_config.py so it looks like

	from celery import Celery
	app = Celery('celery_config', broker='redis://localhost:6379/0', include=['pack.celery_fetch'])

Start celery worker from same level as celery_config.py

	celery worker -A celery_config -l info -c 5

Make sure you see the following in output.

	[tasks]
	  . pack.celery_fetch.fetch_url

Now use `func` from ipython.

	from pack.celery_fetch import func

	func(['https://google.com', 'https://facebook.com'])

### Redis and celery on separate machines

Till now our script, celery worker and redis were running on the same machine. But there is no such necessity. Three of them can be on separate machines.

Celery tasks need to make network calls. So having celery worker on a network optimized machine would make the tasks run faster. Redis is an in-memory database, so very often you'll want redis running on a memory-optimized machine.

In this example let's run redis on a separate machine and keep running script and celery worker on local system.

I have a server at 54.69.176.94 where I have redis running.

So change "broker" in the celery_config.py so it becomes.

	app = Celery('celery_config', broker='redis://54.69.176.94:6379/0', include=['celery_blog'])

Now if I run any task, our script will serialize it and put it on redis running at 54.69.176.94.

Celery worker will also communicate with 54.69.176.94, get the task from redis on this server and execute it.

Note: You will have to use your own server address where redis-server is running. I have stopped redis on my server and so you will not be able to connect to redis.

### Celery and script/web-application on separate machines.

As I told earlier, celery worker and your program are separate processes and are independent of each other. We can run them on different machines.

Suppose you have a server at 54.69.176.94 where you want to run celery but you want to keep running your script on local machine.

So you can copy all the files, in our case celery_config.py and celery_blog.py to the server. And run `celery worker -A celery_config -l info` on the server.

Call any task on the local machine, it will be enqueued wherever the `broker` points. Celery worker on 54.69.176.94 is also connected with same broker, so it will fetch the task from this broker and can execute it.

### Gotchas

In the simplest celery example, i.e where we have configuration and task fetch_url in the same file.

Change app name from `celery_blog` to `celery_blo`.

Run the worker, `celery -A celery_blog worker -l info`

The output tells that task is registered as `celery_blog.fetch_url`

Now try putting a task in queue.

	python celery_blog.py

A KeyError is raised.

Some lines of error:

	[2015-07-05 16:59:22,956: ERROR/MainProcess] Received unregistered task of type 'celery_blo.fetch_url'.
	KeyError: 'celery_blo.fetch_url'

So when putting the task on queue, celery uses the app name i.e `celery_blo`. But worker i.e `celery worker -A celery_blog` registers the task using the module name i.e `celery_blog` and not using the app name i.e `celery_bio`.


