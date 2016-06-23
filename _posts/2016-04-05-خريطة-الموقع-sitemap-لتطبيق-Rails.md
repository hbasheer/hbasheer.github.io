---
title: إضافة خريطة الموقع Sitemap لتطبيق Rails
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: sitemap
category: Rails
date: 2016-04-05T01:56:14+03:00
---

### ما هي خريطة الموقع Sitemap ؟

خريطة الموقع هي أداة مهمة وظيفتها إخبار محركات البحث عن بنية الهيكلية للموقع الخاص بك.
إنشاء خريطة للموقع وتقديمها لمحركات البحث(مثل`Google`و `Bing` ... ) تسمح لهم بمعرفة الصفحات التي فاتتهم من خلال عمليات الزحف إلى الموقع.بالإضافة إلى أنه يسرع عملية الزحف بحد كبير جدا.

في هذه المقالة سوف اوضح كيفية بناء خريطةالموقع الخاص بموقعك او تطبيقك والتي يمكنك بعد ذلك تقدمها إلى مختلف محركات البحث.


### إعداد تطبيق Rails
كمثال سنقوم بإنشاء مدونة بسيطة وإعدادها وبعدها سنضيف التوابع اللازمة لخريطة الموقع. 
لنبدأ حاليا بإنشاء المودل `Model` الخاص بالمدونة blog

```
rails g model Post title body:text
rake db:migrate
```
الآن لننشأ وحدات `Controller` الضرورية لهذا المثال وهي :
1- Post Controller : لعرض قائمة التدوينات والسماح بعرض كل بوست على حدا
2- Sitemap Controller : المتحكم الخاص الذي يقوم بإنشاء خريطة الموقع

نفذ التعليمات التالية لإنشاء المتحكمات السابقة

```
rails g controller Posts index show
rails g controller Sitemap index
```
بعد تنفيذ التعليمة السابقة أضف الأكواد التالي ضمن ملفات `controller`و `views` التي تم إنشائها
لنبدأ بالمتحكم الخاص بالمدونة `posts_controller` 
**`app/controllers/posts_controller.rb`**

```ruby
class PostsController < ApplicationController
  def index
    @posts = Post.order("created_at DESC")
  end

  def show
    @post = Post.find(params[:id])
  end
end
```

**`app/views/posts/index.html.erb:`**

```ruby
<h1>مدونتي</h1>
<%  @posts.each do |post| %>
  <h3><%= link_to post.title, post %></h3>
  <p>
    <%= post.body.html_safe %>
  </p>
  <% if post != @posts.last %>
    <hr />
  <% end %>
<% end %>
```
**`app/views/posts/show.html.erb:`**

```ruby
<h1><%= @post.title %></h1>
<p>
  <%= @post.body.html_safe %>
</p>
```


**`app/controllers/sitemap_controller.rb`**

```ruby
class SitemapController < ApplicationController
  respond_to :xml
  def index
    @posts = Post.order("created_at DESC")
  end
end
```


نقوم بعدها بفتح ملف `routes.rb` وإضافة التوجيهات الضرورية ضمنه
**`config/routes.rb`**

```ruby
Blog::Application.routes.draw do
  resources :posts, only: [:index, :show]
  resources :sitemap, only: [:index]
  root to: "posts#index"
end
```
الآن حان الوقت لخريطة الموقع والتي هي عبارة عن ملف `XML` والذي يتكون من الغلاف `urlset` والذي يتحوي على عدة عناصر `url` والتي بدورها ايضا تحوي على أربع عناصر التالية:

 العنصر | الشرح
--- | --- 
loc|عنوان الصفحة الفعلي (url) الذي ترغب بإضافته إلى قائمة خريطة الموقع|
changefreq|عدد مرات تغير الصفحة ويأخد القيمة التالية always, hourly, daily, weekly, monthly, yearly, or never|
priority|مدى أهمية الصفحة هي في سياق موقعك.
lastmod|تاريخ آخر تعديل لهذه الصفحة.

المثال التالي يوضح بنية ملف `XML` الموضحة سابقا

**`sitemap.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>http://localhost:3000/</loc>
    <changefreq>hourly</changefreq>
    <priority>1.0</priority>
  </url>
  <url>
    <loc>http://localhost:3000/posts/2</loc>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
    <lastmod>2014-03-04T17:01:15.37+00:00</lastmod>
  </url>
  <url>
    <loc>http://localhost:3000/posts/1</loc>
    <changefreq>daily</changefreq>
    <priority>0.8</priority>
    <lastmod>2014-03-04T17:01:15.36+00:00</lastmod>
  </url>
</urlset>
```
الآن لنقوم بكتابة الملف `XML` ضمن المسار `app/views/sitemap/index.xml.builder` وفقا لمحتوى تطبيق Rails الذي تم إنشائه

```ruby
xml.instruct!

xml.urlset(xmlns: "http://www.sitemaps.org/schemas/sitemap/0.9") do
  xml.url do
    xml.loc root_url
    xml.changefreq("hourly")
    xml.priority "1.0"
  end
  @posts.each do |post|  
    xml.url do
      xml.loc post_url(post)
      xml.changefreq("daily")
      xml.priority "0.8"
      xml.lastmod post.updated_at.strftime("%Y-%m-%dT%H:%M:%S.%2N%:z")
    end
  end
end
```
مبروك! هكذا نكون قد انتهنا من إنشاء خريطة للتطبيق السابق.

يمكنك استعراض الملف المولد عبر تنفيذ تعليمة `rails s` وزيارة الرابط التالي 
http://localhost:3000/sitemap.xml.

طبعا بقي خطوة آخيرة وهي إضافة الموقع الخاص (بعد نشره) إلى محرك البحث Google عبر
[Google Webmaster Tools][1] و Bing  عبر [Bing Webmaster Tools][2]



> الطريقة السابقة تقوم بتوليد ملف **`sitemap.xml`** بشكل تلقائي وفقا لمحتوى الذي يتم إضافته إلى Model Post. إذا تم إضافة إو حذف محتوى من Post سيتغير محتوى الملف وفقا لذلك ايضا.



  [1]: http://www.google.com/webmasters/
  [2]: http://www.bing.com/toolbox/webmaster