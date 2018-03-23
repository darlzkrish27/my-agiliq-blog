---
layout: post
title:  "__new__() in python"
date:   2012-06-10 10:30:02+05:30
tags: __new__
author: akshar
---
Lately I started looking into Django code and wish to write about internals of Django. I started with Django models and will be writing about it soon. 
For understanding how Django models work, I had to understand what metaclasses are and how metaclasses work. Metaclasses use method "\_\_new\_\_" and so I looked at what "\_\_new\_\_" does.

###As \_\_new\_\_ is a static method, we will see a lttle bit about static methods and then \_\_new\_\_ in detail. 
1. Understanding static methods.

2. Understanding method "\_\_new\_\_" of any class. We will see how to override method \_\_new\_\_ in a class.

Also, I will be trying all the code we write here on Ipython and I suggest you to try everything on Ipython as well.

###Static methods
A little bit about instance methods first.
Let's write a class.

    In [1]: class A(object):
       ...:     def met(self, a, b):
       ...:         print a, b
       ...:

In this case, met() is an instance method. So, it is expected that we pass an instance of A as the first argument to met.

Let's create an object and call met() on the created object and pass two arguments to met().

    In [4]: obj = A()

    In [5]: obj.met(1,2)
    1 2                #output

####What happened here?
When we called met(), we passed two arguments although met() expects three argument as per its definition. When we wrote obj.met(1, 2), interpreter took care of sending instance *obj* as the first argument to met() and 1 and 2 were passed as second and third arguments respectively.

Let's try calling met() without an instance or in other words let's call the method using class.

    In [6]: A.met(1,2)
    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    /home/akshar/branding_git/netconference/<ipython-input-6-e8b323dba928> in <module>()
    ----> 1 A.met(1,2)

    TypeError: unbound method met() must be called with A instance as first argument (got int instance instead)

We defined met() as an instance method of class A, so it expected an instance of A as the first argument. As its very clear from the error, met() expected an instance of A as the first argument but got an 'int' instead.

If we pass an instance of A as the first argument, it will work as expected.

    In [7]: A.met(obj, 3, 4)
    3 4                  #output

Notice that we called the method on class A and not on an instance of A. But we took care of sending an instance of A as the first argument to met() and it worked as expected.

Let's see static method now.

    In [8]: class B(object):
       ...:     @staticmethod
       ...:     def met(a, b):
       ...:         print a,b
       ...: 

####What does *@staticmethod* above the method definition do?
Its a *decorator* which changes a method to static method. It means the method is no longer an instance method, which means that the method does not expect its first argument to be an instance. So, for our method definition, the method does not expect its first argument to be an instance of B.
Even if we call the method on an instance of B, current instance will not be passed as the first argument to this method, since its a *static method*. For *instance method* that we saw earlier the current instance was passed as the first argument.

    In [9]: B.met(5,6)
    5 6                     #output

Here we were able to call the method on the class and were not required to pass an instance of B as the first argument.

Let's call this method on an instance.

    In [10]: b = B()

    In [11]: b.met(5,6)
    5 6

Here we called the method on an instance and passed two arguments. Since its a static method, the current instance i.e *b* was not passed as the first argument to met().
Had it not been a static method, current instance would have been passed as first argument, 5 as second argument and 6 as third argument.


###Understanding method *\_\_new\_\_* of a class. We will also see how to override method \_\_new\_\_ in a class.
First let's see a little bit about *\_\_init\_\_*

    In [13]: class A(object):
       ....:     def __init__(self, a, b):
       ....:         print "init gets called"
       ....:         print "self is", self
       ....:         self.a, self.b = a,b
       ....: 

In \_\_init\_\_, we print something as we enter the method which is to validate that \_\_init\_\_ has been called. Then we print the first argument which is self and then we perform some assignments. \_\_init\_\_ is an instance method and expects the first argument to be an instance.

Let's call the class passing it two arguments. Keep in mind the part **call the class**, we are going to again use it in next few lines. If you have any confusion about the part *calling the class*, it will be clear in next few lines.

    In [16]: a = A(1,2)
    init gets called                                    #output
    self is <__main__.A object at 0x3357210>            #output

Notice the second line of output which is "self is <\_\_main__.A object at 0x3357210>". As apparent from second line of output, when \_\_init\_\_ is entered, object/instance has already been created by that time. Its only the assignment which is done in \_\_init\_\_, althought you could do some other stuff as well. But \_\_init\_\_ doesn't create the instance. \_\_init\_\_ receives the created instance as the first argument.

####What creates the object?
Method \_\_new\_\_() creates the object.

