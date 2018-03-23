---
layout: post
title:  "How performant is your Python web application"
date:   2017-11-15 12:26:01+05:30
tags: gunicorn
author: akshar
---
This post tries to explain web application performance. **Performance** means the **number of requests per second** that can be served by a deployed application. 

This post would help answer questions like:

* How `performant` is an application.
* How much `load` can it handle.
* How many `concurrent requests` can it serve.
* How can you determine `requests per second` for an application.
* What steps to take to increase `serving capability` for an application.

This post has as much code as theory.

This post assumes that you have a basic understanding of processes, threads. You can read our <a href="http://agiliq.com/blog/2013/09/process-and-threads-for-beginners/" target="_blank">previous post</a> for basic understanding of processes and threads.

In any web application there are many urls and associated handlers/controllers for each url. A url might respond in 200ms while another url might take 3 seconds. While determining performance of an application, choose the url which will be used most often and determine its performance.

### Factors which determine performance

There are many `factors` determining performance of an application. Major factors are:

* Application type
* Application complexity
* Web server
* Physical server, i.e infrastructure
* Web server configuration

In this post we will see how changing `web server configuration` changes performance with other factors remaining constant.

Two major components of `web server configuration` are:

1. Number of processes/workers
2. Number of threads

We will also see what web server configuration should be preferred for which application type. Example: How increasing number of server processes boosts performance for some applications while it reduces performance for other application types.

#### Application type

Applications could be compute intensive or network intensive or I/O intensive. Compute intensive applications can't get any benefit by making number of server workers or threads greater than number of CPU cores.

Network intensive or I/O intensive applications can benefit a lot by making several server workers/threads run on each core.

#### Application complexity

Suppose a url handler makes two db calls and takes a second to respond. In such case reducing number of db calls to 1 will reduce response time by half and number of requests served per second will get doubled.

#### Web server

There are many web servers. eg: Apache, Gunicorn, uwsgi etc. Apache might be better than gunicorn and might be able to handle more requests per second than gunicorn.

#### Physical server/Infrastructure

Increasing the number of cores or memory will improve the performance. If a single core machine is able to handle 10 requests per second for a computationally intensive application, then a machine with 2 cores should be able to handle 20 requests per second.

This might need properly configuring the web server to get maximum utilization from physical server.

#### Application server configuration

Number of running web server processes influences performance. Similarly number of server threads in each process influences performance too.

### Demo server configuration

Our demo physical server is a t1.micro instance with 1 GB RAM. It has a single core.

Demo application for this post uses Django/Python served from a Gunicorn application server.

Familiarity with Django would be helpful but you should be able to follow as long as you have understanding of any web framework.

### Demo application

Our application has the following url:

	http://34.233.117.92:8000/static_content_sleep

The Django handler/controller for url `static_content_sleep` looks like:

	def static_content_sleep(request):
		st = time.time()  # Compute Start time
		print "sleeping"
		time.sleep(1)     # Simulate db call which takes 1 seconds
		print "waking"
		print time.time() - st, "in this function"  # Compute end time
		return HttpResponse("Woke up")

Most web applications would be working with a database to fetch data.

We don't have a database setup. We are assuming that db used by the handler responds in 1 second. We are simulating a db call by making the code execution sleep for 1 second.

This application is being served by Gunicorn web server. Gunicorn configuration looks like:

	workers = 1
	threads = 1
	bind = '0.0.0.0:8000'
	daemon = False

The important configuration variables are `workers` and `threads`. Ignore others.

If you want to setup gunicorn on your physical server, you can refer our <a href="http://agiliq.com/blog/2013/08/minimal-nginx-and-gunicorn-configuration-for-djang/" target="_blank">previous post</a>.

We used default gunicorn configuration for `workers` and `threads`. The default configuration of gunicorn has following characteristics:

#### workers

Default is 1. This means that only one gunicorn process would be running on the physical server.

If we make it 2, it would mean that 2 gunicorn processes would be running on the server.

#### threads

Default is 1. This tells number of threads in each worker process. This means that each gunicorn worker is single threaded and isn't multithreaded.

### Making requests

Let's make two simultaneous request to this url.

Making two requests from browser will not be simultaneous as switching from one browser tab to another tab might take you more than a second. In that case you cannot properly observe time taken by the server to process two simultaneous requests.

Let's write a Python function to make simultaneous requests and log the time.

	In [15]: from threading import Thread
	In [16]: import requests

	In [17]: class UrlThread(Thread):
		...:     def run(self):
		...:         resp = requests.get('http://34.233.117.92:8000/static_content_sleep')

	In [18]: def make_n_requests(num_requests):
		...:     threads = []
		...:     for i in range(num_requests):
		...:         threads.append(UrlThread())
		...:     start = time.time()
				 # The requests will be almost simultaneous.
				 # Second request will be made within nanoseconds of making the first request.
		...:     for thread in threads:
		...:         thread.start() # Threads will be started without waiting for response of previous threads
		...:     for thread in threads:
		...:         thread.join()  # Wait for response for all the requests.
		...:     end = time.time()
		...:     print "Time to get response for %d simultaneous requests" % (num_requests,), end - start

