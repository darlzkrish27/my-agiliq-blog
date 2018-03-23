---
layout: post
title:  "Method decorators in Python"
date:   2014-07-15 19:46:08+05:30
tags: python
author: akshar
---
This post assumes you have a basic understanding of Decorators. If not, you can read our <a href="http://agiliq.com/blog/2012/11/understanding-decorators-2/" target="_blank">introductory post on Decorator</a>

### Method decorators

In introductory post, we learnt about function decorators. In this post we'll see method decorators:

Let's write a Class and a method on it which we will decorate later.

	>>> class Person(object):
	...     def __init__(self, name):
	...         self.name = name
	...     def print_name(self):
	...         print self.name

Create a Person instance. p1 refers to the created instance.

	>>> p1 = Person("eddard")

Call print_name on Person.

	>>> p1.print_name()
	eddard     #output

Something worth noting about previous Python statement. Quoting from <a href="https://docs.python.org/2/tutorial/classes.html" target="_blank">docs</a>

	"The method function is declared with an explicit first argument representing the object, which is provided implicitly by the call"

It means, call p1.print_name() implicitly passes p1 to method print_name. And so **self** in print_name starts referring to our created Person instance.

There is another way in which print_name() can be called on p1. Knowing this way would make understanding method decorator easier.

	>>> Person.print_name(p1)
	eddard     #output

In last statement, print_name() wasn't called on p1, but instead called on class Person. So p1 couldn't be passed to print_name() implicitly. It had to be done explicitly.

This paragraph is important. We can think a method as just a function. If provided with correct arguments, it would work similar to any other function. If called on the instance, instance is passed implicitly as the argument and so method/function works. If called on class, instance has to be passed explicitly and then method/function works. So, most of the things which can be done with a function can be done with the method. Let's play around with it a little.

	>>> some_func = Person.print_name
	>>> some_func(p1)
	eddard     #output

	>>> Person.another_print_name_reference = Person.print_name
	>>> Person.another_print_name_reference(p1)
	eddard     #output

#### Scenario for using decorator
We want Person.print_name code to be executed only if name of the Person instance is not "joffrey". Let's write the decorator for it.

	>>> def decorate_method(method):
	...     def inner(person_instance):
	...         if person_instance.name == "joffrey":
	...             print "What a stupid name, I won't print name for you"
	...         else:
	...             method(person_instance)
	...     return inner

##### Structure of decorate_method

* Signature of function created by the decorator, which in our case is function named **inner**, should be similar to signature of the function it is trying to decorate.
* Here we are trying to decorate a method, print_name, which ultimately is a function. And this function expects a Person instance as its only argument.
* So **inner** is defined to expect one argument too.

Let's use this decorator without using decorator syntax.

	>>> Person.decorated_print_name = decorate_method(Person.print_name)
	>>> p1.decorated_print_name()
	eddard     #output

	>>> p2 = Person("joffrey")
	>>> p2.decorated_print_name()
	What a stupid name, I won't print name for you

Let's use this decorator with decorator syntax.

	>>> class Person(object):
	...     def __init__(self, name):
	...         self.name = name
	...     @decorate_method
	...     def print_name(self):
	...         print self.name

So, this was similar to saying:

	Person.print_name = decorate_method(Person.print_name)

Using our decorated method:

	>>> p1 = Person("eddard")
	>>> p1.print_name()
	eddard
	>>> p2 = Person("joffrey")
	>>> p2.print_name()
	What a stupid name, I won't print name for you

ou

