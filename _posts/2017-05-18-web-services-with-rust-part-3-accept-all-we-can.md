---
layout: post
title: "Web Services with Rust Part 3: Accept All We Can"
description: ""
category: "rust"
tags: ["rust-web"]
excerpt: In the last posting we added metrics to our very basic test Web service. Initially this third part was supposed to be about adding logging to the server, comparing the performance impact of mutex based- and, especially, [lock free data structures](https://github.com/crossbeam-rs/crossbeam).
---
{% include JB/setup %}

In the last posting we added metrics to our very basic test Web service. 
Initially this third
part was supposed to be about adding logging to the server, comparing the
performance impact of mutex based- and, especially,
[lock free data structures](https://github.com/crossbeam-rs/crossbeam).

However, I was also still puzzled by the observation that the number of
connections the server took from the stream of accepted sockets somehow
seemed related to the request processing rate. I just could find no reason
that would explain this observation. Especially not, since the stream and
actually handling the connections definitely happens in different tasks.

As a first step I changed the load testing configuration to have each
request use a fresh connection and thus get a 1:1 ratio of requests and
accepted connections. The [Stormforger](https://stormforger.com/) team
fortunately pointed out to me that their load generators react on the
HTTP request headers  set by test configurations, so
`"Connection": "close"` would trigger the load generator to open a new
connection for every request.

    definition.session("OneRequestPerConnection", function(session) {
      session.forEver( function(session) {
        session.get("/data", {
          headers: {
            "Connection": "close",
          }
        });
      });
    });

I added some further metrics in order to compare the number of
accepted and the number of done connections.

As before, the server was able to handle up to 250 requests per second,
being bound by the roughly 4ms of CPU bound time per request. Still,
the number of accepted requests did correlate with the number of 
handled requests, showing that the server indeed did not simply pull in all
accepted requests from the listening socket.

[The Tokio team suggested](https://github.com/tokio-rs/tokio/issues/8) that
doing the CPU-bound work on the same thread as the task that is taking
connections of the listeing socket might actually yield this behavior, so
next up was having the CPU-bound work been done on a different thread.

The great thing about the [Tokio ecosystem](https://github.com/tokio-rs)
is that it comes with [all the tools](https://github.com/alexcrichton/futures-rs/tree/master/src/sync)
that make working with threads and
asynchronous work
a breeze. 

In order to send work off to a different thread that thread needs to run its
own Tokio Core. Since we cannot send a Core to another thread we create the
Core on the main thread, create a remote handle to it and send that to
a new thread along with the actual server startup.
       
    fn start_server(n: usize, addr: &'static str) {
    
      let mut workerCore = Core::new().unwrap();
      let remote = workerCore.remote();

      thread::spawn(move || {
        let addr = addr.parse().unwrap();
        let protocol = Arc::new(Http::new());
        for i in 0..n - 1 {
          let protocol = protocol.clone();
          let remote = remote.clone();
          thread::spawn(move || serve(&remote, , &addr, &protocol));
        }
        serve(&remote, &addr, &protocol);
    });

The request handler then creates a one-time channel and spawns a task on the
remote handle. This taskperforms the CPU-bound work and sends the result through
the channel to complete the response.

    let f = match (req.method(), req.path()) {
      (&Get, "/data") => {
        let (tx, rx) = futures::sync::oneshot::channel::<Self::Response>();
        self.remote.spawn(|_| {
          let b = cpu_intensive_work().into_bytes();
          let res = Response::new()
                   .with_header(ContentLength(b.len() as u64))
                   .with_body(b);
          let _ = tx.send(res);
          Ok(())
        });
        FutureResponse(rx.or_else(|_| -> Result<Response, Self::Error> {
          Ok(Response::new().with_status(StatusCode::InternalServerError))
        }).boxed())
      }
    }

With this modification, the server can now do roughly 1300 requests per second
until the 99th percentile of response latency starts to increase. At this point
we now indeed see the server taking more accepted connections of the
listen queue than it can complete.

Though the server is now running on two threads instead of one, the performance
gain seen suggests that intense CPU bound work should be run
on a different thread than more IO bound tasks. 

See the [test server source code](https://github.com/algermissen/web-rust/blob/master/src/bin/ts3.rs).



