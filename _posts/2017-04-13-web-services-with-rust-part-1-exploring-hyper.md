---
layout: post
title: "Web Services with Rust Part 1: Exploring Hyper"
description: ""
category: "rust"
tags: ["rust-web"]
---
[All Web-Rust postings](http://www.jalg.net/tags.html#rust-web-ref)

A while ago I have turned my head towards [Rust](http://rustlang.org) for
developing Web Services. Rust's design in my opinion hits a sweet spot between
ease of development and runtime efficiency, especially doing away with a
garbage collector.

Recently sophisticated support for [future-based](https://tokio.rs/docs/getting-started/futures/),
[async](https://tokio.rs/docs/getting-started/reactor/),
and [reactive](https://tokio.rs/docs/getting-started/streams-and-sinks/)
programming has been added to the Rust ecosystem; putting together in the
[Tokio project](https://tokio.rs) some of the most
[intriguing](https://aturon.github.io/blog/2016/08/11/futures/) designs I
have come across so far.

In addition, the [hyper Web server](https://github.com/hyperium/hyper) is
being refactored to natively work with the
async and reactive mechanics provided by Tokio. This work is very closely aligned with
the work on Tokio, making for a very promising HTTP toolkit roadmap.

All in all, this feels a lot like the more lightweight, less bloated alternative to
all I like about Akka-HTTP, that I have been hoping for for quite some time now.

As sugar on top there are several very interesting developments in
related topics, such as
[lock free data structures](https://github.com/crossbeam-rs/crossbeam)
(also see [Lock-freedom without garbage collection](https://aturon.github.io/blog/2015/08/27/epoch/))
and [Actors](https://github.com/carllerche/kabuki). I think there are lots of
opportunities here for Web programming in constrained environments or for reducing
resource consumption in the micro services world to avoid micro services
eating up macro Euros of cloud computing budgets.

In this series of blog postings we'll explore how to build Web services with
hyper, focusing primarily on non functional aspects such as using thread pools with
Tokio and hyper, sharing state across requests, or running background activities.
I think that while Rust forces you to be very close to many details of async and
multi threaded programming it and its existing and upcoming libraries make up for
that with quite gentle programming abstractions. Personally, I am willing to
deprive myself from JVM and go runtime magic, humbly embrace the resulting
learning curve, and hopefully end up with, for one thing, smaller, faster, and cheaper
deployables at sufficient development speed. Not to mention the joy of the
functional programming style...

Having said that, let's dive right in.

### Minimal, Single-Threaded Server (No Shared State, No I/O)

The most simple server to run based on hyper is a single-threaded server that
does not involve shared state or blocking calls. We'll just use the
[echo server example](https://github.com/hyperium/hyper/blob/9605e860ff46aa7cf02d4e29624f887604cb6541/examples/server.rs)
from the hyper repository.

To get some rough idea about the performance impact of our forthcoming modifications
we'll use a simple testing setup using a [4 core 2.4 GHz, 8GB RAM, 1Gbps bare
metal server](https://www.packet.net/bare-metal/servers/type-0/) offered by
[Packet](https://www.packet.net) and [Stormforger](https://stormforger.com/) to
put substantial stress on the Web servers. Test server and load generators are both
located in continental Europe.

The first test runs with the example server capped around 10k req/s without CPU
exhaustion. I added some CPU-bound work to the server, simulating, for example,
template rendering activities you might have in a real Web server and had to
experiment with the load test settings to get to a point where I could drive the
server to the limit in a controlled way.

The [work simulation](https://github.com/algermissen/web-rust/blob/ecc430d558aa60dcf2fd2d7ee89f73b7395e5e9d/src/bin/minimal_single_threaded.rs#L26)
looks like this

{% highlight rust linenos %}
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
{% endhighlight %}

The server code I compiled as a statically linked executable for Linux (resulting
in 4.5 MB).


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

Now that we have a baseline, let's see what happens if we involve multiple cores.

### Minimal Multi-Threaded Server (No Shared State, No I/O)

Running our server on multiple threads (and thus potentially cores) requires
some changes to the approach towards server startup. As there is not
yet any shared state between the individual requests we can spawn the threads
independently, reusing the listening socket for each handler thread. This can
be done with a combination of the net2 and tokio-core libraries and under the
hood uses the operating system's SO_REUSEPORT feature.

Here is the interesting part of the code:

    fn serve(addr: &SocketAddr, protocol: &Http) {
        let mut core = Core::new().unwrap();
        let handle = core.handle();
        let listener = net2::TcpBuilder::new_v4().unwrap()
        .reuse_port(true).unwrap()
        .bind(addr).unwrap()
        .listen(128).unwrap();
        let listener = TcpListener::from_listener(listener, addr, &handle).unwrap();
        core.run(listener.incoming().for_each(|(socket, addr)| {
            protocol.bind_connection(&handle, socket, addr, Echo);
            Ok(())
        })).unwrap();
    }

    fn start_server(nb_instances: usize, addr: &str) {
        let addr = addr.parse().unwrap();

        let protocol = Arc::new(Http::new());
        {
            for _ in 0..nb_instances - 1 {
                let protocol = protocol.clone();
                thread::spawn(move || serve(&addr, &protocol));
            }
        }
        serve(&addr, &protocol);
    }


    fn main() {
        start_server(2, "0.0.0.0:8080");
    }


Check out the [full source code](https://github.com/algermissen/web-rust/blob/master/src/bin/ts1.rs).

This version (using two threads) should give us roughly twice the throughput - let's run
the same test setup and meanwhile take a look at the graph from the single-threaded setup.
It nicely shows how at 6k req/s the latencies start to increase, thereby capping the
throughput at these 6k.

![Results 1-threaded](/assets/img/1-Threaded.jpg)

There, the result came back from the 2-threaded setup and (relief :-) just what we
expected: the server is now giving us about 11k req/s and I indeed saw exactly two
CPUs being saturated. Nice!

![Results 2-threaded](/assets/img/2-Threaded.jpg)

Next up is a 4-threaded version (on the 4 core machine) - will we be able to double
again?

Yes, I am seeing the 4 cores being saturated and the latencies remain pretty ok
until about 20k req/s.

![Results 4-threaded](/assets/img/4-Threaded.jpg)

Yes, the 99th percentile rises somewhere around 15k, but for this initial
verification that what I am doing makes some sort of sense the result is
sufficient.

As a side note: 20k req/s at $ 0.05/h for the server gives roughly
72 million req/h at 5 cents...

So much for getting out feet wet with Rust, hyper and Tokio and for establishing
a baseline for comparing future variations. Next up I'll look into a variant
running on a Tokio-managed thread pool and how to share state across
request handling threads (for example for rate limiting, metrics collection,
logging).






















