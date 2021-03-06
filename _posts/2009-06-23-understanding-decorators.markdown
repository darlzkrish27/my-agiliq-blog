---
layout: post
title:  "Understanding decorators"
date:   2009-06-23 19:34:57+05:30
categories: python
author: shabda
---
If you used Django for any length of time, you would have come across the
`login_required` decorator. You write `@login_required` before a view, and it
magically becomes accessible only to authenticated users.

Decorators were introduced in python 2.4. [PEP 318](http://www.python.org/dev/peps/pep-0318/)
is the [PEP](http://www.python.org/dev/peps/) describing it. At
the simplest decorators are nothing but callables returning other callables, and
the decorator syntax `@decorator` is nothing but `foo = bar(foo)`, where both
`bar` and `foo` are callables.

Let us see some decorators in action.

    
    In [1]: def good_function():
       ...:     print 'I am a good function'
       ...:     
       ...:     
    
    In [2]: def decorator(orig_func):
       ...:     def bad_func():
       ...:         print 'I am a bad function'
       ...:     return bad_func
       ...: 
    
    In [3]: good_function = decorator(good_function)
        
    In [4]: good_function()
    I am a bad function
    
    In [5]: @decorator
       ....: def good_function2():
       ....:     print 'I am a good function'
       ....:     
       ....:     
    
    In [6]: good_function2()
    I am a bad function
    
So you can see that the decorated function depended only on the value of the
function returned by the decorator.

Here we discarded the function we were passed. Any useful decorator, _decorates_
the original function, so it would use the original function. Let us see an
actual decorator which may be useful.

    def is_val_positive_deco(orig_func):
            def temp_func(val):
                    if val < 0:
                            return 0
                    else:
                            return orig_func(val)
            return temp_func
    
    @is_val_positive_deco
    def sqrt(val):
            import math
            return math.pow(val, (1.0/2))
            
    print sqrt(-1)
    print sqrt(4)
    
Here we defined an decorator `is_val_positive_deco` which will make functions
return 0, if the argument passed is negative. We can use this decorator to guard
against MathErrors.

Class based decorators
--------------------------

Decorators are just callables, and hence can be a class which has `__call__`
method. Sometimes they are easier to understand and reason about(than decorators
written as functions).
Lets see one,

    class LogArgumentsDecorator(object):
            def __init__(self, orig_func):
                    self.orig_func = orig_func
                    print 'started logging: %s' % orig_func.__name__
                    
            def __call__(self,  *args, **kwargs):
                    print 'args: %s' % args
                    print 'kwargs:%s'% kwargs
                    return self.orig_func(*args, **kwargs)
                    
    @LogArgumentsDecorator		
    def sum_of_squares(a, b):
            return a*a + b*b
    
    print sum_of_squares(3, b=4)
    
This outputs,

    started logging: sum_of_squares
    args: 3
    kwargs:{'b': 4}
    25


Lets see what happens when we do,

    @LogArgumentsDecorator		
    def sum_of_squares(a, b):

This is equivalent to

    sum_of_squares = LogArgumentsDecorator(sum_of_squares)
    
At this point, we created a new LogArgumentsDecorator object, so
`LogArgumentsDecorator.__init__(self, sum_of_squares)` gets called. Hence
`print 'started logging: %s' % orig_func.__name__` line is executed.

When we do `sum_of_squares(...)`

`LogArgumentsDecorator.__call__` gets called with the passed values, where we
print the arguments and call the original function.

Here is one more example of a class based decorator. Here we rewrite(sort of,
anyway) Django's login_required decorator as a class based decorator.

    class LoginRequiredDecorator(object):
	def __init__(self, orig_func):
		self.orig_func = orig_func
	
	def __call__(self, request, *args, **kwargs):
		if request.user.is_authenticated():
			self.orig_func(request, *args, **kwargs)
		else:
			return HttpResponseRedirect(reverse('...'))
                        

Similar to above, it takes the view function, during `__init__`. When the request
comes, `__call__` makes sure that user is authenticated, and takes action
appropriately.

Resources
* [Decorator Library](http://wiki.python.org/moin/PythonDecoratorLibrary)
* [Pep 318](http://www.python.org/dev/peps/pep-0318/)


--------------------
Subscribe to [The Uswaretech Blog](http://feeds.feedburner.com/UswareBlog) where we will be writing the next part of this tutorial explaining more complex decorators, decorators which take arguments, decorator factories and more. Or follow us on [twitter](http://twitter.com/uswaretech).




