---
layout: post
title: "GlusterFS on Ubuntu"
date: 2013-10-11 09:51
comments: true
categories: 
---

Motivation
----------

One part of my current project needs to write some files to some centralized storage. First I was using rsync, and than NFS to do so. 
Each of the options has its cons. No failover in both cases(Unless you will do that magic yourself). Rsync is poorly managed and it is actually designed for some other things. NFS works almost "fine", but in current setup it has single point of failure, this shared volume itself.

Installation
------------

Ubuntu is an OS of my choice so I will follow the installation instructions for that operations system. Last release version for GlusterFS is 3.4.
The most natural way for Ubuntu is to install stuff from PPA so here it is:

From http://download.gluster.org/pub/gluster/glusterfs/3.4/LATEST/Ubuntu/Ubuntu.README

    sudo add-apt-repository ppa:semiosis/ubuntu-glusterfs-3.4
    sudo apt-get update
    sudo apt-get install glusterfs-server

Lines above will install also glusterfs-client glusterfs-common for you.

Configuration
-------------

After we installed the binaries we need to configure some options to make it work as needed. 

Documentation says that most of the options you will get via [translators](http://europe.gluster.org/community/documentation/index.php/Translators). 

But first I would like we to go the simplest way. Just replicated filesystem with ability to mount it on other hosts. To do so we need to do next steps:

* Get host A and host B where we will run our cluster
* Prepare and mount storage devices on both hosts
* Create volume using glusterfs tools
* Start the gluster FS volume 

After those steps our cluster will be up and running and we will be able to access it from other machines.

For testing things like that and playing arround I'm using [Vagrant](http://www.vagrantup.com/). 

So, basically my Vagrantfile will be next for this small assignment: 

{% gist 6983893 %}

Shell script for provisioning nodes:

{% gist 7040042 %}

Conclusions
-----------

When I checked the glusterfs resources first time, I was very enthousiastic about it. But more I googled, more I got bad expiarences links. Personally I think it is important to understand that its really depends on user case. And you should not apply some one's faults to your setup.

More readings & links
---------------------

Bad experience:
  
* http://www.devco.net/archives/2010/09/22/experience_with_glusterfs.php
* http://www.mail-archive.com/gluster-users@gluster.org/msg08351.html
* http://www.softwareprojects.com/resources/programming/t-6-months-with-glusterfs-a-distributed-file-system-2057.html

Highscalability review:

* http://highscalability.com/product-glusterfs
