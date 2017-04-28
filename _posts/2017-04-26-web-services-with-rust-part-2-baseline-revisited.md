---
layout: post
title: "Web Services with Rust Part 2: Baseline Revisited"
description: ""
category: "rust"
tags: ["rust-web"]
---
{% include JB/setup %}

[All Web-Rust postings](http://www.jalg.net/tags.html#rust-web-ref)

In the last posting I started exploring implementing Web services in Rust, using
hyper with some minimal single- and multi threaded servers.

If you have been paying attention to the test setup description there (as I should have
done) you will have noticed that the load test configuration is limiting the number
of clients to 500. Many requests with few clients are not the common case reality for
most Web services and definitely not what I had in mind when I ran the tests.

Then again, few clients that send many requests is probably exactly what we want to
test for a micro services environment when thinking about not public facing services.

Given that Tokio and hyper allow for controlling what happens at the per-connection-
and thread level let's take another turn at exploring different usage patterns and
establishing a baseline. We can leverage the
[configuration options of StormForger](https://docs.stormforger.com/reference/)
to extend the test scenarios.

A note on [StormForger](https://stormforger.com/): I have worked with them in
a previous client project where they provided their platform and traffic modeling
and result analysis support. We very much benefited from their deep understanding of
load testing issues and their traffic shaping DSL makes creating test scenarios a
breeze. Naturally, I picked them for this exploration and since they graciously
fuel my tests here, I am more than happy to give credit :-)

### Instrumenting the Server

While StormForger is providing excellent reports on the test runs, I still want
to measure directly in the server to get a better picture of what is happening
and to verify the assumptions I make about the Rust and [Tokio](http://tokio.io)
ecosystems.

### Test Case: Many-Clients-One-Request

This test case focuses on maximizing the number of clients while having each
client just send a single request. My goal is to use this to get as many
clients as possible connected in parallel while minimizing the time they spend
being served.

    definition.setTarget("xxx:8080");

    definition.setArrivalPhases([
      { duration: 30, rate: 20},
      { duration: 30, rate: 40},
      { duration: 30, rate: 60},
      { duration: 30, rate: 120},
      { duration: 30, rate: 240},
      { duration: 30, rate: 480},
      { duration: 30, rate: 960},
      { duration: 30, rate: 1920},
    ]);

    definition.setTestOptions({
      cluster: { sizing: "large", },
    });

    definition.session("ManyClientsOneRequest", function(context) {
      context.get("/data", { tag: "root" });
    });



### Test Case:


TBD...

