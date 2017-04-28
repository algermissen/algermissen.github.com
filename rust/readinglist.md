---
layout: page
title: "Rust Readinglist"
description: "My collection bookmarks on various Rust topics."
---
{% include JB/setup %}

### News
[Top Rust Projects](http://oss.io/Rust/index)
[This week in Rust](https://this-week-in-rust.org/)

### Forums
[Rust Gitter Channels](https://medium.freecodecamp.com/best-gitter-channels-on-rust-ad8f5f73b5a2)

### Rust Programmers
[http://seanmonstar.com/]
[https://github.com/Boscop]
[https://github.com/carllerche]



### Learning Rust
[Learning Rust With Entirely Too Many Linked Lists](http://cglab.ca/~abeinges/blah/too-many-lists/book/)
* Rather detailed treatment of Rust basics.
[https://medium.com/@ericdreichert/what-one-must-understand-to-be-productive-with-rust-e9e472116728]

### Tokio
[Tokio Docs](https://tokio.rs/)
[Multi Threading in Tokio](https://users.rust-lang.org/t/when-using-a-tokio-futures-backed-webserver-how-to-manage-threads/10064)


### Hyper and Projects building on tokio or hyper
[Hyper](https://github.com/hyperium/hyper)
* [https://www.reddit.com/r/rust/comments/5qcouc/tokiobased_hyper_performance_serverside/]
[EdgeDNS](https://github.com/jedisct1/edgedns)
* Uses prometheus lib
* 
[Rust asynchronous HTTP server with tokio and hyper(Experiences with hyper (ranting))](https://blog.guillaume-gomez.fr/articles/2017-02-22+Rust+asynchronous+HTTP+server+with+tokio+and+hyper)
* [http://archive.is/6cd6E]
[Clarify Performnace Claims Thread](https://github.com/SergioBenitez/Rocket/issues/140)
[https://hoverbear.org/2017/03/01/the-future-with-futures/]
[https://users.rust-lang.org/t/is-tokio-too-complicated/9497/3]
* Hyper code examples


 
### Concurrency
[Ycombinator thread on concurrency in Rust](https://news.ycombinator.com/item?id=11369170)

### Cassandra Driver
[https://github.com/nhellwig/tokio-cassandra]

### Date and Time
[Chrono](https://github.com/chronotope/chrono)
*

### Web Frameworks
[Sapper](https://github.com/sappworks/sapper)
* Builds on hyper
* Seems more humble than Rocket

[Rocket](https://rocket.rs/)
* Builds on hyper
* Seems a bit naiv when it comes to concurrency; need to check how clever it integrates with Tokio
* Blames hyper for some issues (
* IRC: [https://kiwiirc.com/client/irc.mozilla.org/###rocket]
* Info: [https://news.ycombinator.com/item?id=13245475]
* Usage: [http://matthias-endler.de/2017/rust-url-shortener/index.html]

[JWT Impl](https://github.com/brendanzab?tab=repositories)


### Database Projects
[Mentat](https://github.com/mozilla/mentat)
* By Mozilla


### Commandline Argument Parsers
[Clap](https://clap.rs/)
* Seems like *the* thing to use

### Templating
[List of template engines](http://www.arewewebyet.org/topics/templating/)

### Actor Systems
[Kabuki](https://github.com/carllerche/kabuki)
[Kabuki Fork with extensions](https://github.com/habnabit/kabuki)

### Telemetry and Log Shipping
[Cernan](https://github.com/postmates/cernan)
* [https://users.rust-lang.org/t/cernan-a-telemetry-and-log-aggregation-server/8909] (Useful references)


### Metrics Libraries
[Rust Prometheus](https://github.com/pingcap/rust-prometheus)
* The Prometheus client

### Rust Internals
[An alternative introduction to Rust](http://words.steveklabnik.com/a-new-introduction-to-rust)
[]()

### Neural Networks
[RustNN](https://github.com/jackm321/RustNN)

### File Systems etc.
[TFS File System](https://github.com/redox-os/tfs)

### Cross Compiling
[https://chr4.org/blog/2017/03/15/cross-compile-and-link-a-static-binary-on-macos-for-linux-with-cargo-and-rust/]
[https://github.com/japaric/rust-cross]
[http://grahamenos.com/rust-osx-linux-musl.html]


### Lots of Rust Code
[https://github.com/habnabit?tab=repositories]
[https://github.com/brendanzab?tab=repositories]

### Useful Reads
[ycombinator: Rocket – Web Framework for Rust](https://news.ycombinator.com/item?id=13245475)
[The Problem With Single-threaded Shared Mutability](http://manishearth.github.io/blog/2015/05/17/the-problem-with-shared-mutability/)
[Proxy server with Tokio](https://ayende.com/blog/176705/rust-based-load-balancing-proxy-server-with-async-i-o)
[http://bluejekyll.github.io/blog/rust/dns/2016/08/21/a-year-of-rust-and-dns.html]
[http://bluejekyll.github.io/blog/rust/2016/12/03/trust-dns-into-future.html]
[https://github.com/bluejekyll/trust-dns]
[https://news.ycombinator.com/item?id=12332876]
[]
[]

### Misc Reads
[https://users.rust-lang.org/t/reusing-mutable-closure-arguments/5369]
[]()

### Other Resource Lists
[https://devhub.io/zh/repos/brson-awesome-rust]
[http://awesomeawesome.party/awesome-rust]
[]()
