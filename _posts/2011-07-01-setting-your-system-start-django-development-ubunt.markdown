---
layout: post
title:  "Setting up your system to start with Django development on Ubuntu:"
date:   2011-07-01 22:30:00+05:30
tags: mysql
author: akshar
---
1>Being a Python web framework, Django requires Python. If you are on ubuntu, you probably have python installed on your system.
If you don't have it installed, give the following command from the terminal:

	sudo apt-get install python

This will install Python on your machine.

2>If you will be developing a database driven web application, you need to have a database setup on your system. I feel mysql is good and will tell you about installing mysql. It is pretty easy to install and use. You need to issue the following command from terminal:

	sudo apt-get install mysql-server

While installing mysql, you will be prompted to enter a password which you will require when you try to use mysql. Give a password.
This should install mysql on your system. 
To check whether mysql is properly installed or not, give this command from the terminal:

	mysql -u root -p

This command would prompt you to enter the password. After entering the password you set earlier for mysql, you would be taken to mysql prompt.
If you reach the mysql prompt, it means mysql is successfully installed onto your system.
Now, give the command as "exit". This exits from mysql.

3>Installing mysqldb:
To write Python scripts that connects to mysql, you need Python interface to mysql. MySQLdb is the python interface to mysql.
Give the following command at the terminal:

	sudo apt-get install python-mysqldb

If this command executes successfully, python can access the database server.If you want to test this, go to python interactive environment by typing "python" at the terminal:
Inside python interactive environment, write the following statement:

	import MySQLdb

If this statement excutes without giving an ImportError then you have successfully installed mysqldb and now your python program can interact with mysql.

4>Installing Django
Go to this link:
	https://www.djangoproject.com/download/

I suggest to use option 1 as it worked easily for me

Now, you have django setup on your system and can start developing applications.

