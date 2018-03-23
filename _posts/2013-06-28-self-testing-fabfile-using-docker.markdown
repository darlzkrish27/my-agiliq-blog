---
layout: post
title:  "Self-testing fabfile using docker"
date:   2013-06-28 18:57:35+05:30
tags: docker
author: Javed
---
In my last blog post, I discussed [deploying django on
docker](http://agiliq.com/blog/2013/06/deploying-django-using-docker/). This is
a blog post on a related note and we'll be looking at using docker to test a
fabfile. Note that docker is still under development and not ready for use in
production.


Fabfile and testing:
--------------------


[Fabric](http://docs.fabfile.org/en/latest/index.html) is a great tool for
automating deployments. Unfortunately, the docs on testing fabfiles are a bit
scarce.

With good tests, you can be sure that your fabfile works out of the box and be
prepared for server migrations or upgrades.

One apporach to testing complex fabfiles is using vagrant or using a VM
manually. I think docker is easier to use in scripts and way faster.

In this example, we're testing the deployment of a website using fabric, so our
basic testcase would be that the url to which we're deploying is returning a
200. For the sake of simplicity, let's assume that we have docker installed on
     www.example.com and we're deploying to the same address.

We can use fabric to create a new container and use it to run our provision and
deployment commands (all() does both here). We can use the "dhrp/sshd"
container image as mentioned in the [earlier blog
post](http://agiliq.com/blog/2013/06/deploying-django-using-docker/) to forward
ssh and web ports. One minor quibble might be that the dhrp/sshd image doesn't
have your public key by default so your fab command would be interrupted for a
password, but using ssh-copy-id should fix that. You could commit this change
and use this image next time for a permanent fix.

Here's a rough idea of how our fabfile would look like:

    import requests
    from fabric.api import settings

    def docker(cmd):
        return run("docker %s" % cmd)

    def self_test():
        host = 'www.example.com' #  test server with docker installed
        with settings(host_string='%(user)s@%(host)s' % {'user': 'me', 'host': host}):
            container = docker('run -d -p 22 -p 80 dhrp/sshd /usr/sbin/sshd -D')
            ssh_port = docker('port %s 22' % container)
            web_port = docker('port %s 80' % container)
            container_host = '%(user)s@%(host)s:%(port)s' % {'user': 'root', 'host': host, 'port': ssh_port}
            try:
                with settings(host_string=container_host):
                    all()  # provision and deploy to the container
                    url = "http://%(host)s:%(port)s" % {'host': host, 'port': web_port}
                    assert requests.get(url).status_code == 200
            finally:
                run('docker kill %s' % container)

and voila:

    fab self_test


