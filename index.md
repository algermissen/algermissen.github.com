---
layout: default
title: Applying the Web To Enterprise IT
---
{% include JB/setup %}

{% for post in site.posts %}
<h2>{{ post.title }}</h2>
  <div class="post">
    {{ post.content }}
  </div>
{% endfor %}

