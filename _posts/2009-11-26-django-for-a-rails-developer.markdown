---
layout: post
title:  "Django for a Rails Developer"
date:   2009-11-26 20:12:21+05:30
tags: python
author: dheeru
---
This is not yet another <a href="http://www.djangoproject.com/">Django</a> vs <a href="http://rubyonrails.org/">Rails</a> blog post. It is a compilation of notes I made working with Django after having worked on Rails for years.

In this post I want to give a brief introduction to Django project layout from a Rails developer point of view, on what is there, what is not there and where to look for things. It should help a rails developer working on django be able to find the necessary files and underatnd the layout of the project files.

Once you have both the frameworks installed on your system you can create the projects using the commands

    # creating a rails project
    rails rails_project
    # creating a Django project
    django-admin.py startproject django_project

Lets look at the files and structure created by the respective frameworks

    #rails_project
    README
    Rakefile
    app/
        controllers/
            application_controller.rb
        helpers/
            application_helper.rb
        models/
        views/
            layouts/
    config/
        boot.rb
        database.yml
        environment.rb
        environments/
            development.rb
            production.rb
            test.rb
        initializers/
            backtrace_silencers.rb
            inflections.rb
            mime_types.rb
            new_rails_defaults.rb
            session_store.rb
        locales/
            en.yml
        routes.rb
    db/
        seeds.rb
    doc/
        README_FOR_APP
    lib/
        tasks/
    log/
        development.log
        production.log
        server.log
        test.log
    public/
        404.html
        422.html
        500.html
        favicon.ico
        images/
            rails.png
        index.html
        javascripts/
            application.js
            controls.js
            dragdrop.js
            effects.js
            prototype.js
        robots.txt
        stylesheets/
    script/
        about
        console
        dbconsole
        destroy
        generate
        performance/
            benchmarker
            profiler
        plugin
        runner
        server
    test/
        fixtures/
        functional/
        integration/
        performance/
            browsing_test.rb
        test_helper.rb
        unit/
    tmp/
        cache/
        pids/
        sessions/
        sockets/
    vendor/
        plugins/

that is huge....

lets look at the django project files

	# django_project
    __init__.py
    manage.py
    settings.py
    urls.py

far lesser when compared to the rails project.

In fact a rails project comes with everything a web application needs. When I say everything I mean everything..... base application, routing, database configuration, development, test and production environment specific configurations and their respective log files,  javascript, test, some standard html files and some helpful scripts for developing.

Why then does a Django project have so less number of files? It has got to do with the Django's philosophy and the concept of applications. The django project is not complete without the application, so lets create a application inside the project and have a look at the structure

	django-admin.py startapp app

	# django_project
	__init__.py
    app/
        __init__.py
        models.py
        tests.py
        views.py
    manage.py
    settings.py
    urls.py

even after including this, the number of files is still less than the rails project.

Lets break it down and relate both the frameworks. 

