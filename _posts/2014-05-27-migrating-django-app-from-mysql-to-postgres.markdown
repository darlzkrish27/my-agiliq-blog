---
layout: post
title:  "Migrating django app from MySQL to Postgres"
date:   2014-05-27 13:28:50+05:30
tags: mysqltopostgres
author: manjunath
---
In this tutorial, we will take a django app backed by MySQL and will convert MySQL database to postgres database.
This is useful if we are deploying our app to `Heroku` because `Heroku` uses standard `Postgres`

### Prerequisites:

* We assume that you have a running django app with MySQL as a database.
* Also, your app is running on virtualenv.


### Converting MySQL to Postgres:

Let's assume that you have a django app running with MySQL and you want to convert this to Postgres.

1) Install dependenicies:

    $ pip install psycopg2
    $ pip install py-mysql2pgsql


2) Create Postgres database:
   
    postgres@agiliq-Inspiron-N5010:~$ psql
    psql (9.1.11)
    Type "help" for help.

    postgres=# create database my_database;
    CREATE DATABASE
    postgres=#

3) Run:

    $ py-mysql2pgsql

At initial run this command creates a file named `mysql2pgsql.yml` having the below info:

    mysql:
    hostname: localhost
    port: 3306
    socket: /tmp/mysql.sock
    username: foo
    password: bar
    database: your_database_name
    compress: false
    destination:

    postgres:
    hostname: localhost
    port: 5432
    username: foo
    password: bar
    database: your_database_name

Update the above configuration file with appropriate database credentials for both 'MySQL' and `Postgres`.


4) Run:

    $ py-mysql2pgsql -v -f mysql2pgsql.yml

The above command will transfer the data from `MySQL` database to `Postgres`.

Note: 

* The above command may raise some `Integrity` errors, but no worries it can be fixed. :)
* You can also include or exclude some tables check this [Here](https://github.com/philipsoutham/py-mysql2pgsql)


5) Be sure to update your database `settings.py` file:

    DATABASES = {
    "default": {
       "ENGINE": "django.db.backends.postgresql_psycopg2",
       "NAME": "your_database_name",
       "USER": "your_username",
       "PASSWORD": "your_password",
       "HOST": "localhost",
       "PORT": "5432",
    }
    }

6) Verify the correctness by adding some data to your existing database(Postgres).

