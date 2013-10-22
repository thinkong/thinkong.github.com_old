---
layout: page
title: Nokdu's Dev Blog!
tagline: random things about random programming
---
{% include JB/setup %}

{% assign last_post = site.posts.first %}

# Last Article: {{ last_post.title }}
<hr>

{{ last_post.content }}

[Click to view the comments on this post]({{ last_post.url }}#comments)
<hr>
# Older Posts

<ul class="posts" style="margin-top:10px">
  {% for post in site.posts limit:10 offset:1 %}
    <li><a href="{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>