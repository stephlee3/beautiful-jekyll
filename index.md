---
layout: page
title: Welcome to my homepage!
use-site-title: true
bigimg:
  - "img/Inner_Harbor.jpg" : "Inner Harbor, Baltimore MD"
  - "img/Ithaca.jpg" : "Ithaca NY"
  - "img/Golden_Gate_Bridge.jpg" : "Golden Gate Bridge, San Francisco CA"
  
---
My name is Runzhe Li. I am a first-year PhD student at the Department of Biostatistics, Johns Hopkins University. I graduated from Department of Mathematical Sciences, Tsinghua University in July 2018.

Click [About Me](https://stephlee3.github.io/aboutme) for more details.

<div class="posts">
  {% for post in site.posts %}
    <article class="post">

      <h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1>

      <div class="entry">
        {{ post.excerpt }}
      </div>

      <a href="{{ site.baseurl }}{{ post.url }}" class="read-more">Read More</a>
    </article>
  {% endfor %}
</div>
