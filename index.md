---
layout: page
title: Hello World!
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts %}
<h2>{{ post.title }}</h2>
  <div class="post">
    {{ post.content }}
  </div>
{% endfor %}

