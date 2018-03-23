---
layout: post
title:  "The magic of metaclasses in Python"
date:   2009-11-21 18:07:59+05:30
tags: Uncategorized
author: shabda
---
Metaclasses are a way for you to have a class act as the template for another class. They are simlar to a classfactory,
in that they create new classes. In words of Tim Peters `Metaclasses are deeper magic than 99% of people are going to need`.

However, in right hands they can be a potent tool, in particular Django uses metaclasses beautifully to create
very beautiful declarative models. Without further ado, here is a very simple (and very wrong) metaclass example.


    class Z(type):
        def __new__(cls, name, bases, attrs, ):
            print cls, name, bases, attrs
            
    class A(object):
        __metaclass__ = Z
        
        
    print A
    print A()
    
    
Running this gives this output.

    shabda@shabda-laptop ~/docs> python testmetaclass.py 
    <class '__main__.Z'> A (<type 'object'>,) {'__module__': '__main__', '__metaclass__': <class '__main__.Z'>}
    None
    Traceback (most recent call last):
      File "testmetaclass.py", line 14, in <module>
        print A()
    TypeError: 'NoneType' object is not callable
    
Let what happened,

1. `print cls, name, bases, attrs` got executed, even though we are not creating `Z` or calling it anywhere.
2. `print A` printed `None`
3. `print A()` failed with a traceback

Lets try a slightly modified file,http://uswaretech.com/blog/2009/11/the-magic-of-metaclasses-in-python/

    class Z(type):
        def __new__(cls, name, bases, attrs, ):
            return str
            
    class A(object):
        __metaclass__ = Z
        
This gives me,
        
        
    print A
    print A('Hello')

    shabda@shabda-laptop ~/docs> python testmetaclass.py 
    <type 'str'>
    Hello
    shabda@shabda-laptop ~/docs> 

The output suggests that `A` is behaving like `str`. This should lead us to a few conclusions,

1. If a class has a `__metaclass__`, it is replaced by another class.
2. The function called for creating the new class is `__new__`, it takes 4 parameters.
3. cls is the metaclass being called, name is the name of the original class, bases are bases for original class, and attrs are attributes from original class.
4. The old class is set to the value returned from `__metaclass__.__new__`
5. As we returned nothing in the first snippet, `A` was set to None. In second snippet it was set to `str`.

Here is an actual metaclass being used by Django.

<script src="http://gist.github.com/240103.js"></script>

1. In line 24, `Form. __metaclass__` is set to `DeclarativeFieldsMetaclass`
2. `DeclarativeFieldsMetaclass` is a subclass of `type`.
3. It defines a method called as `__new__` which returns the newly created class.

And that is all there is to metaclasses!

---------------

This is the first in the series of `short and sweet` Django posts we are going to do. You are interested, right. Do follow us on [twitter](http://twitter.com/uswaretech) or [subscribe to our feed](http://feeds.feedburner.com/uswarearticles).

-----

We build *Amazing We Apps*. [Talk to us](http://uswaretech.com/contact/) or email us at sales@uswaretech.com .



