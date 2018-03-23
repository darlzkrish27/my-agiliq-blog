---
layout: post
title:  "Metaclass in Python"
date:   2012-07-02 08:11:46+05:30
tags: Metaclass
author: akshar
---
In this post we will be talking about Metaclass in python. If you are reading some code which uses metaclass, you will probably come across `__new__`. If you are not familiar with what \_\_new\_\_ does, i suggest you first read about \_\_new\_\_.

You can read about `__new__` in our last to <a href="http://agiliq.com/blog/2012/06/__new__-python/">last post</a>

Throughout this post we will be talking about new style classes. Things might differ in old-style classes.

As we go, i will be trying everything on IPython and suggest you as well to try everything on IPython.

Let's see a little bit about normal Objects in Python.

    >>> class A(object):
    ...     pass
    ... 

We created a class named A. A extends from class object (Here by object we mean class named object from which all new style classes extend). 

Hereafter if we use lowercase 'o' in object, it means we want to refer to the class named object. If we use an Object with uppercase 'O', we mean an instance of some class i.e an instance created by calling the class. 

Let's create an instance of A.

    >>> obj=A()

Let's print 'obj' to see what it is.

    >>> print obj
    <__main__.A object at 0x1e34e50>               #output

Output tells that obj is an instance of A. Or, we can say obj is an Object(Pay attention to the capital 'O' we used in Object).

Let's check the **type** of obj.

###What is *type* of an Object?
Roughly speaking, by type of an Object, we mean the class of the concerned Object. By knowing the type of an Object/instance, we know what class was used to create the concerned instance. It is like knowing **concerned Object is an instance of which class**.

###How do we know the type of an Object/instance?
* There is a built-in function named **type()** which tells the type/class of the Object.
* Method signature of type is:
    * type(some_object)
* This method returns the class of the passed argument. So, it tells the class which was used to create the passed argument.
* Documentation of type() says that type() takes an Object as the argument and returns the class of that Object. It means that if you pass some argument to type() and type() executes successfully without raising an error, it means that the argument you passed is an Object. You can consider it as a test of being an Object.

Let's check type of obj now.

    >>> type(obj)
    <class '__main__.A'>                           #output

Output tells that class of obj is A which means that obj is an instance of A.

As we know, A is a class. 
But **is A an Object?** In other words, is A an instance of something? Let's try finding out using type().

    >>> type(A)
    <type 'type'>                                  #output

We passed a class, class A, as an argument to type(). Since type() executed successfully and did not raise any error, it confirms A is an Object. It means even **classes are Objects** in Python. 

Actually, everything is an Object in Python. Classes are Objects, Functions are Objects, Methods are Objects, Literals are Objects. Anything you can define in Python is an Object.

We can verify this by passing anything we can define in Python as an argument to type() and we will see that type() will execute successfully without raising any error.

    >>> type(A)
    <type 'type'>                                 #classes are Objects

    >>> type(2)
    <type 'int'>                                  #integer is an Object

    >>> def met(arg):
    ...     return arg
    ... 
    >>> type(met)
    <type 'function'>                             #functions are Objects

###What is a Metaclass?
We verified that classes are Objects in Python. But Object mean instance. So, a **class must be an instance of some other class** i.e class itself must be an instance of some other class. 

In our example, **class A must be an instance of some other class**. Let's assume the name of that class is SomeClass. SomeClass is a metaclass. So, **the class of a class is known as Metaclass**. Might be a little confusing, Keep reading, it will be clear with the explanation to follow.

####Explanation:
In our earlier examples, class of obj was A which means obj is an instance of A. But, we could even execute type(A) which must have returned the class of A. Or in other words, we can say it told us A was an instance of which class. Let' see it once more.

    >>> type(A)
    <type 'type'>                                 #output

Output means that A is an instance of a class named type. In other words, class of A is type. Don't get confused when we are saying that type is a class and earlier we said that type() is a builtin function. We have the explanation in next few lines.

