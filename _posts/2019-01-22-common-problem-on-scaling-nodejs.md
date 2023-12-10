---
layout: post
title: Common Problem on Scaling Nodejs
---

I have built lots of NodeJS apps and scaled them to millions of requests. The most common problem you will see is Garbage collection pauses and memory leaks which sometime gives random distribution of time response of requests. The problem is not the GC but the way you have written code.

Another is Memory leaks you will find them in production and see your app crashing a lot and no idea why it crashed because locally everything seemed to be fine but you gotta find them.

Sometimes you may have to tweak V8 heap limit and another constraints in order to scale it well and so that it doesn’t crash at its default heap size limit.

Another issue which is quite common is forgetting error handling and crashing the main app you may be using supervisor, forever, pm2 or simply upstart but at high scale the frequency of crash may increase and you have to fix those. Restarting is not a solution it can be bad experience of some users if suddenly while processing their request you app restarts.

Also if you are using single process in production that is also bad and can bite you. Try proxy like Nginx along with node cluster mode. With proxy you can scale across multiple machine without the need of an ELB.

This is a naive problem but happens with users who just started learning javascript, they block the request which has come for client and do something else in background if you do these for many request you are basically filling up the queue which is bad in general.