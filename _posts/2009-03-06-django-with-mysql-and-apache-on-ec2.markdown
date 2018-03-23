---
layout: post
title:  "Django with Mysql and Apache on EC2"
date:   2009-03-06 13:48:17+05:30
tags: mysql
author: shabda
---
What is EC2
----------------

Unless you have been living on Mars these last few years, you are sure to have heard of [EC2](http://aws.amazon.com/ec2/). Amazon's cloud offering,
it offers infinite scalability. Using EC2, you can bring up any number of machines online at minutes notice, and after
you are done with them, bring them down. 

How does EC2 work?
-----------------------

A EC2 machine is nothing but a bare machine. An [Amazon Machine Image](http://aws.amazon.com/ec2/instance-types/) (AMI) is a machine bundled as an Image, with preconfigured software which you can 
start at moments notice. We will take a AMI with a basic Ubuntu installed, and install Django with Mysql and Apache there.

The prerequisites
----------------------
You need to have an [AWS account](http://aws.amazon.com/) with EC2 enabled. We don't like to use not standard tools from the shell, right? So we will install
[ElasticFox](http://sourceforge.net/projects/elasticfox/), a Firefox plugin to enable using AWS services from within Firefox.

Give ElasticFox your EC2 credentials.  Launch ElasticFox and lunch an AMI. This is covered in extreme detail in [ElasticFox owners manual](http://ec2-downloads.s3.amazonaws.com/elasticfox-owners-manual.pdf)
so we will not cover that here.

I started the AMI with AMI id ami-f27c999b. This is a 64 bit Ubuntu Gutsy AMI.

After you start your ami, right click in ElasticFox on your instance and get its Public DNS. This will allow you to ssh to the instance. But you don't have the root password for it, eh? When you launched your AMI, you used a keypair, right? So you can ssh to your server using your private key.

My public dns was ec2-75-101-203-97.compute-1.amazonaws.com, my private key is stored in a file called id-django. So i login as,

	shabda@shabda-laptop:~$ ssh -i id-django root@ec2-75-101-203-97.compute-1.amazonaws.com


	..........

	root@domU-12-31-39-02-BC-E1:~# 

Fine, we are in our brand new EC2 server now! Ok, we do not want to work as root, create a new user and give sudo rights to it.

	root@domU-12-31-39-02-BC-E1:~# adduser shabda
	Adding user `shabda' ...
	Adding new group `shabda' (1000) ...
	Adding new user `shabda' (1000) with group `shabda' ...
	Creating home directory `/home/shabda' ...
	Copying files from `/etc/skel' ...
	Enter new UNIX password: 
	Retype new UNIX password: 
	passwd: password updated successfully
	Changing the user information for shabda
	Enter the new value, or press ENTER for the default
		Full Name []: 
		Room Number []: 
		Work Phone []: 
		Home Phone []: 
		Other []: 
	Is the information correct? [y/N] y
	root@domU-12-31-39-02-BC-E1:~# visudo	

Ok, so we created a new user and gave the new user shabda sudo rights. Logout and login as shabda.


	shabda@shabda-laptop:~$ ssh shabda@ec2-75-101-203-97.compute-1.amazonaws.com
	shabda@ec2-75-101-203-97.compute-1.amazonaws.com's password: 

	...
	shabda@domU-12-31-39-02-BC-E1:~$ 

Ok we are logged in as shabda. Let us install mysql, apache, mod_python, django and associated libraries.

	shabda@domU-12-31-39-02-BC-E1:~$ sudo apt-get install mysql-server mysql-client
	shabda@domU-12-31-39-02-BC-E1:~$ sudo apt-get install apache2
	shabda@domU-12-31-39-02-BC-E1:~$ sudo apt-get install libapache2-mod-python
	shabda@domU-12-31-39-02-BC-E1:~$ sudo apt-get install python-mysqldb
	shabda@domU-12-31-39-02-BC-E1:~$ wget http://www.djangoproject.com/download/1.0.2/tarball/
	shabda@domU-12-31-39-02-BC-E1:~$ tar -xvzf Django-1.0.2-final.tzf
	shabda@domU-12-31-39-02-BC-E1:~$ cd Django-1.0.2-final/
	shabda@domU-12-31-39-02-BC-E1:~$ sudo python setup.py install

Fine, seems like we are done. Lets start a python interpretor and see id we can import Django

	shabda@domU-12-31-39-02-BC-E1:~/Django-1.0.2-final$ python
	Python 2.5.2 (r252:60911, Apr  8 2008, 21:47:16) 
	[GCC 4.2.3 (Ubuntu 4.2.3-2ubuntu7)] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	&gt;&gt;&gt; import django
	&gt;&gt;&gt; 
	shabda@domU-12-31-39-02-BC-E1:~$ django-admin.py startproject testproject
	shabda@domU-12-31-39-02-BC-E1:~$ cd testproject
	shabda@domU-12-31-39-02-BC-E1:~/testproject$ python manage.py validate
	0 errors found

Ok we have django running. Lets checkout something , so apt-get subversion.

	shabda@domU-12-31-39-02-BC-E1:~$ sudo apt-get install subversion

Ok we can run our site on the dev server, lets configure Apache to serve our content.

	shabda@domU-12-31-39-02-BC-E1:~$ sudo vim /etc/apache2/httpd.conf 

	<location>
	    SetHandler python-program
	    PythonHandler django.core.handlers.modpython
	    SetEnv DJANGO_SETTINGS_MODULE settings
	    PythonOption django.root
	    PythonDebug On
	    PythonPath "['/home/shabda/testproject', '/var'] + sys.path"
	</location>

	Alias /media /usr/lib/python2.5/site-packages/django/contrib/admin/media
	<location media="">
	    SetHandler None
	</location>

	Alias /static /var/www_django/static
	<location static="">
	    SetHandler None
	</location>


	

Check ec2-75-101-203-97.compute-1.amazonaws.com, you get a "Congratulations on your first Django-powered page." page


Lets create a database now.

	shabda@domU-12-31-39-02-BC-E1:~$ mysql -u root -p

	mysql&gt; create database djangotest;

	mysql&gt; GRANT ALL PRIVILEGES ON *.* TO 'shabda8'@'localhost' IDENTIFIED BY 'some_pass' WITH GRANT OPTION;

	shabda@domU-12-31-39-02-BC-E1:~/testproject$ mysql -u shabda8 -p
	Enter password: 
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 17
	Server version: 5.0.51a-3ubuntu5 (Ubuntu)

	Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

	mysql&gt; 

Ok we have a database. Lets checkout a reusable django app in our project directory.

	shabda@domU-12-31-39-02-BC-E1:~$ svn co  https://svn.uswaretech.com/djikiki/trunk/djikiki/ djikiki

Edit the settings.py and add database setting and this app to installed apps

	shabda@domU-12-31-39-02-BC-E1:~$ python manage.py validate

Oh and we are done. Navigate to the public url of your instance and you should see your site. Play around with it.


Hmm, we want to test how well does our site perform. We are going to use the awesome benchmarking tool called ab.
ab is Apache Benchmark tool, it comes installed with Apache Httpd server. However to benchmark we need to bring up another instance, as 
running the server and benchmarking tool on same server gives wrong result.

Ok so we bring up another server, login and run the ab tool. Here is the ouput.


	root@domU-12-31-39-03-40-97:~# ab -n 5000 -c 70 http://ec2-75-101-203-97.compute-1.amazonaws.com/djikiki/page/A-title/
	This is ApacheBench, Version 2.3 &lt;$Revision: 655654 $&gt;
	Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
	Licensed to The Apache Software Foundation, http://www.apache.org/

	Benchmarking ec2-75-101-203-97.compute-1.amazonaws.com (be patient)
	Completed 500 requests
	Completed 1000 requests
	Completed 1500 requests
	Completed 2000 requests
	Completed 2500 requests
	Completed 3000 requests
	Completed 3500 requests
	Completed 4000 requests
	Completed 4500 requests
	Completed 5000 requests
	Finished 5000 requests


	Server Software:        Apache/2.2.8
	Server Hostname:        ec2-75-101-203-97.compute-1.amazonaws.com
	Server Port:            80

	Document Path:          /djikiki/page/A-title/
	Document Length:        2703 bytes

	Concurrency Level:      70
	Time taken for tests:   32.344 seconds
	Complete requests:      5000
	Failed requests:        0
	Write errors:           0
	Total transferred:      14465000 bytes
	HTML transferred:       13515000 bytes
	Requests per second:    154.59 [#/sec] (mean)
	Time per request:       452.814 [ms] (mean)
	Time per request:       6.469 [ms] (mean, across all concurrent requests)
	Transfer rate:          436.74 [Kbytes/sec] received

	Connection Times (ms)
		      min  mean[+/-sd] median   max
	Connect:        0    5   6.5      1      61
	Processing:    22  446 228.9    421    2356
	Waiting:       22  444 228.6    419    2356
	Total:         23  450 229.3    426    2356

	Percentage of the requests served within a certain time (ms)
	  50%    426
	  66%    524
	  75%    588
	  80%    631
	  90%    740
	  95%    833
	  98%    935
	  99%   1026
	 100%   2356 (longest request)

150+ requests per second. (This page does about 10 database queries). Not bad for an unoptimized server with even DEBUG = True, eh?

We still have a lot to do to make this server production ready. If you bring down this instance, all you data is lost.
So you need to get an [EBS](http://aws.amazon.com/ebs/) volume and attach it to your instance. You also want to backup your data on [S2](http://aws.amazon.com/s2/). But more about those 
in another post.

-------------------------------------------
Want [Usware](http://uswaretech.com/) to build a high performance web application for you? [Click here](http://uswaretech.com/contact/) to contact us.