obj was an instance of A and we created it like "obj=A()".

A is an instance of type. So, under the hood, A would have got created something like "A=type()". 
This call to class type for creating class A was taken care by the interepreter when interpreter saw the class definition for A. Actually, call to class type is a little different than this, some arguments are also passed while calling type which we will see soon.

A is a class. And A's class is type. So, here **type is a metaclass because it is a class of a class**.

###Earlier we said that type() is a builtin function. Now we are saying that type is a class. How?
* Yes, there is a built-in function named type() in Python as well as there is a class named type in Python.
* How the interpreter interprets it depends on how we use it. 
* If we pass a single argument to type(), it is interpreted as a function and we get the class of the passed argument.
* type is also a class and it can be used to create instance. Infact it must have been used to create class A as we described in last to last paragraph, though it was used under the hood. Since it was used under the hood, we could not see how class type was used to create class A.
Instances of class type turn out to be classes. eg: A is an instance of class type but it is a class as well.


Till here we know that type is the metaclass used for creating class A. Remember **type is the default metaclass** which is used to create classes. We can override this default behaviour. We can specify an attribute named `__metaclass__` on the class we want to be created  and the required class will be created using the metaclass that we specify. If you understood that type is the default metaclass that was used here, you need not worry about other metaclass stuff that we wrote in this para, it will be clear with next few lines.

####Before we go further, a para about `__new__`. 
We know that instance of a class is created by calling that class. Calling that class in turns calls the \_\_new\_\_ method of that class where actual instance creation happens. As we know, a class is an instance of the metaclass. So a class must be getting created by calling the metaclass which in turn would be invoking \_\_new\_\_ of the metaclass where actual class creation would be happening. 

###How was class A created?
* Interpreter sees the class definition which was class A in our case.
* Interpreter checks whether an attribute named `__metaclass__` has been defined for this class. If not then it considers class type as the metaclass and uses it to create the required class.
In our class definition for class A, there was no attribute named \_\_metaclass\_\_ defined on A. So, class type was used as the metaclass and was used to create A.
* As we know a class is an instance of the Metaclass. So it must be getting created in `__new__` of the Metaclass.
So for our case, class A must have got created inside \_\_new\_\_ of class type.
* For creating a class, metaclass is called by the interpreter passing three arguments to the metaclass.
Three arguments are :
    * name : This will be the name of the class that needs to be created. For our case, it would be string 'A'.
    * bases : it is a tuple containing all the bases for the class that needs to be created. For our case, it would be a tuple containing only one class which is class object. So, it is (object,)
    * attrs : It is a dictionary containing all the attributes defined for the class plus one additional key which is \_\_module\_\_. Here we did not define any attribute for A, so dictionary will only have one key value pair. So, attrs is {'\_\_module\_\_':'\_\_main\_\_'}  

So, for creating class A, interpreter would have made the following call under the hood on seeing the class definition of A. 

type('A', (object,), {'\_\_module\_\_':'\_\_main\_\_'})

Remember, interpreter will call the metaclass and will pass it three arguments described above whenever interpreter needs to create a class.

This in turn would have called method \_\_new\_\_ on class type to create an instance of class type which turns out to be class A.

And thus class A was created.

###Using class type explicitly to create a class.
In the previous section we saw that class type was used under the hood to create class A. Also we saw that three arguments were passed while calling type. We can create a class by explicitly using class type and without writing the class definition. Try the next statement in your shell.

    >>> A=type('A', (object,), {})

We called class type and pass it the required arguments to create a class. Pay attention that we passed three arguments to type. 

* First argument is a string which is the name we want for the class. 
* Second argument is a tuple containing the base classes we want for our class. Since we wanted to create a new style class, we want our class to extend from object and hence tuple contains object. If you want your class to extend from some more classes, you can add those classes to the tuple. 
* Third argument is a dictionary containing the attributes that we want to set on the class. Here we did not want to set any attribute on the class, so dictionary is empty.


