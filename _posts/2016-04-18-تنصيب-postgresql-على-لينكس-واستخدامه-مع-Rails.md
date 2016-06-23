---
title: تنصيب Postgresql على لينكس واستخدامه مع Rails
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: postgresql
category: DataBase
date: 2016-04-18T03:08:22+03:00
---

#### تنصيب postgresql حسب الإصدار الخاص به
1- تنصيب الإصدار 9.3  يتم عبر التعليمات التالية

    sudo apt-get update
    sudo apt-get install postgresql postgresql-contrib libpq-dev

2- تنصيب الإصدار 9.5 

- إضافة المستودع الخاص ب postgresql إلى linux 

يتم ذلك عبر إنشاء ملف خاص ضمن `sources.list.d` وليكن  `pgdg.list`



~~~
 sudo touch /etc/apt/sources.list.d/pgdg.list
 sudo gedit /etc/apt/sources.list.d/pgdg.list
~~~
 ومن ثم إضافة الحزمة التالية إليه وحفظ الملف 


~~~
deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main
~~~
- إضافة المفتاح الخاص بمستودع حزمة  postgresql


~~~
sudo apt-get install wget ca-certificates
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
~~~

- تحديث الحزم ضمن النظام 

~~~
sudo apt-get update
~~~
- تنصيب postgresql-9.5


~~~
sudo apt-get install postgresql-9.5 pgadmin3 postgresql-contrib libpq-dev
~~~

#### إعداد postgresql
ينبغي لنا أن نفهم بعض الأشياء عن كيفية عمل postgresql

1- **مستخدم postgresql الافتراضي**: 
بعد تنصيب PostgreSQL يتم إنشاء مستخدم نظام باسم **postgres** بدون كلمة مرور(طبعا لا يمكن الدخول إلى server بدون إنشاء كلمة مرور ) له صلاحية `super admin` يمكنه القيام بجميع المهام والولوج إلى جميع السيرفرات الخاصة بقاعدة البيانات

2- **قاعدة البيانات الافتراضية** :
 PostgreSQL يتم تثبيتها مع قاعدة افتراضية باسم  **postgres** 
قاعدة البيانات الابقة تستخدم لأغراض إدارية `dmin purposes`، ويتم إنشاء قاهدة بيانات آخرى للاستخدامات التي نريدها

3- **سطر الأوامر المساعدة sql**:
PostgreSQL يتضمن sqlسطر الأوامرالمساعد لإدارة القواعد البيانات والسيرفرات.
هناك واجهة بيانية تدعى pgadmin3 التعامل معها اسهل وتقوم بنفس مهام سطر الأوامر السابق.
سوف نستخدم سطر الأوامر السابق لإنشاء مستخدم جديد وقاعدة البيانات ابتدائية

#### إنشاء مستخدم آدمن Super User
سنقوم بالولوج إلى حساب **postgres**  من خلال المستخدم الجذر الخاص بالنظام  بك ، ومن ثم نستخدم حساب **postgres**  لإنشاء حساب مستخدم جديد بصلاحية آدمن.

* الولوج إلى المستخدم **postgres** :

~~~
## switch user to root:
 su - 
## switch user to postgres:
 su - postgres

## or 
sudo -i -u postgres 
~~~

* الدخول إلى سطر الأوامر المساعدة sql:


~~~
psql
~~~
* إنشاء مستخدم من خلال طرفية sql:

~~~
postgres=# CREATE USER youruseraccount
postgres-# WITH SUPERUSER CREATEDB CREATEROLE
postgres-# PASSWORD 'userAccountPassword';
~~~

يتم الخروج من الطرفية عبر التعليمة التالية `q/`

* تسجيل دخول باستخدام psql وإنشاء قاعدة بيانات


~~~
psql postgres
CREATE DATABASE db_name WITH OWNER username;
~~~

* آخيرا الاتصال بقاعدة البيانات

~~~
\connect test_db;
~~~
#### استخدام pg ضمن تطبيق الريلز

* تنصيب الحزمة الخاصة بقاعدة البيانات

~~~
gem 'pg'
~~~
* تعديل ملف database.yml


~~~
default: &default
  adapter: postgresql
  user: username
  password: 
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db_development

test:
  <<: *default
  database: db_test
~~~

#### كيفية عمل Backup & Restore
* Backup & Restore لقاعدة بيانات واحدة فقط


~~~
#Backup
pg_dump -U {user-name} {source_db} -f {dumpfilename.sql}
~~~


~~~
#Restore
psql -U {user-name} -d {desintation_db}-f {dumpfilename.sql}
~~~

* Backup & Restore لجميع قواعد المعطيات 


~~~
#Backup
pg_dumpall > backup_file
~~~


~~~
#Restore
psql -f backup_file postgres
~~~
> النسخة تكون ضمن المسار /var/lib/postgresql