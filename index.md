---
layout: page
title: Home
tagline: Supporting tagline
---
{% include JB/setup %}

{% for post in site.posts %}
  
    <h2>post.title</h2>
    <div class="content">
        {{ post.content }}
    </div>

{% endfor %}

