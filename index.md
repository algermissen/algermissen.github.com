---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}


Das ist [Hi you](http://www.jalg.net). Und text und so.


Und hier kommt code


    aaaaaa
    bbbbbb

    cccccc
    dddddd



XXXXXXXXXXXXXXX


----
wdwdwdwdwd
wdwdwdwd
----


XXXXXXXXXXXXXXX


{% for post in site.posts %}
<h2>{{ post.title }}</h2>
  <div class="post">
    {{ post.content }}
  </div>
{% endfor %}

