---
author: algermissen
comments: false
date: 2003-04-05 11:40:34+00:00
layout: post
slug: building-web-services
title: Building Web Services
wordpress_id: 172
---

In the first issue of [ACM Queue](http://queue.acm.org) there is an interesting [interview](http://queue.acm.org/detail.cfm?id=640150) with [Adam Bosworth](http://adambosworth.wordpress.com). The issue has been lying in my cupboard since March and I only discovered this interview yesterday.

Here is a quote:





> There also are transactional integrity issues to consider. A database can maintain a transaction over a short period of time in a client/server setting. But if you have one application going to another to ask for 1,000 different things at 1,000 different points in time, the odds that consistency will be maintained are pretty slim. So what would make a lot more sense would be to ask for everything you want from that medical record at the same time. That way, it's just a single transaction over the Web. And then inside the hospital system, they can pull together all the needed data and hand it back as one big screen chunk. Now that model is very efficient for the Web and it doesn't require you to maintain state. Basically, you've got to take a coarse-grain approach like that or your system just isn't going to scale. That's true in the database world and it's even truer in the application world.





Interestingly, the interview sounds as if Bosworth was arguing pro REST-style services!
