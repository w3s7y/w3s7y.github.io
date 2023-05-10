---
title:  "New Blog, New Project"
date:   2017-06-02 15:34:01 +0100
categories: blog
---

### Introduction
So barely a few days pass since bringing this blog on the air and already a new
and exciting opportunity has been presented to me.  A few weeks ago I was in
conversation with one of the data scientists I work with, after a while we get
talking about this side project he has been thinking about working on in his spare
time.  I have a lot of patience and respect for people that do this, it shows that
they have a real passion for IT and are not just doing it for the paycheck!

### The plan
The primary deliverable for the project is a virtual machine which can be shipped
around our company internally (or even potentially delivered to the cloud) and
used by the data science community giving them an identical and common development
environment.  This is commonplace amongst development teams in web development,
however it quickly became apparent the more we spoke that data scientists
do not have a common environment on which to develop machine learning models or
perform statistical analysis on data.  This leads to the classical "Well it works
on my machine." scenario and bugs in code which cannot be reproduced due to people
not working in the same environment.

This machine is envisaged to have all of the common data science libraries and languages
pre-installed and configured (R, Python with sklearn, matplot and others) and also
to have a method to run other services which would be needed for a machine learning
scenario (backend databases, code repositories, data generators and the like).
This gives all data scientists a take anywhere, run anywhere toolkit which in theory
should have all of the software they will ever need.  As by his own admission data
scientists are not engineers and therefore getting them to configure a backend
database for example by hand is probably a stretch too far, even with good docs!
The intention is to give them the capabilities to let them use a familiar IDE to
edit their code and run in an environment which can be easily customised for
varying scenarios or machine learning problems.

### The job
Skip forward a few days and we get onto the topic of Ansible and he wondered if
I could give him some basic framework code to get him off the ground.  Obviously
I jumped at the chance to get someone else on the Ansible hype and help out a
colleague with the same passion for hacking (not cracking) and knocked out a
simple Ansible role which would provision a Debian Jessie box by installing
[Python](https://python.org) and doing a few pip installs of common scientific
libraries, [Docker](https://docker.io) and a few other tools like docker-compose
and finally reconfigure the iptables (firewall) to block all traffic and allow
some basic (SSH) ports inbound.

After handing the code over and showing him a test execution on a server running
in the AWS cloud, he was seemingly impressed and said to leave it with him and
that he would come back to me if he needed any more help.  That I thought would
be the sum of my contribution to his great little side project.  Well I couldn't
have been more wrong!

### The take-off
Fast forward to today and I learn that a senior (VP) of the company I work for
has learned of this project and now wants to fund it and run it as a formal internal
project.  We have been given an AWS account to play around in and basically been
told that yesterday is the delivery date! (why does that seem so familiar...).  
There is certainly now a buzz around this project and I have been formally invited
on to deliver the Ansible component of it... happy days!

This is all the information I have on the project right now however as the weeks
progress I will certainly be keeping you all up to date with updates and maybe
even some `code snippets` if your lucky!  

Thanks for reading and please don't hesitate to hit me up via Twitter to discuss!
