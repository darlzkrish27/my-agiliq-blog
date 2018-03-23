---
layout: post
title:  "Iterators and Iterables"
date:   2017-10-12 11:36:19+05:30
tags: python
author: akshar
---
#### Iterable

Iterables are objects that implement the iterator protocol.

Iterator protocol mandates that \_\_iter\_\_ method be implemented on the object.

	class A(object):
		def __iter__(self):
			return B()

An instance of A would be an iterable, because class A has \_\_iter\_\_() defined on it.

\_\_iter\_\_ method mandates that an `iterator` be returned from it. Instance of class B must be an iterator. More on iterators to follow. `iterator` and `iterables` are different things.

	a = A()

Here "a" is an `iterable`. It is not an `iterator`.

There is a built-in method called `iter()`. Only iterables can be passed to built in method `iter()`. If we try to pass a non-iterable to iter(), a TypeError will occur. More on built-in method iter() to follow.

Passing an iterable to built in `iter()` causes \_\_iter\_\_() of the iterable to be called.

#### Iterator

An iterator is an object that has next() method defined.

An iterator doesn't need to have \_\_iter\_\_() defined. Similarly an iterable doesn't need to have next() defined.

To reiterate, iterable must have \_\_iter\_\_() defined and iterator must have next() defined.

Iterator class B could look like:

	class B(object):
		def next(self):
			return "boom"

Class B is an iterator because it has method next().

You can do:

	a_instance = A()	
	a_iter = iter(a_instance)

`a_instance` is an iterable because it has method \_\_iter\_\_. Calling built-in iter() on the iterable `a_instance` internally called a_instance.\_\_iter\_\_(). a_instance.\_\_iter\_\_() returned an iterator which is an instance of class B.

#### Built-in next() and built-in iter()

Built in method `next()` mandates that an iterator be passed to it.

	In [33]: next(a_iter)
	Out[33]: 'boom'

	In [34]: next(a_iter)
	Out[34]: 'boom'

next() works with an iterator. next() doesn't work with iterable. Try it:

	In [60]: iterable = A()

	In [61]: next(iterable)
	---------------------------------------------------------------------------
	TypeError                                 Traceback (most recent call last)
	<ipython-input-61-edd1adac5cd0> in <module>()
	----> 1 next(iterable)

	TypeError: A object is not an iterator

	In [65]: iterator = B()

	In [66]: next(iterator)
	Out[66]: 'boom'

	In [67]: next(iterator)
	Out[67]: 'boom'

iter() works with iterable. iter() doesn't work with iterators.

	In [73]: iter(iterable)
	Out[73]: <__main__.B at 0x1058cef50>

	In [74]: iter(iterator)
	---------------------------------------------------------------------------
	TypeError                                 Traceback (most recent call last)
	<ipython-input-74-035e65827850> in <module>()
	----> 1 iter(iterator)

	TypeError: 'B' object is not iterable

An iterable needs an underlying iterator. In our examples, iterable A needs underlying iterator B.

At the same time iterators are independent of iterables. B isn't dependent on A.

Let's create a class which is not an iterable i.e which doesn't have \_\_iter\_\_() implemented and try to use it with built in iter().

	In [44]: class NotIterable(object):
		...:     pass
		...:

	In [45]: iter(NotIterable())
	---------------------------------------------------------------------------
	TypeError                                 Traceback (most recent call last)
	<ipython-input-45-a8708f85a52f> in <module>()
	----> 1 iter(NotIterable())

	TypeError: 'NotIterable' object is not iterable

Built in iter() can only work with an iterable. And calling iter(iterable) returns an iterator.

#### StopIteration

When using iterators, there is a related concept called StopIteration.

Currently every time you call next() on an instance of B, "boom" is returned. Suppose you only want "boom" to be returned 3 times, then you can do.

	In [79]: class B(object):
		...:     def __init__(self):
		...:         self.i = 0
		...:     def next(self):
		...:         if self.i == 3:
		...:             raise StopIteration()
		...:         self.i += 1
		...:         return "boom"

	In [81]: next(b_instance)
	Out[81]: 'boom'

	In [82]: next(b_instance)
	Out[82]: 'boom'

	In [83]: next(b_instance)
	Out[83]: 'boom'

	In [84]: next(b_instance)
	---------------------------------------------------------------------------
	StopIteration                             Traceback (most recent call last)
	<ipython-input-84-95c74b691bf4> in <module>()
	----> 1 next(b_instance)

	<ipython-input-79-700035006973> in next(self)
		  4     def next(self):
		  5         if self.i == 3:
	----> 6             raise StopIteration()
		  7         self.i += 1
		  8         return "boom"

	StopIteration:

#### How for loop works

For loop expects an iterable to be passed to it. Assuming the classes look like the following:

	In [89]: class B(object):
		...:     def next(self):
		...:         return "boom"
		...:

	In [89]: class A(object):
		...:     def __iter__(self):
		...:         return B()
		...:

	In [89]: iterable = A()

	In [89]: for each in iterable:
		...:     print each
		...:

This would keep printing "boom". What happened here:

* Saying `for each in iterable` causes iter(iterable) to be called. This returns the underlying iterator.
* Then next() of iterator is repeatedly called until next() of iterator raises a StopIteration.
* Since in this case StopIteration() is never raised from the iterator, so "boom" keeps on getting returned.

In case we only want "boom" to be printed 3 times, we could do:

	In [89]: class B(object):
		...:     def __init__(self):
		...:         self.i = 0 # Hard coded currently, but can be made configurable
		...:     def next(self):
		...:         if self.i == 3:
		...:             raise StopIteration()
		...:         self.i += 1
		...:         return "boom"
		...:

	In [90]: iterable = A()

	In [91]: for each in iterable:
		...:     print each
		...:
	boom
	boom
	boom

Here StopIteration() was raised after next() of iterator ran for 3 times. So `for` loop only printed "boom" 3 times.

##### How lists work with for loop

Python lists are iterables. Internally lists implement the \_\_iter\_\_() method. And \_\_iter\_\_() of list returns an iterator which has a next() method.

You can verify that a list object has \_\_iter\_\_():

	In [92]: l = [1, 2, 3]

	In [93]: l.__iter__
	Out[93]: <method-wrapper '__iter__' of list object at 0x1058eaa28>

Let's get the corresponding iterator for this iterable.

	In [94]: iterator_for_list = iter(l)

Since we are expecting it to be an iterator, there must be a method next() on this object.

	In [99]: iterator_for_list.next
	Out[99]: <method-wrapper 'next' of listiterator object at 0x1058dd610>

Calling next() on this object will return different elements of list. When no more elements are left, a StopIteration() would be raised

	In [102]: iterator_for_list.next()
	Out[102]: 1

	In [103]: iterator_for_list.next()
	Out[103]: 2

	In [104]: iterator_for_list.next()
	Out[104]: 3

	In [105]: iterator_for_list.next()
	---------------------------------------------------------------------------
	StopIteration                             Traceback (most recent call last)
	<ipython-input-105-3adc9ab4c81f> in <module>()
	----> 1 iterator_for_list.next()

	StopIteration:

Because iterator protocol is implemented on a list, that's why we are able to iterate over a list.

