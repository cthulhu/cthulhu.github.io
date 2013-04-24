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

  *   Ruby programming language skills - not much, since ruby is quite expressive language
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

Your log files should be placed on amazon S3 so EMR will be able to get access to them. 

For my test cases I used 1 bucket with next structure inside:

    bootstrap.sh  
    browsers.rb
    input  
    output

bootstrap.sh - bash file for software installation on every EMR node. Basically, you can setup gems and libs, compile needed externall resources etc with it.

My file is very simple:

    #!/bin/bash
 
    sudo apt-get -y install rubygems

Since rubygems depends on ruby, we will get it installed as well. I can imagine that in reallife cases bootstrap scripts will be much more complex.

browsers.rb - that is basically our map function. In my example it looks like:

{% gist 5446482 %}

That example will never win first price, but any way it works and shows main principles. 

The main idea behind is that data actually goes to mapper STDIN line by line, after that we parse the lines and generate the emission to the STDOUT. It is quit simple and can be done actually in any language even shell.

To get started go to aws console, your favourite region and select s3. After that create bucket, in my case it was emr.workshop. Upload workshop directory with content above to that bucket. As a result you should see some thing like this:

{% img /images/s3-emr.jpg [width] [height] %}

Input folder should contains access.log file with data.

*Note:* I prefer to create a separate user and group when do some actions in aws. That guaranties some rights and visibility isolation, especially when you have quite a lot of critical and important applications running under same account. You can easily do that with IAM(Identity and Access Management) and policies generators. I will not cover that topic here, since it's quite nice described in aws documentation.

Once we loaded input and created policies(you can skip that if you want), we can build our MR-job. 

Go to aws elastic map reduce and click "Create New Job Flow". Fill the name of the job, in my case is "Browsers Analysis", select hadoop version "Amazon Distribution", Select "Run your own application" as a job flow.

Result should be like that:

{% img /images/create-job-step1.jpg [width] [height] %}

On the next step you should fill in MR-job parameters. For me it was: 

* Input location: s3://emr-workshop2013/input/access.log
* Output location: s3://emr-workshop2013/output/
* Mapper: s3://emr-workshop2013/browsers.rb
* Reducer: aggregate

{% img /images/create-job-step2.jpg [width] [height] %}

Next you need to pick up hardware for the job. That is really depends on your case and expectations. If you have 100 Gb log file, and you want to process it faster than you probably need to think about more nodes. In our example everything is quite simple and log file is not very big so we will pick up few small instances(basically default options). You should also consider [costs](http://aws.amazon.com/elasticmapreduce/pricing/) when you pick up size of computation cluster. 

On Advanced options step you need to configure additional options for the job. I would say use defaults and press continue, since in test example all those settings do not matter much.

On next step you will have to configure bootstrap options for your nodes. All our nodes will be bootstrapped same using our script loaded to s3 bucket above. So Select "Configure your Bootstrap Actions", Action type: "Custom Action", Name: Node bootstrap. Amazon S3 Location: s3://emr-workshop2013/bootstrap.sh.

{% img /images/create-job-step5.jpg [width] [height] %}

On last step of the wisard you will be able to review job's settings.

After you will press "Create Job Flow" Amazon will start instances and the computation will start. It will take few minutes to bootstrap the nodes, and ofcourse some time to process the data from input log file. As a result in output folder you will have a bunch of text files with aggregated counts per browser family. 

How to debug MR-job?

Since job(mapper) gets data from STDIN you can easily do that with just piping in linux command line:

    cat input/access.log | ./browsers.rb

will give you output from the mapper, and with:

    cat input/access.log | ./browsers.rb | sort | uniq -c

You will have results, close to aggregate reducer output. In my case it was:

          1 LongValueSum:Chrome ["10"]  1
         14 LongValueSum:Chrome ["11"]  1
          2 LongValueSum:Chrome ["12"]  1
          1 LongValueSum:Chrome ["14"]  1
          1 LongValueSum:Chrome ["15"]  1
          2 LongValueSum:Chrome ["16"]  1
          3 LongValueSum:Chrome ["17"]  1
         90 LongValueSum:Chrome ["18"]  1
          1 LongValueSum:Chrome ["20"]  1
          7 LongValueSum:Chrome ["21"]  1
         25 LongValueSum:Chrome ["22"]  1
       1826 LongValueSum:Chrome ["23"]  1
         10 LongValueSum:Chrome ["24"]  1
          3 LongValueSum:Chrome ["25"]  1
          1 LongValueSum:Chrome ["5"]   1
          1 LongValueSum:Chrome ["7"]   1
          1 LongValueSum:Chromium ["20"]    1
          1 LongValueSum:Chromium ["22"]    1
         15 LongValueSum:Firefox ["10"] 1
          8 LongValueSum:Firefox ["11"] 1
         26 LongValueSum:Firefox ["12"] 1
         13 LongValueSum:Firefox ["13"] 1
         30 LongValueSum:Firefox ["14"] 1
         23 LongValueSum:Firefox ["15"] 1
         56 LongValueSum:Firefox ["16"] 1
        940 LongValueSum:Firefox ["17"] 1
         31 LongValueSum:Firefox ["18"] 1
          2 LongValueSum:Firefox ["19"] 1
          1 LongValueSum:Firefox ["20"] 1
         22 LongValueSum:Firefox ["3"]  1
          1 LongValueSum:Firefox ["4"]  1
          1 LongValueSum:Firefox ["5"]  1
          4 LongValueSum:Firefox ["6"]  1
         27 LongValueSum:Firefox ["8"]  1
          9 LongValueSum:Firefox ["9"]  1
       3042 LongValueSum:Internet Explorer  1
         96 LongValueSum:Opera ["9"]    1
         65 LongValueSum:Safari     1
          1 LongValueSum:Safari ["3"]   1
        953 LongValueSum:Safari ["4"]   1
        683 LongValueSum:Safari ["5"]   1
       1407 LongValueSum:Safari ["6"]   1
         12 LongValueSum:Safari ["7"]   1

Also during MR-job creation you will be able to set bucket for MR process logging and ability to access nodes to check software etc(Step Advanced Settings).

This was very basic overview of EMR abilities, I hope in the nearest future I will be able to present some more complex usecases and applications of that technologie.