---
layout: post
title: "Elastic MapReduce using streaming and ruby"
date: 2013-03-04 01:57
comments: true
categories: [AWS, EMR, Ruby, MapReduce] 
---

Before we will start to work on the topic itself here are some requirements you need to know to understand the workarround:

  *   Ruby - we will take 1.9.3-head version
  *   Amazon Web Services account - if you don't have one please create, subscribe for EMR(Elastic MapReduce) and S3

Goal
----

The main goal is to be able to process huge amount of data (we will take log files from nginx server from one of my applications) and generate statistic. 
We will investigate amount of hits per browser families per day. I would like to note that it is just an example of how you can use amazon EMR for such kind of cases.

