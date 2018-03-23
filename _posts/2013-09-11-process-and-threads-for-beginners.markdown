---
layout: post
title:  "Process and Threads for Beginners"
date:   2013-09-11 12:33:39+05:30
tags: process
author: akshar
---
We will cover this post in following sequence:

* Process and multiprocessing
* Threads and multithreading

###Process and multiprocessing

A process is an executable instance of a program. 

####Example:
You write a java program and save it as **abcd.java** and run it with **java abcd**. While this program is executing it is a process. File **abcd.java** is not a process, but when it's executing it is a process.

You write a Python program and save it as **abcd.py** and run it with **python abcd.py**. So, while this program is executing it is a process. Once the program terminates, the process ends.

You are reading this blog in a browser. The code for browser must have been written in some programming language. And that code is executing right now to enable you to work in the browser. So, a process is running. When you close the browser, the specific process for the browser ends.

Your program works with some data, and data is stored in some variables. Variables are stored in memory. So, when a process starts, OS assigns it some memory which the process can use.

The way to find all the executing process in a UNIX system is:

    ps aux

Open a file in `vim` on a terminal.

    vim threads.txt
    
Open another terminal and find vim processes.

    ps aux | grep vim

You will find that a vim process is running.

If you close vim on first terminal and try to find the vim processes again, you won't find it anymore.

Now start two instances of vim on two separate terminals:

    vim abcd.txt
    vim efgh.txt 

Check all the vim processes again using `ps aux | grep vim`, and this time you will find that two vim processes are running on the system.

So, this tells that a process is an **executable instance** of a program.

####Multitasking and Multiprocessing
A program is a sequence of instruction. A processor can only execute one instruction at a time.

Multiprocessing means that multiple processes can run simultaneously i.e at any instant of time multiple processes can run. This can never be possible in a system with a single processor, because processor can execute only one instruction at a particular instant of time. And that instruction can only be associated with a single process, so it's like a single process running at a particular instant.

Multitasking means that two or more processes **appear** to run simultaneously.

#####Example:

Suppose you have a system with a single processor. You want to download some files and simultaneously you want to write a program in your editor. So, download mechanism and your editor are two separate processes. But the processor can only run instruction of one process at a time. So, it keeps switching between two processes, and only one process is actually running at any particular instant of time. So, here it **appears** that two processes are running simultaneosly but they aren't. Hence multitasking is at play here, and not multiprocessing.

Now consider you have a system with multiple processor. So, download process can run on one processor and editor process on another processor. In this case, both the processes are actually running simultaneously and hence it is multiprocessing.

Few things about Processes:

* When a process starts, OS assigns it some memory.
* Memory is not shared between different processes.

###Threads and multithreading
Thread exist within a process. So, a process can have multiple threads.

A thread is an independent sequence of execution within a process. Let's consider an example to find out why we require threads.

#####Example:
We want a scraping program, this program will fetch five different urls and parse the response recieved from these different urls and print it. So, we will have a list/array and store the urls in that array. This list is stored in the memory associated with the scraping process.

Usual way of writing it:

* Loop through the array.
* Fetch first url.
* Wait for the response.
* Print the response after recieving it.
* Hit the next url in the list.
* Keep looping through the list until all the urls are hit.
* In this way, processor goes through the urls in sequence and it doesn't hit a url unless it gets a response for the previous url.

But hitting a url and recieving a response from it takes times. So, processor is idle while it is waiting for the response from a url and it is a waste of processor's time. And it is a waste of your time too looking at the terminal for all the urls to be processed sequentially. So, while the processor is waiting for the response, we can ask the processor to hit another url and in this way minimise the processor's idle time. Threads come into picture here and if threads come into picture our program becomes multithreaded.

Multithreaded version for the same program:

* We will create five different threads. Each thread will act on a single url.
* Processor works on one thread and hits the url associated with that thread.
* Processor becomes idle because getting the response will take some time.
* It starts working on another thread and hits the url associated with this thread. This happens before the processor recieves the response from the first url.
* So, our program is not sequential in this case and it is a multithreaded program.
* At some point response for the first url is recieved and processor switches back to first thread and prints it.
* Again it switches to some other thread.
* Processor idle time is greatly minimised in this multithreaded version.

The sequence described here might not match what processor actually does because it is highly unpredictable to tell which thread gets executed when.

#####Threads with single processor and multiple processor
Remember the section where we talked how a machine with single processor can never perform actual multiprocessing.

A machine with single processor might or might not provide any advantage even if the program is multithreaded.

A multithreaded program will perform faster than a single threaded program on a single processor machine in following scenarios:

* Program requires talking over the network i.e getting response from some url
* Program needs to wait for I/O

It happens because network and I/O operations take time during which processor can remain idle in a single-threaded program. In a multithreaded program, another threads can execute while one thread is waiting for some network operation or I/O. So a multithreaded program will be faster in such scenario.

If the operations are quick operations, like arithmetic operation or assigning some variable etc, then a multithreaded program will not perform any better than a single threaded program if the machine has a single processor. Reason being that the processor is not idle and hence no time is saved by switching between threads.

But if the machine has multiple processors, or multiple cores, a multithreaded program will always be faster that single-threaded program becuase different threads can execute concurrently on different processors.

