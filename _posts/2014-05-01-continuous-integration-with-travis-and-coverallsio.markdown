---
layout: post
title:  "Continuous integration with travis and coveralls.io for Django apps"
date:   2014-05-01 16:26:46+05:30
tags: travis
author: Rakesh
---

Maintaining a solid rapport with the ongoing software development process always turns out to be a walk on air.
Ensuring a software build integrity and quality in every single commit makes it much more exciting.

If the current software bulid is constantly available for testing, demo or release isn't it a developer's paradise on earth?

Giving a cold shoulder to "Integration hell" the 'Continuous integration' process stands out to deliver all the above assets.

What's Continuous integration?
-------------------------------------------------

Continuous integration is a development practice that requires a developer to integrate code into a shared repository in which isolated changes are immediately tested and reported on when they are added to a larger code base.

Continuous integration (CI) originated from within the extreme programming paradigm, but the principles can be applied to any iterative programming model, such as agile programming. As critics noted several potential drawbacks on extreme programming many organizations have adopted CI without all of the extreme programming concepts.

Principles of CI:
------------------------

- Maintain a single source repository
- Automate the build
- Make the build self-testing
- Keep the build fast
- Everyone can see the results of the latest build
- Automate deployment

Keywords:
-----------------------

1. Build: All steps necessary to compile and create deliverables.
2. Commit: The operation follows the validation of updates to existing code.
3. Update: This operation allows the update from the repository of the configuration management tool.
4. Checkout: This is the operation to extract a version of a project under development from the repository of the configuration manager.



Spices for the recipe:
--------------------------

- Use of a version control tool (Git, SVN, etc.)
- An automated build and product release process.
- Instrumentation of the build process to trigger unit and acceptance tests for evey change that is commited
- A notifying process that notifies for every single test fail and alerts the team.


The approach:
----------------
1. The developer makes a commit to the repository
2. The integration server detects the commit, makes a checkout, launches operations and testing.
3. Eventually the repository may become so different from the developers baseline referred to as "Merge hell"
4. In case of a failure in the testing process a notification is generated to the team.
5. The developers may discard the changes or fix the bugs to make it progress successfully.

So this makes us fancy trying out continuous integration. Let's study a CI software now.

Travis CI
------------------------

Travis CI is a distributed build system for the open source community. Its a CI service used to build and test projects hosted on Github.

Travis is configured by adding a file named .travis.yml which is a YAML(a human-readable data serialization format) text file.
It automatically detects when a commit has been made and pushed to a GitHub repository that is using Travis CI, and each time this happens, it will try to build the project and run tests. It also builds and runs pull requests and once it is completed it notifies the developer in the way it is configured.

Steps to use Travis CI:
--------------------------------
1. Sign-in: To get started with Travis CI we should sign-in to our github account.
2. Activate Github webhook: Once the Signup process gets completed we need to enable the service hook in the github profile page.
3. Add .travis.yml: We should add the yml file to the project.

Writing .travis.yml file:
----------------------------

In order for Travis CI to build our project we need to tell the system a little bit about it. we will be needed to add a file named .travis.yml to the root of our repository.
The basic options in the .travis.yml should contain are language key which tells which language environment to select for our project and other options include the version of the language and scripts to run the tests, etc.


Example for travis.yml for python project:

    language: python
    python:
      - "2.7"

    # command to install dependencies
    install:
      - "pip install -r requirements.txt"

    # command to run tests
    script: python manage.py test

Explanation:

In the first line we have specified python as the language environment, the next specifies the version of python. Travis will then make sure that the project runs well with all the python versions. If we have any python dependencies Travis will test our code in a virtualenv, so all we need to write a requirments.txt and tell Travis to install it and the script command will run tests when excited.

The complete Lifecycle
----------------------

1. `before_install`
2. `install`
3. `after_install`
4. `before_script`
5. `script`
6. `after_script`
7. `after_success or after_failure`


Configuring .travis.yml
-------------------------------

Build only specific branches:

Per default travis will build once we push to a branch on Github. That behavior can be annoying if we are committing often to development branches. We can restrict which branches Travis is monitoring for changes in the .travis.yml. Either we can blacklist the branches or whitelist them:

.travis.yml

    # Whitelisting example
    branches:
      only:
        - master

    # Blacklisting example
    branches:
      except:
        - develop
        - feature


Getting notified with lint errors:

We can also get notified or make bulid fail if there are any lint errors in the commit. For this we can use the option before-script.
We can create a lint.sh file and we can specify it in the before-script option so that the bulid fails and notifies when lint error occurs.

.travis.yml

    before_script:
        -./lint.sh


Using coveralls with travis CI:
---------------------------------------

Coveralls is web service to help us track our code coverage over time and ensure that all our new code is fully covered.

Steps to use coveralls:

1. First, log in via Github and add your repo on Coveralls website.

2. Add pip install coveralls to install section of .travis.yml

3. Make sure you run your tests with coverage during the build in script part.

    script:
        coverage run --source=yourpackagename setup.py test

4. Execute run `coveralls` in `after_success` section.

    after_success:
        coveralls


Example:

travis.yml

    language: python
    python:
      - 2.7
    install:
      - pip install -r requirements.txt
      - pip install coveralls
    script:
      coverage run --source=moscowdjango,meetup manage.py test
    after_success:
      coveralls

The above code makes custom report for data generated by coverage.py package and sends it to json API of coveralls.io service. All python files in our coverage analysis are posted to this service along with coverage stats

tats

