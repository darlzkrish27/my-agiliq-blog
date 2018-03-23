---
layout: post
title:  "Travis and coveralls for private repo"
date:   2014-08-22 12:59:46+05:30
tags: coveralls.io
author: manjunath
---
Before we begin, i recommend you to read this first [Continous integration with travis and coveralls.io for Django apps](https://agiliq.com/blog/2014/05/continuous-integration-with-travis-and-coverallsio/).


Here is how `.travis.yml` example file looks like:

    language: python
    python:
      - 2.7
    install:
      - pip install -r requirements.txt
      - pip install coveralls
    script:
      coverage run manage.py test
    after_success:
      coveralls
      

Setting up coveralls for private repositories requires you to add just one more file `.coveralls.yml`.

1) Create a `.coveralls.yml` and make sure it resides in your project's root directory.

2) Add the following to this file:

    service_name: travis-pro
    repo_token: ****
    
`service_name` is to specify where Coveralls should look to find additional information about your builds.
    
You can get the `repo_token` from your repository's page on Coveralls, if you have the admin privileges. This is to tell which project on Coveralls your project maps to.

Make sure your `repo_token` remains secret and do not add this to your public repository.

3) Add the file, commit it and make a git push.

4) If everything is OK you should see some thing like the below in your travis build:

    Submitting coverage to coveralls.io...
    Coverage submitted!
    Job #22.1
    https://coveralls.io/jobs/54864565
    
Thats it now get a coverage badge from coveralls and add this badge in your repo's README.md.


