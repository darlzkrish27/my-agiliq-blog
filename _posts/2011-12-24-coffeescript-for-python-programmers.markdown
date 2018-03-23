---
layout: post
title:  "Coffeescript for Python programmers"
date:   2011-12-24 11:54:11+05:30
tags: coffeescript
author: shabda
---
I just learnt Coffeescript, and as a Python programmer loved being able to write Javascript in a Python like language. Coffeescript is inspired by 
Python/Ruby and is very close to these languages. Writing Coffeescript and reading the compiled Javascript also improved my understanding of Javascript.
Without much ado here is translation of some Python code to Coffeescript to get you started.

Defining a variable

	Python
	a = 10

	Coffeescript
	a = 10

Setting scope is done via whitespaces.

	Python

	if i == 10:
		foo()
	
	
	Coffeescript

	if i == 10
		foo()
	

No semicolons

List comprehensions

	Python

	languages = ["Python", "Coffeescript", "Java", "Ruby", "Haskell"]
	languages = [lan.lower() for lan in languages]

	Coffeescript

	languages = ["Python", "Coffeescript", "Java", "Ruby", "Haskell"]
	languages = (lan.lowerCase for lan in languages)

Get a range

	Python
	tillTen = range(1, 10)

	CoffeScript

	tillTen = [1...10]

String Interpolation

	Python

	language = "Python"
	greet = "I love %s" % language

	Coffeescript

	language = "Coffescript"
	greet = "I love #{language}"


Multi Line String

	Python

	"""
	The Quick Brown
	Fox, Jumps over
	The lazy Dog
	"""

	Coffeescript

	"""
	The Quick Brown
	Fox, Jumps over
	The lazy Dog
	"""

Defining a function

	Python

	def double(x):
		return 2 * x
	
	Coffeescript	

	double = (x) -> 
		2 * x
	
	


Here is a larger [python script](https://github.com/shabda/humanhash-coffeescript/blob/master/humanhash.py) converted to [Coffeescript](https://github.com/shabda/humanhash-coffeescript/blob/master/humanhash.coffee). 


You will see that they are very similar and if you know Python learning (and loving) Coffeescript won't take you a long time. Excited? 
[Get started here](http://coffeescript.org/).

----

### Resources

* [CoffeeScript: Accelerated JavaScript Development](http://pragprog.com/book/tbcoffee/coffeescript): This is the book I read Coffeescript from and I highly recommend it.
* [The Little book of Coffeescript](http://arcturo.github.com/library/coffeescript/)
* [Official Coffeescript Site](Http://coffeescript.org/)
* [Another take on Coffeescript for Python people](http://blog.ssokolow.com/archives/2011/05/07/a-python-programmers-first-impression-of-coffeescript/)


-----
Can you believe that it has been only an year since the 1.0 release!


