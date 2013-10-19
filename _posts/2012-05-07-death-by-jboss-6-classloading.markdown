---
author: algermissen
comments: false
date: 2012-05-07 22:19:13+00:00
layout: post
slug: death-by-jboss-6-classloading
title: Death by JBoss 6 Classloading
wordpress_id: 91
categories:
- Java Class Loading
- JavaEE6
- JBoss6
---

During the last week I have spent hour after hour to debug some strange behavior shown by some of my JAX-RS provider classes in two JavaEE6 projects. When I deployed the WARs of the two projects suddenly in one of the projects all providers from both projects where available and in the other none, ... zero, zip.

Naturally, I only saw this by accident, after an extensive two-day search in one of the projects for a potential bug caused by myself. Dang! - When I, frustrated as I was, re-deployed the other project I saw the providers I had been looking for in the first deployed app.

Yes, I know now, that I should have thought about the interference between the two right away...but you know how these things go.

When it finally dawned on me that I fell victim to some default configuration in JBoss that (at least to me) is totally counter intuitive the search went reasonably well. (Though documentation at jboss.org is still way to hard to find for my taste)

I figured I had to isolate my class loaders with something like this:


    
    
    <code><?xml version="1.0" encoding="UTF-8"?>
    <jboss-web>
    <class-loading>
          <loader-repository>com.example:archive=myapp-1.war
            <loader-repository-config>java2ParentDelegation=false</loader-repository-config>
          </loader-repository>
       </class-loading>
    </jboss-web></code>
    



The crappy thing about this is that apparently the reason for the non-intuitive default is that versions prior to JBoss 5 exhibit not-isolated class loading. The designers of JBoss 5+ desired to make the old configurations work in the newer JBosses without adjustments. Makes me shiver.

**Edit**: Turns out you also need to set java2ParentDelegation=false in the example above to make it work.

**Edit**: Nope - the above also did not do the trick. After a wasted week of digging around, I give up now, simply installing another JBoss (unable to switch to Glassfish due to external forces :-(.

Useful references:
[https://community.jboss.org/wiki/ClassLoadingConfiguration](https://community.jboss.org/wiki/ClassLoadingConfiguration)
[https://community.jboss.org/wiki/JBossClassLoadingUseCases](https://community.jboss.org/wiki/JBossClassLoadingUseCases)

