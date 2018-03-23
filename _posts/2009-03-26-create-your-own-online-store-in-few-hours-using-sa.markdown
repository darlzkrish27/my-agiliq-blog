---
layout: post
title:  "Create your own online store in few hours using satchmo (django)"
date:   2009-03-26 09:58:11+05:30
tags: satchmo
author: rama
---
## What is satcmo?
Satchmo is a django based framework which allows  you to  create an online store very quickly.
Satchmo also offers powerful customizations which allows you to tune a store to your specific business needs.

Sample Demo Store : [http://www.xfgw.info/store](http://www.xfgw.info/store)
#### Login details
1. Use below mail and password to login into the site:  <br />
  Email : test@test.com
  password : welcome
2. Use dummy credit card number like 4222-2222-2222-2222 (VISA) to make any purchases.

Above is the sample demo store which is being provided by satchmo outofbox just after installing it.

In this blog post we will discuss very basic things about satchmo like how to install satchmo? , and how to do various customizations ? etc......

## Installing satchmo

Please note below 

1. Installation instructions are applicable to the latest version of satchmo  which is available in the trunk.
2. We assume that python setuptools has already been installed on your system.
3. You must have latest version of Django 1.1 beta.

Just follow the below 5 steps to install various dependencies of satchmo.

    1) easy_install pycrypto
    2) easy_install http://code.enthought.com/enstaller/eggs/rhel/3/Reportlab-2.1.0001-py2.5.egg
    3) easy_install django-registration
    4) easy_install http://tinyforge.org/frs/download.php/1/trml2pdf.tar.gz
    5) easy_install PyYAML


#### install django-threaded-multihost
Go to  : http://banjo.assembla.com/spaces/threaded_multihost/documents. Download latest .tar.gz file and follow below steps

    8  tar xvzf django-threaded-multihost-1.3-0.tar.gz  
    9) cd django-threaded-multihost-1.3-0 
    10) python setup.py install

if you get any error as below

    Traceback (most recent call last):
    File "setup.py", line 1, in <module>
    import ez_setup
    ImportError: No module named ez_setup

#### Fix the above error

Open setup.py and comment out  the first two lines as shown below and repeat the step 10.

    # ez_setup.use_setuptools()
    # from setuptools import setup, find_packages

Next install django-app-plugins app by following below steps.

    11) svn checkout http://django-app-plugins.googlecode.com/svn/trunk/ django-app-plugins-read-only
    12) cd /path/to/your/site-packages/dir (usually it is /usr/lib/python2.X/site-packages/)
    13) ln -s /path/to/django-app-plugins-read-only/app_plugins  . 

12) Installing an app means placing an app on  python path so that django can find it 
#### How do we do it?
13) By convention any directory which is in  /usr/lib/python2.X/site-packages/ is on python path.Because of this we are creating a symlink from the sitepackges directory to the directory where we checkedout the latest version of django-app-plugins.Symlinking is more efficient that copying entire directory under sitepackages.

Install Sorl thumbnail app for generating thumbnails from images

    14) svn checkout http://sorl-thumbnail.googlecode.com/svn/trunk/sorl/
    15) cd /path/to/your/site-packages/dir
    16) ln -s /path/to/sorl  .

14,15,16  We followed similar approach as that of above  to install sorl-thumbnail application.

Optional dependencies 

    17) easy_install elementtree
    18) easy_install docutils


The above are all dependencies which are required for the satchmo.

#### Now we will see how to create our own custom store using satchmo

create new django project 

    19) django-admin.py startproject customstore

check out the latest version of satchmo

    20)  svn co svn://satchmoproject.com/satchmo/trunk satchmo-trunk

Install satchmo

    21) cd satchmo-trunk
    21) sudo python setup.py install

Make sure everything is working fine.

    >>> import django
    >>> django.VERSION
    (1, 1, 0, 'alpha', 1)
    >>> import satchmo_store
    >>> satchmo_store.VERSION
    (0, 9, 'pre')

Go to your project

    cd customstore

copy settings.py and localsetting.py from the sample project which is provided along with satchmo.

    cp  ......../satchmo-trunk/satchmo/projects/base/settings.py  ........../customstore/settings.py
    cp  ......../satchmo-trunk/satchmo/projects/base/local_settings.py  ........../customstore/local_settings.py  

#### Edit customstore/settings.py
1. Change the Django_project name from 'satchmo'  to 'satchmo_store'.In previous versions project is called as "satchmo" but now it being called as "satchmo_store"
    
