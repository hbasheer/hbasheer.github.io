---
title: إعداد مشاريع Rails  الجديدة عن طريق railsrc.
date: 2017-04-18T01:59:12+03:00
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: "/assets/images/nature-grain.jpg"
  show_overlay_excerpt: false
tags: railsrc
category: Rails
---

إذا كنت ممن يستخدمون نظام linux/unix فالتأكيد قد صادفت ملفات تبدأ ب `.` وتنتهي ب `rc` مثل : `vimrc` , `.bashrc`, 

> يمكنك استعراض هذه الملفات من خلال التعليمة التالية:
>
    ls ~/.*rc

هذه الملفات تمكنك من إعداد سلوك البرامج الخاصة بها بالشكل التي تريده تماما عند استخدامها من دون تمرير flags او option التي تقوم بوظيفة معينة. 


### لشرح آلية عمل ملفات rc لنأخذ المثال التالي: 

فرضا لو أردت تثبت gem معينة بدون تثبيت التوثيق الخاص بها استخدم التعليمة التالية :

    gem install gem-name  --no-document
    
  `--no-document` وظيفتها هي منع تثبيت التوثيق الخاص بالجيم
 طبعا لو اردت عدم تثبيت التوثيق دوما عند إضافة كل جيم ستكون مضطرا لاستخدام flag السابق دوما إلا اذا تمت إضافة flag إلى ملف الإعداد `.gemrc`
 
    echo 'gem: --no-document' >> ~/.gemrc

>  Rubygems تحتوي على ملف  `rc` ايضا 
 
 الآن عند كل عملية تنصيب لجيم جديدة عبر  Rubygems سيتم تمرير flag  ` --no-document ` بشكل آلي.


### Ruby on Rails تملك ملف `rc` 
عندما تريد إنشاء تطبيق  rails جديد عبر التعليمة التالية `rails new project_name` غالبا ما تضيف سلسلة طويلة من flag لكي تقوم بتخصيص التطبيق وفقا لحاجاتك  

```ruby
rails new project_name --database=postgresql  --skip-action-cable --webpack --skip-spring --skip-coffee --skip-turbolinks --template=/path/to/rails_template.rb
```
بدلا من استخدام الطريقة يمكنك إضافة flags السابقة ضمن ملف `.railsrc`

```
# ~/.railsrc
--database=postgresql
--skip-bundle
--skip-spring
--skip-test-unit
--skip-turbolinks
--template=/path/to/rails_template.rb
```
لكي يتم تمريرها بشكل آلي عند إنشاء اي تطبيق Rails جديد


