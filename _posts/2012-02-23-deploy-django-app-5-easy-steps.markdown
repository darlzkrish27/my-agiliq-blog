---
layout: post
title:  "Deploy Django App in 5 Easy Steps"
date:   2012-02-23 04:59:50+05:30
tags: vps
author: saket
---
So you just bought a new VPS, have installed [Ubuntu](http://www.ubuntu.com/) and want to deploy
your [django](www.djangoproject.com) app, GREAT!! We shall get your app, up and running in 5 easy steps, using best(arguably) of tools available. The post is targeted to audience who are new to deployment arena, but assumes you are comfortable with developing basic django apps. We shall be using [gunicorn](http://gunicorn.org/) as our server and [nginx](http://nginx.org/en/) nginx as our reverse proxy and static hanlder. Here we go:

### 1. Login and OS Updation:

    $ ssh root@{your_ip}

    # apt-get update  

    # apt-get upgrade

    # apt-get dist-upgrade

    # dpkg-reconfigure tzdata #choose your time zone
        
### 2. Setup Users and Permissions:
 
    # useradd agiliq

    # mkdir /home/agiliq

    # chown agiliq:agiliq /home/agiliq

    # passwd agiliq  #choose a password

    # chsh agiliq -s /bin/bash  #choose a shell

    # visudo #lets give sudo access to the user agiliq

		root     ALL=(ALL) ALL 
		agiliq   ALL=(ALL) ALL 
		
    # su agiliq # switch from root to agiliq
        
### 3. Install necessary stuff and create a dev environment:

	$ sudo apt-get install python-pip git mysql-client mysql-server  

	$ python-mysqldb nginx emacs(you might choose vim or stick to default vi/nano)

	$ pip install virtualenv #please refer to documentation of virtualenv

	$ virtualenv projhq # creates a project directory, 

	$ cd projhq

	$ projhq source /bin/activate

	(projhq)$ pip install django gunicorn 


### 4. Deploy some code for test

you may clone this polls application for the sake of testing

        $ cd ~/projhq #your project directory
        (projhq)$ git clone git://github.com/bitaVersion/Hello-Django.git

### 5. Necessary Configurations

##### i) nginx

        $ cd /etc/nginx/sites-enabled/
        $ touch nginx.conf

and inside the file place the following snippet, make sure to change your 
servername, username and static files directory. 

	server {
    		listen 80;
    		server_name “your server name”;
    		access_log /home/username/access.log;
    		error_log /home/username/error.log;
    		location /static {
        		root /home/path_to_proj_directory_containing static_files;
    		}

    		location / {
        		proxy_pass http://127.0.0.1:8000;
    		}
	}

##### ii)gunicorn configuration

cd into project directory/app 
touch gunicorn_cfg.py 
place the following snippet:


	bind = "127.0.0.1:8000"
	logfile = "/var/log/gunicorn.log"
	workers = 3
	pid = projhq/pid/gunicorn.pid

##### iii)server restart script

make a directory called scripts parallel to project root
and place the following snippet in some file(i name start.sh):

	PROJDIR="/path/to/proj/" #directory which contains settings.py
	PIDFILE="$PROJDIR/pid/gunicorn.pid" #create a directory called pid inside projdir
	cd $PROJDIR
	source /path/to/bin/activate
	if [ -f $PIDFILE ]; then
  		kill `cat -- $PIDFILE`
  		rm -f -- $PIDFILE
	fi
	gunicorn_django -c path_to_gunicorn_file/gunicorn_cfg.py




Few checks if you have missed them:

#####  create mysql db: 

       create a database with the name matching your settings.py 

##### Start nginx:

      sudo service nginx restart

##### Start server:

	go to scripts directory: 
	chmod +x start.sh(only for the first time)
	bash start.sh 

The above post was a general introduction of how you can begin using lightweight yet scalable tools, in the next post we shall be talking about tools as [fabric](http://docs.fabfile.org/en/1.4.0/index.html), [supervisor](http://pypi.python.org/pypi/supervisor) and [Monit](http://pypi.python.org/pypi/MonitManager), which makes things far easier and more secure. Experienced admins can point out flaws in the approach and suggest improvements.











    





