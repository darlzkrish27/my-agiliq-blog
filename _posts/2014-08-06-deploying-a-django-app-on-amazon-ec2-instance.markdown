---
layout: post
title:  "Deploying a Django app on Amazon EC2 instance."
date:   2014-08-06 20:38:22+05:30
tags: Django app deployment
author: Rakesh
---
In this tutorial we are going to launch an EC2 instance and deploy a Django app. I am going to use nginx, gunicorn in this process.

From an eagle's eye this is going to be our walk-through in this blog post.

- Launch an Amazon EC2 instance
- Deploying Django App on Amazon EC2
- Creating a load balancer
- Autoscaling an instance.

Launching an Amazon EC2 instance:
----------------------------------
Before we launch your instance, we should be sure setting up with Amazon EC2. Here we go to set them up following the below steps:

1. Sign Up for AWS
2. Create an IAM User
3. Create a Key Pair
4. Create a Virtual Private Cloud (VPC)
5. Create a Security Group

Creating an instance from an Amazon Machine Image reduces the redundant work of specifying all the details of the instance as the AMI provides the information required to launch an instance. So lets create an AMI in this process of creating an instance.

The following are the steps to create an instance through the console.

1. Open the Amazon EC2 console.
2. In the navigation bar at the top of the screen, the current region is displayed. Click the region's name to select the region for the instance.
3. From the Amazon EC2 console dashboard, click Launch Instance.
4. The Choose an Amazon Machine Image (AMI) page displays a list of basic configurations called Amazon Machine Images (AMIs)
5. Choose the AMI to use and click Select. Start by selecting the type of AMI to use by using these categories on the left pane.
6. On the Choose an Instance Type page, select the hardware configuration and the size of the instance to launch.
7. On the Configure Instance Details page, change the settings as needed, and then click Next: Add Storage.
8. On the Add Storage page, you can specify volumes to attach to the instance, besides the volumes specified by the AMI.
9. On the Tag Instance page, specify tags for the instance by providing key and value combinations. Click Create Tag to add more than one tag to your resource.
10. On the Configure Security Group page, you can define firewall rules for your instance. These rules specify which incoming network traffic is delivered to your instance.
11. On the Review Instance Launch page, check the details of your instance, and make any necessary changes by clicking the appropriate Edit link.
12. In the Select an existing key pair or create a new key pair dialog box, you can choose an existing key pair, or create a new one.

And we are done with the creation of the instance.

Deploying the app:
------------------------

Now lets deploy our Django app on Amazon EC2. There are many ways to deploy it and I am depicting a simple method here.

##### 1. Get the key pair that we have created in the launching process of the Amazon instance and change give execute permissions to it. Save the key-pair in your ~/.ssh directory.
Once you are done with that connect to your instance using the ssh.

        $ ssh -i ~/.ssh/keypair.pem ubuntu@ec2-XX-XX-X-XXX.compute-X.amazonaws.com

##### 2.Update and upgrade the repository.

        $ sudo apt-get update
        $ sudo apt-get upgrade

##### 3.Install python and some basic packages.

        $ sudo apt-get install python-pip
        $ sudo apt-get install python-dev
        $ sudo apt-get install build-essential

##### 4. Install virtualenvwrapper.

        $ pip install virtualenvwrapper

##### 5.Add the below lines in ~/.bashrc

    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh

##### 6.And source it.

    $ source ~/.bashrc


##### 7.Create a Django App

Lets create a sample application to test the deployment process. First, create a virtualenv and activate it.

    $ mkvirtualenv testapp
    $ workon testapp

##### 8.Install django and create the app

    (testapp) pip install django
    (testapp) django-admin.py startproject testapp
    (testapp) cd testapp
    (testapp) python manage.py runserver

##### 9.Working with nginx

Nginx is an open source reverse proxy server, which has the capability to handle our static files and redirect all other stuff to our Django server.
Installing Nginx:

    $ sudo apt-get install nginx

We can test whether our nginx is working or not by following the below steps.

    $ sudo service nginx start

