---
title: 'My NodeJs setup for production'
date: Wed, 15 Jan
draft: false
tags: [Web, Nodejs, CI,CD, Production Deployment]
---
### Post in Progress
At the end of 2019 I decided to learn two new technolgies NodeJs and react while building an App (https://tr.bsid.io). Previuously I used to build my side projects using php and Javascript with no version control. I used to backup these projects to my Google Drive for safety and trust me keeping backup of backup is very important I learnt this the hardway(When my WD Backup drive failed lost 1TB of Data). While researching online on "How to Deploy nodeJs in production"  I came across PM2. I wish I could have learnt about Node+PM2 earlier both have made my life easier.
So in this post I will be explaining How I am using PM2, Travis, CodeCov and git to test and deploy my side projects to production.

As the number of side projects grew. I had to stream line the way I build and deploy these projects with time and cost in mind. I moved all the side projects with static content to Netlify and I use a Digital Ocean VPS for Dynamic sites with Databases.Before you proceed any futher Please note I am a Learner/Beginner and sometimes I may get few things wrong and for that reason do your due diligence when deploying in Production. Let's begin....

What is PM2?

PM2 is a nodeJs process manager for production with built in Load-balancer. It works on Mac, Linux and Windows. With a simple Config file, you specify what processes to run and scale and PM2 takes care of the rest. This allows you to keep your NodeJs Application alive forever, and to reload with zero downtime when you have updates to your application.