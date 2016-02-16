---
layout: team
title: TR7 Team
permalink: /team/
---

<div class="team-posts">

  
  <!-- <h1 class="page-heading">Posts</h1> -->
  
  <!-- <ul class="post-list"> -->
    {% for teammember in site.teammembers %}
      <div class="team-post">
        <!-- <span class="post-meta">{{ teammember.date | date: "%b %-d, %Y" }}</span> -->
        <img src="{{teammember.image}}">
        <div>
          <h2 class="team-link">
            <span>{{ teammember.name }}</span>
            {% if teammember.twitter_username %}
            <span class="team-twitter">{% include icon-twitter.html username=teammember.twitter_username %}</span>
            {% endif %}
          </h2>
          <div class="team-meta">
            <span>{{ teammember.content }}</span>
          </div>

        </div>
      </div>
    {% endfor %}
  <!-- </ul> -->

  <!-- <p class="rss-subscribe">subscribe <a href="{{ "/feed.xml" | prepend: site.baseurl }}">via RSS</a></p> -->
  
</div>