If we go to the URL, i.e.  http://ec2-XX-XX-X-XXX.compute-X.amazonaws.com and it should show us "Welcome to nginx".
Now stop the server to configure it further.

    $ sudo service nginx stop

Lets make Nginx run as ubuntu user.

    $ sudo vim /etc/nginx/nginx.conf

And now edit the first line containing the User name. It should look like the below way at the end.

    user ubuntu ubuntu;

We should configure our app now, to get it done, edit this file:

    $ sudo vim /etc/nginx/sites-enabled/default

Change the values according to our need. Some basic things would be:

    access_log  /home/ubuntu/testapp/nginx-access.log;
    error_log  /home/ubuntu/testapp/nginx-error.log info;

Also change the location of the static file directory.

##### 10.Working with Gunicorn

Gunicorn is a Python web server gateway interface HTTP server for Unix, which natively supports WSGI and Django, helps in process management, creating multiple worker configurations and for creating various server hooks for extensibilty.

    (testapp)$ pip install gunicorn

Let’s create a script to start gunicorn.

    $ cd ~/testapp
    $ vim start_gnicorn.sh

The start_gunicorn should like this:

    APPNAME=testapp
    APPDIR=/home/ubuntu/$APPNAME/

    LOGFILE=$APPDIR'gunicorn.log'
    ERRORFILE=$APPFIR'gunicorn-error.log'

    NUM_WORKERS=3

    ADDRESS=127.0.0.1:8000

    cd $APPDIR

    source ~/.bashrc
    workon $APPNAME

    exec gunicorn $APPNAME.wsgi:application \
    -w $NUM_WORKERS --bind=$ADDRESS \
    --log-level=debug \
    --log-file=$LOGFILE 2>>$LOGFILE  1>>$ERRORFILE &

Now run the script to start gunicorn.


    $ chdmod +x start_gunicorn.sh
    $ ./start_gunicorn.sh

Hurrah!! We are done with the deployment of our app. Go to the app URL and watch it running.


Elastic load balancing:
-------------------------

There is a possibility of getting the instance failed in any condition and there is a need to reroute the traffic into another running EC2 instance, without disrupting the overall flow of information, there comes the concept of Load balancing.  The Elastic load balancer automatically spawns app servers based on the load and assures us with the app running at all times. When we use Elastic load balancing to manage the traffic to our app, it will result in the distribution of requests to Amazon EC2 instances in multiple availability zones so that the risk of overloading one single instance gets minimized. We can associate our domain name to the load balancer as the load balancer is the only computer that is exposed to the internet and also we don't have to create and manage public domain names for the instances to the instance that is managed by load balancer.

Create a Basic Load Balancer in EC2-VPC:

Amazon Virtual Private Cloud (Amazon VPC) enables us to launch Amazon Web Services (AWS) resources, such as, ELB load balancers and Amazon EC2 instances, into a virtual network that we have defined. Here are the steps to deal with it:

There are four main steps in dealing with the launching of a load balancer.

1. Define Load balancer
2. Configure Health Check
3. Assign Security groups
4. Add EC2 instances [Optional as you can skip this step if you have planned to add auto scaling]

#### 1.Define Load Balancer

- Sign in to the AWS Management Console and open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
- Click on Load Balancers in the EC2 dashboard pane of Resources page.
- In the Define Load Balancer Page make the necessary selections like name and listener configurations.
- Click continue to configure Health check for our instance.

#### 2.Configure Health Check

- On the Configure Health Check page of the Create a New Load Balancer wizard set the following configurations:
    1. Leave the Ping Protocol to its default value of HTTP
    2. We can set the Ping Port to its default value 80
    3. In the Ping Path field specify a single forward slash("/") the path to the default home page of the webserver as the Elastic Load balancer sends the health check queries to the path we specified here.
    4. Set the Advanced Options according to the needs.
- Click continue to select the subnet in which we want to launch our load balancer instance.

#### 3.Add Security Groups

- In the Assign Security Groups page, we can assign an existing security group or we can create a new one. If we use an existing security group we should ensure that it allows ingress to the ports that we configured the load balancer to use.
- Click Continue to register EC2 instances with our load balancer.

#### 4.Add EC2 instances [Optional]

