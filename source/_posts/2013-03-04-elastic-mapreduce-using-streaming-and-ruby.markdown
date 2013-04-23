---
layout: post
title: "Elastic MapReduce using streaming and ruby"
date: 2013-03-04 01:57
comments: true
categories: [AWS, EMR, Ruby, MapReduce] 
---

[Amazon Elastic MapReduce](http://aws.amazon.com/elasticmapreduce/) - Is a service provided by amazon web services intended to provide facilities to build fast and easy to maintain big data calculations using MapReduce paradigm. Basically, Amazon EMR is a hosted Hadoop, at first glance. But in combination with aws API and other services, like S3, EMR becomes quite feature full solution. Hadoop requires some extra skills to deploy it locally. At least you need to be familar with JVM infrostructure. EMR allows you to abstract from some technical aspects like deploying cluster and managing nodes.

Also, which is nice about EMR that you can write MR functions in any language, not necessary in Java. 

Before we will start to work on the topic itself, here are some requirements you need to meet to be able to use samples:

  *   Ruby programming language - we will use 1.9.3-p392 version (on the moment of blogging that is 1.9.3-head)
  *   Amazon Web Services account - if you don't have one please create, subscribe for EMR (Elastic MapReduce) and S3

The main goal is to be able to process huge amount of data (we will take log files from nginx server from one of my applications) and generate statistic. 
We will investigate amount of hits per browser families per day. I would like to note that it is just an example of how you can use amazon EMR for such kind of cases.

About MapReduce concept you can read on different sources, starting from [Goolge's whitepaper](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/uk//archive/mapreduce-osdi04.pdf), [wiki](http://en.wikipedia.org/wiki/MapReduce), [Hadoop sources](http://hadoop.apache.org/docs/r1.0.4/mapred_tutorial.html).

I will not cover the theoretical principles of MR, since they covered quite well in sources above. Besides, I think It is easier to get that on realworld example. So please, read the references, at least article in wikipedia.

First our source file - I will use nginx access.log from demo server and the use case will be is to collect amount of hits per browser family. Basilcaly, I dont care about period, let's say we have 1 log file per day and we will use as input file that one only.

Every access.log line formated with "combined" format: 

    '$remote_addr - $remote_user [$time_local]  '
        '"$request" $status $body_bytes_sent '
        '"$http_referer" "$http_user_agent"';

And in real life looks like this:

    127.0.0.1 - - [23/Apr/2013:00:10:02 +0100] "GET / HTTP/1.0" 403 168 "-" "Wget/1.12 (linux-gnu)"
    193.169.146.189 - - [22/Apr/2013:09:07:12 +0100] "GET /login HTTP/1.1" 200 1651 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_3) AppleWebKit/536.28.10 (KHTML, like Gecko) Version/6.0.3 Safari/536.28.10"
    83.161.144.255 - - [22/Apr/2013:09:08:22 +0100] "GET /login HTTP/1.1" 200 1648 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.31 (KHTML, like Gecko) Chrome/26.0.1410.64 Safari/537.31"

Your log