---
layout: post
title:  "How Python generators are similar to iterators"
date:   2017-11-01 11:29:47+05:30
tags: python
author: akshar
---
This post assumes you have a solid understanding of iterables, iterators, \_\_iter\_\_(), next() and built in iter(). If not, read our <a href="http://agiliq.com/blog/2017/10/iterators-and-iterables/" target="_blank">previous post</a>

### Generators

Generators simplify creation of iterables. Anything that can be done with a generator can be done by implementing iterator protocol. Generators require lesser lines of code than solving the problem with iterators and iterables.

Generators are **functions** having an **yield** keyword. Any function which has "yield" in it is a generator.

Calling a generator function creates an iterable. Since it is an iterable so it can be used with iter() and with a for loop.

Let's write a generator function

    def func():
        yield 1
        yield 2

We can use this generator with a for loop:

	In [2]: for each in func():
	   ...:     print each
	   ...:
	1
	2

In this case the numbers 1 and 2 were never stored in memory simultaneously. During first iteration of loop only 1 was stored in memory and during second iteration 2 was stored in memory.

Same thing using iterators and iterables would need the following code:

	In [15]: class Iterable(object):
    ...:     def __init__(self):
    ...:         self.current = 1
    ...:     def __iter__(self):
    ...:         return self
    ...:     def next(self):
    ...:         while self.current < 3:
    ...:             temp = self.current
    ...:             self.current += 1
    ...:             return temp
    ...:         raise StopIteration()
    ...:

	In [16]: for each in Iterable():
		...:     print each
		...:
	1
	2

#### How generators are similar to iterables

* A generator function, i.e func can be considered similar to an iterable class.
* Calling a generator function creates an iterable instance. This instance has an \_\_iter\_\_() method. So this instance can be used with built-in function iter().
* Since calling a generator function creates an iterable, so it can be used with a for-loop.

Let's try it out step by step with the generator func:

	In [10]: f_iterable = func()

Let's verify that it is an iterable. Check for iterables is they have an \_\_iter\_\_() on them.

	In [11]: f_iterable.__iter__
	Out[11]: <method-wrapper '__iter__' of generator object at 0x103fe5f50>

Since it is an iterable, let's get the underlying iterator from it.

	In [12]: f_iterator = iter(f_iterable)

Let's get the values from this iterator using next().

	In [13]: next(f_iterator)
	Out[13]: 1

	In [14]: next(f_iterator)
	Out[14]: 2

	In [15]: next(f_iterator)
	---------------------------------------------------------------------------
	StopIteration                             Traceback (most recent call last)
	<ipython-input-15-c130bb6b8eb9> in <module>()
	----> 1 next(f_iterator)

	StopIteration:

Generator takes care of creating the iterable. It also takes care of creating the underlying iterator. And next() of this iterator() is such that it returns each 'yield' value of generator one after the other. When there is no more 'yield' in the generator function then this iterator raises StopIteration.

And we already know how iterables work with for-loop. To recap:

* Saying `for each in iterable` causes iter(iterable) to be called. This returns the underlying iterator.
* Then next() of iterator is repeatedly called until next() of iterator raises a StopIteration.

That's why when we use the generator instance with for-loop the 'yield' values are returned one by one until there is nothing more to be 'yielded' and at that point **StopIteration** is raised and so for-loop stops.

In last post we saw how to generate fibonacci number upto a maximum using iterators and iterables. Putting it here for reference:

    In [16]: class Fibonacci(object):
        ...:     def __init__(self, max):
        ...:         self.max = max
        ...:         self.loop = 1
        ...:         self.first = 0
        ...:         self.second = 1
        ...:     def __iter__(self):
        ...:         return self
        ...:     def next(self):
        ...:         if self.first + self.second > self.max:
        ...:             raise StopIteration()
        ...:         if self.loop == 1:
        ...:             temp = 0
        ...:         elif self.loop == 2:
        ...:             temp = 1
        ...:         else:
        ...:             temp = self.first + self.second
        ...:             self.first, self.second = self.second, temp
        ...:         self.loop += 1
        ...:         return temp

    In [17]: for each in Fibonacci(20):
        ...:     print each
        ...:
    0
    1
    1
    2
    3
    5
    8
    13

The same fibonacci generator can be written using generators in the following way:

	In [43]: def fibonacci(maximum):
		...:     first, second = 0, 1
		...:     yield first
		...:     yield second
		...:     while (first + second) < maximum:
		...:         yield first + second
		...:         first, second = second, first+second
		...:

	In [44]: for each in fibonacci(20):
		...:     print each
		...:
	0
	1
	1
	2
	3
	5
	8
	13

We saved several lines of code by using generators instead of iterators and iterables.

#### Implementing xrange with generator.

In the <a href="http://agiliq.com/blog/2017/10/real-world-usage-iterators-and-iterables/" target="_blank">last post</a> we implemented Python built-in xrange() using iterators and iterables.

We can implement xrange with generators in following way:

	In [76]: def xrange(stop):
		...:     current = 0
		...:     while current < stop:
		...:         yield current
		...:         current += 1
		...:

You should read <a href="https://stackoverflow.com/questions/2776829/difference-between-pythons-generators-and-iterators#answer-2776865" target="_blank">this SO answer</a> for difference between generators and iterators.


