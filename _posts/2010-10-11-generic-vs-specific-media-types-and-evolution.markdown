---
author: algermissen
comments: false
date: 2010-10-11 22:59:54+00:00
layout: post
slug: generic-vs-specific-media-types-and-evolution
title: Generic vs. Specific Media Types and Evolution
wordpress_id: 178
---

In addition to the loss of self-descriptiveness there is another problem with generic media types that is not so obvious. 

Let's think about the problem from the point of view of someone that needs to conduct a change impact analysis. Suppose there is some REST Web service component C, for example an incident management system, that needs to be changed due to some new business requirement. Also suppose the change applies to the resource implementation that allows for retrieval of incident representations. Specifically, the entity needs to be augmented and changed a little.


    
    
    GET /incidents/554
    
    200 Ok
    Content-Type: application/xml
    
    <incident>
      ... whatever happened to what CI ...
    </incident>
    



With the generic media type approach above change impact analysis is quite challenging because it is difficult to figure out, which assumptions the consumers actually make about the entity. Thorough change impact analysis would require the change manager to contact the client owners and investigate their assumptions. Based on that information it is then possible to make changes without breaking existing clients.


    
    
    GET /incidents/554
    
    200 Ok
    Content-Type: application/incident
    
    <incident>
      ... whatever happened to what CI ...
    </incident>
    



In contrast, when using a specific media type, there is no need for change impact analysis because the media type itself acts as the contract within which the entity can be changed.

In other words, REST's constraint of specifically typed entities provides for a perimeter within which the server can be flexible without a need to consider the effect the changes have on clients.

Viewing media types as a perimeter for server side change is an important consideration when designing media types. How big you make the perimeter directly determines the bounds within which the server can later on evolve and, of course, how much burden you put on the client for dealing with entities of that media type.
