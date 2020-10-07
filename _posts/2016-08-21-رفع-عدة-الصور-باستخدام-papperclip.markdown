---
title: رفع عدة الصور باستخدام  papperclip
date: 2016-08-21T21:49:33+03:00
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: "/assets/images/nature-grain.jpg"
  show_overlay_excerpt: false
tags: paperclip
category: Rails
---


لنفرض لدينا تطبيق ذو البنية التالية:

نموذج للتدوينات Post ونموذج آخر للصور Image العلاقة بينهما (كل تدوينة واحدة تمتلك عدة صورة -  Post has_may: images)   
نموذج الصور يحتوي على صورة واحدة image والتي هي paperclip attachment.  
الذي نريده حاليا هو معرفة طريقة لرفع عدة صور مباشرة عند إنشاء التدوينة وهذا ما سيتم توضيحه ضمن هذه التدوينة
وفقا للخطوات التالية:

1- إنشاء تطبيق Rails  
2- تنصيب الجيم `papperclip` ضمن التطبيق  
3- توليد نموذج Post  
4- توليد نموذج Image  
5- تخصيص  controller & views   


### 1- إنشاء تطبيق Rails
عبر تنفيذ التعليمة التالية:

~~~bash
rails new demo_app
~~~

### 2- تنصيب الجيم  `papperclip` ضمن التطبيق

* الجيم `paperclip` تعتمد على imagemagick لذا يجب تنصيبها ضمن نظام التشغيل ليمكن استخدامها

~~~
sudo apt-get install imagemagick -y
~~~
* إضافة الجيم `papperclip` إلى ملف Gemfile


~~~ruby
gem 'paperclip'
~~~
ثم نطبق تعليمة `bundle install`

### 3- توليد نموذج Post 

~~~bash
rails g scaffold post title:string body:text  --skip-assets
~~~

`app/models/post.rb`

~~~ruby
class Post < ApplicationRecord
	has_many :images, :dependent => :destroy
end
~~~

### 4- توليد  نموذج Image


~~~ruby
rails g model Image image:attachment post_id:integer
rake db:migrate
~~~

 `app/models/image.rb`

~~~ruby
class Image < ApplicationRecord
	belongs_to :post

	# paperclip attachment
	has_attached_file :image,
       :path => ":rails_root/public/images/:id/:filename",
       :url  => "/images/:id/:filename"
       styles: { large: '1200x600>', thumb: '200x120>' }
       #paperclip validation
      validates_attachment :image,presence: true,
      content_type: { content_type: ["image/jpeg", "image/gif", "image/png"]},
      size: { in: 0..3.megabytes }	
end
~~~


### 4- تخصيص controller & views 

الفكرة حاليا تتخلص بالتالي:

* تمرير جميع الصور المراد رفعها دفعة واحدة عبر حقل input إلى مصفوقة ولتكن images[] وذلك ضمن الفورم الخاص بالنموذج Post كالتالي:

`app/views/posts/_form.html.erb`

~~~ruby
<%= form_for @post, :html => { :class => 'form-horizontal', multipart: true } do |f| %>
  <div class="control-group">
    <%= f.label :title, :class => 'control-label' %>
    <div class="controls">
      <%= f.text_field :title, :class => 'text_field' %>
    </div>
  </div>
  <div class="control-group">
    <%= f.label :body, :class => 'control-label' %>
    <div class="controls">
      <%= f.area_field :body, :class => 'text_field' %>
    </div>
  </div>

  <div class="control-group">
    <%= f.label :images, :class => 'control-label' %>
    <div class="controls">
      <%= file_field_tag "images[]", type: :file, multiple: true %>
    </div>
  </div>

  <div class="form-actions">
    <%= f.submit nil, :class => 'btn btn-primary' %>
  </div>
<% end %>
~~~

>  تم إضافة خاصية  HTML5 multipart إلى الفورم لكي نتمكن من تحديد عدة عناصر(صور) ضمن input وتخزينها ضمن المصفوفة images[]  وتمريرها إلى `post_controller` 

> ايضا استخدمنا التابع file_field_tag لان images[]  ليست Post attribute  من معاملات النموذج Post 

* اخيرا تمرير مصفوفة الصور إلى create ضمن  post_controller.rb لإنشائها بعد حفظ post

`app/controllers/post_controller.rb`

~~~ruby
def create
  @post = Post.new(post_params)

  respond_to do |format|
    if @post.save
     
      if params[:images]
        params[:images].each { |image|
          @post.images.create(image: image)
        }
      end

      format.html { redirect_to @post, notice: 'Post was successfully created.' }
      format.json { render json: @post, status: :created, location: @post }
    else
      format.html { render action: "new" }
      format.json { render json: @post.errors, status: :unprocessable_entity }
    end
  end
end
~~~