Remember three arguments need to be passed while calling class type if we are using class type explicitly. Also, the arguments need to be passed in the same order that we described earlier i.e first argument should be the name we want for the class, second argument should be a tuple of bases for the class and third argument must be a dictionary containing the attributes we want on the class.

However, interpreter takes care of adding one key value pair to this dictionary which specifies the module for the class. So, interpreter would have changed our call to class type to:

A = type('A', (object,) {'\_\_module\_\_' : '\_\_main\_\_'})

####Why three arguments need to be passed when calling class type?
class type has a method \_\_new\_\_ where actual class creation happens. But method \_\_new\_\_ of class type has been defined to expect four arguments. 

As you must be knowing, \_\_new\_\_ receives all the arguments passed while calling the class plus one extra argument, which is the class itself, as the first argument. In total, \_\_new\_\_ of type expects four arguments to be passed to it, so we need to call class type passing it three arguments. One extra argument will be added by the interpreter and hence four arguments will be passed to \_\_new\_\_ of class type.

Try printing A and type of A.

    >>> A
    <class '__main__.A'>

This confirms that class A was created by our explicit call to class type.

    >>> type(A)
    <type 'type'>

This confirms that A is an instance of type or the metaclass used to create class A is class type.

So in this section we explicitly used class type to create class A. We did not use the class definition to create the class. This was just to demonstrate that type creates our class under the hood and we could use type explicitly to create our class.

Till here we only talked about the default metaclass type. Let's write our own Metaclass now.

###Some points about Metaclass to keep in mind.
* class type is the default metaclass used for class creation.
* Metaclass is just a class which has the capability to create a class. Like a normal class has the capability to create an instance of that class, similarly Metaclass has the capability to create a class. The created class is an instance of Metaclass.
* Remember, a class is an instance of Metaclass. So we can treat a class similar to how we treat an instance and in that case don't get confused.
* Since type is the default metaclass which knows how to create a class, so if we want to write a metaclass we must extend class type. 


Let's write the Metaclass named MyMeta.

    >>> class MyMeta(type):
    ...     def __new__(cls, name, bases, attrs):
    ...         return super(MyMeta, cls).__new__(cls, name, bases, attrs)
    ... 

Name of our metaclass is MyMeta.

* Our Metaclass extends from class type.
* We override method \_\_new\_\_ of class type in MyMeta.
* Interpreter would call our metaclass with three arguments and hence \_\_new\_\_ of metaclass would be receiving four arguments. So we take care that \_\_new\_\_ of metaclass expects four arguments.
* Inside \_\_new\_\_ of MyMeta we use \_\_new\_\_ of superclass which turns out to be \_\_new\_\_ of class type. As we discussed earlier, actual class creation happens inside \_\_new\_\_ of type if no metaclass is used. It means \_\_new\_\_ of class type knows how class creation works at low level.

So if we are writing a metaclass, we must inherit from class type so that we can inherit the class creation capability of class type. And we must overide \_\_new\_\_ of type and must use \_\_new\_\_ of superclass type in the overridden \_\_new\_\_ for class creation.


Let's use the metaclass we wrote to create some class.

    >>> class B(object):
    ...     __metaclass__ = MyMeta
    ... 

We defined a class named B and set an attribute `__metaclass__` on this class. Attribute \_\_metaclass\_\_ was set as MyMeta, the metaclass we just wrote.

Let's check class of B. Or we can say let's check B is an instance of which class.

    >>> type(B)
    <class '__main__.MyMeta'>                    #output

Output tells that B is an instance of MyMeta. So, MyMeta was used to create class B.

####How was class B created?
* Interpreter sees the class definition.
* Interpreter sees that there is an attribute \_\_metaclass\_\_ defined on B. So now interpreter knows that default metaclass type must not be used to create B. Instead MyMeta must be used to create B.
* So, interpreter makes a call to MyMeta to create class B.
Internally interpreter makes the following call.
    * MyMeta('B', (object,), {'\_\_module\_\_':'\_\_main\_\_', '\_\_metaclass\_\_':MyMeta})