<table width='100%' border='1px'>
	<thead>
		<tr>
			<th></th>
			<th>Rails</th>
			<th>Django</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td width='20%'>Configuration Files</td>
			<td width='40%'>
				<li>config/database.yml for database settings</li>
				config/environments/
				<li>development.rb for development specific settings</li>
				<li>production.rb for production specific settings</li>
				<li>test.rb for test specific settings</li>
			</td>
			<td width='40%'>
			    settings.py, one file for everything, database configuration and any other configuaration or settings will be in this file
			</td>
		</tr>
		<tr>
			<td>URLs</td>
			<td>
				config/routes.rb
			</td>
			<td>
			    urls.py
			</td>
		</tr>
		<tr>
			<td>Schema/Models</td>
			<td>
				<li>db/schema.rb for the ruby version of db schema</li>
				<li>app/models/* for the models</li>
			</td>
			<td>
			    <li>complete schema is not stored any where</li>
				<li>Under every application in your project, models.py file will contain the table specific schema stored as django models</li>
			</td>
		</tr>
		<tr>
			<td>Management Commands</td>
			<td>
				<li>./script/server,  start server</li>
				<li>./script/console, ruby console</li>
				<li>./script/dbconsole, database console</li>
				<li>rake db:migrate, run database migrations</li>
			</td>
			<td>
				manage.py is the file for all your tasks
				<li>./manage.py runserver, start server</li>
				<li>./manage.py shell, python console</li>
				<li>./manage.py dbshell, database console</li>
				<li>./manage.py syncdb</li>
			</td>
		</tr>
		<tr>
			<td>Application Code</td>
			<td>
				app/controllers/* will contain the application logic
			</td>
			<td>
			    views.py file under each application folder is the place to write to your application logic, file can be named with any name, views.py is the general convention
			</td>
		</tr>
		<tr>
			<td>Application templates</td>
			<td>
				app/views/<controller_name>/* is the place for the templates
			</td>
			<td>
			    <li> default: looks inside 'templates' directory, under the application directory</li>
				<li> looks in the directory specified by 'TEMPLATE_DIRS' in settings.py </li>
			</td>
		</tr>

	</tbody>
</table>

and lets have a look at the other things in rails_project
	<ul>
		<li>Logging, unders the logs directory </li>
		<li>Some default html files for some standard http errors, under the public directory </li>
		<li>Rails has very good support for testing, for that bunch of files under tests </li>
		<li>Vendor/Plugin, place for some third party plugins/applications. </li>
	</ul>

Missing pieces in Django (for a rails developer)
	<ul>
		<li>Scaffold magic</li>
		<li>Generate commands</li>
		<li>Migrations</li>
	</ul>
	
and what is Django is providing by default? Sorry no extra files in the project; but you will get an authentication system and Django's killer feature, admin, just by modifying your 'INSTALLED_APPS' in your settings. It is similar to the plugins feature in Rails, Django's philosophy of resusable apps helps you in getting any particular functionality into your project.

Following is a list of what I like in Django (and associated apps):
<li>Admin (almost everybodys favourite)</li>
<li>Generic Views, helps from writing a lot of repetetive code, <a href='http://www.b-list.org/weblog/2006/nov/16/django-tips-get-most-out-generic-views/'>James Bennett's blog on generic views</a></li>
<li>Django DB API & QuerySets, (chaining and the filters that are missing in ActiveRecord)</li>
<li>Forms and above all, my favorite yet: <a href="http://docs.djangoproject.com/en/dev/topics/forms/modelforms/">ModelForm</a>.</li>
<li>You can get the migrations feature in Django using <a href="http://south.aeracode.org/">South</a>. Being an external app, it a bit of pain in setting it up to work with South. Other than that it is more like rails migrations</li>
<li><a href="http://code.google.com/p/django-command-extensions/">django-command-extensions</a></li>
<li><a href="http://haystacksearch.org/">Search the Django way</a></li>

Following are the questions that I keep getting, 
 <li>When a project will almost have a application why creating project & app has to be two different steps?</li>
<li>Why not create a 'templates' directoy and a 'base.html' either in project's directory or in the apps's directory, because creating the same templates directory and same base.html for every project is not DRY :) ?</li>
<li> Why serving static files in development has to be a additional setup, as no developer wants to setup a server for serving static files, I am aware of 'django.static.serve' but still that is an additional setup, why not create a sample media directory and a url for the same in urls.py ?</li>

let me know via comments, if you have any answers 

program used for listing the files in a directory

    import os
    import sys

    try:
        directory = sys.argv[1]
    except IndexError:
        directory = os.path.dirname(os.path.abspath(__file__))

    def r_list_dir(directory, indent=0):
        dir_files = sorted([os.path.join(directory, file_name) for file_name in  os.listdir(directory)])
        for item in dir_files:
            if os.path.isdir(item):
                print " " * indent + os.path.split(item)[1] + '/'
                r_list_dir(item, indent+4)
            else:
                print " " * indent + os.path.split(item)[1]

    r_list_dir(directory)