Note: If you are planning to add autoscaling to Loadbalancer we can skip this step as we can specify the instance while configuring Autoscaling settings.

- On the Add EC2 Instances page, in the Add Instances to Load Balancer table, select the boxes in the Instance column to register instances with our load balancer.
- Set the Enable Cross-Zone Load Balancing and Enable Connection Draining boxes according to your need.

At the end Review all the settings and create the load balancer

We can verify the creation and description of the specifications of our load balancer by clicking on the Load Balancers on the left pane of the Resources page and selecting the Load balancer.

Autoscaling an instance
-------------------------

Amazon constantly monitors the app servers, and if any of them reaches a certain CPU usage, Amazon will automatically launch X new server(s) and associate them with the load balancer when they're up and running. Same thing applies if traffic levels go down and you need to terminate an instance or two. Auto scaling enables us to automatically launch or terminate instances based on user-defined policies, health status checks and schedules. We can autoscale an instance using console and CLI(command line interface) as well. Autoscaling the instance through the console is straight forward and using CLI requires some tools and commands to get going. We can get to know how to autoscale through console in this post.

* Steps to follow for autoscaling an instance through console:

    1. [Optional] Create a launch configuration. Skip this step if you want to use your own launch configuration.
    2. Create an Auto Scaling group with a load balancer.
    3. Verify that our Auto Scaling group has been created with a load balancer


#### 1. Create Launch configuration

- Sign in to the AWS Management Console and open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
- On the Amazon EC2 Resources page, in the EC2 Dashboard pane, under Auto Scaling, click Launch Configurations.
- Click Create launch configuration.
- On the Create Launch Configuration wizard, the 1. Choose AMI page displays a list of basic configurations, called Amazon Machine Images (AMIs), that serve as templates for your instance. Select an AMI for your instance.
- On the 2. Choose Instance Type page, you can select the hardware configuration of your instance.
- On the 3. Configure Details page, in the Name field, enter a name of your launch configuration (my-first-lc) and fill the other details as required
- In the next 2 steps you can add storage devices and configure a security group.
- On the 6. Review page, review the details of your launch configuration.
- After you are done reviewing your Launch Configuration, click Create launch configuration.
- In the Select an existing key pair or create a new key pair dialog box, select one of the listed options.
- Select the acknowledgement check box and then click Create Launch Configuration to create your Launch Configuration.
- The Launch configuration creation status page displays the status of your newly created launch configuration. Click Create an Auto scaling group using this launch configuration.

#### 2. Create an Auto Scaling group with a load balancer using console:

- On the Configure Auto Scaling Group Details page, enter the following details:
    1. In the Group name field, enter a name for your Auto Scaling group my-test-asg-lbs.
    2. In the Group size field, enter 2 for the number of instances you want your Auto Scaling group to start with.
    3. Specify the Network field as required.
    4. Click the Availability Zone(s) dialog box, and select your Availability Zone in that.
- Click Advanced Details.
- In the Load Balancing field, select Receive traffic from Elastic Load Balancer(s).
- Click the empty dialog box and select the load balancer name that we have created.
- In the Health Check Type field, select ELB or EC2 as required.
- The Health Check Grace Period field is pre-populated with the default value. We can type in the field to change the default value.
- Click Next: Configure scaling policies and configure it as required.
- Click Review to verify the details of your Auto Scaling Group and Create Auto Scaling group.

#### 3. Verify that our Auto Scaling group has been created with a load balancer:

For verifying the autoscaling we can check it by clicking the Auto scaling groups and selecting the the autoscaling group that we have created. we can get to see the details tab containing our loadbalancer and in the instance tab we can see them with their status as Inservice.

Updating AMI when there is a change in the code:
--------------------------
Whenever there is a new bit of code that the app needs, i.e. any changes in the code or any necessary package updating we need to update the AMI to get the new code running on the server. So, whenever there is a change in the code we create a new AMI with the instance that we have updated and we associate it to the load balancer and from then the app server will be using the new AMI(i.e. the app with the updated code). We will be terminating the older instances and the loadbalancer updates the newer instances according to the scaling rules and brings up the new instance running.

