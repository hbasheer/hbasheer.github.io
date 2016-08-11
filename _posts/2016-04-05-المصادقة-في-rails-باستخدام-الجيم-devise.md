---
title: المصادقة في Rails باستخدام الجيم Devise
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: devise
category: Rails
date: 2016-04-05T01:53:56+03:00
---

### لمحة بسيطة عن devise
devise هو أحد أشهر الجزم البرمجية(gem) التي تقدم حل متكامل للمصادقة(Authentication) ضمن تطبيقات Rails.
إن أبرز ما يميز devise هو سهولة استخدام(عدة تعليمات برمجية ويتم تثبيته ضمن تطيق Rails وتشغيله) والدعم الكبير من قبل المجتمع الضخم من المطوريين الذي لا يزال يقدم تطويرات وإضافات مفيدة بشكل متزايد.


يتكون devise من 10 وحدات(module) يمكنك من اختيار الضروري منها وفقا للحاجة التي تريدها ضمن تطبيقك، أذكرها باختصار مع وظيفة كل منها:

* **Registrable** تسمح هذه الوحدة للمستخدمين بتسجيل حسابات جديدة وتعديلها وحذفها فما بعد.
* **Database Authenticatable** تخزين وتشفير معلومات المستخدمين في قاعدة البيانات والتحقق منها عند دخول المستخدم الى التطبيق.
* **Omniauthable** تقوم هذه الوحدة بإضافة نظام Omniauth والذي يسمح للمستخدم بالدخول الى تطبيقات rails بإستخدام بيانات دخوله في مواقع أخرى مثلا تويتر - فيسبوك - جووجل وغيرها .
*  **Confirmable** إرسال بريد الى المستخدم بعد تسجيله مباشرة يحتوي على معلومات تفعيل الحساب.
* **Recoverable** إعادة تعيين كلمة المرور وإسترجاعها وذلك بإرسال بريد للمستخدم.
* **Rememberable** إدارة عملية حفظ ومسح معلومات المستخدم من على الجهاز بإستخدام saved cookie .
* **Trackable** تسمح هذه الوحدة بمعرفة عدد مرات وعنوان ip address و تاريخ و وقت الدخول.
* **Timeoutable** إنهاء الجلسة session عندما لا يكون هناك أي نشاط للمشتخدم خلال مدة زمنية محددة.
* **Validatable** تعمل هذه الوحدة على تطبيق شروط Validations  على حقل البريد الالكتروني وكلمة المرور التي يدخلها المستخدم.
* **Lockable** تعمل هذه الوحدة على إيقاف حساب المستخدم بعد عدة محاولات دخول فاشلة.

### تثبيت devise ضمن تطبيق Rails   

لنقم أولا بإنشاء تطبيق Rails جديد باسم devise_demo

    $ rails new devise_demo

بعد تنفيذ الأمر السابق وإنشاء مجموعة الملفات والمجلدات الخاصة بالتطبيق، أدخل إلى مسار التطبيق 

    $ cd devise_demo 
   
واستعرض ملف `Gemfile` ,أضف إليه الجيم `devise`

    gem 'devise'

وبعدها نفذ الأمر التالي

    $ bundle install

>   Bundler
هو برنامج لإدارة الحزم البرمجية Gem في مشاريع روبي.
تستطيع من خلاله تحديد الحزم التي يحتاجها مشروعك ،مع تعين اصدار معين لكل حزمة في ملف واحد يسمى `Gemfile` وتنصيبها في وقت واحد عن طريق تعليمة `bundle install`، دون الحاجة لاستخدام تعليمة gem لتنصيب كل حزمة على حدا  

بعد الانتهاء من تثبيت الحزم الموجودة ضمن ملف Gemfile عبر تعليمة `bundle`، نفذ الأمر التالي لتوليد الملفات الرئيسية لعمل devise ضمن التطبيق

    $ rails generate devise:install

نتيجة تنفيذ التعليمة السابقة ستكون كالتالي: 

	create  config/initializers/devise.rb
	create  config/locales/devise.en.yml


	Some setup you must do manually if you haven't yet:

	1. Ensure you have defined default url options in your environments files. Here
 	is an example of default_url_options appropriate for a development environment
    in config/environments/development.rb:

	       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

	In production, :host should be set to the actual host of your application.

  	2. Ensure you have defined root_url to *something* in your config/routes.rb.
 	    For example:

  	    root to: "home#index"

    3. Ensure you have flash messages in app/views/layouts/application.html.erb.
	     For example:

	      <p class="notice"><%= notice %></p>
 	      <p class="alert"><%= alert %></p>

 	4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

 	      config.assets.initialize_on_precompile = false

    On config/application.rb forcing your application to not access the DB
  	   or load models when precompiling your assets.
	
 	5. You can copy Devise views (for customization) to your app by running:

 	      rails g devise:views

