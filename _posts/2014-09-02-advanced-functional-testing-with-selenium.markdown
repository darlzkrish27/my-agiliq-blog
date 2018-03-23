---
layout: post
title:  "Advanced functional testing with Selenium in Django"
date:   2014-09-02 15:17:48+05:30
tags: functional-testing
author: karambir
---
In the [previous post](http://agiliq.com/blog/2014/09/selenium-testing/), we introduced Selenium and how it can be used to write functional tests in Django Projects. In this post, we will look into some more advanced features of selenium, handling javascript heavy sites and integrate our functional tests with travis, saucelabs and browserstack.

Advanced features of Selenium
-----------------------------

### Back and Forward
We used a simple "get" method to retrieve a webpage. Selenium also lets us navigate back and forth in browser's history. This is just like clicking on "Back" and "Forward" button in a real browser.

    driver.get("http://www.python.org")
    driver.back()
    driver.forward()

### Setting Cookies
Using Selenium, you can also set cookies for domains. This is useful in many scenarios when you are testing your local website.

    # Load a webpage
    driver.get("http://localhost:8000")

    # Set the cookie
    cookie = {"name": "value"}
    driver.add_cookie(cookie)

### Switching between windows and frames
Selenium provides methods like "switch_to_window" and "switch_to_frame" which can be used to run commands on other windows and frames.

    driver.switch_to_frame("frame_name")
    # Now all subsequent commands will be directed to this frame

    # If frame does not have name, we can also use simple locators to locate that frame
    driver.switch_to_frame(driver.find_elements_by_tag_name("iframe")[0])

    # To switch window for following link:
    <a href="/login" target="loginwindow">Login here</a>

    driver.switch_to_window("loginwindow")

### Action Chains
When we want to perform a click operation on any element, selenium checks if that element is visible. This is to conform with the fact that a normal user can only click a visible element in a web browser. But sometimes you have to hover on some element to get the element to be clicked to appear(like [this navigation menu](http://jsfiddle.net/akarambir/wq3gg1ed/2/)). To perform these type of operations, Selenium provides a class called ActionChains. Here is an example code to click on a submenu on that navigation menu:

    # Showing just the essential part of code.
    from selenium import webdriver
    from selenium.webdriver import ActionChains

    driver = webdriver.Firefox()

    # get the page...

    action_chains = ActionChains(driver)
    # Locate main menu to be hovered
    main_menu = driver.find_element_by_id("drop1")
    # Locate sub menu to be clicked
    sub_menu = dirver.find_element_by_id("submenu1")

    # Action Chains stores all the actions until perform method is called
    # Move mouse pointer to main_menu. This will do hover.
    action_chains.move_to_element(main_menu)
    # Click on the sub-menu
    action_chains.click(sub_menu)
    # Perform all the commands
    action_chains.perform()

Likewise, ActionChains can be used to emulate drag and drop behaviour. See [docs](http://selenium.googlecode.com/svn/trunk/docs/api/py/webdriver/selenium.webdriver.common.action_chains.html)

**More information about these methods can be found [here](http://selenium.googlecode.com/svn/trunk/docs/api/py/webdriver_remote/selenium.webdriver.remote.webdriver.html?).**

### Waiting for Javascript processes
Selenium handles waiting for page load and redirects when we click a link. But if a website is using ajax calls to fill data on webpage, we have to wait for the data to be filled before proceeding. For this, selenium provides two types of waits - implicit and explicit:

#### Implcit Waits
This tells the selenium webdriver to wait for a certain amount of time while finding element(s). The default value is 0. So webdriver will raise an exception if does not immediately find the element. Here is an example:

    driver = webdriver.Firefox()
    driver.implicitly_wait(6) # value in seconds
    # Now all find commands will try for the elements for 6 seconds before raising an exception
    driver.get("http://localhost:8000/")
    driver.find_element_by_id("sidebar_menu")
    # It will find element even if this element is loaded by javascript dynamically

#### Explicit Waits
We can define our code to wait for some condition to pass before proceeding further. Webdriver will keep checking that condition(default is after every 0.5sec) until it satisfies.

    from selenium import webdriver
    from selenium.webdriver.support.ui import WebDriverWait
    from selenium.webdriver.support import expected_conditions as EC
    from selenium.webdriver.common.by import By

    driver = webdriver.Firefox()
    driver.get("http://localhost:8000/")

    try:
        element = WebDriverWait(driver, 6).until(
            EC.presence_of_element_located(By.ID, "sidebar_menu"))
        )
    finally:
        print element # This will either print element object or None

    driver.quit()

This code tries locating the sidebar_menu for 6 sec and then throw TimeoutException if cannot be found. *EC* or *Expected Conditions* have some common methods we can use while testing our applications. Some of them are:

+ title_contains
+ presence_of_element_located
+ visibility_of_element_located - An element may be present in DOM but not visible
+ invisibility_of_element_located
+ text_to_be_present_in_element

See all the conditions [here](http://selenium.googlecode.com/svn/trunk/docs/api/py/webdriver_support/selenium.webdriver.support.expected_conditions.html).

**Some of the frequently asked questions about Selenium-Python are listed [here](http://selenium-python.readthedocs.org/faq.html).**

Functional testing on Travis
----------------------------

When running tests on Travis, we can use [phantomjs](http://phantomjs.org/), firefox with [xvfb](http://www.xfree86.org/4.0.1/Xvfb.1.html) or external services(saucelabs and browserstack) to run a browser for our tests.

### Firefox
Add following in your *travis.yml* file:

    before_install:
        - "export DISPLAY=:99.0"
        - "sh -e /etc/init.d/xvfb start"
        - sleep 3 # give xvfb some time to start

Firefox is installed by default in travis. This code will use xvfb to run firefox for us.

### PhantonJS
PhantomJS is a headless WebKit with JavaScript API. It is an optimal solution for fast headless testing, site scraping, pages capture, SVG renderer, network monitoring and many other use cases. PhantomJs is pre-installed in travis, so we just have to change our webdriver in code:

    from selenium import webdriver
    driver = webdriver.PhantonJS()

### Sauce Labs
Sauce Labs provides a Selenium Cloud with variety or OS/browser combinations. Sign up for the service [here](https://saucelabs.com/) and add following to *travis.yml*

    addons:
    sauce_connect:
        username: "Your Sauce Labs username"
        access_key: "Your Sauce Labs access key"

And in our tests file we have to change our browser to a remote one in setUp:

    username = os.environ["SAUCE_USERNAME"]
    access_key = os.environ["SAUCE_ACCESS_KEY"]
    capabilities["tunnel-identifier"] = os.environ["TRAVIS_JOB_NUMBER"]
    hub_url = "%s:%s@localhost:4445" % (username, access_key)
    driver = webdriver.Remote(desired_capabilities=capabilities, command_executor="http://%s/wd/hub" % hub_url)

Here we are using [Remote WebDriver](https://code.google.com/p/selenium/wiki/RemoteWebDriver) instead of Firefox or PhantonJS.

### BrowserStack
Similar to Sauce Labs in offerings, we can use this service instead of Sauce Labs to run our functional tests. Though it does not have addon similar to sauce, we can use [this script](https://gist.github.com/akarambir/46e3e56e2339070ace5f) to start a connection. In our *travis.yml*:

    before_script:
        - ./browserstack.sh
    env:
        global:
            - BS_USERNAME = "Your Broserstack username"
            - BS_ACCESS_KEY = "Your Browserstack access key"

And in our tests file, change our code to this:

    # before setUp
    browser = {
        'os': 'Windows',
        'os_version': '8.1',
        'browser': 'Firefox',
        'browser_version': '32.0',
        'resolution': '1920x1080'
    }
    USERNAME = os.environ.get('BS_USERNAME')
    ACCESS_KEY = os.environ.get('BS_ACCESS_KEY')

    # in setUp
    self.desired_capabilities = browser
    self.desired_capabilities['build'] = 'local'
    self.desired_capabilities['browserstack.local'] = True
    self.desired_capabilities['browserstack.debug'] = True
    url = 'http://%s:%s@hub.browserstack.com:80/wd/hub'
    self.selenium = webdriver.Remote(
        desired_capabilities=self.desired_capabilities,
        command_executor=url % (USERNAME, ACCESS_KEY)
    )

**Note**

The main advantage in using these services is that they collect debug information about selenium commands and save screenshots/video of the tests which comes in handy when you run into bug/errors.


