---
layout: post
title: "Go Bookmarks"
description: "A collection of references for Go programming"
category: "go"
excerpt: This is my collection of references I found especially useful to understand idiomatic solutions to various design problems and Go internals.
---
{% include JB/setup %}

This is my collection of references I found especially useful to understand
idiomatic solutions to various design problems and Go internals.

## Application Architecture

After digging through the considarable amount of articles applying the
topic of Clean Architecture to Go I found this to be the most compelling
one (and actually one of the few in the spirit of the original idea, AFAIK):

[Clean Architecture Guide (with tested examples): Data Flow != Dependency Rule](https://proandroiddev.com/clean-architecture-data-flow-dependency-rule-615ffdd79e29)

## Error Handling

When approaching error handling in Go it is in my opinion very easy to make
the mistake and work against the original ideas of the language. The
approach I found to be naturally aligned with what GHop deliberately provides
for errors out of the box is described in:

[Error Handling Practices in Go](https://banzaicloud.com/blog/error-handling-go/)


## HTTP Handler Functions

At the end of this blog post a proper technique is show how to pass data from
one handler to another in an HTTP handler 'chain'. In my opinion, using a
closure approach and create the actual handling function on the fly and closing
over the pass-id data is far superior than any other approach (eg writing your own
handlers signature or using request context values):

[Pitfalls of context values and how to avoid or mitigate them in Go](https://www.calhoun.io/pitfalls-of-context-values-and-how-to-avoid-or-mitigate-them/)


## Testing

This talk discusses useful strategies for testing that mostly present in Go's standard library but are
are not well known. 
["Advanced Testing with Go by Mitchell Hashimoto"](https://m.youtube.com/watch?v=yszygk1cpEc)


TODO
["xx"](https://talks.golang.org/2014/testing.slide#1)



## Go Gotchas

These links address subtle issues about the Go language, that do not appear
obvious at fiorst sight:

Go has a potential for memory leaks, described in this section
[Delete without preserving order](https://github.com/golang/go/wiki/SliceTricks#delete-without-preserving-order)


## Other Reference Lists

- https://github.com/golang/go/wiki/Articles
- https://awesome-go.com