---
 نلاحظ أن الأمر الأخير قام بإنشاء ملفات خاصة بنظام devise وهي عبارة عن مايلي :

 - ملف devise.rb و الموجود على المسار config/initalizers وهو يحتوي على الإعدادت الرئيسية لل devise والتي يتم تطبيقها بدء تشغيل التطبيق.
 - ملف devise.en.yml والموجود في المسار config/locals ويحتوي على جميع الرسائل التي تظهر للمستخدمين ويمكن التعديل على هذه الرسائل و إنشاء
   ملفات أخرى للغات غير الإنجليزية.

أيضا نلاحظ أن هناك رسالة بمجموعة من الإعدادات يجب أن نتآكد من تنفيذها على التطبيق وهي كما يلي

1- إضافة السطر التالي `config/environments/development.rb ` لتحديد الرابط الافتراضي(default URL) لبيئة التطوير development :

    config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

2- تحديد المسار  root   في ملف `routes.rb`
يتم ذلك عن طريق إنشاء controller  باسم page وتحديد action لها باسم index

    $ rails g controller page index

ويتم تحديد index كصفحة رئيسية للتطبيق بإضافة التعليمة التالية ضمن ملف `config/routes.rb`

    root "page#index"

3- إضافة الأكواد التالية لإظهار رسائل التنبيه ضمن ملف الرئيسي للتطبيق 
`app/views/layouts/application.html.erb`:  


~~~
[...]
<body>
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
<%= yield %>
</body>
[...]
~~~
4- الخطوة الرابعة لن نحتاج لها لأن إصدار rails الذي نستخدمه 4 وليس 3.2

---
وأخيرا إنشاء MODEL ضمن Devise:

    $ rails generate devise user

التعليمة السابقة تولد model  ضمن المسار `app/model/user.rb` ويكون محتواه كالتالي :

~~~ruby
class User < ActiveRecord::Base
	
  	devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end
~~~

> نلاحظ مما سبق انه من ضمن 10 وحدات فقط 6 تكون مفعلة عند تثبيت devise وهي موضحة ضمن الكود السابق

وتولد أيضا ملف migration ضمن المسار التالي `db/migrations/xxx_devise_create_users.rb` الذي يحوي على الأعمدة الضرورية التي يحتاجها devise ضمن قاعدة البيانات

### إضافة الوحدتان  Confirmable,Lockable واستخدامهما ضمن Devise
كما تم التوضيح سابقا أن الوحدات المفعلة بشكل افتراضي عند تثبيت devise هي 6 فقط :

 database_authenticatable | registerable |recoverable  
:-------------: |:-------------:| :-----:
**trackable** |    **rememberable**  |     **rememberable**


لإضافة كل من الوحدتان Confirmable,Lockable  إلى devise واستخدامها يجب أولا إضافتها ضمن ملف 
`app/model/user.rb` 

~~~ruby
devise :database_authenticatable, :registerable,
       :recoverable, :rememberable, :trackable, :validatable, :confirmable, :lockable
~~~
وأخيرا إضافة الأعمدة الضرورية لعملها ضمن  ملف migration الذي تم إنشائه سابقا
`db/migrations/xxx_devise_create_users.rb`

> الأعمدة تم تولديها بالفعل ولكن تم تعطيلها بإضافة رمز`#` ليتم اعتبارها كتعليق 


~~~ruby
      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at
~~~
أحذف الرمز `#` من الأكواد السابقة وأضف أيضا عامود لسماح للمستخدمين بإضافة اسم لهم

~~~ ruby
      ## Database authenticatable
      t.string :name 
~~~
الآن نفذ الأمر التالي لإنشاء جدول باسم user ضمن قاعدة المعطيات يحتوي على الأعمدة الموضحة بالملف 
`db/migrations/xxx_devise_create_users.rb`

    $ rake db:migrate


### ضبط إعدادات  Devise وفقا لعمل كل وحدة
جميع الإعدادات التي تخص عمل الوحدات الخاصة به موجودة ضمن الملف `config/initializers/devise.rb`, افتح الملف السابق وابحث عن القسم الخاص بالوحدة `lockable` الموجود تحت العبارة التالية `Configuration for :lockable`  واحذف الرمز `#` من امام الإعدادت التالية :


 - **config.lock_strategy = :failed_attempts** : هذا يعني بأن الحساب سيتم قفله بعد عدة محاولات فاشلة لتسجيل الدخول، يمكنك جعلها تساوي `none` والتعامل مع آلية القفل بنفسك.
 - **config.unlock_strategy = :both** : إجراء إلغاء قفل الحساب تتم إما عن طريق البريد الالكتروني(من خلال التأكيد على رابط يتم إرساله من قبل devise) أو من خلال الانتظار مدة معينة من الزمن، بإمكانك استبدال `both:` بالقيمة التالية `email:` لجعل إلغاء القفل عبر البريد فقط أو استبدالة بالقيمة `time:` لجعل الإلغاء عبر الانتظار فقط أو ترك القيمة الافتراضية `both:` لجعل الأمر يتم عبر الاثنين معا.
 - **config.maximum_attempts = 20** : عدد المرات المتتالية التي يسمح للمستخدم فيها إدخال كلمة مرور خاطئة قبل إقفال الحساب.
 - **config.unlock_in = 1.hour** : المدة الزمنية التي يجب على المستخدم انتظارها قبل إلغاء القفل بشكل آلي.
 - **config.last_attempt_warning = true** :إصدار رسالة تحذير `flash message` عندما يتبقى للمستخدم محاولة وحيدة قبل إغلاق الحساب.

