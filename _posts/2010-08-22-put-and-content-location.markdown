---
author: algermissen
comments: false
date: 2010-08-22 22:54:07+00:00
layout: post
slug: put-and-content-location
title: PUT and Content-Location
wordpress_id: 159
categories:
- HTTP Tips
---

I noticed some days ago that I totally missed the use of the Content-Location header in responses to PUT requests. Used in this scenario, the client does not need an additional GET to obtain a representation of the resource after the PUT.

And what is even more interesting is that caches can also cache the response of the PUT.



    
    
    
    PUT /orders/42
    Content-Type: application/order
    [...]
    
    200 Ok
    Content-Location: /orders/42
    Content-Type: application/order
    ETag: "1"
    [...]
    



The Content-Location header is telling the client and intermediary: "This is the entity and entity headers you would receive for a GET to /orders/42".
