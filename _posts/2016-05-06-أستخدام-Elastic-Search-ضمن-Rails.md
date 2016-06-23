---
title: استخدام Elastic Search ضمن Rails
layout: single
author_profile: true
date: 2016-05-06T00:00:30+03:00
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: Elasticsearch
category: Rails
---


### المفاهيم الأساسية حول Elasticsearch

* **Near Realtime اختصارا (NRT)** :    
أي أن elasticsearch منصة تعمل بالقرب من الوقت الحقيقي(ليس في الوقت الحقيقي)، والسبب في ذلك أنه هناك فرق طفيف بين الوقت الذي تفهرس index  فيه مستند Docunent حتى يصبح قابل للبحث searchable (عادة يقدر هذا الفرق بثانية واحدة)

* **Cluster**:  
تجميع من عقدة أو عدة عقد التي تعمل معا لتوفر البيانات والوظائف الخاصة بالبحث لجميع العقد.


* **node**:   
هو خدمة واحدة التي هي جزء من Cluster. الذي يخزن البيانات ويشارك الفهرسة indexing وقدرات البحث.


* **index**:    
هو مجموعة من المستندات documents.  يحتوي كل فهرس على تصنيف محدد من البيانات. 
مثلا فهرس للمستخدمين وفهرس آخر للمقالات ...
تكون محدد باسم (lowercase) وهذا الاسم يشير للفهرس index عندما القيام بعمليات indexing, update, delete operations

* **Type**: 
هذا النمط يكون مرفق مع الفهرس index.
بالإمكان إضافة نمط واحد أو عدة أنماط على كل حقل من حقول index.
أنواع الأنماط كثيرة سيتم توضيحها لاحقا .

* **Document**: 
هي الوحدة الأساسية من المعلومات التي يمكن فهرستها. مثلا: لديك مستند عن مستخدم واحد أو عن منتج واحد وغيرها...
يتم التعبير عن المستند Document باستخدام JSON.

* **Shards**: 
الفهرس ربما يخزن كمية كبيرة من البيانات التي تتجاوز حدود البنية الصلبة لعقدة واحدة. 
مثلا فهرس واحد لملايين من البيانات ربما يصل حجمه ل 1TB وهذا إما أن يملأ القرص الصلب لعقدة أو ربما يبطئ طلبات عملية البحث على عقدة واحدة.
أجزاء index.

* **Replicas**:
وهي نسخ طبق الأصل من index
القيم الافتراضي ل Shards & Replicas
Shards = 5, Replicas=1 
5 أجزاء للفهرسة و 5 آخرى نسخة طبق الأصل منها فيكون عدد الكلي 10


#### تطبيق `elasticsearch` ضمن تطبيق rails 

1- تثبيت `elasticsearch` بداية على نظام لينكس
2- استخدام الحزم البرمجية لاستخدام `elasticsearch` ضمن Rails

أضف الحزم التالية إلى Gemfile 

~~~
gem 'elasticsearch-model'
gem 'elasticsearch-rails'
~~~

بعدها bundle

~~~
bundle install
~~~
**`elasticsearch-model`** : تحتوي هذه الحزمة على تكاملات البحث التي تستخدم ضمن model مثل  `ActiveRecord::Base` و ` Mongoid`...

**`elasticsearch-rails`** : تحتوي هذه الحزمة على ميزات مختلفة لاجل تطبيق Ruby on Rails

3- إعداد `Model` المراد البحث فيه
 
* تضمين المكتبة Elasticsearch::Model

~~~ruby
class Article < ActiveRecord::Base
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks
...
...
...
end
Article.import # for auto sync model with elastic search
~~~

* تخصيص mapping


~~~ruby
settings index: { number_of_shards: 1 } do
  mappings  do
    indexes :title, analyzer: 'english'
    indexes :text, analyzer: 'english'
  end
end
~~~

* إنشاء index

~~~ruby
# Delete the previous Posts index in Elasticsearch
Post.__elasticsearch__.client.indices.delete index: Post.index_name rescue nil

# Create the new index with the new mapping
Post.__elasticsearch__.client.indices.create \
  index: Post.index_name,
  body: { settings: Post.settings.to_hash, mappings: Post.mappings.to_hash }

# Index all Post records from the DB to Elasticsearch
Post.import

~~~

* إضافة تابع البحث search

~~~ruby
def self.search(query)
  __elasticsearch__.search(
    {
      query: {
        multi_match: {
          query: query,
          fields: ['title^10', 'text']
        }
      }
    }
  )
end
~~~
 4- إنشاء Controller && Views



~~~ruby
rails g controller search 
~~~
* إضافة تابع البحث  إلى الملف التالي `app/controller/search_controller.rb`

~~~ruby
def search
  if params[:q].nil?
    @posts = []
  else
    @posts = Post.search params[:q]
  end
end
~~~

*  إنشاء ملف view
`app/views/search/search.html.erb:`

~~~ruby
<h1>Posts Search</h1>

<%= form_for search_path, method: :get do |f| %>
  <p>
    <%= f.label "Search for" %>
    <%= text_field_tag :q, params[:q] %>
    <%= submit_tag "Search", name: nil %>
  </p>
<% end %>

<ul>
  <% @Posts.each do |post| %>
    <li>
      <h3>
        <%= link_to post.title, controller: "posts" action: "show", id: post._id%>
      </h3>
    </li>
  <% end %>
</ul>

~~~

*  اخيرا rute search

~~~ruby
get 'search', to: 'search#search'
~~~













