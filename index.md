---
layout: page
title: Home
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts %}
  <h1> {{ post.title }} <br> <font size="3px"> {{ post.datename }} </font> </h1>
  <div class="content">
     {{ post.content }}
  </div>
{% endfor %}

