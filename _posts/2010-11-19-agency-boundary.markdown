---
layout: post
title: Agency Boundary
---
{% include JB/setup %}

A central aspect of the problem space that REST is designed for is that the communicating components are typically not owned by a common authority and that therefore change must be enabled to happen without coordinating the change between the authorities. For example, for Amazon it is simply impossible to ask any of its customers whether such and such a change 'would be ok'.

In my recent [presentation on RESTful HTTP](http://www.slideshare.net/algermissen/res-tful-httppatternsantipatterns) I used the term 'administrative boundary'. Today, I came across the much better term _Agency Boundary_ coined by Rohit Khare and Richard N. Taylor in their ICSE'04 paper [Extending the REpresentational State Transfer (REST) Architectural Style for Decentralized Systems](http://portal.acm.org/citation.cfm?id=999447):

> 
An agency boundary denotes the set of components operating on behalf of a common (human) authority, with the power to establish agreement within that set and the right to disagree beyond it it.
