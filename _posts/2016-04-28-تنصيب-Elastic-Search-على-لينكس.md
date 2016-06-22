---
title: تنصيب Elastic Search على نظام لينكس 
layout: single
author_profile: true
date: 2016-04-28T18:04:01+03:00
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: Elasticsearch
category: rails
---

 
#### 1- تنصيب جافا
elastic search مكتوب بلغة جافا لذا وجود تنصيب بيئة عمل جافا ضمن النظام

~~~
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
java -version
~~~

#### 2- تنصيب elastic search

* إضافة Public Signing Key

~~~
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
~~~
* إضافة مستودع elastic search

~~~
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
~~~

بعدها update وتنصيب elasticsearch عبر apt-get

~~~
sudo apt-get update && sudo apt-get install elasticsearch
~~~

* إعداد elastic search للعمل بشكل اتوماتيكي عند تشغيل النظام
إذا كانت التوزيعة تستخدم `SysV` نفذ الأمر التالي

~~~
sudo update-rc.d elasticsearch defaults 95 10
~~~

وإذا كانت تستخدم `systemd`

~~~
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
~~~

#### 3- تشغيل elastic serach

~~~
sudo service elasticsearch start
~~~

مرجع:  [elasticsearch ](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-repositories.html) 

