---
layout: post
title:  "Heroku Django S3 for serving Media files"
date:   2014-06-05 08:17:35+05:30
tags: heroku
author: manjunath
---
### Why should i store my media files on AWS S3?

Heroku dynos have limited life span, and when they die and get replaced, the files within them are lost. So you should set up Django media handler to put the files somewhere permanent.

### Let's begin:

1) Install dependencies:

    $ pip install django-storages boto
    
(Update your <code>requirements.txt using `pip freeze > requirements.txt`)

2) Update `INSTALLED_APPS` in your settings.py:

    INSTALLED_APPS += ('storages',)
    
3) Sign in to [AWS Console](http://aws.amazon.com/console/), select `S3` under `Storage & Content Delivery` and create a bucket.

4) Don't forget to generate `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

5) Add the below to your settings file:

    AWS_QUERYSTRING_AUTH = False
    AWS_ACCESS_KEY_ID = os.environ['AWS_ACCESS_KEY_ID']
    AWS_SECRET_ACCESS_KEY = os.environ['AWS_SECRET_ACCESS_KEY']
    AWS_STORAGE_BUCKET_NAME = os.environ['S3_BUCKET_NAME']
    MEDIA_URL = 'http://%s.s3.amazonaws.com/your-folder/' % AWS_STORAGE_BUCKET_NAME
    DEFAULT_FILE_STORAGE = "storages.backends.s3boto.S3BotoStorage"
    
Note: You should never expose your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` there is some reason why they are called SECRET.

6) Install [AWS CLI](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-set-up.html) on your machine.

7) Its time to move your files to AWS S3 bucket.

    $ aws s3 cp myfolder s3://mybucket/myfolder --recursive
    
8) After the previous step you can use the following command to view the contents of your bucket.

    $ aws s3 ls s3://mybucket
    
9) Grant public-read permission to everyone, add the below to your S3 bucket policy.

    {
      "Version":"2008-10-17",
      "Statement":[{
        "Sid":"AllowPublicRead",
            "Effect":"Allow",
          "Principal": {
                "AWS": "*"
             },
          "Action":["s3:GetObject"],
          "Resource":["arn:aws:s3:::bucket/*"
          ]
        }
      ]
    }
    
You can also grant full access to specific IP addresses check this [Bucket Policies](http://s3browser.com/working-with-amazon-s3-bucket-policies.php)

Thats it now your django app should be able to serve your media files.If not please check your bucket policy, you can check the url link under S3 Management Console->Properties.

Any problem or feedback? feel free to leave your comments below.