####What is \_\_new\_\_?
1. \_\_new\_\_ is a static method which creates an instance. We will see the method signature soon. One reason i could think of having \_\_new\_\_ as a static method is because the instance has not been created yet when \_\_new\_\_ is called. So, they could not have had it as an instance method.
2. \_\_new\_\_ gets called when you *call the class*. *Call the class* means issuing the statement "a=A(1,2)". Here A(1,2) is like calling the class. A is a class and we put two parenthesis in front of it and put some arguments between the parenthesis. So, its like "calling the class" similar to calling a method.
3. \_\_new\_\_ must return the created object.
4. Only when \_\_new\_\_ returns the created instance then \_\_init\_\_ gets called. If \_\_new\_\_ does not return an instance then \_\_init\_\_ would not be called. Remember \_\_new\_\_ is always called before \_\_init\_\_.
5. \_\_new\_\_ gets passed all the arguments that we pass while calling the class. Also, it gets passed one extra argument that we will see soon.

####How was the instance created in the last example when we didn't define \_\_new\_\_?
class A extends from object(Here we mean the class named object) i.e subclasses from object. object defines a method \_\_new\_\_, so A gets this method from object since its extending object. This inherited \_\_new\_\_ created the instance of A.

####Method signature of \_\_new\_\_
\_\_new\_\_ receives the class whose instance need to be created as the first argument. This statement could be a little confusing, just continue reading and see the next example and again read it after seeing the example, it will be clear. The other arguments received by \_\_new\_\_ are same as what were passed while calling the class.

So, \_\_new\_\_ receives all the arguments that we pass while calling the class. Also, it receives one extra argument. This extra argument is the class whose instance need to be created and it will be received as first argument by \_\_new\_\_.

So, signature of \_\_new\_\_ could be written as:
    
    __new__(cls, *args, **kwargs)

Let's see an example.

    In [22]: class A(object):
       ....:     def __new__(cls, *args, **kwargs):
       ....:         print cls
       ....:         print "args is", args
       ....:         print "kwargs is", kwargs
       ....: 

Here we override \_\_new\_\_ that we inherit from the superclass. We are printing all the arguments that this method receives so that we can check what gets passed to \_\_new\_\_.
Let's try to create an instance of A by calling the class.

    In [23]: a=A()
    <class '__main__.A'>                              #output
    args is ()                                        #output
    kwargs is {}                                      #output

As we mentioned earlier, \_\_new\_\_ gets called when we call the class. As is apparent from the output \_\_new\_\_ was called and it printed three lines of output.

First line of output prints the first argument received by \_\_new\_\_. As we can see, it is  *class A* itself. **We tried to create an instance of A and \_\_new\_\_ of A received class A itself as the first argument.** 
This is what we meant when we said "\_\_new\_\_ receives the class whose instance need to be created as the first argument". Now go back to the section "Method signature of \_\_new\_\_" and read it again.

While calling the class we did not pass any arguments. So our output shows that args and kwargs did not receive anything.

You can verify that all the arguments passed while calling the class gets sent to \_\_new\_\_. Just call the class passing it some arguments.

    In [25]: a=A(1,2,named=5)
    <class '__main__.A'>                              #output
    args is (1, 2)                                    #output
    kwargs is {'named': 5}                            #output

So, whatever arguments we passed while calling the class were passed to \_\_new\_\_ and were received by args and kwargs in \_\_new\_\_.

Let's check whether an object really gets created with how we have currently overridden \_\_new\_\_.

    In [26]: a = A(1,2)
    <class '__main__.A'>                              #output
    args is (1, 2)                                    #output
    kwargs is {}                                      #output

    In [27]: print a
    None                                              #output

We tried to create an instance and then tried printing the instance. But an instance of *A* was not created as apparent from the last print statement which printed None.

####Why did this happen?
As we know if we don't return any value from a method, it implicitly returns None.
Under the section "What is \_\_new\_\_", we mentioned that \_\_new\_\_ must return the created instance.
Here we did not return the created instance from \_\_new\_\_, so None was implicitly returned and was assigned to *a*.

Let's combine \_\_new\_\_ and \_\_init\_\_.

    In [29]: class A(object):
       ....:     def __new__(cls, *args, **kwargs):
       ....:         print cls
       ....:         print args
       ....:         print kwargs
       ....:     def __init__(self, a, b):
       ....:         print "init gets called"
       ....:         print "self is", self
       ....:         self.a, self.b = a, b
       ....: 

Let's try to create an instance of A.

    In [31]: a=A(1,2)
    <class '__main__.A'>                              #output
    (1, 2)                                            #output
    {}                                                #output

As we mentioned earlier when a class gets called, first \_\_new\_\_ is called. Only when \_\_new\_\_ returns an instance then \_\_init\_\_ is called.

In our previous example \_\_new\_\_ did not return an instance. So \_\_init\_\_ was not called. Had \_\_init\_\_ been called we would have seen the print statements that we have inside \_\_init\_\_.

Also since \_\_new\_\_ did not return an instance, *a* will still be None. Verify that.

    In [32]: print a
    None                                              #output

