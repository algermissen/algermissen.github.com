---
layout: post
title: Beyond OAuth
---
{% include JB/setup %}

[Eran Hammer](http://hueniverse.com)'s [noisy departure](http://hueniverse.com/2012/07/oauth-2-0-and-the-road-to-hell/) from [OAuth 2](http://tools.ietf.org/html/rfc6749) woke me up to finally engage in that HTTP security investigation that had been buried in my todo list for years. Thanks a bunch for that, Eran!

Starting from close-to-zero security knowledge it took some time to understand the [points](http://hueniverse.com/2010/09/oauth-bearer-tokens-are-a-terrible-idea/) [he](http://www.ietf.org/mail-archive/web/oauth/current/msg00507.html) [is](http://hueniverse.com/2010/09/oauth-2-0-without-signatures-is-bad-for-the-web/) [making](http://hueniverse.com/2010/09/more-oauth-nonsense/) but finally it all came together: Yep - sadly he's spot on with his [criticism](http://hueniverse.com/2012/11/fuckoauth-realtimeconf/). If in doubt, look at the [OAuth 2.0 Threat Model](http://tools.ietf.org/html/rfc6819) or the [bloating OAuth related I-Ds](https://datatracker.ietf.org/doc/search/?name=oauth&activeDrafts=on).

Regarding his IETF criticism, I can see where he is coming from, given the amount of Big Co representatives in that WG, but I don't think the IETF is to blame in general. Personally I like written standards and I like the process to be slow to buffer change. In the end, it depends on the staffing I'd say, not the organization as such.

Suggestions? Steer clear of OAuth 2, it's a waste of time. Stick to [OAuth 1](http://tools.ietf.org/html/rfc5849) if you need something right now, or, much better, follow the new thing...

Fortunately [Eran resumed productivity](https://groups.google.com/forum/?fromgroups=#!forum/oz-protocol) and started work on a set of _the-code-is-the-specification_ modules for node.js that each cover a distinct aspect of what really smells like a useful successor of OAuth: [iron](https://github.com/hueniverse/iron), [Hawk](https://github.com/hueniverse/hawk), and [Oz](https://github.com/hueniverse/oz).

[iron](https://github.com/hueniverse/iron) implements a cryptographic procedure for encrypting arbitrary data and ensuring the integrity of the result. In the context of coming up with an OAuth successor it serves as a tool to provide a stateless way of authenticating a request and checking authorization. "Stateless" as in "no shared state between hundreds of scaled-out auth-checking servers". That in itself is big.

[Hawk](https://github.com/hueniverse/hawk) implements an HTTP HMAC-based authentication scheme. It builds upon the [work](http://tools.ietf.org/html/draft-ietf-oauth-v2-http-mac-01) Eran has done on this in the OAuth 2 context and addresses the requirement of doing authentication without passing passwords around the Web.

Finally, [Oz](https://github.com/hueniverse/oz) addresses the use case of delegated authorization, the stuff you have in mind when you are looking for an OAuth successor. Oz builds upon iron and Hawk to solve some of the statefulness problems of OAuth 1 and to employ a mandatory, sound, yet simpler-than-OAuth-1 signature mechanism.

Over the past months I have been sort of reengineering the procedures implicitly defined by the three modules to create implementations of them in Java and C. I also work on creating written specs for them to reduce the impact of programming language specifics on the procedures. This should help to create interoperable implementations in other languages. For example, there is quite some implicit encoding stuff going on in the node.js implementation that needs to be nailed down when you do the Java thing.

So far, I have a Java implementation of iron ready, called [jiron](https://github.com/algermissen/jiron) and one in C called [ciron](https://github.com/algermissen/ciron).

There is also an implementation of Hawk in Java, named [hawkj](https://github.com/algermissen/hawkj) and I am working on example JAX-RS 2 filter implementations to show how to use it.

The situation regarding Oz is a bit difficult because Eran has not yet produced a writeup of his ideas and I do not feel capable of extracting these ideas from the existing node.js module code base. AFAIU his ideas are still not completely stable either So I will put this on hold for now. He's activelu on it so it won't be too long.

Another thing is that, while I am excited with what you can see in Oz already, I have use cases in mind for an OAuth successor myself. While Eran is explicitly focussing on mobile and steering clear of typical enterprise use cases ("boss wants to grant secretary access to his calendar" , "Report generator client must have read-access to all calendars of all users") I am interested in a solution that covers all the use cases you can think of for RESTful APIs. Public-facing as well as intra-enterprise.

I think I have something there that addresses arbitrary use cases in the HTTP-API space and is not just an [essentially meaningless framework](http://tools.ietf.org/html/rfc6749) and also doesn't end up being an enterprisey, complexity-bloated monster. I am not yet clear whether it can be an evolution of Oz or needs to be a competitor. We'll see about that.

In any case, it is a fascinating space to explore once you get over the initial crypto-confusion barrier.


