---
layout: post
title: "Web Services with Rust Part 1bis: Baseline Revisited"
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




 I think we
should have scenarios for

* very few clients
* medium number of clients (e.g. the above 500) that attempt maximum number of requests
* very many clients that do few requests (the true Web scenario)

With 10 or 100 clients the limiting factor is the sizing of the load test generators. We'll
get back to this scenario after upgrading the load test generator in another posting.

The 500 clients scenario we have already covered, so let us go with the very many clients
variant next.

For this we alter the StormForger test case to start many more clients that all do some
work and then stop.

    definition.setTarget("147.75.101.27:8080");

    definition.setArrivalPhases([
      { duration: 1 * 5, rate: 100, },
      { duration: 1 * 20, rate: 200, },
      { duration: 1 * 20, rate: 400, },
      { duration: 1 * 20, rate: 600, },
      { duration: 1 * 20, rate: 800, },
      { duration: 1 * 20, rate: 1000, },
      { duration: 1 * 20, rate: 1200, },
    ]);

    definition.setTestOptions({
      cluster: { sizing: "small", },
    });

    definition.session("VeryManyClients", function(context) {
      context.times(8, function(context){
        context.get("/data", { tag: "root" });
        context.wait(0.1, { random: true });
      });
    });

This gives us gradually increasing number of arriving clients, each calling the
static resource eight times, waiting a random number of milliseconds up to 100ms
between calls.

When running this with a single thread version of the server, we see the 99th
latency percentile to increase at about 4000 req/s and 1000 parallel clients.

![Results Many Clients - 1 Thread](/assets/img/ManyClients_1Thread.jpg)

Let's verify we get twice the throughput with 2 threads.

![Results Many Clients - 2 Thread](/assets/img/ManyClients_2Thread.jpg)

