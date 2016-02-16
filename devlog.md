---
layout: devlog
title: TR7 Devlog
permalink: /devlog/
---

<div class="devlog-posts">

  
  <!-- <h1 class="page-heading">Posts</h1> -->
  
  <!-- <ul class="post-list"> -->
    {% for post in site.posts %}
      <div class="devlog-post">
        <!-- <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span> -->
        {% if post.image %}
        <img src="{{post.image}}" alt="{{post.title}}">
        {% endif %}
        <div>
          <h2 class="devlog-link">
            <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a>
          </h2>
          <div class="devlog-meta">
            <span>{{ post.date | date: "%b %-d, %Y" }}</span>
          </div>
        </div>
      </div>
    {% endfor %}
  <!-- </ul> -->

  <!-- <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p> -->
  
</div>