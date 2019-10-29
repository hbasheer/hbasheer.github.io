---
layout: archive
permalink: /categories/
author_profile: true
title: التصنيفات
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: unsplash.jpg
---

{% include base_path %}
{% include group-by-array collection=site.posts field="categories" %}

{% for category in group_names %}
  {% assign posts = group_items[forloop.index0] %}
  <h2 id="{{ category | slugify }}" class="archive__subtitle">{{ category }}</h2>
  {% for post in posts %}
    {% include archive-single.html %}
  {% endfor %}
{% endfor %}