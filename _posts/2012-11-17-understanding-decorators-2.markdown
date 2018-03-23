---
layout: post
title:  "Understanding decorators"
date:   2012-11-17 14:43:53+05:30
tags: Function as objects
author: akshar
---
Understanding decorators in Python

###We will divide this post into two topics
1. Functions are objects and they can be passed around.
2. Writing few decorators

As usual, I will be trying out the code in ipython and I suggest you to do the same.

###Functions can be passed around similar to any other object in Python.

If you are already familiar with this, you can skip to next section.

Let's first write a class.

    >>> class A(object):
    ...     def __init__(self, a):
    ...         self.a = a
    ...

Let's create an instance of this class.

    >>> obj = A(5)    
    >>> print obj.a
    5                 #output

Let's write a function that takes an object, increase the value of instance variable 'a' by 10 and return the object.

    >>> def func(some_obj):
    ...     some_obj.a = some_obj.a + 10
    ...     return some_obj
    ... 

In function **func**, our assumption is that we will always pass it an object as argument and this object will have an instance variable 'a'.

Let's pass the created object **obj** of class A to this function and take the return value of this function in a variable named obj2.

    >>> obj2 = func(obj)
    >>> print obj2.a
    15

We wrote a function 'func' that takes an object as an argument, manipulates it and also returns that object. **We passed around the object 'obj' of class A**.

In Python, functions are objects. They can be passed around similar to any other object. Let's see this through an example.

Let's write a function that prints something.

    >>> def hello_world():
    ...     print "Hello World"
    ...  

Use this function.

    >>> hello_world()
    Hello World            #output

Since functions are objects they can be assigned to any variable.

    >>> hello_world_two = hello_world

We assigned hello_world, which is a function, to another variable named hello_world_two.

Use hello_world_two

    >>> hello_world_two()
    Hello World             #It gives the same output as hello_world()

Now we want to see how to pass around this function.

For that, we want to write a new function that takes function 'hello_world' as an argument and then uses it.

    >>> def use_hello_world(some_func):
    ...     print "This function takes a function as an argument and uses it"
    ...     print "We will send function hello_world to this function as an argument and will use it in this function"
    ...     some_func()          #This is a function call. So our assumption is 'some_func' must be a function.
    ...

Let's use function **use_hello_world**.

    >>> use_hello_world(hello_world)
    This function takes a function as an argument and uses it
    We will send function hello_world to this function as an argument and will use it in this function
    Hello World                 #This is because of the print stament we have in hello_world()

####What happened here?
1. hello_world is the name of the first function we wrote.
2. We wanted to see how to pass around functions. 
3. We wrote another function named **use_hello_world**. use_hello_world expects one argument. 
4. use_hello_world has three lines in its body.
5. The first two lines of use_hello_world are just print staments.
6. Third line of use_hello_world used the argument that it recieved. Notice the **parenthesis** after **some_func**. So. assumption with use_hello_world is that the argument that will be passed to it will be a function.

####Calling use_hello_world
1. We passed one argument which is function **hello_world** to **use_hello_world**.
2. In use_hello_world, received argument hello_world was assigned to variable named some_func.
3. Two print statements in use_hello_world were printed.
4. some_func which refers to the function pointed to by hello_world was called and the print statement in hello_world was executed.

Also functions can be returned from another function. Let's see it.

    >>> def some_func():
    ...     print "This function defines an inner function"
    ...     def say_hello():
    ...         print "Hello Duniya"
    ...     print "A function can be returned from a function"
    ...     return say_hello
    ... 

####What happens in function **some_func**?
1. First line is a print statement
2. We define an inner function named **say_hello** inside this function.
3. We have a print stament inside say_hello.
4. Fourth line is a print statement of function some_func.
5. say_hello has only one line in its body which is **Hello Duniya**
6. We return function **say_hello**.

Let's see how say_hello can be used.

    >>> abc = some_func()
    This function defines an inner function                             #output
    A function can be returned from a function                          #output

####What happened on calling some_func?
1. As we call **some_func**, two print statements inside it gets executed.
2. However, we do not see third line inside some_func i.e the print statement of **say_hello** in the output. This is because when we call some_func, the inner function inside it is only defined but it is not called.
3. Last line of some_func returns the function defined inside it which is say_hello. Then this say_hello gets assigned to abc. Now abc and say_hello are referring to the same function.

