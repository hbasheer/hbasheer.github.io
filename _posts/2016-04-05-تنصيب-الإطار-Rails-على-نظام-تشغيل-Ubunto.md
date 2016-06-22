---
title: تنصيب الإطار Rails على نظام تشغيل Ubunto
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: Ubunto
category: rails
date: 2016-04-05T00:47:26+03:00
---



### لماذا اخترت Ubuntu  بالتحديد ؟
Ubuntu هو أكثر منصة شائعة تستخدم لتطوير تطبيقات Rails من قبل عدد كبير جدا من المطورين، بالإضافة إلى سهولة تنصيبها واستخدامها ودعمها الواسع من قبل مجتمع مطوريين Rails


<iframe width="560" height="315" src="http://www.youtube.com/embed/PWf4WUoMXwg" frameborder="0"> </iframe>


### تجهيز النظام 
سوف تحتاج إلى تحديث مدير الحزم الخاص بنظام التشغيل قبل تنصيب Ruby on Rails من خلال تنفيذ التعليمة التالية:

    $ sudo apt-get update

بعد الانتهاء من تنفيذ التعليمة السابقة نفذ التعليمة التالية عبر الطرفية لتثبيت [Curl](https://en.wikipedia.org/wiki/CURL)

    $ sudo apt-get install curl

> Curl سيتم استخدامه لتنصيب مدير إصدارات روبي RVM

### تنصيب Ruby باستخدام RVM

> إذا كانت منصبة لديك، حدثها ونصب روبي من خلال التعليمات التالية:
$ rvm get stable --autolibs=enable
$ rvm install ruby
  
قبل تنصيب RVM لابد من استيراد GPG Key الخاص بها عن طريق تنفيذ الأمر التالي :

    $ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3

بعدها نفذ الأمر التالي لتنصيب [RVM](https://rvm.io/)
  
    $ \curl -sSL https://get.rvm.io | bash -s stable

بعد الانتهاء من تنصيب RVM لابد من تنفيذ الأمر التالي لتحميل RVM script واستخدامه ضمن الطرفية

    $ source ~/.rvm/scripts/rvm

أخيرا تنصيب Ruby  

    $ rvm install ruby

التعليمة السابقة تقوم بتنصيب الإصدار الأخير من اللغة Ruby وهو `2.3.0`.  
يمكنك الإطلاع على إصدارات روبي المتوفرة والمدعومة  من خلال الموقع الرسمي للغة [Ruby ](https://www.ruby-lang.org/en/downloads/)

> عند تنفيذ التعليمة السابقة ستظهر لك رسالة خطأ التالية "rvm command not found"
لحل المشكلة:
> من الطرفية gnome-terminal اختار Edit ▸ Profile Preferences
> علم المربع عند Run command as login shell
> بالنهاية أعد تشغيل الطرفية https://rvm.io/integration/gnome-terminal

استخدام الإصدار السابق بشكل افتراضي

    $ rvm use 2.3.0 --default 

### تنصيب  Ruby on Rails
 
يتم تنصيب Rails باستخدام Rubygems ولكن يفضل ان يتم إخبار Rubygems بعدم تحميل التوثيق الخاص بكل حزمة ليتم تنصيب الحزم بشكل أسرع عبر التعليمة التالية:

    $ echo "gem: --no-ri --no-rdoc" > ~/.gemrc
    
أخيرا تنصيب Rails

    $ gem install rails
    $ rails -v

التعليمة السابقة تقوم بتنصيب آخر إصدار مستقر من [Rails][1]، الإصدار Rails 4.2.5 هو الإصدار الآخير حاليا وفقا لتاريخ التدوينة.

> مدير حزم روبي (RubyGems): أداة صممت لتسهيل عملية تنصيب مكتبات روبي او ما يسمى ب Gems.
> يستخدم تعليمة `gem`من اجل تنصيب او حذف او تعديل الحزم Gems .
> عمل (RubyGems) يشبه إلى  حد كبير عمل كل من `yum` - `apt-get` في أنظمة لينكس .
> يمكنك زيارة دليل  [ RubyGems Guides](http://guides.rubygems.org/)  لتعرف عليه أكثر

منذ الإصدار `3.1` لإطار Rails وبيئة التطوير ضمن توزيعات لينكس  تتطلب JavaScript runtime التي تسمح باستخدام Coffeescript و [ Asset Pipeline][2]
يتم تجاوز هذه المشكلة من خلال تنصيب  nodejs

    $ sudo apt-get install nodejs

إذا لم يتم تنصيب nodejs  سوف تحتاج إلى إضافة الجيم التالية إلى ملف Gemfile لكل تطبيق رولز تقوم بإنشائه:

    gem 'therubyracer'

### إنشاء تطبيق جديد 

إنشاء تطبيق Rails يتم من خلال تعليمة واحدة فقط 

    $ rails new app

> حيث `app` هو اسم التطبيق الذي تريد إنشائه

بعد الانتهاء من تنفيذ التعليمة السابقة نفذ الأمر التالي `rails s` لتشغيل `web server` والذي يسمى WEBrick (مبني على لغة روبي)

الآن افتح متصفح الوب وادخل http://localhost:3000/ لترى التطبيق الذي قمت بتنصيبه. 

  [1]: https://rubygems.org/gems/rails
  [2]: http://guides.rubyonrails.org/asset_pipeline.html