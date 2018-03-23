---
layout: post
title:  "Better Python package management using source and version control systems"
date:   2009-06-24 22:16:13+05:30
tags: python
author: lakshman
---
Thanks to awesome [django community](http://www.djangoproject.com/community/), there is plenty of open source [django](http://code.google.com/hosting/search?q=django&projectsearch=Search+projects) [code](http://bitbucket.org/repo/all/?name=django) [around](http://github.com/search?q=django&type=Everything&repo=&langOverride=&start_value=1).

These packages get updated quite often and if you use it often like we do, you'd have possibly realized the need to manage these packages better.

Thankfully, all python ever needs is the source, and all you need to do is to place the source in the python path. 

Most projects use Distributed Version Control Systems like Mercurial and Git, and they come locally with the entire history of the source which provides a lot of control to use any version of the code. For code that we use often, like django packages, using the source from a version control system seems to be the best way to manage.

In order to install [django-registration](http://bitbucket.org/ubernostrum/django-registration/), a popular django app by [James Bennett](http://www.b-list.org/), you need to:

Clone the repository:

	$ hg clone https://becomingguru@bitbucket.org/ubernostrum/django-registration/ django_registration

Create a symbolic link in the python path:

	$ sudo ln -fs ~/django_registration/registration /usr/lib/python2.6/dist-packages/django_registration

Thats it. `django_registration` is now on the python path, courtesy of symbollic link, that links the django-registration source to python path, in this case the `dist-packages/` folder; thus it will be possible to import this app within the python environment.

You now have the latest version of the source in the folder `~/django_registration`, which you can check, and modify. With an editor/IDE that has _go to source_ option, you can browse the source by using a simple short cut.(which may not have been simple if the code were is some egg file in a less well known folder)

Because of cloning of the mercurial repository, you now have access to all the revisions of the application. So you can easily update, or change to other versions.
    
To update to a newer version when one exists:

	$ hg pull -u
    
To move to any particular revision:

	$ hg update -r200
    
To check the tags:

	$ hg tags

	tip                              221:b360801eae96
	v0.7                             205:d073602dc103
	v0.6                             154:e263c551ef7b
	v0.5                             139:dc2bf754aa94
	v0.4                             110:4d2a725d8c18
	v0.3                              75:f41e8a239016
	v0.2                              58:e5110bb8d48a
	v0.1                              42:d28e5a770ff8

If you want to update to a recent release, say 0.7,

     $	hg update v0.7
	7 files updated, 0 files merged, 0 files removed, 0 files unresolved
	
The same is also applicable while dealing with SVN repositories; alebit some of the revision changes go thro the network to the central server.

In order to install and work with [django trunk](http://code.djangoproject.com/svn/django/trunk/), check out the trunk and add a symlink, as before

	$ svn co http://code.djangoproject.com/svn/django/trunk/ django-trunk
	$ cd django-trunk/django
	$ sudo ln -fs . /usr/lib/python2.6/dist-packages/django-extensions
    
You will now be able to access the latest version of django:

	$ python
	>>> import django
	>>> django.get_version()
	u'1.1 beta 1 SVN-11092'

To use any other version of django you can do some SVN manipulation, as following:

	#To update to a latest version
	svn up
	At revision 11092.

	#To update to a particular revision
	svn update -r11090
	At revision 11090.

	#To update to a last committed version
	svn up -r'COMMITTED'
	At revision 11079.

	#Change to a tagged version - to release 1.02
	svn switch http://code.djangoproject.com/svn/django/tags/releases/1.0.2/

	#Change to any branch
	svn switch http://code.djangoproject.com/browser/django/branches/soc2009

`svn switch` commands talk to the central servers and can be time consuming.

### Possible shortcomings of using source and symbollic links:

Setuptools distribution adds to the python standard library [Distutils](http://docs.python.org/distutils/) module, by adding a lot of functionality, among others that stores the package meta data like the version, location of download. If you perform

	easy_install django
	
after installing it by source as above, `easy_install` again downloads the latest released version of django 1.02, listed in the Python Package Index, as it hasn't been informed of django installation. 

If you are not going to use `easy_install` at all, this isn't a problem; but it is good to also update setup tools that a version of this application exists, if you at all need. You can then use the `pkg_resources` module installed with setup tools distibution, to query to find all the installed applications, from python.

If the application contains a `setup.py` file, you could manually inform setup tools to update using the following command

	python setup.py develop

### The workflow

To install packages, in a way with high control following needs to be done:
	
	* Checkout the repository
	* Add a symbollic link to the repository
	* Update setup tools of this installation.

### Enter PIP

PIP is a package management tool that does the workflow we need, well. It automatically checks out from the repos and runs `python manage.py develop` and updates few other resources, so that python package management tools in the system are well aware of the installed packages.

If you are checking out the source and creating symbolic links, it is kind of awkward to have to create symbolic links all the time; Pip automates that. Pip also updates the setuptools with the metadata of the application, so that next time if you ask the same application to be installed, via `easy_install` or `pip`, it doesn't download and install again.

So the following command:

	pip install -e hg+http://bitbucket.org/ubernostrum/django-registration/#egg=django_registration

* Clones the mercurial repository django-registration locally, into `Env/src/django_registration`
* Creates the required symbollic links in the python path, in this case in the `site-packages` folder
* Updates the local package management tools by setup tools of the new package.

`-e` option tells pip to install the package as editable, that is to leave the repository undeleted. The `hg+` and `egg=django_registration` indicate the repository and the local folder to install to.

Pip works well with git, SVN, Hg an Bzr. Provided you inform the pip of the repository and the folder name, pip completes the work-flow.

Since the source is present in the `Env/src` folder,  along with the SVN, Git or Hg data, one can modify the versions using version control systems, as illustrated in some of the commands above

### Multiple environments

Most often the requirement will be just to update the package into the latest version, for which simple revision control update commands seem good enough. Its good to also mess around with the code changing into different versions and trying different things.

But if you seem to be changing of the versions too often, for compatibility between different environments, you might want to using [virtualenv](http://pypi.python.org/pypi/virtualenv), that isolates a python working environment. [virtualenvwrapper](http://pypi.python.org/pypi/virtualenvwrapper) adds a bash wrapper to use virtualenv conviniently.

<hr />
Do you also want your app development managed well? [Look us up](http://uswaretech.com/contact/)