الآن بعد الانتهاء من إعدادات الوحدة `lockable` ننتقل إلى الوحدة `confirmable` بنفس الطريقة السابقة، ابحث عن القسم الخاص به الموجود تحت العبارة التالية `Configuration for :confirmable`  واحذف الرمز `#` من امام الإعدادت التالية ايضا:

 - **config.confirm_within = 3.days** : المدة الزمنية التي يستطيع المستخدم تنشيط حسابه ضمنها عن طريق رمز التفعيل المرسل إلى البريد الالكتروني (بعد انتهاء هذه المدة يصبح رابط التنشيط غير فعال ويتوجب على المستخدم الحصول على رمز جديد). يمكنك إضافة القيمة التالية "nil" إذا أردت ان لا تنتهي مدة تنشيط الحساب.
 - **config.reconfirmable = true** : يجب على المستخدم إعادة تأكيد اي تغير على ملفه الشخصي عبر البريد الإلكتروني. هذه العملية تشبه عملية تنشيط الحساب بعد التسجيل.فلو مثلا قام بتعديل حقل البريد في ملفه الشخصي ولم يقم بتأكيد التعديل عبر البريد الالكتروني، يبقى البريد الجديد الذي تم إضافته غير مؤكد، ويجري تخزينه في الحقل unconfirmed_email حتى يقوم المستخدم بزيارة رابط التفعيل. وخلال هذه الفترة يتم استخدام البريد الإلكتروني القديم لتسجيل الدخول.

**إذا كنت تقوم ببناء تطبيق حقيقي، لا ننسى أيضا أن تعدل هذه الإعدادات:**

 - **config.mailer_sender** : اسم البريد الذي سوف يظهر لكل مستخدم في حقل `from`.
 - **config.secret_key** : المفتاح السري `Secret key`. بإمكانك استخدام المفتاح الموجود بشكل افتراضي او توليد مفتاح جديد عن طريق التعليمة التالية:

~~~
$ rake secret 
~~~

### تخصيص Devise  وإضافة اللمسات الأخيرة لعمله

#### توليد ملفات الإظهار (Generating Views)

يتم توليد جميع ملفات الإظهار الخاصة بعمل الوحدات المفعلة عن طريق الأمر التالي:

    $ rails generate devise:views

التعليمة السابقة سوف تنسخ ملفات إظهار Devise الافتراضية بمجلد اسمه `devise` ضمن المسار التالي `app/views/layouts/devise`.
لنلقي نظرة سريعة على محتوى المجلد السابق الذي تم توليده الذي بدورة يحتوي على عدة مجدات وهي:

 - **confirmations** : يحتوي على ملف وحيد `new.html.erb` يتم تحميله عندما يريد المستخدم إعادة إرسال رسالة التفعيل عبر البريد الالكتروني.
 - **mailer** : جميع نماذج البريد الالكتروني التي يتم إرسالها موجودة ضمن هذا المجلد.
 - **passwords** : يحتوي على ملفين الأول `new.html.erb` يتم استدعائه عندما يريد المستخدم استعادة كلمة المرور عبر البريد الالكتروني(يحتوي على form للبريد الالكتروني الذي سيتم إرسال تعليمات الإستعادة إليه)، والآخر `edit.html.erb` يتم استدعائه بعد ضغط المستخدم على رابط المرسل عبر البريد ويحتوي على `form` لكلمة المرور الجديدة وتأكيدها.
 - **registrations** : يحتوي على ملفين الأول `new.html.erb` يتم استدعائه عندما يقوم المستخدم بتسجيل جساب وجديد والملف الأخر `edit.html.erb` يتم استدعائه  عندما يقوم المستخدم بتعديل بيانات ملفه الشخصي.
 - **sessions** :يحتوي على ملف واحد  ويتم استدعائه عندما يريد المستخدم تسجيل دخول(`login form`)
 - **shared** : يحتوي على جميع الروابط التي تظهر في صفحة من صفحات Desvise  مثل (هل نسيت كلمة المرور، أعد إرسال تعليمات تفعيل الحساب ...)
 - **unlocks** :يحتوي على ملف واحد(`form` للبريد الالكتورني) يتم استدعائه عندما يريد المستخدم إلغاء القفل عن حسابه.


