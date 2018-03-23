---
layout: post
title:  "Introduction to functional testing with Selenium in Django"
date:   2014-09-01 15:40:00+05:30
tags: functional-testing
author: karambir
---
In this tutorial we will learn what selenium is and how it can be used in writing functional tests for our Django projects.

What is Selenium?
----------------
Selenium is a Web Browser automation tool, which basically means you can use it to surf web programmatically on real web browsers. But mostly selenium is used for testing web applications. It has packages for most programming languages. For python, we can try selenium by installing its package:

    pip install selenium

For a quick demo, we will open Django Documentation website, perform some search and check the first result. Save and run the following script:

    from selenium import webdriver
    from selenium.webdriver.common.keys import Keys

    driver = webdriver.Firefox()
    driver.maximize_window()
    driver.get("https://docs.djangoproject.com/")
    search_box = driver.find_element_by_name("q")
    search_box.send_keys("testing")
    search_box.send_keys(Keys.RETURN)
    assert "Search" in driver.title
    # Locate first result in page using css selectors.
    result = driver.find_element_by_css_selector("div#search-results a")
    result.click()
    assert "testing" in driver.title.lower()
    driver.quit()

Most of the code is self explanatory. It checks for the search box on the page, enter the search term and hit return key. Then it clicks on the first result and do an assert. *Keys* class is used for keyboard simulation and can also be used to enter special keys(we used RETURN key in example to submit form).

Selenium provide many methods to locate an element on a page:

+ find_element_by_id
+ find_element_by_name
+ find_element_by_tag_name
+ find_element_by_xpath
+ find_element_by_link_text
+ find_element_by_partial_link_text
+ find_element_by_class_name
+ find_element_by_css_selector

If multiple matches are found for a query then it returns the first match(note how we used css selector to find first result in above example). You can also find multiple elements with a given logic by similar methods. For example to locate all the links on the page you can do:

    all_links = driver.find_elements_by_tag_name("a") # note the 's' in find_elements

Each element is an object of WebElement class and have many methods associated with it. Most commonly used are:

+ click - perform a mouse click on element
+ text - gets the text of the element
+ location - Returns the location of the element in the renderable canvas
+ is_displayed - Whether the element would be visible to a user
+ get_attribute(name) - Gets the attribute value.

There are several more. You can read about them at the [Selenium Python docs](http://selenium.googlecode.com/svn/trunk/docs/api/py/webdriver_remote/selenium.webdriver.remote.webelement.html).

Testing Django Web application
---------------------

We will setup a minimal Django project with just admin activated:

    pip install Django==1.6.6 selenium
    django-admin.py startproject myproject
    cd myproject
    python manage.py syncdb --noinput

###Note:
*Django 1.6 onwards, admin is activated in urls.py. If you are using django<1.6, you should add the admin urls.*

Now we will write a test to create a user in Django admin. Add a file *test.py* with following code:

    from django.test import LiveServerTestCase
    from django.contrib.auth.models import User

    from selenium import webdriver

    class AdminTestCase(LiveServerTestCase):
        def setUp(self):
            # setUp is where you instantiate the selenium webdriver and loads the browser.
            User.objects.create_superuser(
                username='admin',
                password='admin',
                email='admin@example.com'
            )

            self.selenium = webdriver.Firefox()
            self.selenium.maximize_window()
            super(AdminTestCase, self).setUp()

        def tearDown(self):
            # Call tearDown to close the web browser
            self.selenium.quit()
            super(AdminTestCase, self).tearDown()

        def test_create_user(self):
            """
            Django admin create user test
            This test will create a user in django admin and assert that
            page is redirected to the new user change form.
            """
            # Open the django admin page.
            # DjangoLiveServerTestCase provides a live server url attribute
            # to access the base url in tests
            self.selenium.get(
                '%s%s' % (self.live_server_url,  "/admin/")
            )

            # Fill login information of admin
            username = self.selenium.find_element_by_id("id_username")
            username.send_keys("admin")
            password = self.selenium.find_element_by_id("id_password")
            password.send_keys("admin")

            # Locate Login button and click it
            self.selenium.find_element_by_xpath('//input[@value="Log in"]').click()
            self.selenium.get(
                '%s%s' % (self.live_server_url, "/admin/auth/user/add/")
            )

            # Fill the create user form with username and password
            self.selenium.find_element_by_id("id_username").send_keys("test")
            self.selenium.find_element_by_id("id_password1").send_keys("test")
            self.selenium.find_element_by_id("id_password2").send_keys("test")

            # Forms can be submitted directly by calling its method submit
            self.selenium.find_element_by_id("user_form").submit()
            self.assertIn("Change user", self.selenium.title)

Django LiveServerTestCase is extended from TransactionTestCase class. It launches a Django server(on port 8081) in the background on setup and shuts it down on teardown. Note in the code above we are instantiating the selenium webdriver in the setUp method and call quit on it in tearDown. We can get the server url from live_server_url attribute provided by TestCase.

Now just run the test command:

    python manage.py test

You will see a firefox browser opens and run the test. This is a very simple example but you can use selenium to check every major functionality of your website.

We will cover some advanced functionality like testing javascript heavy site and running tests on travis, [Sauce](https://saucelabs.com), [BrowserStack](https://www.browserstack.com) in our [next post](http://agiliq.com/blog/2014/09/advanced-functional-testing-with-selenium/). Meanwhile read more about selenium at the Python selenium docs [here](http://selenium-python.readthedocs.org/index.html)(unofficial but good for beginners) and [here](http://selenium.googlecode.com/svn/trunk/docs/api/py/index.html)(official). Happy Testing


