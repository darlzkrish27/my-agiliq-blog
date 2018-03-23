---
layout: post
title:  "Deploying django using docker"
date:   2013-06-14 19:16:51+05:30
tags: docker
author: Javed
---
In this blog post we'll take a look at [Docker](http://docker.io) and
discuss ways to deploy django using docker.


About Docker:
-------------

Docker is a tool that helps you manage lxc containers and images. You can
create an image, launch container from the existing image, attach to a running
container, and generally play around with the containers.

The most important benefit of using docker and containers in general is that
you have a clean, hygeinic and portable runtime enivorment for your app. This
means you don't have to worry about missing dependecies, packages and other
pain points during subsequent deployments. You can take a snapshot of a running
container and restore it again when required. Plus, each app runs in it's own
isolated container so you can have various versions of libraries and other
dependecies for each app without worrying about it's effect on other apps.
Coming from the python world where virtualenv is ubiqitous, it's a huge relief
to me that there's such a better and cleaner solution.

>  Note: Docker is still under heavy development. It should not yet be used in
>  production. Check the repo for recent progress.

To know more about docker and install it, go to the ["Getting Started"](http://www.docker.io/gettingstarted/) guide.

Here I'll discuss two ways of deploying django using docker:

Vanilla deployment:
-------------------

Nothing fancy here, we just instantiate a container image and open up an ssh
port and a web port, then we can use our usual fabfile script to deploy it to
the container. Here I'll use the container image "dhrp/sshd" which I found by
searching [the container index](https://index.docker.io/search?q=ssh).

    > docker pull dhrp/sshd

This will fetch the container image dhrp/sshd

Now let's run the container `-d` for daemonize, `-p 22 -p 8000` for telling docker
to open port 22 and 8000, dhrp/sshd is the image we want to use, and `/usr/sbin/sshd -D`
is the command we want to run inside the container - here we want to run the sshd daemon

    > docker run -d -p 22 -p 8000 dhrp/sshd /usr/sbin/sshd -D
    >> c4cee8e86fa0

The above command will give an hash by which you can refer to the running
container instance in the future.

Now you can query for the NATed ports associated with our newly launched
container using:

    > docker port c4cee8e86fa0 22
    >> 49185

    > docker port c4cee8e86fa0 8000
    >> 49186

or you can also use the generic `docker ps` command for a human readable and
prettier output:

    > docker ps

Now, we can ssh into our container through the NATed port - the password for
this container is `screencast`:

    > ssh root@localhost -p 49185
    >> root@c4cee8e86fa0:/#

Note that the password - `screencast` was set by the container image author.
For better security, you should login, change the password and commit your
changes immediately:

    > docker commit c4cee8e86fa0 dhrp/sshd

Now, you can use this container just like any other server. You'll just need to
make sure to proxy the NATed web port through nginx. You can update your
fabfile to point to the container by updating the ssh port number and deploy as
usual.

Heroku-like deployment:
-----------------------

A few days back I came across [dokku](https://github.com/progrium/dokku) and
decided to give it a try by deploying a small django app. You'll need to clone
the dokku repository and use a buildpack like
[heroku-buildpack-django](https://github.com/jiaaro/heroku-buildpack-django) as described
in the [dokku docs](https://github.com/progrium/buildstep#adding-buildpacks). The
django buildpack I found worked quite well, except that it requires python-dev to be installed.
Luckily, it's quite easy to modify the buildstep script to handle this. Once I had this set-up,
I just had to create a tarball of my code and build it:

    cat django-pastebin.tar | ./buildstep django-pastebin

This will build a container from scratch and install all dependecies required
for the buildpack and prepare the container for deployment.

Here are the repos I've used for this example:

* [https://github.com/tuxcanfly/buildstep](https://github.com/tuxcanfly/buildstep)
* [https://github.com/tuxcanfly/heroku-buildpack-django](https://github.com/tuxcanfly/heroku-buildpack-django)
* [https://github.com/agiliq/django-pastebin](https://github.com/agiliq/django-pastebin)


Verify that it works:

    > docker run -p 8000 django-pastebin /start web
    >> 2013-06-14 13:31:22 [9] [INFO] Starting gunicorn 0.17.4
    >> 2013-06-14 13:31:22 [9] [INFO] Listening at: http://127.0.0.1:8000 (9)
    >> 2013-06-14 13:31:22 [9] [INFO] Using worker: sync
    >> 2013-06-14 13:31:22 [14] [INFO] Booting worker with pid: 14

Now, you can daemonize it and let it run in the background:

    > docker run -d -p 8000 django-pastebin /start web
    >> 4dc243483b56

Conclusion:
-----------

I've found docker and containers very useful in managing multiple apps on a
single server even if it's a little rough around the edges. Tools like
[container index](https://index.docker.io/) are a great way of sharing
development enviroments and avoid getting bogged down by installation
processes. I'm eagerly looking forward to future versions and upcoming features
in docker.


