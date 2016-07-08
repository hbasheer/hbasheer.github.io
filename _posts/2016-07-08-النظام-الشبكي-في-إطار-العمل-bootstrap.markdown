---
title: النظام الشبكي في إطار العمل Bootstrap
layout: single
author_profile: true
header:
  overlay_color: "#000"
  overlay_filter: "0.4"
  overlay_image: nature-grain.jpg
tags: Bootstap
category: CSS
date: 2016-07-08T11:28:06+03:00
---

يحتوي النظام الشبكي للإطار Bootstap على اربعة تصنيفات (xs, sm, md, lg )، كل تصنيف مخصص لأحجام محددة من الشاشات. 

* **xs : **  للشاشات الصغرى (الهواتف المحمولة) ذات الحجم 768px أو أقل
* **sm : ** للشاشات الصغيرة (الأجهزة اللوحية) من حجم 768px إلى حجم 992px.
* **md : ** للشاشات المتوسطة الحجم (الأجهزة المكتبية والحواسيب المحمولة) من حجم 993px إلى حجم 1200px.
* **lg :**  للشاشات الكبيرة (الأجهزة المكتبية الكبيرة) ذات حجم أكبر من 1200px.

يقسم النظام الشبكي الصفحة مهما كان حجمها إلى 12 عامود أفقي كما هو موضح بالصورة التالية:

![Grid-system](https://www.safaribooksonline.com/library/view/bootstrap/9781449344573/httpatomoreillycomsourceoreillyimages1640513.png  "Grid sysytem")

الأمثلة التالية توضح كيفية استخدام التصفيات الأربعة السابقة مع بعضها او كلا على حدة :
 
### تقسيم الصفحة إلى قسمين بنسبة (50%-50%) لكل قسم باستخدام تصنيف `md`
~~~html
<div class="container">
	<div class="row">
		<div class="col-md-6">50%</div>
		<div class="col-md-6">50%</div>
	</div>
</div>
~~~


### تقسيم الصفحة إلى قسمين (33%-66% )باستخدام تصنيف `md`
~~~html
<div class="container">
	<div class="row">
		<div class="col-md-4">33%</div>
		<div class="col-md-8">66%</div>
	</div>
</div>
~~~


> يجب ان يكون مجموع الأعمدة المستخدمة ضمن كل صف يساوي 12 عامود أو أقل إذا أردنا استخدام الإزاحة

###  تقسيم الصفحة إلى ثلاثة أقسام (25%-50%-25%) باستخدام تصنيف `md`

~~~html
<div class="container">
	<div class="row">
		<div class="col-md-3">25%</div>
		<div class="col-md-6">50%</div>
		<div class="col-md-3">25%</div>
	</div>
</div>
~~~
### استخدام أعمدة داخل أعمدة
فقط يتم إنشاء صف جديد وداخله يتم إضافة 12 عامود بالتقسيم المناسب 

~~~html
<div class="container">
	<div class="row">
		<div class="col-md-3">25%</div>
		<div class="col-md-6">
			<div class="row">
				<div class="col-md-4"></div>
				<div class="col-md-8"></div>
			</div>
		</div>
		<div class="col-md-3">25%</div>
	</div>
</div>
~~~

### استخدام عدة تصنيفات(sm,md,lg) مختلفة معا  
~~~html
<div class="container"> 
  <div class="row"> 
    <div class="col-sm-6 col-md-7 col-lg-9"></div> 
    <div class="col-sm-6 col-md-5 col-lg-3"></div> 
  </div> 
  <hr /> 
  <div class="row"> 
    <div class="col-sm-6 col-md-2 col-lg-5"></div> 
   	<div class="col-sm-6 col-md-10 col-lg-7"></div> 
  </div>
</div>
~~~

### إزاحة offset الأعمدة
هذه الخاصية لا تدعم الشاشات الصغيرة أي التي تنتمي للصنف من النوع xs 
> يجب ان يكون حجم الأعمدة والإزاحة يساوي 12

~~~html
<div class="container">
	<div calss="row"> 
	  <div class="col-xs-2 col-md-offset-1"></div> 
	  <div class="col-xs-6 col-md-offset-3"></div>
	</div>
</div>
~~~

### ترتيب الأعمدة
وهو ظهور الأعمدة بترتيب مغاير لترتيب الكود المكتوب ويتم ذلك عبر `col-*-pull-*` و `col-*-push-*`

~~~html
<div class="container">
	<div class="row"> 
	  <div class="col-md-4 col-md-push-8"></div> 
	  <div class="col-md-8 col-md-pull-4"></div>
	</div>
</div>
~~~