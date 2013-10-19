---
layout: default
title: Applying the Web To Enterprise IT
---
{% include JB/setup %}

{% for post in site.posts limit:6 %}
<h2>{{ post.title }}</h2>
<div>
{{ post.content | strip_html | truncatewords:75}}<br>
<a href="{{ post.url }}">Read more...</a><br/><br/>
</div>
{% endfor %}
<!--
<ul >
    {% for post in site.posts limit:6 %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
        {{ post.content | strip_html | truncatewords:75}}<br>
            <a href="{{ post.url }}">Read more...</a><br><br>
    {% endfor %}
</ul>
-->

