---
layout: post
title: "Web Services with Rust Part 1: Exploring Hyper"
description: ""
category: "rust"
tags: []
---
{% include JB/setup %}

A while ago I have turned my head towards [Rust](http://rustlang.org) for developing Web Services. Rust's
design in my opinion hits a sweet spot between ease of development and runtime
efficiency, especially doing away with a garbage collector.

Recently sophisticated support for [future-based](https://tokio.rs/docs/getting-started/futures/),
[async](https://tokio.rs/docs/getting-started/reactor/),
and [reactive](https://tokio.rs/docs/getting-started/streams-and-sinks/)
programming has been added to the Rust ecosystem; putting together in the
[Tokio project](https://tokio.rs) some of the most
[intriguing](https://aturon.github.io/blog/2016/08/11/futures/) designs I have
come across so far.

In addition, the [hyper Web server]() is being refactored to natively work with the
async and reactive mechanics provided by Tokio. This work is very closely aligned with
the work on Tokio, making for a very promising HTTP toolkit roadmap.

All in all, this feels a lot like the more lightweight, less bloated alternative to
all I like about Akka-HTTP that I have been hoping for for quite some time now.

As sugar on top there are several very interesting developments regarding
related topics, such as
[lock free data structures](https://github.com/crossbeam-rs/crossbeam)
(also see [Lock-freedom without garbage collection](https://aturon.github.io/blog/2015/08/27/epoch/))
and [Actors](https://github.com/carllerche/kabuki).

In this series of blog postings we'll explore how to build Web services with
hyper, focusing primarily on non functional aspects such as using thread pools with
hyper, shared state,

# Minimal, Single-Threaded Server (No Shared State, No I/O)

The most simple server to run based on hyper is a single-threaded server that
does not involve shared state or blocking calls. We'll just use the
[echo server example](https://github.com/hyperium/hyper/blob/9605e860ff46aa7cf02d4e29624f887604cb6541/examples/server.rs)
from the hyper repository.

To get some rough idea about the performance impact of our forthcoming modifications
we'll use a simple testing setup using a [4 core 2.4 GHz, 8GB RAM, 1BGbps bare
metal server](https://www.packet.net/bare-metal/servers/type-0/) offered by
[Packet](https://www.packet.net) and [Stormforger](https://stormforger.com/) to
put substantial stress on the Web servers. Test server and load generators are both
located in continental Europe.

The first test runs with the example server capped around 10k req/s, running out of
TCP connections. I added some CPU-bound work to the server, simulating, for example,
template rendering activities you might have in a real Web server and had to
experiment with the load tests setting to get to a point where I could drive the
server to the limit in a controlled way.

The [work simulation](https://github.com/algermissen/web-rust/blob/ecc430d558aa60dcf2fd2d7ee89f73b7395e5e9d/src/bin/minimal_single_threaded.rs#L26)
looks like this

    fn cpu_intensive_work() -> String {
        let mut y = "X".to_string();
        for x in 0..100 {
            y = format!("Value: {}", x);
        }
        let address = Address {
            street: "10 Downing Street".to_owned(),
            city: y.to_owned(),
        };

        let j = serde_json::to_string(&address).unwrap();
        return j;
    }

The server code I compiled as a statically linked executable for Linux.


The Stormforger load test looks like this:

    definition.setTarget("[host:port]");

    definition.setArrivalPhases([
      {
        duration: 60,
        rate: 8.0,         // clients per second to launch
        max_clients: 500,
      },
    ]);

    definition.setTestOptions({
      cluster: { sizing: "small", },
    });

    definition.session("Load Test 1", function(session) {

      session.forEver(function(context){
        context.get("/data", { tag: "root" });

      });
    });



With that I was able to get about 6000 req/s out of the server before seeing
the 99th percentile latency degrading. modifying the simulated work showed
corresponding variations in req/s numbers so I am now sure I am not hitting
any unwanted capacity limits.




























