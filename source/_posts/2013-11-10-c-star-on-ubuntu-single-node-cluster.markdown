---
layout: post
title: "C* On Ubuntu Single-Node Cluster"
date: 2013-11-10 23:07
comments: true
categories: NoSQL, Cassandra, C\*, DBMS
---

Due to http://cassandra.apache.org/download/ - The latest stable release of Apache Cassandra is 2.0.2 (released on 2013-10-28).

I'v googled quite a while to get the most true way of C\* installation under the Ubuntu. No luck actually, for a some reason C\* doesn't have any well supported PPA or so. And looks like the only way to get it is to download from site and go for a manual.

Update: DataStax hosts some Debian packages for C\*, but 1 - they are looks old, 2 - they are for Debian not for Ubuntu. So lets go for a manual.  

Update 2: My colleague showed me that [link](http://www.datastax.com/documentation/gettingstarted/getting_started/gettingStartedDeb_t.html), which describes the process for debian/ubuntu packages. 

Installation on Ubuntu 12.04:
=============================

Lets make our host up to date:

    sudo apt-get update && sudo apt-get upgrade -y

Before we will get Cassandra we need to install Java. I will go fo open-jdk:

    sudo apt-get install openjdk-7-jdk openjdk-7-jre -y

Lets check our java:

    java -version

You should get some thing like:

    java version "1.7.0_40"
    Java(TM) SE Runtime Environment (build 1.7.0_40-b43)
    Java HotSpot(TM) 64-Bit Server VM (build 24.0-b56, mixed mode)

Now we can get Cassandra:

    cd ~ && wget http://apache.mirror1.spango.com/cassandra/2.0.2/apache-cassandra-2.0.2-bin.tar.gz
    tar xvf apache-cassandra-2.0.2-bin.tar.gz
    sudo mv apache-cassandra-2.0.2 /opt

Now C\* looks like "Installed", we still need to create startup script and configure some directories with correct permissions.

Since my interest is purely academic - I'm not going to build HA solutions so far, but just want to play with C\* a bit I will handle start/stop manually.

So, start daemon with:

    sudo /opt/apache-cassandra-2.0.2/bin/cassandra -f

After first run Cassandra will be happy to tell you that it created some dirs and files there and here. And it will start to listen some port, I belive that's 9160. Also -f option says foreground to daemon so it will attach to your screen and you probably need to open new one. BTW this one can be stopped with CTRL+C. Lets connect and check if everything works:

    /opt/apache-cassandra-2.0.2/bin/cqlsh

Should promt you next:

    Connected to Test Cluster at localhost:9160.
    [cqlsh 4.1.0 | Cassandra 2.0.2 | CQL spec 3.1.1 | Thrift protocol 19.38.0]
    Use HELP for help.
    cqlsh>

Basics
======

Cassandra operates with keyspaces - namespaces for tables. Which I think are databases in other DBMSes. Hehe, Cassandra is KSMS - keyspace management system.

Lets create one keyspace:

    CREATE KEYSPACE mykeyspace WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 1 };  

So far that's black magick and we don't know anything about meaning of the words in command above. 

To switch to newerly created keyspace do:

    USE mykeyspace;

Now we can create our first table. Example says it should be users. We don't have any problems with that, so:

    CREATE TABLE users (
      user_id int PRIMARY KEY,
      fname text,
      lname text
    );

Unlike mongodb you have to explicitly specify the schema. As well as you have to create tables.


Lets insert some users. Again from examples:

    INSERT INTO users (user_id,  fname, lname)
      VALUES (1745, 'john', 'smith');
    INSERT INTO users (user_id,  fname, lname)
      VALUES (1744, 'john', 'doe');
    INSERT INTO users (user_id,  fname, lname)
      VALUES (1746, 'john', 'smith');

And get all the users:

    SELECT * FROM users;

If we want to search by particular field not primary key we need to specify an index on that field. Basically, If we want to query we need to have covered indexes, as far as I understand.

so without extra indexes:

    SELECT * FROM users WHERE lname = 'smith';

Will produce an error: Bad Request: No indexed columns present in by-columns clause with Equal operator.

Probably thats good, since from 1st query we can see already if we need indexes or not. 

Adding indexes is very simple, just:

    CREATE INDEX ON users (lname);

After that it works like it should.

Conclusions
-----------

It was very fast and brief of how you can start with Cassandra on Ubuntu. I really hope that I will have more time to dig deeper and get actually what is so big deal about C\*.
