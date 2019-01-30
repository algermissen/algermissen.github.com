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


## Go Gotchas

These links address subtle issues about the Go language, that do not appear
obvious at fiorst sight:

Go has a potential for memory leaks, described in this section
[Delete without preserving order](https://github.com/golang/go/wiki/SliceTricks#delete-without-preserving-order)


## Other Reference Lists

- https://github.com/golang/go/wiki/Articles
- https://awesome-go.com
- 