You can use curl or a shell script or any programming language using which you can simulate **n** simultaneous requests.

Make 2 simultaneous requests

	In [19]: make_n_requests(2)
	Time to get response for 2 simultaneous requests 2.622205019

Gunicorn log on server looks like:

	sleeping
	waking
	1.00131487846 in this function
	sleeping
	waking
	1.00130581856 in this function

#### Observation

* Print statements of the handler are printed. Print statements are printed again.
* Based on prints, we can infer that second request's execution started after completion of first request.

Let's make 5 simultaneous requests and see what happens

	In [20]: make_n_requests(5)
	Time to get response for 5 simultaneous requests 5.65813708305

Gunicorn log looks like:

	sleeping
	waking
	1.00114989281 in this function
	sleeping
	waking
	1.0011370182 in this function
	sleeping
	waking
	1.00114202499 in this function
	sleeping
	waking
	1.00130295753 in this function
	sleeping
	waking
	1.00131988525 in this function

You can see 5 sets of print statements.

#### Observation

* As we increase number of simultaneous requests, response time is proportionately increasing.
* For 2 requests, server takes around 2 seconds to respond.
* For 5 requests, server takes around 5 seconds to respond.
* For 10 requests, it `would take` around 10 seconds to respond.

Let's change gunicorn `threads`from its default value of 1 to 5.

	threads = 5

Gunicorn process will have 5 threads now. And each thread is capable of serving a request. Hence 5 requests can be simultaneosly served.

Restart gunicorn.

Let's make 5 simultaneous requests again.

	In [27]: make_n_requests(5)
	Time to get response for 5 simultaneous requests 1.77244091034

** While earlier it was taking around 5 seconds to get 5 simultaneous requests processed, now it takes less than 2 seconds. **

Server log looks like:

	sleeping
	 sleeping
	sleeping
	sleeping
	sleeping
	waking
	1.001278162 in this function
	 waking
	1.00262784958 in this function
	waking
	1.00353288651 in this function
	waking
	1.00325298309 in this function
	waking
	1.0035841465 in this function

#### Important Observation

* Each gunicorn thread could handle a request.
* Processor kept switching between all these 5 threads.
* This way all 5 requests being processed by 5 different threads got a chance to execute concurrently without waiting for completion of any request.
* Print pattern also suggests that execution of 5 requests started before waiting for completion of first request.

We can accomplish this performance boost by increasing number of workers instead of number of threads. We will change threads back to 1 and increase `workers` to 5 instead.

	workers = 5
	threads = 1

Make 5 simultaneous requests

	In [22]: make_n_requests(5)
	Time to get response for 5 simultaneous requests 1.76864600182

Server gunicorn log looks like:

	sleeping
	sleeping
	sleeping
	sleeping
	sleeping
	waking
	1.00134015083 in this function
	waking
	1.00125384331 in this function
	waking
	1.00277590752 in this function
	waking
	1.00121593475 in this function
	waking
	1.00092220306 in this function

Here, processor executed 5 processes parallely.

Let's make 10 simultaneous requests:

	In [23]: make_n_requests(10)
	Time to get response for 10 simultaneous requests 2.74328804016

Server log looks like:

	sleeping   -> First 5 requests are assigned to 5 workers
	sleeping
	sleeping
	sleeping
	sleeping
	waking     -> One worker, say worker 1, finished execution.
	waking     -> Another worker, say worker 2, finished execution. This print was executed before last print statement of worker 1 could execute.
	1.00396203995 in this function  -> Worker 1 last print statement. Worker 1 free now and can serve another request.
	1.00669813156 in this function  -> Worker 2 last print statement. Worker 2 free now
	sleeping   -> Assigned to probably worker 1
	waking     -> Probably worker 3 woke up.
	1.00510120392 in this function
	sleeping
	waking
	1.00605106354 in this function
	sleeping
	sleeping
	waking
	1.00563693047 in this function
	sleeping
	waking
	1.00345993042 in this function
	waking
	1.00495409966 in this function
	waking
	1.00393104553 in this function
	waking
	1.0027141571 in this function
	waking
	1.00151586533 in this function

#### Observation

* We made 10 simultaneous requests.
* First 5 requests were assigned to 5 running gunicorn processes.
* First 5 requests were concurrently handled during 1st second. 5 remaining requests were waiting to be executed during this time. During next second each gunicorn process picked up another request after completing a request. That's why it took around 2 seconds to get response for 10 requests.

Let's use 2 threads with 5 workers

	threads = 2
	workers = 5

Let's restart gunicorn. Now there are 5 gunicorn processes running and each process is running 2 threads.

	In [16]: make_n_requests(10)
	Time to get response for 10 simultaneous requests 1.46275486946

