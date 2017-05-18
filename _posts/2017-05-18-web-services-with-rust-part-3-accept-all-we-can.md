---
layout: post
title: "Web Services with Rust Part 3: Accept All We Can"
description: ""
category: "rust"
tags: ["rust-web"]
---
{% include JB/setup %}

[All Web-Rust postings](http://www.jalg.net/tags.html#rust-web-ref)

In the last posting we added metrics to our very basic test Web service. This
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

I added some further metrics to being able to compare the number of
accepted and the number of done connections.

As before, the server was able to handle up to 250 requests per second,
being bound by the roughly 4ms of CPU bound time per requests. Still,
the number of accepted requests did correlate to the number of 
handled requests, showing that the server indeed did not simply pull in all
accepted requests from the listening socket.

[The Tokio team suggested](https://github.com/tokio-rs/tokio/issues/8) that
doing the CPU-bound work on the same thread as the task that is taking
connections of the listeing socket might actually yield this behavior, so
next up was having the CPU-bound work been done on a different thread.

The great thing about the [Tokio ecosystem](https://github.com/tokio-rs)
is that it comes with all the [tools that make working with threads and
asynchronous work](https://github.com/alexcrichton/futures-rs/tree/master/src/sync)
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

The request handler then creates a one time channel and spawns a task on the
remote handle that performs the CPU-bound work and sends the result through
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

With this modification, the server can now do roughly 1300 requests per seconds
until the 99th percentile of response latency starts to increase. At this point
we now indeed see the server taking more accepted connections of the
listen queue than it can complete.

Though the server is now running on two threads instead of one the performance
gain seen suggests that for intense CPU bound work it makes sense to run it
on a different thread than more IO bound tasks. 

















pub struct FutureResponse(Box<Future<Item = Response, Error = hyper::Error>>);

impl Future for FutureResponse {
    type Item = Response;
    type Error = hyper::Error;

    fn poll(&mut self) -> Poll<Self::Item, Self::Error> {
        self.0.poll()
    }
}

impl Service for Srv {
    type Request = Request;
    type Response = Response;
    type Error = hyper::Error;
    //type Future = futures::Future<Response, hyper::Error>;
    type Future = FutureResponse;

    // Serves the dummy data created from our CPU intensive work
    // (and the metrics endpoint)
    fn call(&self, req: Request) -> Self::Future {
        let f = match (req.method(), req.path()) {
            (&Get, "/data") => {
                self.request_counter.inc();
                let (tx, rx) = futures::sync::oneshot::channel::<Self::Response>();
                self.remote
                    .spawn(|_| {

                        let b = cpu_intensive_work().into_bytes();
                        let res = Response::new()
                            .with_header(ContentLength(b.len() as u64))
                            .with_body(b);
                        let _ = tx.send(res);
                        Ok(())
                    });
                //rx
                FutureResponse(rx.map_err(|_| hyper::error::Error::Method).boxed())
            }
            // Standard Prometheus metrics endpoint
            (&Get, "/metrics") => {
                let encoder = TextEncoder::new();
                let metric_familys = self.registry.gather();
                let mut buffer = vec![];
                encoder.encode(&metric_familys, &mut buffer).unwrap();
                let res = Response::new().with_body(buffer);
                //futures::future::ok::<hyper::Response, hyper::Error>(res)
                FutureResponse(futures::finished(res).boxed())
            }
            _ => {
                let res = Response::new().with_status(StatusCode::NotFound);
                FutureResponse(futures::finished(res).boxed())
            }
        };
        f
    }
    //fn call(&self, req: Request) -> Self::Future {
    //    futures::future::ok(match (req.method(), req.path()) {
    //                            (&Get, "/data") => {
    //                                self.request_counter.inc();
    //                                //info!("GET {} [{}]", req.path(), self.thread_id);
    //                                let timer = self.request_histogram.start_timer();
    //                                let b = cpu_intensive_work().into_bytes();
    //                                timer.observe_duration();
    //                                Response::new()
    //                                    .with_header(ContentLength(b.len() as u64))
    //                                    .with_body(b)
    //                            }
    //                            // Standard Prometheus metrics endpoint
    //                            (&Get, "/metrics") => {
    //                                let encoder = TextEncoder::new();
    //                                let metric_familys = self.registry.gather();
    //                                let mut buffer = vec![];
    //                                encoder.encode(&metric_familys, &mut buffer).unwrap();
    //                                Response::new().with_body(buffer)
    //                            }
    //                            _ => Response::new().with_status(StatusCode::NotFound),
    //                        })
    //}
}

impl Drop for Srv {
    fn drop(&mut self) {
        self.connection_gauge.dec();
        self.finished_conn_counter.inc();
    }
}


fn serve(remote: &Remote,
         thread_id: String,
         addr: &SocketAddr,
         protocol: &Http,
         registry: &Registry) {
    let mut core = Core::new().unwrap();
    let handle = core.handle();

    let listener = net2::TcpBuilder::new_v4()
        .unwrap()
        .reuse_port(true)
        .unwrap()
        .bind(addr)
        .unwrap()
        .listen(4)
        .unwrap();

    // =================== START METRICS SETUP =====================
    let acg_opts = Opts::new("conn_number", "conn number help")
        .const_label("threadId", thread_id.as_str());
    let active_conn_gauge = Gauge::with_opts(acg_opts).unwrap();
    registry
        .register(Box::new(active_conn_gauge.clone()))
        .unwrap();

    let cc_opts = Opts::new("conn_counter", "conn counter help")
        .const_label("threadId", thread_id.as_str());
    let accepted_conn_counter = Counter::with_opts(cc_opts).unwrap();
    registry
        .register(Box::new(accepted_conn_counter.clone()))
        .unwrap();

    let fcc_opts = Opts::new("finished_conn_counter", "finished_conn counter help")
        .const_label("threadId", thread_id.as_str());
    let finished_conn_counter = Counter::with_opts(fcc_opts).unwrap();
    registry
        .register(Box::new(finished_conn_counter.clone()))
        .unwrap();

    let rc_opts = Opts::new("req_counter", "req counter help")
        .const_label("threadId", thread_id.as_str());
    let rc = Counter::with_opts(rc_opts).unwrap();
    registry.register(Box::new(rc.clone())).unwrap();

    let buckets = vec![0.001, 0.002, 0.005, 0.007, 0.010, 0.020, 0.030, 0.040, 0.050, 0.1, 0.2,
                       0.3, 0.5, 1.0, 5.0];
    let h_opts = HistogramOpts::new("http_request_duration_seconds",
                                    "request duration seconds help")
            .const_label("threadId", thread_id.as_str())
            .buckets(buckets);
    let h = Histogram::with_opts(h_opts).unwrap();
    registry.register(Box::new(h.clone())).unwrap();
    // =================== END METRICS SETUP =====================

    let listener = TcpListener::from_listener(listener, addr, &handle).unwrap();
    core.run(listener
                 .incoming()
                 .for_each(|(socket, addr)| {
            //info!("Connection");
            let s = Srv::new(remote,
                             registry,
                             &thread_id,
                             &active_conn_gauge,
                             &finished_conn_counter,
                             &rc,
                             &h);
            accepted_conn_counter.inc();
            protocol.bind_connection(&handle, socket, addr, s);
            Ok(())
        })
                 .or_else(|e| -> FutureResult<(), ()> {
                              panic!("TCP listener failed: {}", e);
                          }))
        .unwrap();
    //    core.run(
    //        listener.incoming()
    //        //        .or_else(|e| {
    //        //            println!("Connection error: {}", e);
    //        //            Err(e)
    //        //        })
    //        .sleep_on_error(Duration::from_millis(1000), &handle)
    //        .map(move |(socket, addr)| {
    //            let s = Srv::new(registry, &thread_id, &active_conn_gauge, &finished_conn_counter, &rc, &h);
    //            accepted_conn_counter.inc();
    //            protocol.bind_connection(&handle, socket, addr, s);
    //            //            Proto::new(socket, &scfg,
    //            //                       BufferedDispatcher::new(addr, &h1, || service),
    //            //                       &h1)
    //            // Log error, effectively making error type of a future nil `()`
    //            //            .map_err(|e| { println!("Connection error: {}", e); })
    //            Ok(())
    //        })
    //        .listen(100000)  // max connections
    //).unwrap();
}

//fn start_server(n: usize, addr: &str) {
fn start_server(n: usize, addr: &'static str) {

    let mut workerCore = Core::new().unwrap();
    let remote = workerCore.remote();

    thread::spawn(move || {
        let reg = Registry::new();
        let addr = addr.parse().unwrap();
        let protocol = Arc::new(Http::new());
        for i in 0..n - 1 {
            //info!("Starting Thread");
            let protocol = protocol.clone();
            let r = reg.clone();
            let thread_id = format!("thread-{}", i);
            let remote2 = remote.clone();
            thread::spawn(move || serve(&remote2, thread_id, &addr, &protocol, &r));
        }
        let thread_id = format!("thread-{}", n - 1);
        serve(&remote, thread_id, &addr, &protocol, &reg);
    });

    workerCore.run(empty::<(), ()>()).unwrap()
}

static A: &'static str = "0.0.0.0:8080";

fn main() {
    //tlog::init_lock_free_logger().unwrap();
    //    tlog::init_mutex_logger().unwrap();
    //info!("Starting Server");
    let args: Vec<_> = env::args().collect();
    if args.len() < 1 {
        panic!("Please state the number of threads to start");
    }

    let n = usize::from_str_radix(&args[1], 10).unwrap();

    start_server(n, A);
}
#![deny(warnings)]
extern crate futures;
extern crate hyper;
extern crate pretty_env_logger;
extern crate webrust;
extern crate net2;
extern crate tokio_core;
extern crate tk_listen;
//extern crate futures;
//extern crate hyper;
//extern crate num_cpus;
//extern crate reader_writer;

use futures::Stream;
use net2::unix::UnixTcpBuilderExt;
use tokio_core::reactor::Core;
use tokio_core::net::TcpListener;
use std::thread;
use std::net::SocketAddr;
use std::sync::Arc;
use futures::future::FutureResult;
use hyper::{Get, StatusCode};
use hyper::header::ContentLength;
use hyper::server::{Http, Service, Request, Response};
use tokio_core::reactor::Remote;

use futures::Future;
use futures::Poll;
use futures::future::*;
//use std::time::Duration;
//use tk_listen::ListenExt;

use std::env;
//use tk_http::{Status};
//use tk_http::server::buffered::{Request, BufferedDispatcher};
//use tk_http::server::{self, Encoder, EncoderDone, Proto, Error};

//use hyper::{Delete, Get, Put, StatusCode};
//use hyper::header::ContentLength;
//use hyper::server::{Http, Service, Request, Response};
//#[macro_use]
//extern crate log;
//extern crate tlog;


use webrust::cpu_intensive_work;

extern crate prometheus;
use prometheus::{Registry, Gauge, Opts, Counter, Histogram, HistogramOpts, Encoder, TextEncoder};

// Srv struct bundles instruments for metrics collection and a thread ID to include
// in the metrics.
#[derive(Clone)]
struct Srv {
    remote: Remote,
    thread_id: String,
    registry: Registry,
    request_counter: Counter,
    request_histogram: Histogram,
    connection_gauge: Gauge,
    finished_conn_counter: Counter,
}

impl Srv {
    fn new(rem: &Remote,
           r: &Registry,
           thread_id: &String,
           cg: &Gauge,
           fcc: &Counter,
           rc: &Counter,
           rh: &Histogram)
           -> Srv {
        let s = Srv {
            remote: rem.clone(),
            registry: r.clone(),
            thread_id: thread_id.clone(),
            request_counter: rc.clone(),
            request_histogram: rh.clone(),
            connection_gauge: cg.clone(),
            finished_conn_counter: fcc.clone(),
        };
        s.connection_gauge.inc();
        s
    }
}

pub struct FutureResponse(Box<Future<Item = Response, Error = hyper::Error>>);

impl Future for FutureResponse {
    type Item = Response;
    type Error = hyper::Error;

    fn poll(&mut self) -> Poll<Self::Item, Self::Error> {
        self.0.poll()
    }
}

impl Service for Srv {
    type Request = Request;
    type Response = Response;
    type Error = hyper::Error;
    //type Future = futures::Future<Response, hyper::Error>;
    type Future = FutureResponse;

    // Serves the dummy data created from our CPU intensive work
    // (and the metrics endpoint)
    fn call(&self, req: Request) -> Self::Future {
        let f = match (req.method(), req.path()) {
            (&Get, "/data") => {
                self.request_counter.inc();
                let (tx, rx) = futures::sync::oneshot::channel::<Self::Response>();
                self.remote
                    .spawn(|_| {

                        let b = cpu_intensive_work().into_bytes();
                        let res = Response::new()
                            .with_header(ContentLength(b.len() as u64))
                            .with_body(b);
                        let _ = tx.send(res);
                        Ok(())
                    });
                //FutureResponse(rx.map_err(|_| hyper::error::Error::Method).boxed())