* When MyMeta is called, \_\_new\_\_ of MyMeta is invoked.
* \_\_new\_\_ of MyMeta gets passed three arguments that were passed while calling MyMeta. Additionally, it also gets passed an extra argument as the first argument. This extra argument is the class in which \_\_new\_\_ is defined.
* So, \_\_new\_\_ of MyMeta gets passed four arguments. We have defined \_\_new\_\_ of MyMeta to expect four arguments.
* Inside \_\_new\_\_ of MyMeta, we use \_\_new\_\_ of superclass type to create the instance of MyMeta. This instance of MyMeta turns out to be class B.


This is the reason our metaclass should extend from type. So that we could use \_\_new\_\_ of superclass i.e \_\_new\_\_ of type in the overridden \_\_new\_\_ . We want to use \_\_new\_\_ of type for actual class creation because it already knows how to do it and we don't need to bother about how actual class creation works at low level.

We return this created instance of MyMeta i.e class B from \_\_new\_\_ of MyMeta. 

We can print everything inside \_\_new\_\_ of MyMeta to verify what all is received by \_\_new\_\_ of MyMeta for creating class B.

Let's redefine MyMeta.

    >>> class MyMeta(type):
    ...     def __new__(cls, name, bases, attrs):
    ...         print "cls is", cls
    ...         print "name is", name
    ...         print "attrs is", attrs
    ...         return type.__new__(cls, name, bases, attrs)
    ... 

Let's again define class B and set its \_\_metaclass\_\_ as MyMeta.

    >>> class B(object):
    ...     __metaclass__ = MyMeta
    ...     
    cls is <class '__main__.MyMeta'>                                                 #output
    name is B                                                                        #output
    attrs is {'__module__': '__main__', '__metaclass__': <class '__main__.MyMeta'>}  #output

####What happened here?
* Interpreter saw the class definition. It sees that there is an attribute named \_\_metaclass\_\_.
* Its uses this \_\_metaclass\_\_ to create class B. So, it makes the following call:
    * MyMeta('B', (object,), {'\_\_module\_\_':'\_\_main\_\_', '\_\_metaclass\_\_':MyMeta})
* This call in turn would invoke \_\_new\_\_ of MyMeta.
* First argument received by \_\_new\_\_ of any class is the class itself. So, in \_\_new\_\_ of MyMeta, first argument received is class MyMeta itself. We can verify this by the output of first print statement.
* Second argument received by \_\_new\_\_ of MyMeta is the name of the class that need to be created. From the output, we verify that it is 'B'.
* In the last statement in \_\_new\_\_ of MyMeta, we pass all the received arguments to \_\_new\_\__ of type where actual class creation happens.

Earlier when we were not using any metaclass, default metaclass type was being used. At that time \_\_new\_\_ of type was used and the first argument being passed to \_\_new\_\_ of type was class type itself. So, the class created used to be an instance of class type.

But now, in our call to \_\_new\_\_ of class type from inside \_\_new\_\_ of MyMeta, we are passing cls as the first argument. As we verified by printing cls, cls refers to class MyMeta. So now our created class is an instance of MyMeta.

Remember our last post about \_\_new\_\_ where we override \_\_new\_\_ of class object, and used \_\_new\_\_ of object in overridden \_\_new\_\_. The created instance was an instance of the class that we passed as first argument to \_\_new\_\_.
That's why here the created instance which is class B is an instance of MyMeta since the first argument passed to \_\_new\_\_ of class type is MyMeta.

Till this point, we did not do anything special in our Metaclass. If we don't use any metaclass, our class would be an instance of class type. When we set a metaclass attribute on our class, the class turns out to be an instance of the metaclass that we set. But apart from it we did not do any special operation on our class.

Let's perform some operation during class creation.

Let's consider a weird example. This example is only for illustration purpose. This example is in part taken from how class Meta is implemented in Django models.