Server log looks like:

	sleeping -> 'sleeping' repeated 10 times before any 'waking'.
	sleeping
	sleeping
	sleeping
	sleeping
	sleeping
	sleeping
	sleeping
	sleeping
	sleeping
	waking
	1.00119280815 in this function
	waking
	1.00499987602 in this function
	waking
	1.00233006477 in this function
	waking
	1.00144195557 in this function
	waking
	1.00114202499 in this function
	waking
	1.00117397308 in this function
	waking
	1.00114512444 in this function
	waking
	1.00120997429 in this function
	waking
	1.00124192238 in this function
	waking
	1.00117397308 in this function

### Inference

Based on these examples we can infer that making number of threads/workers greater than number of cores improves the performance for a network intensive and I/O intensive application.

#### Can we keep increasing workers and threads

You have to keep RAM usage under consideration while tuning the number of workers and threads.

Code execution for a handler needs memory. While worker is processing a request, sufficient memory must be available. If handling each request needs 50 MB and you have 5 workers and 1 thread running, you must ensure that 250 MB free RAM is there after starting gunicorn.

#### Performance with current configuration

Curent configuration has 2 workers and 5 threads for each worker. So 10 requests will be handled concurrently.

Each request takes around 1 second to respond.

So with current configuration server can handle 10 requests per second.

If the handler is changed to `time.sleep(0.5)`, i.e if each request could respond in approximately 0.5 seconds then server would be able to handle 20 requests per second.

If we change number of workers to 3 and 5 threads for each worker, then with time.sleep(0.5), server would be able to handle 3\*5\*2, i,e 30 requests per second.

#### Increase performance by increasing number of cores

Suppose we use a machine with 2 cores.

On single core machine if we were using 2 workers, then on double core machine we should use 4 workers. This assumes that there is sufficient RAM available to be allocated to 4 workers.

A dual core machine would be able to handle 4\*5, i.e 20 requests per second, assuming each request responds in a second.

If we found that 3 workers with 5 thread each is an optimum combination on a single core machine, then we should use 6 workers on a dual core machine.


### Computationally bound applications

Earlier sections discussed about I/O bound applications. Let's talk about CPU bound applications.

App has a following url:

	http://34.233.117.92:8000/list_of_dict

The Django handler/controller for url `list_of_dict` looks like:

	def list_of_dict(request):
		print "entered function"
        st = time.time()
        for i in xrange(30000000):
                pass
        print time.time() - st, "in this function"
        return HttpResponse(json.dumps("a"))

Let's change `UrlThread` used by `make_n_requests` to work with this new url.

	In [14]: class UrlThread(Thread):
		...:     def run(self):
		...:         resp = requests.get('http://34.233.117.92:8000/list_of_dict')

Let's change gunicorn configuration to have a single worker and single thread.

	workers = 1
	threads = 1

Let's make 1 request to this url

	In [28]: make_n_requests(1)
	Time to get response for 1 simultaneous requests 1.49780297279

Server log looks like:

	entered function
	0.454895019531 in this function

It takes around 0.5 seconds in the handler as you can see from server log. And it takes around 1 second for request and response to travel on the wire. So in total it takes around 1.5 seconds.

Let's make 5 requests to this url

	In [32]: make_n_requests(5)
	Time to get response for 5 simultaneous requests 2.94885587692

Server log looks like:

	entered function
	0.460360050201 in this function
	entered function
	0.459032058716 in this function
	entered function
	0.462526082993 in this function
	entered function
	0.45965385437 in this function
	entered function
	0.460576057434 in this function

Each request takes around 0.5 seconds to complete. Plus there is an additional time for request and response to move over the wire. In total it takes around 3 seconds.

Time to get response for n requests is increasing linearly as we increase n.

With a single worker and single thread, time to get response for simultaneous requests was increasing linearly with number of requests in I/O bound handler too.

Let's change gunicorn `threads`from its default value of 1 to 5.

	threads = 5

Let's make 5 simultaneous requests again.

	In [33]: make_n_requests(5)
	Time to get response for 5 simultaneous requests 3.37198781967

We didn't get any advantage by increasing number of threads. Instead the performance deteriorated.

Server log looks like:

	entered function
	entered function
	entered function
	entered function
	entered function
	2.68351387978 in this function
	 2.67835998535 in this function
	2.67423701286 in this function
	 2.67234015465 in this function
	2.6794462204 in this function

Event different requests didn't complete in 0.5 second as was happening with single thread. The CPU time was split between 5 threads and so instead of 0.5 seconds it took 2.6 seconds for each request to complete.

#### Observation

* In CPU bound applications, CPU isn't idle. So multiple threads don't provide an advantage.
* CPU time is split between threads, which infact leads to longer response time for each request.
* Thread switching brings a performance hit. That's why we saw response time for 5 requests going up from 2.9 seconds to 3.3 seconds for 5 simultaneous requests.

### Takeaways

* There is no definite answer for how many workers and threads will provide maximum performance. It's a matter of tuning and finding out.
* Making number of workers/threads greater than number of cores almost always leads to higher number of requests per second handling in a I/O bound application.
* Making number of workers/threads greater than number of cores almost always leads to reduced number of requests per second handling in a CPU bound application.
* Making number of workers/threads greater than number of cores leads to increase in response time for each request in a CPU bound application.

