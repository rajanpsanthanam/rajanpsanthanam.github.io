---
title: Healthchecks.io - Cron Monitoring
layout: post
author: Rajan Santhanam
date: '2020-04-22'
categories:
- product
---

**Hey there!**

Let me talk about cron first and also explain why being a product person i am kind of talking about cron monitoring. Then we can jump into healthchecks.io to see how we can monitor cron job

![](https://rajanpsanthanam.s3.ap-south-1.amazonaws.com/healthchecks-post.png)

**Basics on cron and cron expression**

A simple defintion on cron from the web

> Cron Jobs are used for scheduling tasks to run on the server. They're most commonly used for automating stuff

 Cron is something that you see everywhere, it comes handy in terms of automating things, schedule jobs etc and there is very likely chance of it getting used in the place or product you work or deal with.
 
** Cron expressions** - its nothing but how you schedule the cron job. It has a simple syntax.
 
` * * * * * [command]`

each * from left to right mean the below
* minute
* hour
* day of month
* month
* day of week


Lets check on example 

`0 * * * 1 [command]`

This will run on mondays, every hour. 24 times a day but only on monday.

Let me not get into more on cron, you can read about them on the web.


**Need on cron monitoring**

Monitoring such cron jobs is key because once you schedule something important and it stops suddenly, you should be knowing about it.

Let me talk in terms of usecase. Lets say, you are in need of some important report and it has to come from different server/system. There is a cron running on it which extracts the report and sends you an email. This calls for a monitoring as you will miss the report if cron stops

Being a product person, you would end up in many usecases where you would be talking help of some scheduling system to get something done. Cron would be obvious first choice for most developers. Knowing about cron in general and knowing how to monitor it gives you extra wing :P

**Healthchecks.io**

[healthchecks.io](https://healthchecks.io/) is a simple application which solves the problem effectively.

All you need to do is create a check for your job in healthchecks.io dashboard and add few lines of code (you can fetch this as well from their dashboard based on the programming language in which your job is written)at the end of your job. Thats it.


It has lots of integration to send alerts when you have a prob on the job.


Its obvious question that on what basics i decide my cron is not working, i mean how would healthchecks.io know my cron's running frequency. You can set it while creating the check, you can say if this job is not running for 3 continuous hour, do send me an alert


healthchecks.io is a open source project, you can host it in your own server as well.


**Thanks for your time.**