Using abc

    >>> abc()
    Hello Duniya                                        #output

So now we are comfortable with passing functions as arguments as well as defining an inner function inside another function and returning it.

###Writing a simple decorator

Decorators are functions that recieve a function as an argument and return another function as return value.

Let's first write a function without decorator.

    >>> def divide(numerator, denominator):
    ...     return numerator/denominator
    ... 

Let's use this function few times.

    >>> divide(4, 2)
    2                   #output

    >>> divide(4, 0)
    Traceback (most recent call last):
      File "<ipython-input-11-9552c1bdcb43>", line 1, in <module>
        divide(4, 0)
      File "<ipython-input-9-e63f86a52419>", line 2, in divide
        return numerator/denominator
    ZeroDivisionError: integer division or modulo by zero

So, our second run of divide() raised an error.

We can fix it by having an if condition in divide() and making sure that the denominator is not zero. However here we want to understand decorators, so we will fix it using a decorator. We will see a practical use of decorator later.

So, what we intend to do is, if the denominator is 0 then we do not perform the division and just return 0. If the denominator is not 0 then perform the division.

Let's write a decorator. We will go through every line of the decorator once we have defined the decorator, so just go ahead and define the decorator written below.

    >>> def decorate(function_to_be_decorated):
    ...     def another_function(num, den):
    ...         if den==0:
    ...             return 0
    ...         else:
    ...             result = function_to_be_decorated(num, den)
    ...             return result
    ...     return another_function

####Explanation:
1. As we said at beginning, a decorator is just a function which recieves a function as an argument and returns some other function as a return value.
2. A decorator is just a function, so everything applicable to a function is applicable to a decorator. A decorator is a special type of function.
3. We defined a function named **decorate**. It is a decorator as well but we will keep calling it as function for sometime.
4. This function named decorate expects one argument to be passed to it as we can see from the signature of decorate.
5. The argument that we pass to this function decorate will be captured in the variable function_to_be_decorated.
6. The argument passed to decorate must be a function itself. (The assumption for a decorator is that we will pass a function as argument to it). 
7. Since a function will be passed as an argument to decorate, **function_to_be_decorated** will refer to that passed function.
8. We define a function named **another_function** inside decorate.
9. The purpose of writing this decorator is to help us fix function divide().
10. So the signature of **another_function** must match the signature of **divide**. Since divide has two parameters, so another_function must also have two parameters. We have named those parameters num and den.
11. Ignore the body of another_function for now, we will come to it very soon. 
12. At the end we return function named another_function from decorate. So we see that a decorator also returns a function. And it returns the same function that it defined inside itself. So here we defined another_function inside our decorator decorate and returned that same function from the decorator.
13. Read the first point again and verify that everything we said about decorators is true.

Let's use the decorator that we just defined.

    >>> divide = decorate(divide)

####What happened when we wrote divide = decorate(divide)?

#####Only concentrate on left hand side of this assignment for now i.e decorate(divide) part.
1. We passed the function **divide** to decorator named **decorate**.
2. In decorator decorate, function divide gets assigned to variable function_to_be_decorated.
3. So function_to_be_decorated start referring to divide. Let's start calling it old divide now. So, function_to_be_decorated refers to old divide. Old divide has only one statement in it which is "return numerator/denominator". Make sure you understood this statement, we need it later.

#####Talking about right hand side of this assignment.
1. Whatever is returned from decorate is assigned to divide.
2. decorate returns the reference to function named another_function, which gets assigned to divide. Now, divide starts referring to another_function and is no more referring to the version of divide as defined by us initially.
3. But no worries, old/actual version of divide i.e the one which had only one statement "return numerator/denominator" is still being referenced by function_to_be_decorated. So, it is still reachable and usable.

Using new divide.

    >>> divide(4, 2)
    2                                 #output

####Explanation:
1. Now divide refers to another_function. 
2. So, calling divide actually calls another_function.
3. another_function has parameters num and den.
4. Passing the arguments, assigns 4 to num and 2 to den.
5. We check if den equals 0. Here, den did not equal 0. So, we reach else part of another_function.
6. In else part, we use function_to_be_decorated. But, remember function_to_be_decorated refers to older/actual version of divide.
7. The arguments recieved in another_function i.e num and den are passed as it is to function_to_be_decorated i.e actual version of divide.
8. So, 4 and 2 are passed to function_to_be_decorated i.e to old divide. 
9. In old divide 4 gets assigned to variable numerator and 2 gets assigned to variable denominator.
10. And the actual version of divide just divides the numerator by denominator and returns it.
11. We get the value returned from function_to_be_decorated i.e old divide into varaible result and return it from function another_function.

