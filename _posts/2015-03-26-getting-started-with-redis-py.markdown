---
layout: post
title:  "Getting started with redis-py"
date:   2015-03-26 17:43:25+05:30
tags: redis
author: akshar
---
This post explains how to interface with Redis from Python and how to use different Redis commands from Python using redis-py.

What this post is not about

* Comparing relational and non-relational database
* Comparing MySQL and Redis
* Introduction to Redis


I assume you know the following

* Basics of Redis
* Basic redis commands for working with Strings, Lists and Maps as described at <a href="http://redis.io/topics/data-types-intro" target="_blank">Redis docs</a>.

I did not find enough examples of using redis-py provided methods and so wrote this blog post.

### Getting started

Assuming you have Redis installed and your redis server is running.

We will follow the same sequence as followed in Redis data-types-intro at <a href="http://redis.io/topics/data-types-intro" target="_blank">Redis docs</a>

Install redis-py

	pip install redis

Start redis-cli on some shell, so we can relate our Python redis commands and plain redis command.

#### Redis strings

For setting a string value, we use the following from redis-cli

	127.0.0.1:6379> set mykey myvalue

And you can get it using

	127.0.0.1:6379> get mykey
	"myvalue"

Now we want to do the same from Python. Do the following from Python shell.

	>>> import redis
	>>> r = redis.StrictRedis()

We created an instance of **StrictRedis**. This is required for communication with redis-server.

Try getting "mykey", which was set using redis-cli, from Python

	>>> r.get("mykey")
	'myvalue'   #output

So we are able to communicate properly with redis-server. Things which were inserted into redis using redis-cli can be read using Python.

Try setting a value into Redis from Python shell.

	>>> r.set("anotherkey", "anothervalue")
	True       #output

Check from redis-cli if this key was set properly.

	127.0.0.1:6379> get anotherkey
	"anothervalue"

So it seems Python is able to insert values into Redis properly.

You can read the same value using Python again.

	>>> r.get("anotherkey")
	'anothervalue'

#### INCR and INCRBY

Redis provides **incr** and **incrby** on integer values.

So let's set an integer value first which we will try to increment.

	>>> r.set("num", 10)
	True

	>>> r.get("num")
	'10'

redis-py's equivalent of Redis' **incr** is

	>>> r.incr("num")
	11

Get it to verify that num has been incremented

	>>> r.get("num")
	'11'

You should verify it using redis-cli too.

	127.0.0.1:6379> get num
	"11"

Python's equivalent of **incrby** is

	>>> r.incrby("num", 10)
	21

Verify again using redis-cli

	127.0.0.1:6379> get num
	"21"

#### MSET and MGET

redis-cli's mset equivalent in Python

	>>> r.mset(first_num=1, second_num=2, third_num=3)
	True

Verify these keys are set from redis-cli

	127.0.0.1:6379> get first_num
	"1"
	127.0.0.1:6379> get second_num
	"2"

reidis' mget equivalent

	>>> r.mget(["first_num", "second_num"])
	['1', '2']

#### EXISTS

	>>> r.exists("first_num")
	True
	>>> r.exists("fourth_num")
	False

#### DEL

	>>> r.delete("first_num")
	1

As this key is deleted now, you will get "nil" and "None" if you try to retrieve it.

From redis-cli

	127.0.0.1:6379> get first_num
	(nil)

From Python shell

	>>> r.get("first_num")

#### EXPIRE

Mark key **second_num** to expire after 10 seconds

	>>> r.expire("second_num", 10)
	True

Check this immediately

	>>> r.get("second_num")
	'2'     #output

Check after 10 seconds

	>>> r.get("second_num")
    It returns None after 10 seconds meaning this key doesn't exist in redis anymore

#### Redis Lists

reidis' lpush equivalent in Python

	>>> r.lpush("mylist", 1)
	1L

Verify from redis-cli that a list is created in redis

	127.0.0.1:6379> lrange mylist 0 -1
	1) "1"

Pushing multiple values to list

	>>> r.lpush("mylist", 2, 3)

Check the new list from redis-cli

	127.0.0.1:6379> lrange mylist 0 -1
	1) "3"
	2) "2"
	3) "1"

redis-cli's lrange equivalent in Python

	>>> r.lrange("mylist", 0, -1)
	['3', '2', '1']

	>>> r.lrange("mylist", 0, 1)
	['3', '2']

rpush

	>>> r.rpush("mylist", 4)

Check that element is pushed on right

	>>> r.lrange("mylist", 0, -1)
	['3', '2', '1', '4']

#### Redis hashes

hmset allows saving of dictionary as values. You should already be knowing this from redis docs.

	127.0.0.1:6379> hmset user name ned age 31
	OK

	127.0.0.1:6379> hgetall user
	1) "name"
	2) "ned"
	3) "age"
	4) "31"

redis-py way of achieving the same would be

	>>> r.hmset("another_user", {'name': 'robert', 'age': 32})
	True

Try reading this using redis-cli

	127.0.0.1:6379> hgetall another_user
	1) "age"
	2) "32"
	3) "name"
	4) "robert"

Try reading this using redis-py

	>>> r.hgetall("another_user")
	{'age': '32', 'name': 'robert'}

	>>> r.hget("another_user", "name")
	'robert'

	>>> r.hget("another_user", "age")
	'32'

	>>> r.hgetall("user")
	{'age': '31', 'name': 'ned'}

There are several other methods provided by redis-py too and now you should be in a good position to related redis' commands with redis-py methods.


