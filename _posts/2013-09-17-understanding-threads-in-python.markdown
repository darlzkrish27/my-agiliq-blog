---
layout: post
title:  "Understanding Threads in Python"
date:   2013-09-17 23:53:01+05:30
tags: threads
author: akshar
---
We will see some examples of using threads in Python and how to avoid race conditions:

You should run each example several times to notice that threads are unpredictable and that your results differ every time.

Disclaimer: Forget anything you heard about GIL for now, because GIL is not going to mess up with scenarios I want to show.


###Example 1:

We want to fetch five different urls:

####Single threaded way:

    def get_responses():
        urls = ['http://www.google.com', 'http://www.amazon.com', 'http://www.ebay.com', 'http://www.alibaba.com', 'http://www.reddit.com']
        start = time.time()
        for url in urls:
            print url
            resp = urllib2.urlopen(url)
            print resp.getcode()
        print "Elapsed time: %s" % (time.time()-start)

    get_response()

Output is:

    http://www.google.com 200
    http://www.amazon.com 200
    http://www.ebay.com 200
    http://www.alibaba.com 200
    http://www.reddit.com 200
    Elapsed time: 3.0814409256

#####Explanation:

* All the urls are fetched in sequence.
* Unless the processor got response from a url, it didn't fetch the next url.
* Network operations are time taking, so processor was idle during the time it was expecting the response from a url.

Even in a single threaded program, there is one thread of execution. Let's call it **main thread**. So, last example had only one thread, i.e main thread.

####Multi threaded way:

You need to create a class which subclasses Thread:

    from threading import Thread

    class GetUrlThread(Thread):
        def __init__(self, url):
            self.url = url 
            super(GetUrlThread, self).__init__()

        def run(self):
            resp = urllib2.urlopen(self.url)
            print self.url, resp.getcode()

    def get_responses():
        urls = ['http://www.google.com', 'http://www.amazon.com', 'http://www.ebay.com', 'http://www.alibaba.com', 'http://www.reddit.com']
        start = time.time()
        threads = []
        for url in urls:
            t = GetUrlThread(url)
            threads.append(t)
            t.start()
        for t in threads:
            t.join()
        print "Elapsed time: %s" % (time.time()-start)

    get_responses()


Output is:

    http://www.reddit.com 200
    http://www.google.com 200
    http://www.amazon.com 200
    http://www.alibaba.com 200
    http://www.ebay.com 200
    Elapsed time: 0.689890861511

#####Explanation:

* Appreciate the improvement in running time of this program.
* We wrote a multi threaded program to decrease processor's idle time. While waiting for response of a particular thread's url, processor can work on some other thread and fetch the other thread's url.
* We wanted one thread to act on one url, so overridden the constructor of thread class to pass it a url.
* Execution of a thread means execution of a thread's **run()**.
* So, whatever we want the thread to do must go in its **run()**.
* Created one thread for each url and called **start()** on it. This tells the processor that it can execute the particular thread i.e the **run()** of thread.
* We don't want the elapsed time to be evaluated until all the threads have executed, **join()** comes in picture here.
* Calling **join()** on a thread tells the **main thread** to wait for this particular thread to finish before the main thread can execute the next instruction.
* We call join() on all the threads, so elapsed time will be printed only after all the threads have run.


#####Few things about threads
* Processor might not execute run() of a thread immediately after **start()**.
* You can't say in which order run() of different threads will be called.
* For a specific thread, it's guaranteed that the statements inside run() will be executed sequentially.
* It means that first the the url associated with the thread will be fetched and only then the recieved response will be printed.


###Example 2:

We will demonstrate race condition with a program and then fix it:

Read [wikipedia example](http://en.wikipedia.org/wiki/Race_condition#Example) to understand what race condition means.

    #define a global variable
    some_var = 0

    class IncrementThread(Thread):
        def run(self):
            #we want to read a global variable
            #and then increment it
            global some_var
            read_value = some_var
            print "some_var in %s is %d" % (self.name, read_value)
            some_var = read_value + 1 
            print "some_var in %s after increment is %d" % (self.name, some_var)

    def use_increment_thread():
        threads = []
        for i in range(50):
            t = IncrementThread()
            threads.append(t)
            t.start()
        for t in threads:
            t.join()
        print "After 50 modifications, some_var should have become 50"
        print "After 50 modifications, some_var is %d" % (some_var,)

    use_increment_thread()

Run this and you will find different result every time.


#####Explanation:

* There is a global variable and all the threads will modify it.
* All threads should add 1 to the existing value of the variable.
* There are 50 threads, so at end the value of some_var should become 50, but it doesn't.

#####Why some_var didn't reach 50?
* At some point thread **t1** read the value of some_var as 15 and then processor took the control from this thread and gave it to thread **t2**.
* t2 also reads some_var as 15.
* Both t1 and t2 reset the value of some_var to **15+1** i.e 16.
* But when two threads act on some_var we expected it's value to be increased by 2.
* So, we have a race condition here.
* A similar race condition might have occurred few more times and so value of **some_var** at end remains something like 41 or 42 or anything less than 50.

####Fix this race condition

Change the **run()** of IncrementThread to:

    from threading import Lock
    lock = Lock()

    class IncrementThread(Thread):
        def run(self):
            #we want to read a global variable
            #and then increment it
            global some_var
            lock.acquire()
            read_value = some_var
            print "some_var in %s is %d" % (self.name, read_value)
            some_var = read_value + 1
            print "some_var in %s after increment is %d" % (self.name, some_var)
            lock.release()

You should run use_increment_thread again and the result will match your expectation.

#####Explanation:

* Lock is used to guard against race condition.
* If thread t1 has acquired the lock before performing a set of operations, no other thread can perform the same set of operation until t1 releases the lock.
* We want to make sure that once t1 has read some_var, no other thread can read some_var until t1 is done with modifying the value of some_var.
* So reading some_var and modifying it are logically related operations here.
* And that is why we keep read and modify part of some_var guarded by a Lock instance.
* Lock is a separate object and it will be acquired by the thread from whose context it is called.

###Example 3

In last example we saw how a global variable gets affected in multithreading. Let's see an example to verify that one thread cannot affect the instance variable of some other thread.


Let's introduce **time.sleep()** in this example. It will make sure that a thread goes in suspended state and hence enforces thread switching to occur.

    import time

    class CreateListThread(Thread):
        def run(self):
            self.entries = []
            for i in range(10):
                time.sleep(1)
                self.entries.append(i)
            print self.entries

    def use_create_list_thread():
        for i in range(3):
            t = CreateListThread()
            t.start()

    use_create_list_thread()

Run it few times and you will notice that the list do not get printed properly.

Possibly the entries of one thread was getting printed and during this operation, processor switched to some other thread and started printing the entries for other thread. We want to ensure that entries get printed one after another for separate threads.

Change run() of CreateListThread to use lock.

    class CreateListThread(Thread):
        def run(self):
            self.entries = []
            for i in range(10):
                time.sleep(1)
                self.entries.append(i)
            lock.acquire()
            print self.entries
            lock.release()

So, we put the print operation inside a lock. When one thread has acquired the lock and printing its **entries**, no other thread can print its entries. And so you will see entries of different threads printed on separate lines.

This will show that all threads' **entries**, which is an instance variable, is a list with numbers from 0 to 9. So, thread switching doesn't affect the instance variable of a thread.

