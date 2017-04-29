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

*A note on [StormForger](https://stormforger.com/): I have worked with them in
a previous client project where they provided their platform and traffic modeling
and result analysis support. We very much benefited from their deep understanding of
load testing issues and their traffic shaping DSL makes creating test scenarios a
breeze. Naturally, I picked them for this exploration and since they graciously
fuel my tests here, I am more than happy to give credit :-)*

### Instrumenting the Server

While StormForger is providing excellent reports on the test runs, I still want
to measure directly in the server to get a better picture of what is happening
and to verify the assumptions I make about the Rust and [Tokio](https://tokio.rs/)
ecosystems.

Recently [Prometheus](https://prometheus.io) has become my application metrics
system of choice and since a [Prometheus client for Rust](https://github.com/pingcap/rust-prometheus)
already exists (apparently developed and used by the [TiKV Team](https://github.com/pingcap/tikv))
I'have used this to implement an [instrumented minimal hyper server](https://github.com/algermissen/web-rust/blob/master/src/bin/testserver.rs)

One [gauge](https://prometheus.io/docs/concepts/metric_types/#gauge)
metric [keeps track of the number of connected clients](https://github.com/algermissen/web-rust/blob/51019037c4b6478e953e51a1c016a5dd7ada2b1a/src/bin/testserver.rs#L93)
and a [histogram](https://prometheus.io/docs/concepts/metric_types/#histogram) is
used to [measure the individual requests](https://github.com/algermissen/web-rust/blob/51019037c4b6478e953e51a1c016a5dd7ada2b1a/src/bin/testserver.rs#L119).

So, let's see what the instruments report when we throw load at the server.

### Test Case: Many-Clients-One-Request

This test case focuses on maximizing the number of clients while having each
client just send a single request. My goal is to use this to get as many
clients as possible connected in parallel while minimizing the time they spend
being served.

The StormForger test case setup:

    definition.setTarget("xxx:8080");

    definition.setArrivalPhases([
      { duration: 20, rate: 120},
      { duration: 20, rate: 160},
      { duration: 20, rate: 180},
      { duration: 20, rate: 200},
      { duration: 20, rate: 220},
      { duration: 20, rate: 230},
      { duration: 20, rate: 240},
      { duration: 20, rate: 250},
      { duration: 20, rate: 260},
      { duration: 20, rate: 270},
      { duration: 20, rate: 280},
      { duration: 20, rate: 290},
      { duration: 20, rate: 300},
      { duration: 20, rate: 310},
      { duration: 20, rate: 320},
    ]);

    definition.setTestOptions({
      cluster: { sizing: "large", },
    });

    definition.session("ManyClientsOneRequest", function(context) {
      context.get("/data", { tag: "root" });
    });

This test runs for 5 minutes and gradually increases the rate of arriving clients from 120/s to 320/s,
each doing a single request.

When running this against a single threaded version of our test server, we

![Test 1 - Client Side Graph](/assets/img/web-rust/test1/test1_sf_graph.jpg)

![Test 1 - Server Side Graph](/assets/img/web-rust/test1/test1_gf_graph.jpg)



### Test Case:


TBD...