Let's consider that we are writing some framework. People would be writing classes in that framework. We want to force some limitations on the class that people are going to write. We only want to allow a limited number of attributes on any class that can be defined. Say we only want people to set attribute named first and second on any class they define. Any other attributes they try to set on the class must be discarded during class creation.

Say a person writes a class and defines three attributes namely first, second and third on the class. However we only want to allow two attributes namely first and second on the class. We don't want any attribute apart from attributes named first and second to be available on the created class. So in this case we need to discard attribute named third.

Also we should not be touching any attributes that start with \_. Attributes that start with \_ are private. Also there are few attributes that are generated by Python which start with \_\_. An example is \_\_module\_\_. We should not be touching those attributes as changing them would lead to weird behaviour.

Let's write our metaclass.

    >>> allowed_attributes = ['first', 'second']

We defined a list of attributes that we will allow on the class.

Next comes the metaclass that will be used to create the class.

    >>> class Meta(type):
    ...     def __new__(cls, name, bases, attrs):
    ...         print "Given attributes are", attrs
    ...         attrs_list = list(attrs)
    ...         for each_attr in attrs_list:
    ...             if not each_attr.startswith('_') and each_attr not in allowed_attributes:
    ...                 del attrs[each_attr]
    ...         print "Attributes after deleting non allowed attributes", attrs
    ...         return type.__new__(cls, name, bases, attrs)
    ... 

###What all happens in this metaclass?
1. First line prints the dictionary that we get in \_\_new\_\_. If we don't make any manipulation in this dictionary, all the keys defined in this dictionary will be available as attributes on the class that is created using this metaclass.
2. We create a list from this dictionary in line 2 of \_\_new\_\_. Reason is that we we want to manipulate the dictionary while looping over the contents of dictionary. But changing dictionary size during iterating over the dictionary is not allowed in Python. So, we create a list from this dictionary on which we can iterate and while iterating we make changes to the dictionary from which this list was created.
3. We loop over the created list.
4. We first check that the attribute does not start with \_. Then we check whether the attribute should be allowed for the class. If the attribute does not start with \_ and also if it should not be allowed as an attribute, in such case our condition evaluates to True and we reach the next line.
5. Here we delete this attribute from the dictionary if the condition in last statement evaluates to True.
6. We print the attributes after we are done manipulating the dictionary.
7. Actual class creation happens at this line.

Let's use the metaclass that we just wrote to create some class.

    >>> class C(object):
        __metaclass__ = Meta
        first = 1
        second = 2
        third = 3
    ...     
    Given attributes are {'__module__': '__main__', '__metaclass__': <class '__main__.Meta'>, 'second': 2, 'third': 3, 'first': 1}                  #output
    Attributes after deleting non allowed attributes {'__module__': '__main__', '__metaclass__': <class '__main__.Meta'>, 'second': 2, 'first': 1}  #output

We tried setting three attributes on class C. 

Compare the two lines of output. In the second line of output, it can be seen that key named 'third' has been removed. So when we passed this dictionary during actual class creation, key 'third' was not present in attrs and hence attribute 'third' will not be available on class C.

Let's confirm that.

    >>> C.first
    1                      #output

    >>> C.second
    2                      #output

    >>> C.third
    Traceback (most recent call last):
      File "<ipython-input-55-b760afd6f6d5>", line 1, in <module>
        C.third
    AttributeError: type object 'C' has no attribute 'third'

That was all about Metaclasses. Hope you liked the post.

P.S : I never had to write any metaclass in my project and i believe one would never need to write it unless one is writing a framework. Quoting Tim Peters

Metaclasses are deeper magic than 99% of users should ever worry about. If you wonder whether you need them, you don't (the people who actually need them know with certainty that they need them, and don't need an explanation about why).

---
You may also like to read [The magic of metaclasses in Python](http://agiliq.com/blog/2009/11/the-magic-of-metaclasses-in-python/)

