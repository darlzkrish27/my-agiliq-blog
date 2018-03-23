---
layout: post
title:  "Provisioning Made Easy With Chef"
date:   2012-05-29 13:30:00+05:30
tags: provisioning
author: saket
---
Managing Servers and scaling at will is hard. The bigger web players
have put in years of effort, brute-forcing at times and re-iterating
their methodologies, have come up with the secret sauce which they use
to keep their cloud cool. The article is meant to be introductory rather than
comprehensive. Chef has a steep learning curve, and in this post I
wish to simplify few of the basic concepts. So, instead of going to
the Chef Docs, finding searching, reading and re-reading, few of the beginner
concepts are put at one place, which will help you get started on your
own.  


SysAdmins have started adopting programming paradigms to manage
infrastructure. Infrastructure is managed as code. Preparing a server
is just running few codes, that's how easy it becomes with a
provisioning tool as chef. Chef is a  library for configuration
management its an API for your entire infrastructure. Using Chef helps
you improve scalability and manageability. It helps you save time,
money and the pains of doing the same repetitive tasks time and
again. 

## The Chef Architecture:

Chef Server:  Central Store of your infrastructure’s configuration
data. It stores data necessary to configure your nodes, a REST API
makes this data accessible to the nodes.  Optionally, it has a WebUI
which helps you manage your infrastructure via a web interface.

Nodes: Systems managed by Chef Servers. A node is any host that is
configured using chef-client. Chef-client runs on your node which
contacts the server for information necessary to configure the node. 
In simple terms nodes are your servers in the cloud which you want to
configure using chef. They can be your webservers, database server
etc. 

Workstations - The machine where the sysadmin works. Cookbooks are
maintained at the workstation. The workstation has two key components:

* the knife executable - helps upload data to chef server or individual nodes using ssh. 
* a repository containing cookbooks, data bags, roles etc.(infrastructure’s configuration documents)


### Few Basic Definitions: 

Lets look at few of the basic definitions which we will use later:

Resources are  a cross platform abstraction of the things you are
configuring on the host.

Parameters are the piece of data that the resources contain. 

Action in a resource is what you want to do with the resource. 

Providers are the specific implementation of what the resource
abstracts. They run the system commands. 

Roles provide a way to describe a particular function or type of
node. Roles have run_lists and attributes. 

Templates are all the files with all the important data substituted
with code, so we can fill them later. 

Recipes are where you write your resources. The execution is done in
top down order. 

Mentioned below is how a cookbook folder looks like:
<pre>
.
|-- attributes
|   -- default.rb
|-- CHANGELOG.md
|-- CONTRIBUTING
|-- LICENSE
|-- metadata.rb
|-- README.md
-- recipes
   -- default.rb
</pre>

Cookbooks can be found at Github, they are actively maintained by
opsCode and the community support. This was a basic intro to Chef, in
later posts we will see about the flavors of chef, and which one to
use when. This shall act as a building block to the next series of
posts. Stay Tuned!

