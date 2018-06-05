---
layout: archive
permalink: /latest
author_profile: true
title: التدوينات الآخيرة
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: unsplash.jpg
---

{% for post in site.posts limit:10 %}  
  <li><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>  
{% endfor %}  