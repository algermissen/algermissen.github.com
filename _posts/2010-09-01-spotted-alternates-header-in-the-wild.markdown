---
layout: post
title: Spotted Alternates Header in the Wild
---
{% include JB/setup %}

Just spotted an [Alternates header](http://www.ietf.org/rfc/rfc2295.txt) in the wild:


    
    
    $ curl http://www.w3.org -HAccept:text/foo -I
    
    HTTP/1.1 406 Not Acceptable
    Date: Tue, 31 Aug 2010 22:26:53 GMT
    Server: Apache/2
    Alternates: {"Home.html" 1 {type text/html} {charset utf-8} {length 28437}}, {"Home.xhtml" 0.99 {type application/xhtml+xml} {charset utf-8} {length 28437}}
    Vary: negotiate,accept
    TCN: list
    Connection: close
    Content-Type: text/html; charset=iso-8859-1
    
