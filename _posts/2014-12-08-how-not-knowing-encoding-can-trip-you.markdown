---
layout: post
title:  "How not knowing encoding can trip you"
date:   2014-12-08 16:47:52+05:30
tags: python
author: akshar
---
Our Scenario:

* You are provided with a link to a file on the internet.
* Download and read this file
* Insert read value into the database

Make sure your terminal encoding is set to utf-8. Most probably that would be the default so you would not require any change.

There could be other similar scenarios. Another collaborator on your project adds a file containing some data in version control. You have to read the file and insert this data in database.

I have added one file to Dropbox. We will use this file in our example. Link for the file is <a href="https://www.dropbox.com/s/arcz0jsinipuixp/latin_encoded.txt?dl=0" target="_blank">https://www.dropbox.com/s/arcz0jsinipuixp/latin_encoded.txt?dl=0</a>

You can't read the contents of this file using 'urllib' or 'requests' because then you will get html page source for this link. So download the file on your machine.

Read this file:

	>>> f = open('/Users/akshar/Downloads/latin_encoded.txt')
	>>> read_value = f.read()
	>>> type(read_value)
	<type 'str'>
	>>> print read_value
	�

As you are aware, reading a file gives a 'str' and doesn't give a 'unicode'. And remember, 'str' means encoded 'unicode'? We try to print it. It seems the file content was encoded using a encoding scheme which our terminal doesn't understand, that's why it shows question mark. Terminal understands 'utf-8' encoded content by default. Seems file content was encoded with an encoding other that 'utf-8'. And you don't know what encoding that was.

Anyways without caring about it, let's try to write our read_value into PostgreSQL db.

If you require help setting up the database, see <a href="#database-setup">database setup section</a> at bottom.

Let's write the script to read from the file and insert in db.

	# You can save it in psyco.py
	import psycopg2

	conn_string = "host='localhost' dbname='hack' user='jack' password='crack'"
	print "Connecting to database\n ->%s" % (conn_string)
	conn = psycopg2.connect(conn_string)
	cursor = conn.cursor()
	print "Connected!\n"
	# You need to change file path to your ~/Downloads folder
	f = open('/Users/akshar/Downloads/latin_encoded.txt')
	read_value = f.read()
	f.close()
	cursor.execute("insert into names values ('%s')" % (read_value,))
	conn.commit()
	print "Inserted\n"

Run the script:

	python psyco.py

Boom! You must have got an error:

	(hack)/private/tmp $ python psyco.py
	Connecting to database
	 ->host='localhost' dbname='hack' user='jack' password='crack'
	Connected!

	Traceback (most recent call last):
	  File "psyco.py", line 15, in <module>
		cursor.execute("insert into names values ('%s')" % (read_value,))
	psycopg2.DataError: invalid byte sequence for encoding "UTF8": 0xe4 0x27 0x29

So Python tried to insert read_value to db. read_value was encoded using some encoding scheme which we don't know. Database's default encoding scheme is UTF8. So db tries to decode whatever you are trying to insert with utf8. But in utf8 the byte sequence we are trying to insert doesn't correspond to any codepoint and so decoding fails. And hence an error was raised.

So we must know what is the encoding of read_value. After reading from file, read_value should be decoded using the known encoding scheme and then encoded back to utf8. It should be encoded back to utf8 because our db works with utf8.

Now I tell you, the file which you have downloaded is encoded with encoding scheme '8859'.

So we will have to make following change:

	decoded_read_value = read_value.decode('8859')
	read_value = decoded_read_value.encode('utf-8')

So final script becomes:

	import psycopg2

	conn_string = "host='localhost' dbname='hack' user='jack' password='crack'"
	print "Connecting to database\n ->%s" % (conn_string)
	conn = psycopg2.connect(conn_string)
	cursor = conn.cursor()
	print "Connected!\n"
	f = open('/Users/akshar/Downloads/latin_encoded.txt')
	read_value = f.read()
	f.close()
	decoded_read_value = read_value.decode('8859')
	read_value = decoded_read_value.encode('utf-8')
	cursor.execute("insert into names values ('%s')" % (read_value,))
	conn.commit()
	print "Inserted\n"

Run the script:

	(hack)/private/tmp $ python psyco.py
	Connecting to database
	 ->host='localhost' dbname='hack' user='jack' password='crack'
	Connected!

	Inserted

Seems insert operation worked properly.

Check db content now:

	hack=> select * from names;
	 name
	------
	 ä
	(1 row)

So you had to know the encoding of file to make it work. Unless you knew the encoding of file you wouldn't have been able to decode it.


<h3><a name="database-setup">Setting up the database.</a></h3>
	/private/tmp $ psql

	akshar=# create database hack;
	CREATE DATABASE

	akshar=# create user jack with password 'crack';
	CREATE ROLE

	akshar=# grant all privileges on database hack to jack;
	GRANT

Close db connection.

Connect to this db as 'jack' and create a table. You should create the table as user 'jack' otherwise you'll not be able to insert into the table when connected as jack.

	/private/tmp $ psql hack --username=jack -W
	Password for user jack: crack

	hack=> create table names(name varchar(10));
	CREATE TABLE

Close db connection.


