---
layout: post
title:  "Test Driven Development in Python "
date:   2012-05-02 20:41:12+05:30
tags: Development
author: anoop
---
> We can either prevent bad things from
> happening or fix it, once it is
> detected.

It is your choice to select any of these methodology, while developing a software. You can either develop based on a test driven process or the recover from a fiasco with tests.

Test driven development, as the name suggests, is development based on tests. Tests for core features are written prior to the implementation for the expected output, and then necessary modules are written to satisfy the needs define the 


----------



**Advantages of Test Driven Development**

 - application is determined by using it
 - written minimal amount of application code
 - total application + tests is probably more smaller
 - simpler, stand-alone, minimal dependencies
 - tends to result in extensible architectures
 - instant feedback
 - future development won't break existing features.
 


----------



**About Test Driven Development**

 - Write tests for the use case Run it
 - (make sure it fails and fails
 - miserably) Write code and implement
 - the required functionality with relevant level of detail 
 - Run the test
 - Write test for addition features 
 - Run all test Watch it succeed. Have a cup of coffee !



**Basic Unittest**


----------



    import unittest
    class MyTest(unittest.TestCase):
        def testMethod(self):
            self.assertEqual(hello(5), 5, "Hello din't return 5.")

    if __name__ == '__main__':
        unittest.main()

To write a basic unittest in python, is pretty straight forward. The code above is self explainatory. It is trying out hello function, and expected output is given as 5. If somehow, hello() din't return 5, then the error message is printed to the console.


Now, let's try writing the function hello to satisfy the test.

    def hello(value):
        return 5

or

    def hello(value):
        return value

**This is a problem!**

The problem is, how to select the best. Solution is, to get the tests more detailed and covering more test cases to satisfy the requiements of the function. As we have more precise requirements in the tests, we can easily rule out either the more complex solutions or the more simpler solutions, and get the perfect balance between development and requirement.


One thing to keep in mind while doing Test Driven Development is, ***Don't Overkill***. Just write tests for the core features expected, and write it expecting all the complete range. For example, if it is a multiplication function, make sure that you need make sure that you handle strings as you need it.

Another Example

An example for doing test driven development for greater than function is given below.

The requirement is crazy, to compare the *ascii sum of the text string* if argument is a string.

**The Test**

    import unittest
    from demo import Greater
    class DemoTest(unittest.TestCase):
        def test_number(self):
            comparator = Greater()
            result = comparator.greater(10,5)
            self.assertTrue(result)
        def test_char(self):
            comparator = Greater()
            result = comparator.greater('ABCabcxyz', 'ABa')
            self.assertTrue(result)
        def test_char_equal(self):
            comparator = Greater()
            result = comparator.greater('4', 3)
            self.assertTrue(result)
    if __name__ == '__main__':
        unittest.main()


**Now the Code**


    class Greater(object):
        def greater(self, val1, val2):
            if type(val1) ==str or type(val2) == str:
               val1 = str(val1)
               val2 = str(val2)
               sum1 = sum([ord(i) for i in val1])
               sum2 = sum([ord(i) for i in val2])
               if sum1 > sum2:
                   return True
               else:
                   return False
            if val1>val2:
                return True
            else:
                return False

The function returns True or False, based on the ascii values if any of the argument is  string, else give the greater.


Note: A presentation on the topic is available [http://www.slideshare.net/atmb4u/test-driven-development-in-python][1]

> **TL; DR:  Write tests to satisfy requirements, then write code. Have a peaceful life !**

  [1]: http://www.slideshare.net/atmb4u/test-driven-development-in-python

Have a happy Test Driven Development!!!

