---
layout: archive
permalink: /tags/
author_profile: true
title: الوسوم
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: unsplash.jpg
---

{% include base_path %}
{% include group-by-array collection=site.posts field="tags" %}

{% for tag in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ tag | slugify }}" class="archive__subtitle">{{ tag }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}