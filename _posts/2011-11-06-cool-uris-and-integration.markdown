---
layout: post
title: Cool URIs and Integration
---
{% include JB/setup %}

A key design aspect that made the very success of the World Wide Web possible was the removal of the referential integrity constraint from the hypermedia paradigm. In a system of the scale of the Web the problem of ensuring that no hyperlink would ever be broken cannot be solved and the only way to make the system work was to relax the constraint.

Instead, the constraint was replaced by a best practice, namely that [Cool URIs Don't Change](http://www.w3.org/Provider/Style/URI). Meaning that an origin server that exposes a given URI should continue to do so forever. (This really only applies to URIs that serve as [application entry points](http://tech.groups.yahoo.com/group/rest-discuss/message/12180), aka _bookmarkable URIs_, but that is a different discussion).

Looking at the title, you might wonder what this has to do with integration? Well, here is the story:

Suppose an integration scenario with two systems, a content management system (CMS) and a system that manages information about companies (let's call that system _CIMS_). Authors create article pages in the CMS to be published on the Web as part of a financial news site. Whenever an article talks about a certain company the authors search the CIMS for that company and a reference to the company information in the CIMS is placed in the article. When the article is published, the reference is resolved and some HTML fragment with the company information is embedded in the Web page.

Since the company implementing this scenario does [cutting edge integration](http://paulprescod.sys-con.com/node/40442) the CIMS exposes an HTTP interface and the obtained company information references naturally are URIs.

The bottom line: URIs obtained at some point in time are remembered to be dereferenced in the future. This is what is commonly called a [_bookmark_](http://tech.groups.yahoo.com/group/rest-discuss/message/13606). 

The designers of this system, as well as its users, make the assumption that any of the embedded URIs will continue to be the URI of the same company information it was when obtained from the CIMS. In other words, that the mapping of URI to company information will remain stable. This assumption is backed by the aforemetioned best practice of keeping URIs cool.

The consequence is an implicit (you could equally well argue: _explicit_) obligation on the owner of the origin server (the CIMS in this case) to maintain the URIs - well - indefinitely. And this is something service designers and service owners need to be aware of, especially in an enterprise integration context.

The obligation even exceeds the (probably limited) lifetime of the referenced concept itself (in our case the company information) because the service needs to inform potential clients when there is currently no information available (404 Not Found) or when the information ceased to exist at all (410 Gone). The former case yields a requirement for the client to work around the missing information the latter indicates that the client system should remove the bookmark (here likely indicating a need to rewrite the article).

This last paragraph, BTW, touches only briefly on the topic of how even HTTP error responses are useful (and essential) to the integration quality. (As opposed to a tightly coupled integration style where the client just goes berserk if some piece of information is missing).