2. In installed apps if you find an app called "satchmo" rename it to "satchmo_store" .

#### Edit customstore/local_settings.py
1.Uncomment DATEBASE_NAME,DATABASE_PASSWORD,DATABASE_USER and add corresponding details of your database

    DATABASE_NAME = 'satchmostore' #Name of the database which you create.Please refer to an additional note for mysql users at the end
    DATABASE_PASSWORD = 'welcome'
    DATABASE_USER = 'operations' 

The characters which are being transferred  from database server to client should be in UTF-8.if your database is MYSQL you need to add
the following in local_settings.py.

    DATABASE_OPTIONS = { "init_command": 'SET NAMES "utf8"' ,  "init_command":'SET storage_engine=INNODB' , } 

The above command also sets INNODB as storage engine for database tables. we need to use INNODB instead of ISAM to support transactions.
Uncomment SECRET_KEY and provide some value

    SECRET_KEY = 'asdf'

if you are using MYSQL  please note that you need to create database in UTF-8 character set as follows
 
     create database satchmostore  CHARACTER SET UTF-8 

#### Edit urls.py
Copy paste the below code in the urls.py

    from django.conf.urls.defaults import *
    from django.conf.urls.defaults import *
    from satchmo_store.urls import urlpatterns 

if you have any other addtional urls you need to add at the end of urls.py as 

    urlpatterns += patterns('',.......)

Copy over the static content

    22) python manage.py satchmo_copy_static

Check whether there are any errors in the configuration using.if there are any errors you need to resolve them before proceeding further.

    23) python manage.py satchmo_check
    Checking your satchmo configuration.
    Using Django version 1.1 alpha 1
    Using Satchmo version 0.9-pre-SVN-unknown
    Your configuration has no errors.

Syncdb and load intial data

    24) python manage.py syncdb
    25) python manage.py satchmo_load_l10n
    26) python manage.py satchmo_load_store
    27) python manage.py satchmo_load_us_tax

Runserver

    28) python manage.py runserver

Your store is ready.

29) [http://localhost:8000/store](http://localhost:8000/store)

when you enter above URL you will be seeing a store and all the functionality which is being provided by satchmo out of box.

You can delete all the data which is provided by satchmo by running the below command

    30) python manage.py delete_all_dbs

You can see all the intial settings and configuration details at 

31) [http://localhost:8000/settings](http://localhost:8000/settings)

YOu can see the admin interface by entering the below url

32) [http://localhost:8000/admin](http://localhost:8000/admin)

##Satchmo Customization:
Satchmo has powerfull customization mechanism using which you can tune the store to suit your business needs.The below are basic customizations which you can 
do.Please refer to documentation for more complex customizations and features.

####To add new products and category etc
You need to go to admin interface to add new categories,products,pricing,discounts,option groups,images etc.....

More information about this details at:
[http://www.satchmoproject.com/docs/svn/product.html](http://www.satchmoproject.com/docs/svn/product.html)

####Selling different product types
Using satchmo you can also sell products such as Downloadable Product where customer downloads data/other information after paying amount,
you can also sell subscription product for which customers have to pay every month.

####Custom product types
YOu can also create custom product types by following the below approach.
YOu need to create a model which participates in one-to-one relation ship with the default product model as below

    from django.db import models
    from django.utils.translation import ugettext_lazy as _

    from product.models import Product

    class MyNewProduct(models.Model):
        product = models.OneToOneField(Product, verbose_name=_('Product'),
        primary_key=True)

    def _get_subtype(self):
        return 'MyNewProduct'

    def __unicode__(self):
        return u"MyNewProduct: %s" % self.product.name

    class Admin:
        pass

    class Meta:
        verbose_name = _('My New Product')
        verbose_name_plural = _('My New Products')

1. Thereafter you need tell satchmo about this product type in config.py,add to admin.py to make it visible via admin etc..More details you can find in the below link:
 [http://www.satchmoproject.com/docs/svn/custom-product.html](http://www.satchmoproject.com/docs/svn/custom-product.html)

In addition to above you can do various other customizations like
 CSS,Template,URL,Checkout process,Shipping module  customization,Payment module.

More details at satchmo documentation [http://www.satchmoproject.com/docs/svn/index.html](http://www.satchmoproject.com/docs/svn/index.html)

And important details related to above customizations in the next blog post.        

Stay Tuned.









  









 










      
    
     



 







    





     



 


  

















   








     






