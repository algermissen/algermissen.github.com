---
author: algermissen
comments: false
date: 2010-11-12 23:56:56+00:00
layout: post
slug: interesting-changes-must-surface
title: Interesting Changes *Must* Surface
wordpress_id: 176
---

The human brain is truly strange. It happens surprisingly often that you are (well, at least I am) so tied-up in a false assumption that the ability to understand something new is seriously limited.

Then, when the clouding assumption disappears you have these special, enlightening aha-moments.

In this particular case I have been thinking a lot about change impact lately. The false assumption I made was that change would ideally be kept hidden and not surface towards the consumers of some provided capability.

Given that assumption, it seemed strange to maintain a certain RPC criticism I am trying to rationalize: if the goal is to hide change, what is the problem with RPC in the first place?

Then it dawned some minutes ago: Change to a component is essentially driven by requirements to do new stuff and in order to do new things with a component, the changes must surface. Changes that remain hidden are not worth talking about - these are called implementation details. Details that per definition do not matter at the capability level.

Now I can joyfully continue working on my RPC criticism: since changes must surface to be interesting we can compare how they can surface in the various architectural styles.

REST is the only style that requires changes to surface in a way that is guaranteed not to break clients, because REST is rather specific about what can change in which ways. Except for add-on changes (that are not a problem for existing clients) all surfacing changes to a REST server component are bound by the possibilities to vary representations (you may change as long as you do stick to rules of the media type used)

In RPC-based architectures, change is (you guessed it :-) not constrained at all and might surface in a million ways. Welcome to change impact analysis hell.