Using new divide again.

    >>> divide(4, 0)
    0                               #output

####Explanation:
1. Passing the arguments, assigns 4 to num and 0 to den.
2. We check if den equals 0. Here if condition evaluates to True and we return 0.
3. So, here function_to_be_decorated i.e old divide was not called after we saw that variable den is 0.

The part which had to be understood is done. If you have understood till here, everything here onwards is as easy as pie.

Though we achieved what we wanted to, we did not use the decorator syntax for changing the behaviour of our original/old divide function.

Let's redefine divide so that it becomes as it was initially.

    >>> def divide(numerator, denominator):
    ...     return numerator/denominator

Pass denominator as 0 to this function to verify that this doesn't work.

    >>> divide(4, 0)
    Traceback (most recent call last):
      File "<ipython-input-19-9552c1bdcb43>", line 1, in <module>
        divide(4, 0)
      File "<ipython-input-18-e63f86a52419>", line 2, in divide
        return numerator/denominator
    ZeroDivisionError: integer division or modulo by zero

Earlier, to fix this thing we used "divide=decorate(divide)". This time we will use the exact decorator syntax.

Redefine divide

    >>> @decorate
    ... def divide(numerator, denominator):
    ...     return numerator/denominator

Yes, you spotted it. We need to put @decorate above the function definition.

Now try dividing by 0.

    >>> divide(4, 0)
    0                                              #output

Voila, we fixed what we intended using decorator syntax.

####What is this @decorate?
1. Interpretor comes across the statement @decorate. So, it knows the function which follows @decorate i.e function named divide needs to be decorated using the decorator named decorate.
2. It reads the function definition on next line i.e function definition for divide and does the usual stuff as it does for any undecorated function. Nothing special happens at this stage.
3. But once the function definition is read, interpreter will do an extra operation. It will execute one more statement after function definition.
4. And the extra statement it executes is **divide=decorate(divide)**. This is the step which we performed manually earlier when we were not using the decorator syntax. But with the decorator syntax, the interpreter takes care of that step for us.

Let's write one more decorator. This decorator needs to check that the denominator is greater than numerator and then only perform the division, otherwise division should not be performed and just return 0. Weird requirement, isn't it?

    >>> def decorate_den_less_than_num(function):
    ...     print "Decorator to check denominator less than numerator applied"
    ...     def check_den_less_than_num(num, den):
    ...         if den>num:
    ...             print "Denominator > Numerator. So dont call the actual function"
    ...             return 0
    ...         else:
    ...             print "Denominator <= Numerator. So call actual function"
    ...             return function(num, den)
    ...     return check_den_less_than_num
    ... 

We wrote the decorator keeping in mind that this decorator has to be applied to a function which expects two arguments. So the function defined inside decorator i.e check_den_less_than_num should also take two arguments and then it should make proper check and based on that it should call the actual function. We have some print statements to see what happens at different stages.

Let's rewrite function divide applying the above written decorator to it.

    >>> @decorate_den_less_than_num
    ... def divide(numerator, denominator):
    ...     return numerator/denominator
    ... 
    Decorator to check denominator less than numerator applied     #output

####Why we got the output "Decorator to che............ applied" ?
1. As we discussed, interpreter saw a decorator syntax above the function divide.
2. It did nothing till the function body is defined.
3. But after function body was defined, interpreter would have executed one more statement which is **divide=decorate_den_less_than_num(divide)**. This would have made a call to decorate_den_less_than_num and so the body of decorate_den_less_than_num was executed and so we saw the print statement inside the decorator.

Make some calls to divide to see how it behaves.

    >>> divide(4, 2)
    Denominator <= Numerator. So call actual function
    2

    >>> divide(4, 5)
    Denominator > Numerator. So dont call the actual function
    0

I wanted this tutorial to be acesible to beginners so I was unable to do justice to this topic.  I plan to write more about decorators. Please let me know if this would help you. Things I want to write about.

1. Chaining decorators
2. Understanding `__call__` and class based decorators
3. Some practical examples of decorator

