---
layout: post
title:  "Deploying Django apps on Heroku"
date:   2012-02-22 02:06:10+05:30
tags: django
author: shabda
---
Deploying Django apps on Heroku:

Read this first: http://devcenter.heroku.com/articles/django.

This is a great article by the Heroku. I am just filling in some more details and making this step-by-step.

1. Get your Django project code.
2. Create a virtualenv with a no-site-packages command ::

    virtualenv vent --no-site-packages
    
3. Install Django, psycopg2 (postgres connector), gunicorn and any other required Django libraries.
4. Confirm that you have all the required libraries and you can run your code locally using ``manage.py runserver``.
5. Create a requirement.txt by using 
    ::
        
        pip freeze > requirements.txt
        
6. Make sure you have a requirements.txt at the root of your repo. Heroku uses this to identify that the app is a Python app.
7. Create a Procfile. Put this entry: ::

    web: python mysite/manage.py run_gunicorn -b "0.0.0.0:$PORT" -w 3shabda
    
8. This is what your directory structure will look like

    ::

        [root-of-heroku-folder]
            requirements.txt
            Procfile
            [Django-project-folder]
                __init__.py
                manage.py
                settings.py
                [app1]
                [app2]


9. (If you aren't tracking your files with git yet.) Track your files with git. ::

    git init
    git add .
    git commit -am "First commit"

10. Make sure you have the heroku toolbelt installed. If not go to http://toolbelt.herokuapp.com/ and install.

11.You should have these commands available now: ::

    heroku
    foreman

12. Authenticate to heroku with 

    ::

        heroku auth:login
    

13. Run this command. 
    ::

        heroku create --stack cedar    
    
    
This will create a new Heroku app and create a new remote in your git repo.

14. Push your code to heroku. 
    ::

        git push heroku master
    
    
15. Your app should be working on Heroku now. ``heroku open`` will show your site.


    
    

