---
title: Title والوسوم Meta لكل ملفات Views ضمن تطبيق Rails
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: seo
category: Rails
date: 2016-04-05T01:54:57+03:00
---

### 1- إنشاء التوابع المساعدة

`helpers/application_helper.rb`

~~~ruby
module ApplicationHelper
  def title(text)
    content_for :title, text
  end

  def meta_tag(tag, text)
    content_tag :"meta_#{tag}", text
  end

  def yield_meta_tag(tag, default_text='')
    content_for?(:"meta_#{tag}") ? content_for(:"meta_#{tag}") : default_text
  end
end
~~~

2- استخدام التوابع السابقة ضمن ملف `application.html.erb` الخاص بالتطبيق كاملا
 
`app/views/layouts/application.html.erb`

~~~
<title>
  <%= if content_for?(:title) then yield(:title) + ' | ' end %>
  اسم الموقع 
</title>

<meta name='description'
      content='<%= yield_meta_tag(:description, 'Default description') %>' />
<meta name='keywords'
      content='<%= yield_meta_tag(:keywords, 'defaults,ruby,rails') %>' />
~~~

3- آخيرا استخدام التوابع السابقة ضمن ملفات `views`

~~~
<% title @post.title %>
<% meta_tag :description, @post.description %>
<% meta_tag :keywords, @post.keywords.join(',') %>
~~~