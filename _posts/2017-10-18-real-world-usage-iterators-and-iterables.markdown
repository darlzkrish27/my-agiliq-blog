---
layout: post
title:  "Real world usage of __iter__ and next"
date:   2017-10-18 10:53:28+05:30
tags: python
author: akshar
---
This post assumes that you have a basic understanding of the iterator protocol. If not read our <a href="http://agiliq.com/blog/2017/10/iterators-and-iterables/" target="_blank">last post.</a>

### Real world usage of \_\_iter\_\_

#### Implementing xrange

There is a builtin function called xrange. It is very similar to list. xrange yields the same values as the corresponding list, without actually storing them all simultaneously. Since it doesn't store all values simultaneously, so it saves memory.

If xrange were not there, it could have been implemented in following way using iterators:

	In [12]: class xrange_iterator(object):
		...:     def __init__(self, max):
		...:         self.max = max
		...:         self.current = 0
		...:     def next(self):
		...:        if self.current == self.max:
		...:             raise StopIteration()
		...:        temp = self.current
		...:        self.current += 1
		...:        return temp


	In [13]: class xrange_2(object):
		...:     def __init__(self, max):
		...:         self.max = max
		...:     def __iter__(self):
		...:         return xrange_iterator(self.max)


We wrote an iterable called xrange_2. And wrote an iterator called xrange_iterator which is used by xrange_2. Let's use the iterable we just wrote:

	In [14]: for each in xrange_2(5):
		...:     print each
		...:
	0
	1
	2
	3
	4

Here values from 0 to 4 weren't stored in the memory simultaneously but instead were transient.

#### Allowing looping over any arbitrary object 

Assume there is a class which encapsulates a list and encapsulates several other things too. We want the elements of list to be retrieved when a user loops over the instance of this class.

	n [15]: class ListContainer(object):
		...:     def __init__(self, fruits):
		...:         self.fruits = fruits

		...:     def __iter__(self):
		...:         return iter(self.fruits)
		...:

	In [16]: l = ListContainer(["orange", "mango", "banana"])

	In [17]: for fruits in l:
		...:     print fruits
		...:
	orange
	mango
	banana

\_\_iter\_\_ returns the iterator of the enacapsulated list. That's why looping over the instance calls next() of the encapsulated list.

Django does something similar with Forms and fields.

A form is implemented using a class in Django. Any form has fields. Form fields are also implemented using class in Django. A Django Field class has several functionality implemented in it like how the widget corresponding to field should look, how the POSTed value should be validated etc.

Django code for forms look something like: (Note that this is not exact code, I have simplified it to keep the post shorter)

	In [1]: class Field(object):
	   ...:     pass

	In [2]: class CharField(Field):
	   ...:     pass

    In [3]: class DecimalField(Field):
     ...:     pass

A form class looks something like

    In [4]: class Form(object):
       ...:     def __init__(self, fields):
       ...:         self.fields = fields
       ...:
       ...:     def __iter__(self):
       ...:         return iter(self.fields)

Django want to return field instances when a form is looped over. That's why it implements an \_\_iter\_\_.

A form instance with two field can be created like:

    In [6]: fields = [CharField(), DecimalField()]

    In [7]: form = Form(fields)

You should be seeing the field instances when you loop over the form

    In [8]: for field in form:
       ...:     print field
       ...:
      <__main__.CharField object at 0x107cd7850>
      <__main__.DecimalField object at 0x107cd7710>

#### Generating transient values

An iterator can be used to generate transient values which can be consumed as the loop is running, and which don't need to be stored in a list. We did the same thing in xrange_2.

Suppose we want to generate fibonacci numbers till any maximum value:

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

We can then use the Fibonacci class we just wrote in following way:

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