إذا كنت ترغب في توليد ملفات الإظهار `views` لبعض وحدات فقط (مثلا registerable و confirmable)نفذ هذا الأمر:

    $ rails generate devise:views -v registrations confirmations
 
أيضا إذا كان لديك عدة model ضمن تطبيق Rails تستخدم Devise (مثلا users, admin) بإمكانك توليد ملفات الإظهار `views` الخاصة بكل Model عن طريق التعليمة التالية:

    $ rails generate devise:views users
    $ rails generate devise:views admin

#### Strong_params وحقل الخاص باسم المستخدمين `name:` في  Devise
إذا كنت تستخدم الإصدار 4 أو 5 من الإطار Rails فهذا يعني انك تستخدم [strong_params][1] بشكل افتراضي
لذلك يجب تمرير `name:` إلى parameter الخاصة ب Devise بإضافة الأسطر التالية إلى الملف
`app/controllers/application_controller.rb`

~~~ruby
before_action :configure_permitted_parameters, if: :devise_controller?

protected

def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_up) << :name
  devise_parameter_sanitizer.for(:account_update) << :name
end
~~~

الآن تبقى إضافة حقل `name:` إلى ملفات الإظهار الخاصة بالوحدة `registrations` ليتمكن المستخدم من إضافة اسمه عند تسجيله وتعديله أيضا إذا أراد تعديل ملف الشخصي.

~~~
  <div class="field">
    <%= f.label :name %><br />
    <%= f.email_field :name, autofocus: true %>
  </div>
~~~

#### إعدادات إرسال البريد من قبل Devise
ضمن بيئة `development` عملية إرسال البريد (بريد التفعيل واستعادة كلمة المرور ...) غير مفعلة بشكل افتراضي ضمن Devise. لتفعليها أضف السطر التالي ضمن الملف `config/environments/development.rb`
مع إعدادات `stmp:` لمخدم البريد gmail (بإمكانك استخدام مخدم بريد آخر مثل sendgrid أو mandrill)

~~~ruby
config.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:              'smtp.gmail.com',
  port:                 587,
  domain:               'example.com',
  user_name:            '<username>',
  password:             '<password>',
  authentication:       'plain',
  enable_starttls_auto: true  }
~~~
حيث أن `user_name` و `password` هي اسم المستخدم وكلمة مرور حسابك في gmail

يمكن إيضا إضافة نفس الإعدادات السابقة إلى بيئة `production` مع بعض الاختلافات البسيطة 
`config/environments/production.rb`

~~~ruby
config.action_mailer.default_url_options = { host: 'devise_demo.herokuapp.com' }
onfig.action_mailer.delivery_method = :smtp
config.action_mailer.smtp_settings = {
  address:              'smtp.gmail.com',
  port:                 587,
  domain:               'heroku.com',
  user_name:            '<username>',
  password:             '<password>',
  authentication:       'plain',
  enable_starttls_auto: true  }
~~~

> بالنسبة لهذا المثال تم استخدام منصة التطبيقات السحابية [Heroku][2] كبيئة `production`

الاختلاف يكون فقط في 

 - الرابط الافتراضي للتطبيق وهو
 
~~~ruby 
config.action_mailer.default_url_options = { host: 'devise_demo.herokuapp.com' }
~~~

حيث `devise_demo` هو اسم التطبيق الذي تم إنشائه على منصة Heroku

 - والدومين :

~~~ruby
domain:               'heroku.com'
~~~

 وهذا يقودنا إلى نهاية هذه التدوينة.

---

### الخلاصة:
هذه التدوينة ليست إلا مدخل تعريفي مع بعض الشروحات البسيطة عن وحدات وإعدادت نظام Authentication  المسمى Devise
هناك نقاط كتيرة لم أتطرق لها وربما تكون مهمة بالنسبة إليك لذلك أدعوك إلى الإطلاع على توثيق الجيم [Wiki][3]  وبشكل خاص على الصفحة [How-to’s][4] إذا يوجد بها أكثر من 90 تدريب بسيط ومفيد جدا

 
سأحاول إن شاء الله من خلال تدوينة آخرى شرح بعض الإضافات التي تستخدم مع devise وكيفية استخدام الوحدة Omniauthable أيضا




  [1]: http://edgeapi.rubyonrails.org/classes/ActionController/StrongParameters.html
  [2]: https://www.heroku.com/
  [3]: https://github.com/plataformatec/devise/wikihttps://github.com/plataformatec/devise/wiki
  [4]: https://github.com/plataformatec/devise/wiki/How-Tos
  