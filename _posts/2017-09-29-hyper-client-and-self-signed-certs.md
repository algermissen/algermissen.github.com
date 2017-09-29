---
layout: post
title: "hyper client and self-signed certs"
description: ""
category: ""
tags: ["rust-web"]
---
{% include JB/setup %}

Fortunate are those that are chosen to ride down the happy path...

This is, however, for the warriors like me, with n00b-level Rust knowledge, armed with razor sharp, but early-stage tools,
brave enough to enter the evil land of the enterprise. (Honestly, it also did not help that I had passed on
reading up on TLS and certificates in depth all those years :-/ )

Taken out of a larger context, the battle at hand was to talk to a REST API using
[Tokio](https://tokio.rs/) and [hyper client](https://hyper.rs/guides/client/basic/). Simple enough:
copy/paste the docs, change the URI and shoot.

Argrr, yeah, sure - it's an https: URI! And, yeah, I remember that hyper's design is clever enough to
be decoupled from SSL dependencies completely. So, how to go about that TLS protected request now?

Fortunately [the docs are ready at hand](https://hyper.rs/guides/client/configuration/)
 and victory just another copy/paste away...


    extern crate hyper;
    extern crate hyper_tls;
    extern crate tokio_core;
    
    use hyper::Client;
    use hyper_tls::HttpsConnector;
    use tokio_core::reactor::Core;
    
    let mut core = Core::new()?;
    let handle = core.handle();
    let client = Client::configure()
        .connector(HttpsConnector::new(4, &handle)?)
        .build(&handle);
    
    let req = client.get("https://hyper.rs".parse()?);
    let res = core.run(req)?;
    assert!(res.status().is_success());


But then, while the compiler runs it dawns on me: the evil cert master of
this enterprise is into the self-signing witch craft (to save money or for
whatever crazy reason). And while one would normaly go ahead, ignore that
exception and proceed (because it is after all just an evaluation I am
doing) I cannot really remember having seen anything like that in the docs
so far. Hmmm....

Searches and chat questions (thanks [Sean](https://twitter.com/seanmonstar)) eventually
produced [this](https://github.com/sfackler/rust-native-tls/issues/13) 
which made its way into the crate in a
[nicely explicit way](https://twitter.com/algermissen/status/913438875250458624)
and [this](https://github.com/sfackler/rust-native-tls/issues/13#issuecomment-293628743).
While the former does not really do the trick, the later got me all excited - so
simple.


> "Build a Certificate from the DER-encoded self signed cert and pass it to
> TlsConnectorBuilder::add_root_certificate."


But...


- How do I get the certificate?
- WTF does "Build a Certificate from the DER-encoded self signed cert" mean?
- Haven't seen a `TlsConnectorBuilder` yet.
- Will that really be it?


What my browser can do, I can also do, I figure; Google turns up


    openssl s_client -showcerts -connect example.org:443 > server.crt


Great - let's stick that into that method mentioned above and get this over with.
Where I dug up the code for that I cannot remember, but it goes like this:


    let mut f = File::open("/...my cert file").unwrap();
    let mut buffer = vec![];
    f.read_to_end(&mut buffer).unwrap();
    let cert = Certificate::from_der(buffer.as_slice()).unwrap();

    let mut http = HttpConnector::new(4, handle);
    http.enforce_http(false);

    let mut tls = TlsConnector::builder().unwrap();
    tls.add_root_certificate(cert);
    let mut tls = tls.build().unwrap();

    let ct = HttpsConnector::from((http, tls));
    let client = Client::configure().connector(ct).build(handle);


(It is also especially great to do this past midnight :-)

Victory? - You guessed it: not yet. Why is that method called
from_der() and my file is a crt? This can't end well. At this
point I honestly just shoved as many things into the search box
as I could possibly think of to make sense. 

Eventually yielding


    openssl x509 -in server.crt -outform der -out server.der


I am skipping all the detours I took do to late night errors,
byte array mistakes, wrong assumptions about company root cert
I should get hold of and misinterpreted timeouts that actually
weren't any.

Putting the DER file in the right spot solves the problem and
your worrior can go to bed, exhausted, but still in honest awe
of the language Rust as such.







