####
Docker containers deployment with OpenStack Heat
####


:Version: 1.0
:Authors: Marouen Mechtri and Chaima Ghribi 
:License: Apache License Version 2.0
:Keywords: Docker, Heat, OpenStack, Icehouse, Ubuntu 14.04


===============================

**Authors:**

Copyright (C) `Marouen Mechtri <https://www.linkedin.com/in/mechtri>`_


Copyright (C) `Chaima Ghribi <https://www.linkedin.com/profile/view?id=53659267&trk=nav_responsive_tab_profile>`_


================================

.. contents::


1. Overview
============

We’ve all heard about Docker and the immense amount a buzz around it from last year !

What’s Docker ? why is it useful ? how can we use it and integrate it into Openstack ? 
and how to deploy our applications with it ? 

All these questions came to our minds when we started discovering Docker.

So, we wrote this manual to share with you our experience with Docker and OpenStack.
We will first introduce Docker and show you how it can be integrated into Openstack. Then,
we will explain how to successfully deploy your docker containers with Openstack Heat. 

All the steps of installation and deployment are described in details.
Hope our step by step instructions are easy to follow, even for beginners.


2. What Is Docker?
==================

Docker is an open source project to automatically deploy applications into containers. 
It commoditizes the well known LXC (Linux Container) solution that provides operating system
level virtualization and allows to run multiple containers on the same server. 

To make a simple analogy, a Docker is like an hypervisor.  But unlike traditional Vms,
docker containers are lightweight as they  don’t run OSes but share the host’s operating system (see the figure below).

A docker container is also portable, it hosts the application and its dependencies and it is able
to be deployed or relocated on any Linux server.

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/docker-vs-hypervisor.jpg

The Docker element that manages containers and deploys applications on them is called Docker Engine.

The second component of Docker is Docker Hub. We found it really Awesome ! 
It's the Docker's repository of application. You can share your application with your team
members and you can ship and run it anywhere... It's really amazing :)

It was a quick introduction to Docker ! Now, let's see how to use it with OpenStack ;) 

3. OpenStack & Docker
======================

Openstack can be easily enhanced by docker plugins. 
Docker can be integrated into OpenStack Nova as a form of hypervisor (Containers used as VMs).
But there is a better way to use Docker with OpenStack.
It to orchestrate containers with OpenStack Heat !

You have just to install the docker plugin on Heat and you will be able to create
containers and deploy your applications on the top of them.
You need to identify the required resources, edit your template and deploy it on Heat. Your stack will be created!
(For detailed information on Heat and template creation, you can see our `Heat usage guide <https://github.com/MarouenMechtri/OpenStack-Heat-Installation/blob/master/Create-your-first-stack-with-Heat.rst>`_). 


If you want to test Docker with Heat, we recommand you to deploy OpenStack using a flat networking model (see our `guide <https://github.com/ChaimaGhribi/Icehouse-Installation-Flat-Networking>`_).
One issue we encountered when we considered a multi-node architecture with isolated neutron networks is that 
instances were not able to signal to Heat. So, stack creation fails because it dependens on 
connectivity from VMs to the Heat engine. 

The figure below shows the communication between Heat, Nova, Docker and the instance when creating a stack. 
In this example we have deployed apache and mysql on two Docker containers. Stack deployment fails
if the signal (3) can not reach Heat API.

I think this limitation will be overcome in the next versions to allow
users using isolated neutron networks to deploy and test Docker!

But now you can enjoy docker on OpenStack with Flat-networking ;)
  

.. image:: https://raw.githubusercontent.com/MarouenMechtri/OpenStack-Heat-Installation/master/images/docker-plugin.jpg

4. Deploy Docker containers with OpenStack Heat
===============================================

Now that we have discussed about Docker and Heat, let's move to practice !
We will show you how to install the Docker plugin, how to write your template and how to deploy it with Heat ;)


4.1. Install the Docker Plugin 
--------------------------------

* To get the Docker plugin, download the Heat folder available on GitHub::

    download heat (the ZIP folder) from here
    https://github.com/openstack/heat/tree/stable/icehouse

* Unzip it::

    unzip heat-stable-icehouse.zip


* Remove the tests folder to avoid conflicts::

    cd heat-stable-icehouse/contrib/
    rm -rf docker/docker/tests

* create a new directory under /usr/lib/heat/:: 

    mkdir /usr/lib/heat 
    mkdir /usr/lib/heat/docker-plugin

* Copy the docker plugin under your new directory::

    cp -r docker/* /usr/lib/heat/docker-plugin
  
* Now, install the docker plugin::

    cd /usr/lib/heat/docker-plugin
    apt-get install python-pip
    pip install -r requirements.txt  
    
    
* Edit /etc/heat/heat.conf file::

    vi /etc/heat/heat.conf
    (add)
    plugin_dirs=/usr/lib/heat/docker-plugin/docker
 
 
* Restart services::

    service heat-api restart
    service heat-api-cfn restart
    service heat-engine restart    
    

* Check that the DockerInc\::Docker\::Container resource was successfully added and appears in your resource list::

    heat resource-type-list | grep Docker 
    

4.2. Create your Heat template
-------------------------------

Before editing the template, let's discuss a bit about the content and the resources we will define ;)

In this example, we want to dockerize and deploy a lamp application. So, we will create a docker 
container running apache with php and another one running mysql database. 

We define an OS::Heat::SoftwareConfig resource that describes the configuration and an OS::Heat::SoftwareDeployment resource to
deploy configs on OS::Nova::Server (the Docker server). We associate 
a floating IP to the Docker server to be able to connect to Internet ( using OS::Nova::FloatingIP and OS::Nova::FloatingIPAssociation resources). 
Then, we create two docker containers of type DockerInc::Docker::Container on the Docker host. 

Note: here we provide a simple template, many other interseting parameters ( port_bindings, name, links...) can enhance 
the template and enable more sophisticated use of Docker. These parameters are not supported by the current Docker plugin.  
We will provide more complex templates with the next plugin version. 