Let's redefine the class to make it proper. We should return an instance from \_\_new\_\_, so that \_\_init\_\_ gets called and we get the desired behaviour.

In case we don't override \_\_new\_\_, \_\_new\_\_ of parent class creates the instance and then \_\_init\_\_ gets called. In case we are overriding \_\_new\_\_, we should call the \_\_new\_\_ of parent class to get the created instance. However if you know how object creation works at the low level and you can implement it in your overridden \_\_new\_\_, you don't need to call parent \_\_new\_\_ to get the created instance. I don't know such details of how object creation work and will use the parent \_\_new\_\_ to get the created instance.

Once we get the created instance we can perform any extra operations we wish before returning the instance from \_\_new\_\_ method.

For demonstration purpose, let us take a weird example where we need to add an attribute named 'created_at' to the created instance. For our case lets consider it needs to be done inside the \_\_new\_\_ method, althought we could have done it inside \_\_init\_\_.

    In [33]: import datetime

    In [35]: class A(object):
       ....:     def __new__(cls, *args, **kwargs):
       ....:         new_instance = object.__new__(cls, *args, **kwargs)
       ....:         setattr(new_instance, 'created_at', datetime.datetime.now())
       ....:         return new_instance
       ....:     def __init__(self, a, b):
       ....:         print "inside init"
       ....:         self.a, self.b = a, b
       ....:

In the first line of \_\_new\_\_, we called the \_\_new\_\_ of parent class to get the created instance. \_\_new\_\_ of parent class should be passed the same arguments that we received in the overridden \_\_new\_\_. \_\_new\_\_ of parent class i.e \_\_new\_\_ of class *object* knows how to create an instance and it returns the created instance.

In the next line, we used inbuilt method setattr() to set an attribute 'created_at' on the newly created instance. The value we set for this attribute is the current time. This line is equivalent to writing new_instance.created_at=datetime.datetime.now().

In the final line we returned the newly created instance. Since we are returning an instance from \_\_new\_\_, \_\_init\_\_ will be called passing it whatever arguments were used in the class call. Let's verify this.

    In [36]: obj1 = A(1,2)
    inside init                                    #output

This statement suggests that \_\_init\_\_ was called. Let's print the created instance.

    In [37]: obj1
    Out[37]: <__main__.A at 0x3357390>

Notice that earlier when we were not returning anything from \_\_new\_\_ and were trying to print it, we were getting output as None. But this time the output shows that obj1 refers to an instance of A.

We can verify that obj1 has an attribute 'created_at' and \_\_init\_\_ was properly executed by printing the three attributes of obj1.

    In [37]: print obj1.created_at
    2012-06-09 22:44:30.376914                    #output

    In [38]: print obj1.a, obj1.b
    1 2                                           #output

Let's see our final example.

    In [60]: class B(object):
       ....:     pass
       ....: 

    In [61]: class A(object):
       ....:     def __new__(cls, *args, **kwargs):
       ....:         new_instance = object.__new__(B, *args, **kwargs)
       ....:         return new_instance
       ....: 

Pay attention to first line of A's new. Instead of passing *cls* as the first argument to *object.\_\_new\_\_*, we pass *class B* as first argument.
Let's see what happens in such case.

    In [62]: a = A()

    In [63]: print a
    <__main__.B object at 0x7f912c036750>                    #Output. Tried creating an instance of A but got an instance of B

We tried to create an instance of A. But when we printed it, we realise that an instance of B has been created. 

This happened because we passed *class B* as the first argument to *object.\_\_new\_\_*. This shows that whatever class we pass to superclass' \_\_new\_\_, an instance of that class will be created.

Remember \_\_new\_\_ receives the class whose instance need to be created as first argument. So for any \_\_new\_\_, the first argument (which is *cls* for our case) will always refer to the class inside which \_\_new\_\_ is defined. So, for our case, *cls* will be *class A*.

Here we wanted to create an instance of A. So, *class A* must be passed as first argument to *object.\_\_new\_\_*. Inside \_\_new\_\_ of *class A*, *cls* refers to *class A*. So, we need to pass *cls* as first argument to *object.\_\_new\_\_*.

That's why if we want proper behaviour we need to pass the same arguments to the superclass' \_\_new\_\_ as it was received by the overridden \_\_new\_\_.

We can make that single line change in A's \_\_new\_\_ and our code will behave as expected.

    In [64]: class A(object):
       ....:     def __new__(cls, *args, **kwargs):
       ....:         new_instance = object.__new__(cls, *args, **kwargs)
       ....:         return new_instance
       ....:   

    In [65]: a=A()

    In [66]: print a
    <__main__.A object at 0x7f912c0368d0>                    #Output. We got an instance of A

That was all about method \_\_new\_\_. Hopefully next post would be about metaclasses and there we can see some more useful uses of \_\_new\_\_.

Hope you liked the post.

post.

